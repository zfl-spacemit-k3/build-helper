# fastboot 刷机

1. 连接上串口，注意，板子附带的 usb 的线白色接板子上的 uart-txd， 绿色的线接 uart-rxd；
2. 连接上串口后，先登陆进入它的 bianbu 系统，然后使用 reboot 让其重启；
3. 重启之后，在串口最开始打印 u-boot 的信息时，按 s 键进入 u-boot 的 console；
4. 输入 `fastboot 0` 开始进入刷机模式；
5. 后续的操作按照官方的 [刷机文档](https://spacemit.com/community/document/info?lang=zh&nodepath=tools/user_guide/flasher_user_guide.md) 来进行，需要配置刷机分区文件为 `partition_universal.json`；
6. 刷机工具显示开始烧录固件后，再输入 `fastboot 0`，同时，串口会打印刷机过程中读写的分区等信息；
7. 如果刷机时勾选了自动开机，刷机完成后，会自动进入到官方的 bianbu 系统；