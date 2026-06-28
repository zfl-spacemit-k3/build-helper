# 在进迭时空 k3 上运行 arceos helloworld

## 启动顺序

根据串口的打印日志逐步分析，最终得到的启动顺序如下：

```cpp
FSBL -> OPENSBI -> U-boot -> Linux
```

1. FSBL 初始化 DDR （DDR 初始化函数在 U-boot 源码的 drivers/ddr/spacemit/k3/ddr_init.c 中）之后，会通过 `spl_invoke_opensbi(&spl_image);`（代码在 common/spl/spl.c） 跳转至 opensbi；

```bash
sys: 0x10000200
try sd...
bm:3
ERROR:   CMD8
ERROR:   sd f! l:90
bm:4
nor m:0xc8 d:0x6017
j...

U-Boot SPL 2022.10 (Apr 28 2026 - 15:32:04 +0000)
DDR Part Number: MT62F1G32D2DS, Size: 4096MB, Data Rate: 6400MT/s
zfl debug spacemit_ddr_probe 252
DDR quick boot consume 9ms
zfl debug spl_board_init_f 183
Jumping to U-Boot via RISC-V OpenSBI
```

2. `spl_invoke_opensbi` 函数会构造出 `opensbi_info` 并跳转至 opensbi，由于默认没有使能 opensbi 的 log，因此串口不会打印任何 opensbi 的信息，需要修改 opensbi 的 config（`CONFIG_ENABLE_LOGGING=y`）。

```bash
Jumping to U-Boot via RISC-V OpenSBI
WARNING: riscv,rpmi-hsm driver is experimental and may change
WARNING: riscv,rpmi-shmem-mbox driver is experimental and may change
WARNING: riscv,rpmi-system-reset driver is experimental and may change
WARNING: riscv,rpmi-system-suspend driver is experimental and may change

OpenSBI k3-br-v1.0.0
   ____                    _____ ____ _____
  / __ \                  / ____|  _ \_   _|
 | |  | |_ __   ___ _ __ | (___ | |_) || |
 | |  | | '_ \ / _ \ '_ \ \___ \|  _ < | |
 | |__| | |_) |  __/ | | |____) | |_) || |_
  \____/| .__/ \___|_| |_|_____/|____/_____|
        | |
        |_|

WARNING: riscv,rpmi-mpxy-clock driver is experimental and may change
WARNING: riscv,rpmi-mpxy-voltage driver is experimental and may change
WARNING: riscv,rpmi-mpxy-domain driver is experimental and may change
WARNING: riscv,rpmi-mpxy-rtc driver is experimental and may change
WARNING: riscv,rpmi-mpxy-pwrkey driver is experimental and may change
Platform Name               : spacemit k3 com260 ifx board
Platform Features           : medeleg
Platform HART Count         : 16
Platform IPI Device         : aia-imsic
Platform Timer Device       : aclint-mtimer @ 24000000Hz
Platform Console Device     : uart8250
Platform HSM Device         : rpmi-hsm
Platform PMU Device         : ---
Platform Reboot Device      : rpmi-system-reset
Platform Shutdown Device    : rpmi-system-reset
Platform Suspend Device     : rpmi-system-suspend
Platform CPPC Device        : ---
Firmware Base               : 0x100000000
Firmware Size               : 642 KB
Firmware RW Offset          : 0x40000
Firmware RW Size            : 386 KB
Firmware Heap Offset        : 0x86000
Firmware Heap Size          : 106 KB (total), 6 KB (reserved), 18 KB (used), 81 KB (free)
Firmware Scratch Size       : 8192 B (total), 5816 B (used), 2376 B (free)
Runtime SBI Version         : 2.0
Standard SBI Extensions     : time,rfnc,ipi,base,hsm,srst,susp,pmu,dbcn,legacy
Experimental SBI Extensions : fwft,dbtr,sse,mpxy

Domain0 Name                : root
Domain0 Boot HART           : 0
Domain0 HARTs               : 0*,1*,2*,3*,4*,5*,6*,7*,8*,9*,10*,11*,12*,13*,14*,15*
Domain0 Region00            : 0x00000000cac90c00-0x00000000cac90cff M: (I,R,W) S/U: ()
Domain0 Region01            : 0x00000000d4017000-0x00000000d4017fff M: (I,R,W) S/U: (R,W)
Domain0 Region02            : 0x00000000f1800000-0x00000000f1803fff M: (I,R,W) S/U: ()
Domain0 Region03            : 0x0000000100800000-0x0000000100803fff M: (I,R,W) S/U: ()
Domain0 Region04            : 0x00000000f1000000-0x00000000f100ffff M: (I,R,W) S/U: ()
Domain0 Region05            : 0x00000000f1810000-0x00000000f181ffff M: (I,R,W) S/U: ()
Domain0 Region06            : 0x0000000100000000-0x000000010003ffff M: (R,X) S/U: ()
Domain0 Region07            : 0x0000000100000000-0x00000001000fffff M: (R,W) S/U: ()
Domain0 Region08            : 0x0000000000000000-0xffffffffffffffff M: () S/U: (R,W,X)
Domain0 Next Address        : 0x0000000102000000
Domain0 Next Arg1           : 0x000000010214cb40
Domain0 Next Mode           : S-mode
Domain0 SysReset            : yes
Domain0 SysSuspend          : yes

Boot HART ID                : 0
Boot HART Domain            : root
Boot HART Priv Version      : v1.12
Boot HART Base ISA          : rv64imafdcbvhx
Boot HART ISA Extensions    : smaia,smstateen,sscofpmf,sstc,zicntr,zihpm,smcntrpmf,zicboz,zicbom,svpbmt,sdtrig
Boot HART PMP Count         : 1
Boot HART PMP Granularity   : 2 bits
Boot HART PMP Address Bits  : 23
Boot HART MHPM Info         : 16 (0x0007fff8)
Boot HART Debug Triggers    : 4 triggers
Boot HART MIDELEG           : 0x0000000000003666
Boot HART MEDELEG           : 0x0000000000f0b509
[   1.341]
```

3. U-boot 会根据 `env.bin` 镜像中的脚本读取 `bootfs` 分区中的 env_k3.txt 配置信息，并从中获取 `bootfs` 中的 kernel，dtb 以及 ramdisk，并依次加载，最后跳转至 Linux。

```bash
# env_k3.txt
knl_name=vmlinuz-6.18.3-generic
ramdisk_name=initrd.img-6.18.3-generic
dtb_dir=spacemit/6.18.3-generic
ramdisk_addr=0x130000000
loglevel=8
commonargs=setenv bootargs plymouth.prefer-fbcon plymouth.ignore-serial-consoles splash
```

```bash
U-Boot 2022.10 (Apr 28 2026 - 15:32:04 +0000)

[   1.344] CPU:   rv64imafdcvh
[   1.346] Model: spacemit k3 com260 ifx board
[   1.351] DRAM:  8 GiB
[   1.377] reset driver probe finish
[   1.537] eSPI not ready, skipping EC probe
[   1.541] Core:  626 devices, 35 uclasses, devicetree: board
[   1.544] WDT:   Started watchdog@d4014000 with servicing (60s timeout)
[   1.550] MMC:   spacemit_sdhci sdh@d4280000: probe done.
[   1.555] sdh@d4280000: 0
[   1.558] Loading Environment from mtdENV... k1x_qspi spi@d420c000: qspi iobase:0x0x00000000d420c000, ahb_addr:0x0x00000000b8000000, max_hz:26000000Hz
[   1.571] k1x_qspi spi@d420c000: rx buf size:128, tx buf size:256, ahb buf size=512
[   1.578] k1x_qspi spi@d420c000: AHB read enabled
[   1.583] k1x_qspi spi@d420c000: AHB buf size: 512
[   1.588] k1x_qspi spi@d420c000: Speed Change: 26000000 Hz -> 26500000 Hz
[   1.594] SF: Detected gd25lq64c with page size 256 Bytes, erase size 4 KiB, total 8 MiB
OK
[   1.852] ufs: bRefClkFreq current=0 expected=0
[   1.907] ufs-spacemit_k3 ufs@c0e00000: [RX, TX]: gear=[3, 3], lane[2, 2], pwr[FAST MODE, FAST MODE], rate = 2
[   1.924] bootlogo loaded from ufs_fs (0:2)
[   1.927] Found device 'mipi@d421a800', disp_uc_priv=00000002fbea2310
[   1.931] spacemit_panel_of_to_plat panel lcd_tc358762xbg_dpi_800x480
[   1.949] pll_ctrl_reg0 = 0x3A155555
[   1.949] pll_ctrl_reg1 = 0x4582820B
[   1.953] Panel is lcd_tc358762xbg_dpi
[   1.957] fb=2fe000000, size=800x480
[   1.961] dpi_panel_probe: Warning: cannot get enable GPIO: ret=-2
[   2.076] dpi_panel_probe: Atmel I2C read failed: 0
[   2.078] panel device error -1
[   2.082] Found device 'dp1@cac88000', disp_uc_priv=00000002fbea2250
[   2.253] fb=2fe000000, size=1920x1080
[   2.255] [DP PHY INFO] DPCD: Rev 1.2, MaxRate 5400000 kHz, MaxLanes 4, EnhFrame 128
[   2.261] [DP PHY INFO] DP: Mode Set 1920x1080 (PCLK: 148500 kHz)
[   2.289] [DP PHY INFO] Link Training: Using TPS3
[   2.320] [DP PHY INFO] DP: Training successful for R:2700000 L:2
[   2.323] [DP PHY INFO] MSA: 1920x1080, Rate:2700000 kHz, Lanes:2, BPP:24, TU:52.8
[   2.334] In:    serial
[   2.334] Out:   serial
[   2.336] Err:   serial
[   2.338] zfl debug board_late_init 1054
[   2.343] CTF2301: Start probing ctf2301@4c
[   2.347] Found 4 valid MAC addresses.
[   2.349] TLV item: product_name = k3_com260_ifx
[   2.354] TLV item: serial# = COM3K3081280115
[   2.358] ## Error: Can't overwrite "serial#"
[   2.362] ## Error inserting "serial#" variable, errno=1
[   2.367] TLV item: ddr_partnumber = MT62F1G32D2DS
[   2.373] spacemit reboot: read PMIC reg 0xab value 0xf0
[   2.378] SRAM cleared: addr=0xc0800000 size=0x80000
[   2.382] 238 bytes read in 0 ms
[   2.385] ## Info: input data size = 239 = 0xEF
[   2.389] load env_k3.txt from bootfs successful
[   2.394] Autoboot in 0 seconds
[   2.406] Try to boot from scsi0 ...
[   2.408] product_name: k3_com260_ifx
[   2.410] match dtb by product_name: spacemit/6.18.3-generic/k3_com260_ifx.dtb
[   2.417] select spacemit/6.18.3-generic/k3_com260_ifx.dtb to load
[   2.423] Loading kernel...
[   2.426] 61504 bytes read in 0 ms
[   2.429] Loading dtb...
[   2.432] 142063 bytes read in 0 ms
[   2.435] Loading ramdisk ...
[   2.463] 15146950 bytes read in 25 ms (577.8 MiB/s)
```

## 启动 arceos helloworld

按照上述的启动顺序，只需要将 arceos helloworld 的镜像添加至 `bootfs` 分区，修改 `env_k3.txt` 中的 `knl_name`，最后由于 U-boot 与 Linux 之间没有特权级切换等操作，可以直接通过 go 指令跳转至 arceos helloworld 镜像所在的地址运行（这里要修改 `env.bin`，具体是在 U-boot 源码的 `board/spacemit/k3/k3.env` 中根据文件名是否包含 vmlinuz 进行判断，修改方式在本文最后进行了描述）。

但是由于很久没有跟踪 arceos 的进展了，arceos 使用的 `axplat` 的用法对我产生了一点困扰，以及 AI 工具给我制造了很大的困扰。

1. 尽管 axplat 的 readme 已经较为清晰了，但还是造成了一定的困扰，我很长一段时间认为自己定义的 platform 依赖是添加到 arceos 项目的根目录的，
但实际上需要添加到 `examples/helloworld` 目录下；并且需要在 `main.rs` 中声明 `extern crate axplat_riscv64_spacemitk3;`，否则编译的产物是空的，
这个是我根据 arceos 的 ci 里的 `aarch64-phytium-pi` 脚本才明白的。并且 通用的 platform 中的定义有几个常量我也不太熟悉了，例如 `kernel-aspace-base`，
导致我调试时错误的硬编码虚拟地址。
1. AI 工具给我造成的困扰是在使能 MMU 的时候，由于 arceos 会在最开始建立一个恒等映射以及高位虚拟地址的映射并且马上使能 MMU，最初 AI 工具写的代码计算出错误的 SV39 页表项导致 U-boot 跳转至 arceos 后没有任何输出，过段时间会直接重启；我尝试直接使用 ecall 来调用 opensbi 提供的 print 来进行 debug。最后发现，一旦使能 MMU 之后就会失败重启；
   1. 尝试了只建立对等映射，让虚拟地址等于物理地址，但使能 MMU 之后仍然会失败；最后尝试直接跳过使能 MMU 的部分才完整运行了 helloworld。
   2. U-boot 会开启 Dcache，在 `init_boot_page_table` 后，还需要通过 `core::arch::asm!("fence iorw, iorw", "fence.i");` 确保数据写到 DDR 上；
   3. 最后使用其他的 AI 工具重新计算页表项之后，代码才正常运行。

## 如何增加调试日志

### U-boot

在 yocto 的编译系统中，在编译过一次完整的镜像之后，yocto 会将 U-boot 的源码下载下来（路径为 `build/tmp/work/k3-poky-linux/u-boot-k3/2022.10/sources/u-boot-k3-2022.10`），并且对其应用上 yocto bb 文件（路径为 `layers/meta-riscv/recipes-bsp/u-boot/u-boot-k3_2022.10.bb`）中规定的 patch（路径为 `layers/meta-riscv/recipes-bsp/u-boot/u-boot-k3/0001-Disable-source-tree-clean-check.patch`）。

bb 文件中可以看到

```bash
SRC_URI = "git://github.com/spacemit-com/uboot-2022.10.git;protocol=https;tag=k3-br-v1.0.0;nobranch=1 \
           file://0001-Disable-source-tree-clean-check.patch \
           file://env_k3.txt \
           file://spacemit.bmp"
```

这表示 yocto 在编译 U-boot 的时候，会从 github 上拉取代码，并且应用上 patch 以及文件。因此增加 U-boot 的调试日志就是对 `0001-Disable-source-tree-clean-check.patch` 进行修改，具体的方式如下：

1. 在 U-boot 源码（`build/tmp/work/k3-poky-linux/u-boot-k3/2022.10/sources/u-boot-k3-2022.10`，这里会有 yocto 之前的 patch）下对应的文件增加你的调试信息；
2. 使用 git diff 生成具体的 patch；
3. 使用上一步中的 patch 的内容来替换 `0001-Disable-source-tree-clean-check.patch` 中的内容，需要保留 `Upstream-Status` 信息，否则会出现检查 bb 文件格式报错；

### opensbi

与 U-boot 不同的是，在 `layers/meta-riscv/recipes-bsp/opensbi` 目录下只有 `layers/meta-riscv/recipes-bsp/opensbi/opensbi-k3_1.4.bb` bb 文件而没有对应的 patch；因此操作略有不同：
1. 此时需要手动创建一个 `opensbi-k3` 目录（`opensbi-k3_1.4.bb` 去掉 `_1.4.bb`）；
2. 按照上一小节 U-boot 的方式将 git diff 的 patch 放到这个 `opensbi-k3` 目录下；
3. 修改 `opensbi-k3_1.4.bb` 文件中的 `SRC_URI`，增加上 patch 的相关信息；

### bootfs

涉及 bootfs 的修改为：

1. 将 arceos helloworld 的镜像打包到 bootfs；
2. 将 env_k3.txt 的 knl_name 替换；

具体的操作如下：

1. 将 arceos helloworld 的镜像复制到 yocto 最终编译产物的目录（`build/tmp/deploy/images/k3`）；
2. 修改最终编译产物的 bb 文件，具体修改如下图所示；
3. 使用 `BB_NO_NETWORK=0 MACHINE=k3 bitbake core-image-weston -c clean && BB_NO_NETWORK=0 MACHINE=k3 bitbake core-image-weston` 重新生成最终编译产物，最终编译产物的目录下可以看到 `bootfs` 目录下已经有了 arceos helloworld 镜像，并且 a 打包到 bootfs.ext 中；

！[alt text](。/assets/image-k3-diff.png)

## 注意

刷机时，涉及到 FSBL，U-boot 或者 opensbi 的修改，需要使用 `partition_4M.json` 分区表，刷新 bootfs 分区以及 rootfs 分区的时候，需要使用 `partition_universal.json` 分区表；

在使用 `partition_universal.json` 刷新 U-boot 等分区时，不会生效。