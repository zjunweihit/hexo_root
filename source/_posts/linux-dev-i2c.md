---
title: I2C的前生今世
date: 2016-10-27 22:45:18
categories:
  - linux-dev
tags:
  - linux-dev-misc
---

# Abstract #
* I2C是常用的总线，关于什么是I2C以及I2C怎么传输数据的（时序）这里就不再赘余，可以在百度里google一下。
* I2C传输并不很复杂，但是在Linux的驱动框架中，为了有更好的扩展性，出现了很多相对独立的模块。下面一一介绍。
* 以下内容基于：linux-3.4, i.MX6 SABRESD platform, RTC(rs5c372) driver.
* Referring to http://blog.csdn.net/w2baby1314/article/details/8081993

<!--more-->

# Overview #
* I2C是一个独立的总线，其类型为i2c_bus_type，类似于platform_device一样。所以先要有I2C总线设备
  - 注册I2C总线设备, 对应的driver为dummy_driver
  - 注册I2C主控制器（i2c_adapter）到I2C总线上，在注册过程中，依据板级代码(board-mx6q_sabresd.c)中初始化的i2c_board_info（devinfo）创建i2c_client。i2c_client的父设备为i2c_adapter。
    - i2c_adapter与i2c_client为一对多的关系。
    - 在i2c-imx.c中，先注册platform_driver i2c_imx_driver到platform总线上，如果找到of_match_table中的"fsl,imx6q-i2c"则，调用i2c_imx_probe()，其中，创建i2c_imx->adapter设备，并注册到I2C总线上（使用i2c_add_numbered_adapter()）。
  - 注册i2c_driver到I2C总线上，遍历i2c_adapter的i2c_client，如果有name一致的。则match成功，调用probe函数（rs5c372_probe())
  - 至此，形成如下结构：
```
Device:
              i2c_bus_type       i2c_adapter  <--- i2c_client
                    |               |                  |
I2C bus             V               V                  V
================================================================
Driver:             A               A                  A
                    |               |                  |
                    |               +------------------|
                    |               |
               dummy_driver    i2c_driver(find i2c_adapter's i2c_client according to .id_table)
```
# I2C驱动结构 #
* 主要包括：I2C核心，I2C主控制器（adapter），I2C slave（从设备）

* I2C核心(drivers/i2c/i2c-core.c)：
  - 主控制器与从设备使用的函数与数据结构，由Linux完成，主要依据I2C通信协议（支持I2C标准协议和SMBus协议）
* I2C主控制器(drivers/i2c/busses/i2c-imx.c)：
  - 一般集成在主芯片中，通常由CPU厂商完成代码。依据芯片的User Manual，操作I2C数据的流程。
* I2C slave(drivers/rtc/rtc-rs5c372.c):
  - 挂载到I2C主控制器上的外设（如触摸屏，eeprom，rtc，传感器等）。从设备做为client与主控制器adapter通信。

```
user space       user app                        i2c user mode
                 (update rtc)
                    |                                  |
--------------------|----------------------------------|---------
kernel space        |                                  |
                    |                                  |
                /dev /sys                           i2c-dev
                (/dev/rtc)                             |
                    |                                  |
                    |                                  |
                 slave driver                          |
                 (rtc driver) ----|    |---------------|
               (2)  |             |    |
          only for  | device init |    |
                    |            i2c-core
                    |               |
                    |          主控制器驱动
                    |          (i2c-adapter)
                    |               | (3)
                    |               |------------------|
                    |                                  |
                    |                                  |
--------------------|----------------------------------|---------
hardware device     |                                  |
                    |                                  |
                i2c slave device  <------------>   i2c 主控制器
                (rtc device)

(1) 如果app通过/dev/rtc读/写rtc的设置,那么就会走(1)路线.
    rtc_set_time (rtc driver)
      i2c_smbus_write_i2c_block_data (i2c-corc.c)
        i2c_smbus_xfer
          i2c_smbus_xfer_emulated (如果不支持smbus的话,模拟smbus)
            i2c_transfer
              adp->algo->master_xfer (调用具体i2c adapter的xfer函数)
                i2c_imx_xfer
                  根据msg.flags
                  if (msg.flags & I2C_M_RD)
                    i2c_imx_read
                  else
                    i2c_imx_write

(2) 一般只有在设备初始化时走(2)路线,之后user对rtc设备的操作都要经过(1),即i2c进行读/写控制.
(3) 在i2c读/写过程中,信息的载体就是i2c_msg,它在slave driver和i2c-dev中会实例化.
    主要包括,读写标志,读/写buf,size等.
struct i2c_msg {
        __u16 addr;     /* slave address                        */
        __u16 flags;
#define I2C_M_TEN               0x0010  /* this is a ten bit chip address */
#define I2C_M_RD                0x0001  /* read data, from slave to master */
#define I2C_M_NOSTART           0x4000  /* if I2C_FUNC_PROTOCOL_MANGLING */
#define I2C_M_REV_DIR_ADDR      0x2000  /* if I2C_FUNC_PROTOCOL_MANGLING */
#define I2C_M_IGNORE_NAK        0x1000  /* if I2C_FUNC_PROTOCOL_MANGLING */
#define I2C_M_NO_RD_ACK         0x0800  /* if I2C_FUNC_PROTOCOL_MANGLING */
#define I2C_M_RECV_LEN          0x0400  /* length will be first received byte */
        __u16 len;              /* msg length                           */
        __u8 *buf;              /* pointer to msg data                  */
};
```

# I2C核心 #
* 一般Linux驱动的核心层做两件事情：
  - 为了方便管理与归类，注册总线或class。
  - 为相关的设备驱动提供统一的接口。
* I2C core先要创建I2C bus的设备
```
i2c-core.c
static int __init i2c_init(void)
{
        int retval;

        retval = bus_register(&i2c_bus_type);
        if (retval)
                return retval;
#ifdef CONFIG_I2C_COMPAT
        i2c_adapter_compat_class = class_compat_register("i2c-adapter");
        if (!i2c_adapter_compat_class) {
                retval = -ENOMEM;
                goto bus_err;
        }
#endif
        retval = i2c_add_driver(&dummy_driver);
        if (retval)
                goto class_err;
        return 0;

class_err:
#ifdef CONFIG_I2C_COMPAT
        class_compat_unregister(i2c_adapter_compat_class);
bus_err:
#endif
        bus_unregister(&i2c_bus_type);
        return retval;
}
```
* 这里创建了bus_type类型的I2C总线i2c_bus_type
```
i2c-core.c
struct bus_type i2c_bus_type = {
        .name           = "i2c",
        .match          = i2c_device_match,
        .probe          = i2c_device_probe,
        .remove         = i2c_device_remove,
        .shutdown       = i2c_device_shutdown,
        .pm             = &i2c_device_pm_ops,
};
```
  - 当向I2C总线注册driver时，总终会调用到bus->match(dev,drv)来匹配slave device和slave driver。
```
module_i2c_driver(rs5c372_driver)
  module_driver
    i2c_add_driver(rs5c372_driver)
      i2c_register_driver(THIS_MODULE, rs5c372_driver) <i2c-core.c>
        driver_register() <drivers/base/driver.c>
          bus_add_driver() <drivers/base/bus.c>
            driver_attach() <drivers/base/dd.c>
              __driver_attach() <drivers/base/dd.c>
                driver_match_device(drv, dev) <drivers/base/base.h>
                static inline int driver_match_device(struct device_driver *drv,
                                                      struct device *dev)
                {
                        return drv->bus->match ? drv->bus->match(dev, drv) : 1;
                }
```
  - 对于有些注册名字与实际名字不一样的情况，我们可以设置driver的id_table来描述设备的名字。
```
i2c-core.c
static int i2c_device_match(struct device *dev, struct device_driver *drv)
{
        struct i2c_client       *client = i2c_verify_client(dev);
        struct i2c_driver       *driver;

        if (!client)
                return 0;

        /* Attempt an OF style match */
        if (of_driver_match_device(dev, drv))
                return 1;

        driver = to_i2c_driver(drv);
        /* match on an id table if there is one */
        if (driver->id_table)
                return i2c_match_id(driver->id_table, client) != NULL;

        return 0;
}
```
* i2c_init 流程：
```
i2c_init --> bus_register(&i2c_bus_type) --> create /sys/bus/i2c/
```

# I2C 主控制器（adapter） #
* I2C主控制器驱动，即由adapter来表示。它必须要带有algorithm，实现访问从设备的方法。不同的CPU I2C主控制器，具体实现的传输方法细节也不完全一样，虽然都是依据I2C的协议。
## i2c_adapter ##
```
include/linux/i2c.h
/*
 * i2c_adapter is the structure used to identify a physical i2c bus along
 * with the access algorithms necessary to access it.
 */
struct i2c_adapter {
        struct module *owner;
        unsigned int class;               /* classes to allow probing for */
        const struct i2c_algorithm *algo; /* the algorithm to access the bus */
        void *algo_data;

        /* data fields that are valid for all devices   */
        struct rt_mutex bus_lock;

        int timeout;                    /* in jiffies */
        int retries;
        struct device dev;              /* the adapter device */

        int nr;
        char name[48];
        struct completion dev_released;

        struct mutex userspace_clients_lock;
        struct list_head userspace_clients;
};
```
  * algo_data: 保存其算法的私有数据，在i2c_algorithm的方法中使用。i2c-imx.c没有使用。i2c-s3c2410.c使用了。
  * timeout: 传递数据超时时间，单位jiffies。如果没有初始化，默认值为HZ，即1秒。
  * retires: 传输一次消息重试的次数。传输失败，返回-EAGAIN，然后重试retires - 1次，即一共传输retires次。
  * nr: I2C总线的编号，从0开始。如i.MX6 sabresd dual lite 有4个I2C。0，1，2，3。
  * name: I2C 从设备的名字。
  * dev_released: 释放i2c_adapter的completion。
## i2c_algorithm ##
* i2c_algorithm提供了主从设备通信方法。支持SMBus和标准I2C协议。如果从设备和adapter支持SMBus的话，推荐使用SMBus，它更加灵活方便。
```
include/linux/i2c.h
struct i2c_algorithm {
        /* If an adapter algorithm can't do I2C-level access, set master_xfer
           to NULL. If an adapter algorithm can do SMBus access, set
           smbus_xfer. If set to NULL, the SMBus protocol is simulated
           using common I2C messages */
        /* master_xfer should return the number of messages successfully
           processed, or a negative value on error */
        int (*master_xfer)(struct i2c_adapter *adap, struct i2c_msg *msgs,
                           int num);
        int (*smbus_xfer) (struct i2c_adapter *adap, u16 addr,
                           unsigned short flags, char read_write,
                           u8 command, int size, union i2c_smbus_data *data);

        /* To determine what the adapter supports */
        u32 (*functionality) (struct i2c_adapter *);
};
```
  * master_xfer和smbus_xfer分别表示I2C和SMBus的传输方法，它们的不同细节，参考以下。这里主要说明master_xfer
    - https://i2c.wiki.kernel.org/index.php/Main_Page
    - http://blog.csdn.net/wenlifu71022/article/details/4465339
  * num表示每次传输数据的个数。
```
drivers/i2c/busses/i2c-imx.c
static int i2c_imx_xfer(struct i2c_adapter *adapter,
                                                struct i2c_msg *msgs, int num)
{
        <snip>
        for (i = 0; i < num; i++) {
                <snip>
                if (msgs[i].flags & I2C_M_RD)
                        result = i2c_imx_read(i2c_imx, &msgs[i]);
                else
                        result = i2c_imx_write(i2c_imx, &msgs[i]);
                <snip>
        }
}
```
  * 每次传输的数据用i2c_msg表示。
```
include/linux/i2c.h
struct i2c_msg {
        __u16 addr;     /* slave address                        */
        __u16 flags;
#define I2C_M_TEN               0x0010  /* this is a ten bit chip address */
#define I2C_M_RD                0x0001  /* read data, from slave to master */
#define I2C_M_NOSTART           0x4000  /* if I2C_FUNC_PROTOCOL_MANGLING */
#define I2C_M_REV_DIR_ADDR      0x2000  /* if I2C_FUNC_PROTOCOL_MANGLING */
#define I2C_M_IGNORE_NAK        0x1000  /* if I2C_FUNC_PROTOCOL_MANGLING */
#define I2C_M_NO_RD_ACK         0x0800  /* if I2C_FUNC_PROTOCOL_MANGLING */
#define I2C_M_RECV_LEN          0x0400  /* length will be first received byte */
        __u16 len;              /* msg length                           */
        __u8 *buf;              /* pointer to msg data                  */
};
```
    - addr: 从设备地址，可以是7（通常）或10位。传输地址时，要记得右移一位。
      - 可以使用I2C_BOARD_INFO("ov564x", 0x3c)来写入slave adress。ov564x的地址为0x78，右移后为0x3c。
      - 这是因为在传输时，会有左移的操作，并将最低位写为R/W位。无论是read/write。
```
static int i2c_imx_read(struct imx_i2c_struct *i2c_imx, struct i2c_msg *msgs)
{
<snip>
        /* write slave address */
        writeb((msgs->addr << 1) | 0x01, i2c_imx->base + IMX_I2C_I2DR);
               ^^^^^^^^^^^^^^^^^
<snip>
```
      - 如果是10位，flag必须设置为I2C_M_TEN，adapter也必须支持I2C_FUNC_10BIT_ADDR
    - I2C_M_RD: 表示读操作，主设备从从设备中读数据。注，如果I2C主控制器工作在slave mode的话，则表示外设（主设备）从主控制器（从设备）读数据。
    - 其他详细内容再补充
  * 总结一下master_xfer: 一般主控制器里有一个字节大小的寄存器（其实就是移位器）来循环发传输数据。通常是将要发送的数据保存到i2c_msg->buf中，然后初始化I2C总线信号（产生start信号），发送第一个字节数据（从设备地址与R/W位）到数据寄存器，等待I2c主控制器的中断到来，在中断中判断发送完数据（一般情况下，中断到来时，数据发送完，判断是为了保护及确认数据发送完成），然后唤醒写数据的进程，继续写下一下数据，再次wait in queue等待中断的I2C主控制器的中断到来。
  * functionality: I2C主控制器支持的功能。
```
include/linux/i2c.h
#define I2C_FUNC_I2C                    0x00000001
#define I2C_FUNC_10BIT_ADDR             0x00000002
#define I2C_FUNC_PROTOCOL_MANGLING      0x00000004 /* I2C_M_NOSTART etc. */
#define I2C_FUNC_SMBUS_PEC              0x00000008
#define I2C_FUNC_SMBUS_BLOCK_PROC_CALL  0x00008000 /* SMBus 2.0 */
#define I2C_FUNC_SMBUS_QUICK            0x00010000
#define I2C_FUNC_SMBUS_READ_BYTE        0x00020000
#define I2C_FUNC_SMBUS_WRITE_BYTE       0x00040000
#define I2C_FUNC_SMBUS_READ_BYTE_DATA   0x00080000
#define I2C_FUNC_SMBUS_WRITE_BYTE_DATA  0x00100000
#define I2C_FUNC_SMBUS_READ_WORD_DATA   0x00200000
#define I2C_FUNC_SMBUS_WRITE_WORD_DATA  0x00400000
#define I2C_FUNC_SMBUS_PROC_CALL        0x00800000
#define I2C_FUNC_SMBUS_READ_BLOCK_DATA  0x01000000
#define I2C_FUNC_SMBUS_WRITE_BLOCK_DATA 0x02000000
#define I2C_FUNC_SMBUS_READ_I2C_BLOCK   0x04000000 /* I2C-like block xfer  */
#define I2C_FUNC_SMBUS_WRITE_I2C_BLOCK  0x08000000 /* w/ 1-byte reg. addr. */
```
## i2c_adapter 注册 ##
* i2c adapter device通过以下某一个函数完成：
  * i2c_add_adapter()：用于动态分配的bus号，即adap->nr。
  * i2c_add_numbered_adapter()：用于静态分配的bus号。多数情况下，用这个函数。
* 这两个函数操作是类似的，都是先确定bus的号（IDR分配），然后调用i2c_register_adapter()进行注册。
* 在i2c_register_adapter()中，把adapter当成device注册到i2c bus总线上。使用i2c bus的match函数进行匹配，如果匹配成功，则调用i2c bus的probe函数。
```
i2c_register_adapter()
  device_register() <drivers/base/core.c>
    device_add() <drivers/base/core.c>
      bus_probe_device() <drivers/base/core.c>
        device_attach() <drivers/base/bus.c>
          bus_for_each_drv() <drivers/base/dd.c>
            __device_attach() <drivers/base/dd.c>
              driver_match_device(drv, dev) <drivers/base/base.h>
              static inline int driver_match_device(struct device_driver *drv,
                                                    struct device *dev)
              {
                      return drv->bus->match ? drv->bus->match(dev, drv) : 1;
              }
```
* i2c_register_adapter()详细代码如下：
```
drivers/i2c/i2c-core.c
static int i2c_register_adapter(struct i2c_adapter *adap)
{
        int res = 0;

        /* Can't register until after driver model init */
        /* 判断i2c_bus_type的driver是否注册，因为需要将adapter device 注册到其上 */
        if (unlikely(WARN_ON(!i2c_bus_type.p))) {
                res = -EAGAIN;
                goto out_list;
        }

        /* Sanity checks */
        if (unlikely(adap->name[0] == '\0')) {
                pr_err("i2c-core: Attempt to register an adapter with "
                       "no name!\n");
                return -EINVAL;
        }
        if (unlikely(!adap->algo)) {
                pr_err("i2c-core: Attempt to register adapter '%s' with "
                       "no algo!\n", adap->name);
                return -EINVAL;
        }

        rt_mutex_init(&adap->bus_lock);
        mutex_init(&adap->userspace_clients_lock);
        INIT_LIST_HEAD(&adap->userspace_clients);

        /* Set default timeout to 1 second if not already set */
        if (adap->timeout == 0)
                adap->timeout = HZ;

        /* 注册主设备device设备模型, /sys/bus/devices/下出现"i2c-%d"的主设备
         * 即i2c主控制器设备 */
        dev_set_name(&adap->dev, "i2c-%d", adap->nr);
        adap->dev.bus = &i2c_bus_type;
        adap->dev.type = &i2c_adapter_type;
        res = device_register(&adap->dev);
        if (res)
                goto out_list;

        dev_dbg(&adap->dev, "adapter [%s] registered\n", adap->name);

#ifdef CONFIG_I2C_COMPAT
        res = class_compat_create_link(i2c_adapter_compat_class, &adap->dev,
                                       adap->dev.parent);
        if (res)
                dev_warn(&adap->dev,
                         "Failed to create compatibility class link\n");
#endif

        /* create pre-declared device nodes */
        /* 如果为真，就是说明使用的是静态注册的i2c_adapter
         * 当使用i2c_register_board_info注册从设备时，其参数busnum指定的总线的i2c_adapter必须要静态注册。*/
        if (adap->nr < __i2c_first_dynamic_bus_num)
                i2c_scan_static_board_info(adap);

        /* Notify drivers */
        mutex_lock(&core_lock);
        bus_for_each_drv(&i2c_bus_type, NULL, adap, __process_new_adapter);
        mutex_unlock(&core_lock);

        return 0;

out_list:
        mutex_lock(&core_lock);
        idr_remove(&i2c_adapter_idr, adap->nr);
        mutex_unlock(&core_lock);
        return res;
}
```
  * i2c_scan_static_board_info()遍历__i2c_board_list链表，找到与主控制器adapter相同busnum的从设备i2c_devinfo结构。然后调用i2c_new_device()创建i2c_client结构并注册到i2c bus。i2c_board_list与i2c_devinfo会在slave driver中再次阐明。
  * 创建完i2c_client，遍历i2c bus的驱动，调用__process_new_adapter()。完成从设备的侦测。
* 注册流程
```
                  define i2c_adapter
                         |
                         |
                  i2c_register_adapter
                         |
      |------------------|-----------------------------|
      |                  |                             |
      |                  |                             |
device_register    i2c_scan_static_board_info    bus_for_each_drv
 (&adap->dev)         (adap)                     (&i2c_bus_type, NULL,
                                                  adap, __process_new_adapter);
                         |                             |
                         |                             |
                   list_for_each_entry          __process_new_adapter
                         |                             |
                         |                             |
                     i2c_new_device                i2c_do_add_adapter
                         |
                         |
                  malloc and initialize
                     i2c_client
```
* i2c-imx.c中将i2c_imx_driver注册成为platform driver，然后在i2c_imx_probe()函数中创建并初始化adapter，然后将adapter注册到i2c bus上。
```
/* Add I2C adapter */
ret = i2c_add_numbered_adapter(&i2c_imx->adapter);
```
# i2c slave #
## Register slave device ##
* 一个slave device用i2c_client描述。
```
include/linux/i2c.h
struct i2c_client {
        unsigned short flags;           /* div., see below              */
        unsigned short addr;            /* chip address - NOTE: 7bit    */
                                        /* addresses are stored in the  */
                                        /* _LOWER_ 7 bits               */
        char name[I2C_NAME_SIZE];
        struct i2c_adapter *adapter;    /* the adapter we sit on        */
        struct i2c_driver *driver;      /* and our access routines      */
        struct device dev;              /* the device structure         */
        int irq;                        /* irq issued by device         */
        struct list_head detected;
};
```
  * flag:
    - I2C_CLIENT_TEN: 地址为十位。
```
--------------------------------------------------------------------------
S | Slave addr 1st part r/w | A | Slave addr 2nd part | A | Data | A | P |
--------------------------------------------------------------------------
   1 1 1 1 0 x x                  x x x x x x x x
```
    - 第一个字节为1110b+addr高2位，第二个字节为低8位addr
* 一般向bus注册i2c从设备的时候是在板级代码中加入的，其是在arch_initcall()时调用的。而主控制器adapter驱动使用的是module_init()或subsys_initcall()，总之都是会晚于arch_initcall()。初始化i2c_client需要指定其主控制器i2c_adapter，而i2c_adapter晚于i2c_client。所以，在板极程序中，还不能直接注册i2c_client，而只是保存在一个全局的链表__i2c_board_list中，格式为i2c_devinfo。
  * 在板级代码中，使用i2c_board_info数据结构定义从设备。
```
include/linux/i2c.h
struct i2c_board_info {
        char            type[I2C_NAME_SIZE];
        unsigned short  flags;
        unsigned short  addr;
        void            *platform_data;
        struct dev_archdata     *archdata;
        struct device_node *of_node;
        int             irq;
};
```
    - type: i2c_client的name。设备的名字。
    - flag与addr对应为i2c_client的flag与addr。
    - platform_data: 私有数据。
    - irq: 使用的irq号。
  * 可以使用宏定义来简单定义i2c_board_info
```
include/linux/i2c.h
#define I2C_BOARD_INFO(dev_type, dev_addr) \
        .type = dev_type, .addr = (dev_addr)
```
  * 每一个i2c主控制器挂载的从设备，都保存在相应i2c_board_info的数组里。如：I2C0的所有从设备list。
```
static struct i2c_board_info mxc_i2c0_board_info[] __initdata = {
        {
                I2C_BOARD_INFO("wm89**", 0x1a),
        },
        {
                I2C_BOARD_INFO("ov564x", 0x3c),
                .platform_data = (void *)&camera_data,
        },
        {
                I2C_BOARD_INFO("mma8451", 0x1c),
                .platform_data = (void *)&mma8451_position,
        },
};
```
  * 定义完从设备，就可以向i2c bus注册了。
```
drivers/i2c/i2c-boardinfo.c
int
i2c_register_board_info(int busnum, struct i2c_board_info const *info,
                        unsigned n);
```
    - busnum: 指定要向哪个i2c bus(i2c adapter)加入该设备
    - info: 上面定义的设备list
    - n: 为设备list的个数
  * i2c_register_board_info()为每一个i2c_board_info创建一个i2c_devinfo结构，然后添加到__i2c_board_list中。所有设备都可以在__i2c_board_list中找到。
```
drivers/i2c/i2c-boardinfo.c
int __init
i2c_register_board_info(int busnum,
        struct i2c_board_info const *info, unsigned len)
{
        int status;

        down_write(&__i2c_board_lock);

        /* dynamic bus numbers will be assigned after the last static one */
        if (busnum >= __i2c_first_dynamic_bus_num)
                __i2c_first_dynamic_bus_num = busnum + 1;

        for (status = 0; len; len--, info++) {
                struct i2c_devinfo      *devinfo;

                devinfo = kzalloc(sizeof(*devinfo), GFP_KERNEL);
                if (!devinfo) {
                        pr_debug("i2c-core: can't register boardinfo!\n");
                        status = -ENOMEM;
                        break;
                }

                devinfo->busnum = busnum;
                devinfo->board_info = *info;
                list_add_tail(&devinfo->list, &__i2c_board_list);
        }

        up_write(&__i2c_board_lock);

        return status;
}
```
  * i2c_register_board_info的流程
```
    define i2c_board_info
           |
           |
    i2c_register_board_info
           |
           |
    malloc and initialize
        i2c_devinfo
           |
           |
    add i2c_devinfo to
       __i2c_board_list
```
  * 什么时候，从设备才会真正地注册到i2c bus上呢？在主控制器注册到i2c bus时，使用了i2c_register_adapter()。
    - i2c_register_adapter() --> i2c_scan_static_board_info()遍历__i2c_board_list
    - 这时，adapter与i2c device匹配完成。
* 以上提到的都是i2c从设备的静态注册，即使用i2c_register_board_info()。另外，还有一种方法，侦测从设备。下面进行说明。
## Register slave driver ##
* slave driver使用的是i2c_driver数据结构。其结构与platform_driver类似。
```
struct i2c_driver {
        /* Standard driver model interfaces */
        int (*probe)(struct i2c_client *, const struct i2c_device_id *);
}
```
  - probe 函数中的i2c_device_id为device的id table，即i2c_driver->id_table。
  - device id有两个功能：一个是用于slave driver与slave device匹配，当name相同的时候，匹配成功，触发probe函数。另一个作用是传递给probe函数，作为私有数据，使device driver可以支持多个device的情况。
  - device id使用如下：
```
enum rtc_type {
        rtc_undef = 0,
        rtc_r2025sd,
        rtc_rs5c372a,
        rtc_rs5c372b,
        rtc_rv5c386,
        rtc_rv5c387a,
};

static const struct i2c_device_id rs5c372_id[] = {
        { "r2025sd", rtc_r2025sd },
        { "rs5c372a", rtc_rs5c372a },
        { "rs5c372b", rtc_rs5c372b },
        { "rv5c386", rtc_rv5c386 },
        { "rv5c387a", rtc_rv5c387a },
        { }
};
```
* i2c_driver是通过i2c_add_driver()注册到i2c bus上的。
```
module_i2c_driver(rs5c372_driver)
  module_driver
    i2c_add_driver(rs5c372_driver)
      i2c_register_driver(THIS_MODULE, rs5c372_driver) <i2c-core.c>
```
  - 在i2c_register_driver()中，主要做了两件事。
    - driver_register()注册device_driver()
    - 遍历i2c bus上的所有设备，执行__process_new_driver()
```
drivers/i2c/i2c-core.c
static int __process_new_driver(struct device *dev, void *data)
{
        if (dev->type != &i2c_adapter_type)
                return 0;
        return i2c_do_add_adapter(data, to_i2c_adapter(dev));
}
```
    - 如果device是adapter，则i2c_do_add_adapter()，这个就是动态侦测函数。
    - 这里判断的是i2c_adpater，因为我们要在i2c_driver驱动中侦测所有i2c_adapter上的符合该驱动的i2c_client。
* 在i2c bus上有两种设备：i2c_adapter和i2c_board_info静态注册的从设备。

# Reference #
* [Linux I2c Driver的整体分析](http://blog.csdn.net/w2baby1314/article/details/8081993)
* [linux设备驱动那点事儿之I2C驱动理论篇](http://blog.chinaunix.net/uid-26993600-id-3266678.html)
