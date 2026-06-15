# 通过 yocto 编译系统编译刷机镜像

官方编译指南：https://forum.spacemit.com/t/topic/1239

编译步骤具体如下：

1. 按照官方编译指南，在 ubuntu 上安装相应的依赖库，并且使用 repo 工具同步对应的代码，拉取代码结束后，目录下只有 .repo 和 layers 两个目录；
2. 在创建的 riscv-yocto 目录下执行 `cp -r ./layers/openembedded-core .` 复制 openembedded-core 到 riscv-yocto 目录下；
3. 按照官方编译指南，在 riscv-yocto 目录下执行 `. layers/meta-riscv/tools/envsetup.sh`, 执行结束后会进入到脚本创建的 build 目录下；
4. 按照官方编译指南，编译出需要的镜像，可能会出现网络错误，这里需要使用 ` BB_NO_NETWORK=0 MACHINE=k3 bitbake core-image-weston` 来开启网络；

## 单独编译某个模块

上述的编译过程会持续很久，接下来以编译 u-boot 为例子来描述编译某个模块的方法。

yocto 编译系统通过 bitbake 的 recipes 来编译某个模块，u-boot 的对应的路径在 `riscv-yocto/layers/meta-riscv/recipes-bsp/u-boot` 下。

```bash
➜  u-boot git:(work) pwd
/home/zfl/spacemit/riscv-yocto/layers/meta-riscv/recipes-bsp/u-boot
➜  u-boot git:(work) ll
总计 72K
drwxrwxr-x 3 zfl zfl 4.0K  6 月 15 22:45 boot-bundle
-rw-rw-r-- 1 zfl zfl 1.3K  6 月 15 22:45 boot-bundle.bb
drwxrwxr-x 4 zfl zfl 4.0K  6 月 15 22:45 files
drwxrwxr-x 2 zfl zfl 4.0K  6 月 15 22:45 u-boot-allwinnerd1
-rw-rw-r-- 1 zfl zfl 1.6K  6 月 15 22:45 u-boot-allwinnerd1.bb
-rw-rw-r-- 1 zfl zfl 5.0K  6 月 15 22:45 u-boot_%.bbappend
drwxrwxr-x 2 zfl zfl 4.0K  6 月 15 22:45 u-boot-beaglev-ahead
-rw-rw-r-- 1 zfl zfl 1.3K  6 月 15 22:45 u-boot-beaglev-ahead.bb
drwxrwxr-x 2 zfl zfl 4.0K  6 月 15 22:45 u-boot-k3
-rw-rw-r-- 1 zfl zfl 1.5K  6 月 15 22:45 u-boot-k3_2022.10.bb
drwxrwxr-x 5 zfl zfl 4.0K  6 月 15 22:45 u-boot-orangepi
-rw-rw-r-- 1 zfl zfl 1.7K  6 月 15 22:45 u-boot-orangepi.bb
-rw-rw-r-- 1 zfl zfl 1.2K  6 月 15 22:45 u-boot-spl-k1.bb
drwxrwxr-x 2 zfl zfl 4.0K  6 月 15 22:45 u-boot-starfive
-rw-rw-r-- 1 zfl zfl 1.6K  6 月 15 22:45 u-boot-starfive_v2021.04.bb
-rw-rw-r-- 1 zfl zfl  927  6 月 15 22:45 u-boot-starfive_v2021.07.bb
-rw-rw-r-- 1 zfl zfl 1.2K  6 月 15 22:45 u-boot-starfive_v2021.10.bb
```

这里的 `u-boot-k3_2022.10.bb` 文件就是编译 u-boot 所需要的配方。下面将开始单独编译 u-boot；

```bash
# 进入 envsetup.sh 脚本创建的 build 目录
cd riscv-yocto/build
# 将 core-image-weston 替换成那个 u-boot-k3
# 这个 u-boot-k3 指的是配方文件名 u-boot-k3_2022.10.bb 去掉 _2022.10.bb 的部分
BB_NO_NETWORK=0 MACHINE=k3 bitbake u-boot-k3
```

不出意外，这里将会单独编译出 u-boot，编译的结果在 `riscv-yocto/build/tmp/work/k3-poky-linux/u-boot-k3/2022.10/image` 目录下：

```bash
➜  image pwd
/home/zfl/spacemit/riscv-yocto/build/tmp/work/k3-poky-linux/u-boot-k3/2022.10/image
➜  image tree .
.
├── boot
│   ├── u-boot.bin -> u-boot-k3-2022.10-r0.bin
│   └── u-boot-k3-2022.10-r0.bin
└── etc
    ├── u-boot-k3-initial-env -> u-boot-k3-initial-env-k3-2022.10-r0
    ├── u-boot-k3-initial-env-k3 -> u-boot-k3-initial-env-k3-2022.10-r0
    └── u-boot-k3-initial-env-k3-2022.10-r0

2 directories, 5 files
```

在 `riscv-yocto/build/tmp/deploy/images/k3` 目录下也有，两者的 md5 码是相同的：

```bash
➜  k3 pwd
/home/zfl/spacemit/riscv-yocto/build/tmp/deploy/images/k3
➜  k3
➜  k3 ll
总计 4.4M
-rw-r--r-- 2 zfl zfl   80  6 月 15 23:18 bootinfo_block.bin
-rw-r--r-- 2 zfl zfl   80  6 月 15 23:18 bootinfo_spinand.bin
-rw-r--r-- 2 zfl zfl   80  6 月 15 23:18 bootinfo_spinor.bin
-rw-r--r-- 2 zfl zfl  176  6 月 15 23:18 env_k3.txt
-rw-r--r-- 2 zfl zfl 443K  6 月 15 23:18 FSBL.bin
-rw-r--r-- 2 zfl zfl 469K  6 月 15 23:18 spacemit.bmp
lrwxrwxrwx 2 zfl zfl   24  6 月 15 23:18 u-boot.bin -> u-boot-k3-2022.10-r0.bin
-rw-r--r-- 2 zfl zfl  16K  6 月 15 23:18 u-boot-env-default.bin
-rw-r--r-- 2 zfl zfl 2.1M  6 月 15 23:18 u-boot.itb
-rw-r--r-- 2 zfl zfl 1.4M  6 月 15 23:18 u-boot-k3-2022.10-r0.bin
lrwxrwxrwx 2 zfl zfl   24  6 月 15 23:18 u-boot-k3.bin -> u-boot-k3-2022.10-r0.bin
lrwxrwxrwx 2 zfl zfl   35  6 月 15 23:18 u-boot-k3-initial-env -> u-boot-k3-initial-env-k3-2022.10-r0
lrwxrwxrwx 2 zfl zfl   35  6 月 15 23:18 u-boot-k3-initial-env-k3 -> u-boot-k3-initial-env-k3-2022.10-r0
-rw-r--r-- 2 zfl zfl  11K  6 月 15 23:18 u-boot-k3-initial-env-k3-2022.10-r0
➜  k3
➜  k3 md5sum u-boot-k3-2022.10-r0.bin
be387b55ff8581264aab762f198b014c  u-boot-k3-2022.10-r0.bin
➜  k3
➜  k3 cd ../../../work/k3-poky-linux/u-boot-k3/2022.10/image
➜  image
➜  image cd boot
➜  boot
➜  boot ll
总计 1.4M
lrwxrwxrwx 1 zfl zfl   24  6 月 15 23:18 u-boot.bin -> u-boot-k3-2022.10-r0.bin
-rw-r--r-- 3 zfl zfl 1.4M  6 月 15 23:18 u-boot-k3-2022.10-r0.bin
➜  boot md5sum u-boot-k3-2022.10-r0.bin
be387b55ff8581264aab762f198b014c  u-boot-k3-2022.10-r0.bin
```
