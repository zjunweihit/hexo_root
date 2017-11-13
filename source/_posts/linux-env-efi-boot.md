---
title: Ubuntu EFI
date: 2017-11-13 16:37:00
categories:
tags:
  - ubuntu
---

* EFI overview
* Boot ubuntu in EFI shell.
* Build EFI shell App.

<!--more-->

# EFI overview
* [uefi arch and tech](https://software.intel.com/en-us/articles/uefi-architecture-and-technical-overview/)
* [EFI load file protocol](http://wiki.phoenix.com/wiki/index.php/EFI_LOAD_FILE_PROTOCOL)
* [command in EFI shell](http://dreamfromars.blog.sohu.com/276531848.html)
* [Build hello in EFI shell](http://www.rodsbooks.com/efi-programming/hello.html)
* [UEFI development(get fb address by GOP)](https://www.tuicool.com/articles/7ZNRnm)

# EFI boot stub
* More info
  - [efi-stub.txt](https://www.mjmwired.net/kernel/Documentation/efi-stub.txt)
* Preparation
  - build kernel
```
make-kpkg -j 8 --rootcmd fakeroot --initrd --append-to-version=-custom kernel_image kernel_headers
```
  - copy efi, in EFI shell only /boot/efi/ will be accessed
```
sudo cp arch/x86/boot/bzImage /boot/efi/EFI/BOOT/bzImage.efi
```
* Boot into initramfs in EFI shell
```
FS1:
cd EFI
cd BOOT
bzImage.efi initrd=\EFI\BOOT\initrd.img-4.13.0-custom
```
* Boot rootfs in EFI shell
```
FS1:
cd EFI
cd BOOT
bzImage.efi initrd=\EFI\BOOT\initrd.img-4.13.0-custom root=/dev/sda2
```
  - by default the sda is separated as below in Ubuntu
```
/dev/sda1       2048   1050623   1048576   512M EFI System
/dev/sda2    1050624 217819135 216768512 103.4G Linux filesystem
/dev/sda3  217819136 234440703  16621568   7.9G Linux swap
```

# Build EFI shell App
* Create main.c
```
#include <efi.h>
#include <efilib.h>

EFI_STATUS
efi_main (EFI_HANDLE image_handle, EFI_SYSTEM_TABLE *systab)
{
	EFI_STATUS status;
	EFI_GRAPHICS_OUTPUT_PROTOCOL *gop;
	EFI_GUID gop_guid = EFI_GRAPHICS_OUTPUT_PROTOCOL_GUID;

	InitializeLib(image_handle, systab);

	status = uefi_call_wrapper(systab->BootServices->LocateProtocol,
			3,
			&gop_guid,
			NULL,
			&gop);

	Print(L"Framebuffer base is at %lx\n", gop->Mode->FrameBufferBase);

	return status;
}
```
* Create Makefile(take care of <tab> in the file)
```
ARCH            = $(shell uname -m | sed s,i[3456789]86,ia32,)

OBJS            = main.o
TARGET          = check_fb

EFIINC          = /usr/include/efi
EFIINCS         = -I$(EFIINC) -I$(EFIINC)/$(ARCH) -I$(EFIINC)/protocol
LIB             = /usr/lib
EFILIB          = /usr/lib
EFI_CRT_OBJS    = $(EFILIB)/crt0-efi-$(ARCH).o
EFI_LDS         = $(EFILIB)/elf_$(ARCH)_efi.lds

CFLAGS          = $(EFIINCS) -fno-stack-protector -fpic \
		-fshort-wchar -mno-red-zone -Wall 
ifeq ($(ARCH),x86_64)
	CFLAGS += -DEFI_FUNCTION_WRAPPER
endif

LDFLAGS         = -nostdlib -znocombreloc -T $(EFI_LDS) -shared \
		  -Bsymbolic -L $(EFILIB) -L $(LIB) $(EFI_CRT_OBJS) 

		  all: $(TARGET).efi

$(TARGET).so: $(OBJS)
	ld $(LDFLAGS) $(OBJS) -o $@ -lefi -lgnuefi

%.efi: %.so
	objcopy -j .text -j .sdata -j .data -j .dynamic \
		-j .dynsym  -j .rel -j .rela -j .reloc \
		--target=efi-app-$(ARCH) $^ $@
```
* make it
* copy it to /boot/BOOT/efi or an USB storage, any of which you can access in EFI shell
* run it directly
```
check_fb.efi
```

