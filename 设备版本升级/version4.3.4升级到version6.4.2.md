## 一、注意事项

## 1、version4.3.4升级到version6.4.2属于大版本升级，升级的时候需要有一个中间版本进行过渡（version 5.3.2及以上版本）

## 2、32-bit IOS XR Corrupt Pie File (SWIMS)证书到期

```
Affected Releases
•	Pre-5.3.2 does not contain the SWIMS signing and only supports Abraxas or the legacy Code Signing Server (CSS) software which is now completly deprecated
•	5.3.2 to 6.3.1 support both Abraxas and SWIMS
•	6.3.2 and above supports only SWIMS signing
•	Some latest SMU created after Abraxas server decommission (after 5.3.4 SP9) are also signed with SWIMS only.
Since 5.3.1 and prior releases supports only Abraxas (after Oct 17 2015) and 6.3.2 and above supports only SWIMS signing, routers cannot be upgraded from one to the other.
If you are running 5.3.1 or prior then you must upgrade to 5.3.2-6.3.1 first and then to 6.3.2 or higher.
Examples
Question: I am running 5.3.1 and wish to upgrade to 6.4.2. Will this work?
Answer: No, you must first upgrade to an intermediary release that supports SWIMS.
Question: I am running 5.3.4 and wish to upgrade to 6.4.2. Will this work?
Answer: Yes, as 5.3.4 supports both Abraxas and SWIMS.
Question: I am running 5.3.1 and wish to upgrade to 5.3.4 plus latest SMU. Will this work?
Answer: Upgrade to 5.3.4 first and then installing SMU on top will work. However activating both 5.3.4 + latest SMU at once will fail as 5.3.1 won't understand SMU signage.
```

version4.3.4证书文件已经上传！

## 3、 Cisco IOS-XR 5.3.0 及更早版本的 ASR9000、CRS 和 XR12000 - 2015 年 10 月 17 日 SW 证书到期（从5.3.X升级到6.x.x会出现）

```
Symptom:
When trying to install or add a SMU/PIE post October 17 2015, you will run into the below error due to the expiration of the CSS certificate on Oct 17, 2015.
Error: Cannot proceed with the add operation because the code signing
Error: certificate has expired.
Error: Suggested steps to resolve this:
Error: - check the system clock using 'show clock' (correct with 'clock set' if necessary).
Error: - check the pie file was built within the last 5 years using '(admin) show install pie-info
Error: /tftp://202.153.144.25/auto/tftp-sjc-users3/jamohamm/IMAGES/asr9k-mcast-px.pie-4.3.2'.
Post this expiration date, provided no new image, SMU or pie installs are required, existing customer installations will continue to operate as expected even if the customer reboots a presently installed image with existing SMUs.
```

## 4、新版本激活组件需要和旧版本激活组件保持一致，不然没办法正确激活

```
RP/0/RSP0/CPU0:RT1-A9010(admin)#install activate disk0:asr9k-doc-px$
Thu Aug 29 01:38:43.460 PeKing
Install operation 37 '(admin) install activate disk0:asr9k-doc-px-4.3.4'
started by user 'xp' via CLI at 01:38:43 PeKing Thu Aug 29 2024.
Error:    Cannot activate the following packages as they are already active on
Error:    the router:
Error:     
Error:        disk0:asr9k-doc-px-4.3.4
RP/0/RSP0/CPU0:Aug 29 01:38:44.180 : instdir[257]: %INSTALL-INSTMGR-3-INSTALL_OPERATION_USER_ERROR : User error occurred during install operation 37. See 'show install log 37 detail' for more infError:        disk0:asr9K-doc-supp-4.3.4
ormation. 
Error:     
Error:    Suggested steps to resolve this:
Error:     - check the name of the packages being activated.
Error:     - check the set of active packages using '(admin) show install
Error:       active'.
Error:    Cannot proceed with the activation as there are no packages to
Error:    activate
Error:    Suggested steps to resolve this:
Error:     - check the names of the packages being activated.
Install operation 37 failed at 01:38:44 PeKing Thu Aug 29 2024.
```



# 二、安装前的准备

## 1、配置console口

```
line console 
 exec-timeout 0 0
 stopbits 1
!
```

## 2、配置FPD自动升级

```
router(admin-config)#fpd auto-upgrade   
router(admin-config)#commit
！
```

或者是选择手动升级  

```
router(admin-config)upgrado hw-modulo fpd all location all   
```

注：
FPD在升级过程中如果编程失败，可能会导致硬件需要RMA

## 3、Cost-out IGP操作

Cost-out IGP：为了尽量减少升级过程中的流量损失，请按照以下步骤操作：
确保所有流经路由器的流量都需要升级有一条备用路径。 在这种情况下，可以采取其中之一冗余路由器停止服务，升级它，然后重新投入使用没有任何重大的流量损失（这应该适用于核心路由器，对于边缘设备通常冗余路径可能不可用）
 将 IGP 指标设置为可能的最高值，以便 IGP 尝试路由流量通过备用路径。

```
For OSPF use “max-metric” command. router(config-ospf)# max-metric router-lsa
For ISIS use “spf-overload-bit” command. router(config-isis)# set-overload-bit
```

## 4、将业务板卡进行卸载（升级过程中会重启多次）可选



# 三、安装系统组件

## 1、安装version4.3.4证书文件

```
RP/0/RSP0/CPU0:ROUTER# copy tftp://10.10.1.1/tftpboot-location/css-root.cer disk0:
Wed Dec 21 06:01:16.464 UTC
Destination filename [/disk0:/css-root.cer]?
Accessing tftp://10.10.1.1/tftp-location/Location/css-root.cer
C
1217 bytes copied in   0 sec

RP/0/RSP0/CPU0:ROUTER# sam add certificate /disk0:/css-root.cer root trust
Wed Dec 21 06:09:43.207 UTC
SAM: Successful adding certificate /disk0:/css-root.cer

OR

RP/0/RSP0/CPU0:ROUTER# run
Wed Dec 21 06:19:43.207 UTC
\# samcmd sam add certificate /disk0:/css-root.cer root trust
SAM: Successful adding certificate /disk0:/css-root.cer

RP/0/RSP0/CPU0:ROUTER(admin)# install add tftp://10.10.1.1/tftpboot-location/asr9k-px-5.1.3.CSCut52232-1.0.0 sync
Wed Dec 21 06:12:33.087 UTC
Info:   The following package is now available to be activated:
Info:
Info:     disk0:asr9k-px-5.1.3.CSCut52232-1.0.0
Info:
Info:   The package can be activated across the entire router.
Info:
Install operation 20 completed successfully at 06:12:40 UTC Wed Dec 21 2016.

RP/0/RSP0/CPU0:ROUTER(admin)# install activate disk0:asr9k-px-5.1.3.CSCut52232-1.0.0 sync
Wed Dec 21 06:41:02.086 UTC
Info:   Install Method: Parallel Process Restart
/ 15% complete: The operation can still be aborted (ctrl-c for options)
Info:   The changes made to software configurations will not be persistent
Info:   across system reloads. Use the command '(admin) install commit' to
Info:   make changes persistent.
Info:   Please verify that the system is consistent following the software
Info:   change using the following commands:
Info:     show system verify
Info:     install verify packages

\ 100% complete: The operation can still be aborted (ctrl-c for options)RP/0/RSP0/CPU0:Wed 21 06:42:37.856 : instdir[253]: %INSTALL-INSTMGR-4-ACTIVE_SOFTWARE_COMMITTED_INFO : The currently active software is not committed. If the system reboots then the committed software will be used. Use 'install commit' to commit the active software.

Install operation 7 completed successfully at 06:42:37 UTC Wed Dec 21 2016.
RP/0/RSP0/CPU0:ROUTER(admin)#
注：
如果出现其他安装包无法安装的问题 需使用install commit  将之前安装的组件和证书提交
安装失败提示设备系统时间不对导致无法安装，需要校准系统时间（证书需要时间校准）
```

## 2、安装、激活version5.3.4系统版本

```
安装组件：
router(admin)#install add disk0:asr9k-mini-px.pie-5.3.4 synchronous
或者router#admin install add disk0:asr9k-mini-px.pie-5.3.4 synchronous
注：
如果不识别路径的话改变/disk0:/

激活组件：
A9K-PE1(admin)#install activate disk0:asr9k-mini-px-5.3.4 synchronous
或者
A9K-PE1(admin)# install activate disk0:*5.3.4*
或者
A9K-PE1(admin)# install activate disk0:asr9k-mini-px-6.4.2 disk0:asr9k-mpls-px-6.4.2 ....
```

## 3、version6.4.2保持以上类似操作

```
version6.x.x以上的版本，安装完需要的组件和补丁包以后  需要对设备在进行一次整机重启操作。（旧的session在新系统中没有完全进行释放，直接使用会触发奇怪的bug。重启后能减少故障的排查时间）
```

