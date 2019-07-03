# uboot2016 开发过程
### 1. 建立单板
```sh
1. 下载源代码：
wget ftp://ftp.denx.de/pub/u-boot/u-boot-2016.05.tar.bz2

2. 解压：
tar xvf u-boot-2016.05.tar.bz2

3. 建立新单板的相关文件
cd u-boot-2016.05/board/samsung/
  cp smdk2410 smdk2440 -rf
cd xxx/u-boot-2016.05
  cp include/configs/smdk2410.h  include/configs/smdk2440.h
  cp configs/smdk2410_defconfig  configs/smdk2440_defconfig

4. 修改相关参数
  因为我们的新板（2440）是以2410为模板进行建立的，所以我们在uboot的根目录下搜索2410的相关信息，然后将其改成2440即可。

  cd xxx/u-boot-2016.05
  grep "smdk2410" -nR
scw@scw-VirtualBox:~/weidongshan/high/u-boot-2016.05$ grep "smdk2410" -nR
arch/arm/Kconfig:96:	bool "Support smdk2410"
arch/arm/Kconfig:856:source "board/samsung/smdk2410/Kconfig"
arch/arm/include/asm/mach-types.h:1646:# define machine_is_smdk2410()	(machine_arch_type == MACH_TYPE_SMDK2410)
arch/arm/include/asm/mach-types.h:1648:# define machine_is_smdk2410()	(0)
board/samsung/smdk2440/Kconfig:4:	default "smdk2410"
board/samsung/smdk2440/Kconfig:13:	default "smdk2410"
board/samsung/smdk2440/MAINTAINERS:4:F:	board/samsung/smdk2410/
board/samsung/smdk2440/MAINTAINERS:5:F:	include/configs/smdk2410.h
board/samsung/smdk2440/MAINTAINERS:6:F:	configs/smdk2410_defconfig
board/samsung/smdk2440/Makefile:8:obj-y	:= smdk2410.o (若是需要修改此处，请将smdk2410.c改为smdk2440.c)
board/samsung/smdk2410/Kconfig:4:	default "smdk2410"
board/samsung/smdk2410/Kconfig:13:	default "smdk2410"
board/samsung/smdk2410/MAINTAINERS:4:F:	board/samsung/smdk2410/
board/samsung/smdk2410/MAINTAINERS:5:F:	include/configs/smdk2410.h
board/samsung/smdk2410/MAINTAINERS:6:F:	configs/smdk2410_defconfig
board/samsung/smdk2410/Makefile:8:obj-y	:= smdk2410.o


5. 编译
make smdk2440_config
make -j10
```

![](../image/uboot/1.png)
