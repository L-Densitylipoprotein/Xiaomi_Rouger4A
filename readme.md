# 小米路由器刷机并实现自动登录

前排提醒：**刷机有风险**！！

---

小米路由器4A-百兆版（R4Ac）

- 官方路由器后台：`192.168.31.1`

- breed恢复控制台：`192.168.1.1`

- openwrt：`192.168.1.1`

安装WinSCP：[WinSCP :: Official Site :: Free SFTP and FTP client for Windows](https://winscp.net/eng/index.php)

本质上使用telnet，进入路由器的系统，刷入breed并修改boot loader选项。

---

## 1. 开启Telnet

   打开“控制面板”，进入“程序与功能”，点击左侧“启用或关闭Window功能”，勾选“Telnet客户端”，最后点击确定。

<img src="https://gitee.com/li-Density-Lipoprotein/li-lmage-bed/raw/master/image-20221002144301728.png" alt="image-20221002144301728" style="zoom:50%;" />

​	打开script文件夹下的“0.start_main.bat”，输入路由器后台的登陆密码，看到Done即表示成功。

​	cmd输入

```cmd
telnet 192.168.31.1
```



```cmd
XiaoQiang login: root
```

不用输入密码，回车

显示以下内容即为开启了Telnet功能。

<img src="https://gitee.com/li-Density-Lipoprotein/li-lmage-bed/raw/master/image-20221002111404794.png" alt="image-20221002111404794" style="zoom:67%;" />



失败的重复上述步骤。

【或进入“控制面板”—“网络和共享中心”—左侧“更改高级共享选项”—“启用网络发现”和“启用文件和打印机共享”】~（未验证是否有用）~

## 2.备份

仍在cmd窗口，键入：

```cmd
cat /proc/mtd  #查看路由器分区
```

```cmd
dd if=/dev/mtd0 of=/tmp/all.bin         #备份所有
dd if=/dev/mtd1 of=/tmp/Bootloader.bin  #仅备份Bootloader
```


- mtd0为备份bootloaderd

- mtd1为备份所有文件

备份bootloaderd方便以后刷回官方固件。

一般备份bootloaderd即可



## 3WinSCP

​	双击打开一个站点，协议选择“FTP”；主机名`192.168.31.1`，用户名“root”，没有密码。

<img src="https://gitee.com/li-Density-Lipoprotein/li-lmage-bed/raw/master/image-20221002120908382.png" alt="image-20221002120908382" style="zoom:67%;" />

​	双击进入tmp文件夹，将备份好的bootloaderd文件（压缩包）保存到电脑【选择备份位置，直接拉入的左侧即可】

###   将breed文件上传到tmp文件夹中

​	由于某种未知的原因，并未在WinScp中能查看tmp内文件，但仍可以上传breed。只需将对应的breed文件（.bin）上传到tmp下即可。

### 确认已经breed刷入到tmp内

```cmd
cd /tmp  ##进入tmp文件
ls -l    ##查看该目录下文件详细信息，找到对应的breed即说明成功
```

### 将官方引导文件替换成breed引导文件

```cmd
mtd -r write xxx.bin Bootloader  ##xxx.bin是目标breed文件名
```

路由器会自动重启，等待即可。



浏览器输入`192.168.1.1`

<img src="https://gitee.com/li-Density-Lipoprotein/li-lmage-bed/raw/master/image-20221002143100572.png" alt="image-20221002143100572" style="zoom:50%;" />

看到如上图显示，说明刷入breed成功。

不成功？

- 断电，按住reset键（重置孔）再接入电源。

变砖？

- 打开浏览器，进入漫长的救砖之旅~以下省略5000字T_T



#### 注意备份eeprom！

点击“固件备份”—“EEPROM”下载到电脑

#### MAC地址截图



<img src="https://gitee.com/li-Density-Lipoprotein/li-lmage-bed/raw/master/image-20221002145146570.png" alt="image-20221002145146570" style="zoom:33%;" />

## 4.刷入固件

点击左侧“固件更新”—勾选“固件”，打开需要刷入的固件，上传即可。

<img src="https://gitee.com/li-Density-Lipoprotein/li-lmage-bed/raw/master/image-20221002143449729.png" alt="image-20221002143449729" style="zoom:50%;" />

刷写完成会再次自动重启，最后在浏览器输入`192.168.31.1`便可进入路由器后台。

提供的openwrt固件

- 用户名：root   

- 密码：password

## 参考

1.[小米路由4免拆机无脑刷PADAVAN固件包-小米无线路由器以及小米无线相关的设备-恩山无线论坛 (right.com.cn)](https://www.right.com.cn/forum/thread-4089487-1-1.html)

2.[【2020-11-4】小米4A百兆版 R4A-100M breed 直刷 openwrt 固件-OPENWRT专版-恩山无线论坛 (right.com.cn)](https://www.right.com.cn/forum/thread-4057583-1-1.html)

#### 不重要的

~不知道是幸运还是不幸运，在刷固件的时候，就是到刷如breed的时候，路由器居然没有重启，然后。。然后它就寄了，是连指示灯都没有那种（“真”成砖了T_T）然后就去茫茫的互联网找解决办法，但好像大多是对指示灯是黄色或红色的对策。无奈就去找京东了，幸运的是我的路由器还在保！【所以说电子产品就尽量jd吧】所以重新换了一个还挺方便的。在这里感谢东哥！为我这种折腾埋单，我自己怪不好意思的~~
