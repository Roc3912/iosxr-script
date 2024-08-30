1、**安装前的准备**

格式为FAT32的U盘一个

cisco官网下载有turboboot字样的安装文件

## 2、USB安装步骤如下

对于ASR9001，USB插入后显示为disk1；对于CRS，USB插入后显示为disk2。

使用如下的命令重启设备，会自动进入rommon模式，或者直接断电重加电，CTRL+C进入rommon。

```
admin  config-register 0x0  
admin  reload location all  
```

下面是使用USB启动一台设备时的配置，

```
rommon 15 > unset BOOT  <<< 取消参数BOOT的值
rommon 16 > TURBOBOOT=on,disk0,format
rommon 17 > sync     <<<保存
rommon 18 > set     <<<查看rommon的配置
PS1=rommon ! > 
?=0
TURBOBOOT=on,disk0,format
rommon 19 > 
rommon 19 > 
rommon 19 > boot usb:asr9k-mini-px.vm-6.2.3  <<<相对于普通的install方式升级的文件，可以看到多了vm字样
Beginning Media boot:
**** check image validation ****
.......BIOS CODE SIGN ENTRY ...
Image ASR9K-Tomahawk verified successfully
boot_disk2 - Launching image.
```

## 3、TFTP配置如下

```
rommon1>IP_ADDRESS=10.75.49.45
rommon2>IP_SUBNET_MASK=255.255.255.0
rommon3>TFTP_SERVER=10.75.49.254
rommon4>DEFAULT_GATWAY=10.75.49.1
rommon5>TFTP_RETRY_COUNT=4
rommon6>TFTP_TIMEOUT=60
rommon7>TFTP_CHECKSUM=1
rommon8>priv
rommon9>diswd
rommon10>unset BOOT
rommon11>TURBOBOOT=on,disk0,format
rommon12>sync
rommon13>boot tftp://10.75.49.254/asr9k-mini-px.vm-6.2.3
```

 #more  nvram:classic-rommon-var  可以查看turboboot设置的信息  

#Show  bootvar  查看设备是如何去boot的  