---
title: X window
date: 2016-07-03 14:17:20
categories:
  - linux-dev
  - Graphics
tags:
  - Xorg
---

Everything about X window in code.
<!--more-->

# X Window Extension #
## Initialize Extension ##

```
main() (dix/stubmain.c)
  dix_main (dix/main.c)
    InitOutput (hw/xfree86/common/xf86Init.c)
      xf86ExtensionInit (hw/xfree86/common/xf86Extensions.c)
        to initialize the extension in list of extensionModules via LoadExtension()
    InitExtension (mi/miinitext.c)
      AddStaticExtensions (mi/miinitext.c)
        to initialize the extension in list of staticExtensions via LoadExtension()
```
* The list of extensionModules includes merely DRI2 and related lagency modules
* The list of staticExtensions includes most of extesion modules
  * Randr
  * Composite
  * Damage
  * Present
  * DRI3
* LoadExtension
  * Create(realloc) a new module in global module list of ExtensionModuleList
  * Register name, initFunc, disablePtr
  * Call (*initFunc), such as dri3_extension_init()
    * To add the Extension via AddExtension()

## AddExtension ##
* Add the extension into the ProcVector
  * Before adding the extension to the ProcVector, there are some extension available. AddExtension() will add the exension module dynamically to the ProcVector.
  * The X server will recieve the request from the Client and dispatch it to the coresponding extension looking through the ProcVector.
```
	i = NumExtensions;
	newexts = (ExtensionEntry **) realloc(extensions,
			(i + 1) * sizeof(ExtensionEntry *));
	if (!newexts) {
		free(ext->name);
		dixFreePrivates(ext->devPrivates, PRIVATE_EXTENSION);
		free(ext);
		return ((ExtensionEntry *) NULL);
	[[}]]
	NumExtensions++;
	extensions = newexts;
	extensions[i] = ext;
	ext->index = i;
	ext->base = i + EXTENSION_BASE;
	ext->CloseDown = CloseDownProc;
	ext->MinorOpcode = MinorOpcodeProc;
	ProcVector[i + EXTENSION_BASE] = MainProc;
	SwappedProcVector[i + EXTENSION_BASE] = SwappedMainProc;
```
* AddExtension()
```
AddExtension(const char *name, int NumEvents, int NumErrors,
             int (*MainProc) (ClientPtr c1),
             int (*SwappedMainProc) (ClientPtr c2),
             void (*CloseDownProc) (ExtensionEntry * e),
             unsigned short (*MinorOpcodeProc) (ClientPtr c3))
```
  * Name: 插件的名称。
  * NumEvents: 为扩展保留的事件数。
  * NumErrors: 为扩展保留的错误码数。
  * **MainProc**: 扩展的处理函数。
  * SwappedMainProc: 扩展的处理函数，在处理前先交换字节顺序。
  * CloseDownProc: 扩展的析构函数。
  * MinorOpcodeProc: 用来得到子处理号，一般没有什么用处，在出错时，设置到错误信息里。
* Take DRI3 as an example:
  * After AddExtension, DRI3 proc interfaces are available in ProcVector as a sperate portion.
```
extension = AddExtension(DRI3_NAME, DRI3NumberEvents, DRI3NumberErrors,
                         proc_dri3_dispatch, sproc_dri3_dispatch,
                         NULL, StandardMinorOpcode);

int (*proc_dri3_vector[DRI3NumberRequests]) (ClientPtr) = {
    proc_dri3_query_version,            /* 0 */
    proc_dri3_open,                     /* 1 */
    proc_dri3_pixmap_from_buffer,       /* 2 */
    proc_dri3_buffer_from_pixmap,       /* 3 */
    proc_dri3_fence_from_fd,            /* 4 */
    proc_dri3_fd_from_fence,            /* 5 */
};

int
proc_dri3_dispatch(ClientPtr client)
{
    REQUEST(xReq);
    if (stuff->data >= DRI3NumberRequests || !proc_dri3_vector[stuff->data])
        return BadRequest;
    return (*proc_dri3_vector[stuff->data]) (client);
}
```

## Referrence ##
* [10.X Window扩展机制－－扩展(Extension)](http://blog.csdn.net/absurd/article/details/1796672)

# devPrivate #
* 有很多结构体都有自己的Private数据，X server提供了一套Key-Value的模式来管理所有结构体与Private数据的关联。
* Overview
  * global_keys: 所有key的集合list
  * 这些key分为以下几类：
    * XSELINUX: XSELinux uses the same private keys for numerous objects.
    * Others: get a private in just the requested structure
      * These can have objects created before all of the keys are registered
      * These cannot have any objects before all relevant keys are registered
      * extension privates
```
global_keys[PRIVATE_LAST]
---------------
|  XSELINUX   | .key--> key(n) -> key(n-1) -> ... -> key(0)
---------------
|  SCREEN     |
---------------
|    ...      |
---------------
|  [Type]     |
---------------
|             |
---------------

the type of global_keys is below:

typedef struct _DevPrivateSetRec {
	DevPrivateKey key;	/* related key, 指向最新注册的key，当然是当前同一类型的 */
	unsigned offset;	/* 主要用于XSELINUX，下一组key的offset */
	int created;		/* 每次初始化后加1 */
	int allocated;		/* 每次申请Private后加1 */
} DevPrivateSetRec, *DevPrivateSetPtr;
```
* 在使用所有devPrivate前，X server将所有数据清零dixResetPrivates()
  * 相关数据都清为0
* 申请Private空间dixAllocatePrivate
  * 初始化Private(_dixInitPrivates)时，
    * global_keys[type].created++
    * private = addr = 0, 这是因为之后要使用realloc申请内存
  * ++global_keys[type].allocated
* 注册Private key(dixRegisterPrivateKey)时会创建key，并添加到当前type的key list中去。
  * key list 是一个单向逆序的list，global_keys[type].key始终指向最后一个key
  * allocate key from private via dixReallocPrivates()
```
	/* above is XSELINUX */
    else {
        /* Resize if we can, or make sure nothing's allocated if we can't */
        if (!allocated_early[type])
            assert(!global_keys[type].created);
        else if (!allocated_early[type] (dixReallocPrivates, bytes))
            return FALSE;
        offset = global_keys[type].offset;
        global_keys[type].offset += bytes;
        grow_screen_specific_set(type, bytes);
    }

    /* Setup this key */
    key->offset = offset;
    key->size = size;
    key->initialized = TRUE;
    key->type = type;
    key->allocated = FALSE; /* Why it is not TRUE?? */
    key->next = global_keys[type].key;

Question:
目前来看只有dixRegisterScreenPrivateKey()在调用了dixRegisterPrivateKey()后，将key->allocated=TRUE.
其它情况下，并没有理会该值。可能没什么特别的影响??
```
* 通常，调用dixAllocatePrivate()申请某一type的key in global_keys[]，然后可以多次注册到相同type中许多不同的key。
* 以上，完成了初始化与注册的部分。其实平时使用较多的是以下函数：
  * dixSetPrivate: Associate 'val' with 'key' in 'privates' so that later calls to dixLookupPrivate(privates, key) will return 'val'.
  * dixLookupPrivate: Lookup a pointer to the private record.
    * dixGetPrivate: Fetch a private pointer stored in the object.

# XID #
* XID是X系统中区别于他物的识别号，例如一个Window会有一个唯一的XID。
* XID由两个有效内容组成。
  * 共32bit，其中高3位为无效位，其它29位为XID有效内容: mask as 0x1FFF_FFFF
```

     | <---------------------- (a) ------------------------------> |
 111b| <----- (b) ------> | <--------------- (c) ----------------> |
-----+--------------------------------------------------------------
| 3  |       6 bits       |               23 bits                  |
--------------------------------------------------------------------
           Client ID                     Source ID
      RESOURCE_CLIENT_MASK         RESOURCE_ID_MASK
      (0x1F80_0000)                (0x007F_FFFF)

(a) RESOURCE_AND_CLIENT_COUNT		29 /* The total bits of XID */
(b) RESOURCE_CLIENT_BITS		6  /* Client ID bits */
    also can be 7, 8, 9, according to MAXCLIENTS as 128, 256, 512
(c) CLIENTOFFSET			(RESOURCE_AND_CLIENT_COUNT - RESOURCE_CLIENT_BITS)
    in another word, the Source ID
```

## Client ID ##
* 每个Client都有唯一的ID，可与其它Client区别开来，也可以在引用时方便查找。
* 对于SeverClient，ID是0，其它Client的ID都是从1开始依次累加生成。
* Client ID是在创建新的连接Connection时创建
```
WaitForSomething
  EstablishNewConnections
    AllocNewConnection
      NextAvailableClient
        nextFreeClientID (Start from 1 in Dispatch(), always set by the next available Client ID)
```

## Resource ID ##
* 对于每个XID都可以拥有许多resources，为了方便管理多个resource资源，建立了resource专用的HashTable
* InitClientResources()初始化每个Client的resource by clientTable, static global array in dix/resource.c
  * client->index: Client ID
  * clientTable以Client ID为index，保存每个client ID相关的resource hash table
  * 用.resources[]作为.hassize bits的hash table，每个element之间以单链表方式联接。
  * 当AddResource时，以HashResourceID方式来生成hash table的id，可以能是顺序（刚开始时），也可能是跳序的（溢出之后），如1,3,2,5,4,7。而生成的hash id是由XID决定的。如果需要查找的resource XID在hash table中，那么就应该在以其生成的hash ID处，如果不在的话，那么，就要遍历其all elements with the same hash ID via "->next"。
    * 通常，resource的XID为fake ID也是每次新创建时加1得到的，所以新加的resource依次添加入hash table当hash table不够用的时候，才会overwrite某个element。所以，如果生成的hash ID不能在hash table中直接命中所需要的resource，那就遍历吧。
    * 根据HashResourceID的规则得到的相同的hash ID的resource，hash table(.resources[hash ID]) 将会保留最近一次添加的resource，之前相同hash ID的resource将以单链表联接。
  * Fake ID: 用来标识属于当前client的resource，也是XID，但添加了无效位，SERVER_BIT(0x4000_0000)
    * 如果Client XID is 0x0100_0000，那么fakeID=0x4100_0000, endFakeID=0x4200_0000。
```
InitClientResources()
{
    ...
    clientTable[i = client->index].resources =	/* Hash table for resources */
        malloc(INITBUCKETS * sizeof(ResourcePtr));
    if (!clientTable[i].resources)
        return FALSE;
    clientTable[i].buckets = INITBUCKETS;	/* 64 elements in the hash table */
    clientTable[i].elements = 0;		/* the total number of resources related to the client ID, adding 1 once AddResourc() */
    clientTable[i].hashsize = INITHASHSIZE;	/* the number of bits for has table, 6 bits for 64 elements stored in hash table */
    /* Many IDs allocated from the server client are visible to clients,
     * so we don't use the SERVER_BIT for them, but we have to start
     * past the magic value constants used in the protocol.  For normal
     * clients, we can start from zero, with SERVER_BIT set.
     */
    clientTable[i].fakeID = client->clientAsMask |
        (client->index ? SERVER_BIT : SERVER_MINID);
    clientTable[i].endFakeID = (clientTable[i].fakeID | RESOURCE_ID_MASK) + 1;                                                          |
    ...
}
```

# Common Functions #

* dixAllocateObjectWithPrivates
  * allocate a void* memory block with devPrivate registered
  * returen the void* for the specific type as assigned
* dixLookupWindow
* swapl
* WriteToClient
* Prepare for reply
* AddResource
* DevPrivate
  * http://www.x.org/wiki/Development/Documentation/DevPrivates/

# Reference #
* [DDX design X11R7.7](http://www.x.org/releases/X11R7.7/doc/xorg-server/ddxDesign.html)
* [The X New Developer’s Guide](http://www.x.org/wiki/guide/)

# Acronym #

* BDF
  * Bitmap Distribution Format
* DBE
  * Double Buffer Extension
* DIX
  * Device Independent
* DDX
  * Device Dependent
* ICCCM
  * The Inter-Client Communication Conventions Manual
* DMX
  * distributed multihead X system
* DMPS
  * Display Power Management Signaling
* DPS
  * Display Postscript
* DRI
  * Direct Rendering Interface
* EVI
  * Extended Visual Information
* FS
  * Font Service
* ICE
  * Inter-Client Exchange
* RX
  * remote execute
* CUP
  * Colormap Utilization Policy and Extension
* DMCP
  * Display Manager Control Protocol
* XIM
  * X Input Method Protocol
* XI
  * X11 Input Extension Protocol
* XSMP
  * X Session Management Protocol
* XP
  * X Print Service
* XPM
  * X PixMap Format
* XTrans
  * X Transport Interface
* GC
  * Graphic context
