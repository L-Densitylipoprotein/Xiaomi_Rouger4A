# 小米路由器刷机并实现自动登录

前排提醒：**刷机有风险，三思而后行**！！

---

小米路由器4A-百兆版（R4Ac）

默认地址（自行修改的按照自己设置的地址进入）

- 官方路由器后台：`192.168.31.1`

- breed恢复控制台：`192.168.1.1`

- openwrt：`192.168.1.1`

- Padavan：`10.32.0.1`

安装WinSCP：[WinSCP :: Official Site :: Free SFTP and FTP client for Windows](https://winscp.net/eng/index.php)

本质上使用telnet，通过漏洞进入路由器的系统，刷入breed并修改boot loader选项，刷入目标固件实现脚本或其它功能。

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



## 3.WinSCP

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

Padavan固件

- 用户名：admin
- 密码：admin

## 5.自动登录

### 提醒：不要用openwrt！！！（限于这次小米路由器R4Ac而言）

因为openwrt并没有内置curl命令，要自己下载，但是就会出那么一点点怎么都解决不了的问题，`opkg update`&`opkg install curl`。但就是这两个命令却怎么都无法正常运行：1.更新opkg总是会出现下载失败，换源了依旧无法解决。2.安装curl没有环境变量配置，所以就。。很复杂。附件里仍会提供curl离线的安装包，可以使用`opkg install xxx.ipk`的命令进行安装。但仍无法正常安装。

而且改路由的可操作空间太小了，几个软件包下来空间就不足了，故不建议使用openwrt固件。

这个坑前前后后踩了快两天了吧~一直在尝试解决curl的问题，但就是无法解决，如果通过openwrt实现curl命令的，欢迎一起讨论！

Padvan固件的下载，请见**参考3。**

###1.刷入Padavan

### 2.浏览器输入`10.32.0.1`

​	输入用户名（admin）和密码（admin）进入的后台

### 3.SSH

本方法使用Window自带的Power Shell工具。

输入：

```cmd
ssh admin@10.32.0.1
```

如果重新刷机，需要先输入

```cmd
ssh-keygen -R xxx.xxx.xxx  #该地址为之前使用ssh通信过的地址
```

键入“yes”，随后输入密码“admin”

<img src="C:/Users/86158/AppData/Roaming/Typora/typora-user-images/image-20221005010311422.png" alt="image-20221005010311422" style="zoom:50%;" />

见此说明与路由器通信成功。

### 4.校园网登录分析

以本人学校——SZU为例

打开网页端的上网验证页面，输入对应的用户名和密码，并按下“F12”打开`DevTools`，选择“网络”界面。

点击“登录”，在窗口处找到“login xxx”的文件，点击，查看标头。

![image-20221015154215128](https://gitee.com/li-Density-Lipoprotein/li-lmage-bed/raw/master/image-20221015154215128.png)

复制请求URL后面的部分：

![image-20221015154516426](https://gitee.com/li-Density-Lipoprotein/li-lmage-bed/raw/master/image-20221015154516426.png)

使用“Win”+R打开cmd，验证字段是否有效：

```cmd
curl "http://172.xxxxxxx" #这里填的是刚刚复制的字段
```

如果能正常使用curl命令登录，即可进行后续的步骤。

不行，参考自己学校对应的字段分析进行处理。【实测SZU使用本方法可行】

### 5.基本的Linux命令

```cmd
#详细说明请自行搜索
df -h 	#查看分区
cd    	#移动操作位置  cd .. ##返回上一级
pwd   	#查看此刻位置
mkdir 	#在当前位置新建文件夹
vi xxx.sh   #脚本写入
	{
	o --输入
	按“Exit”后
		:q 	  --退出
		:q!   --强制退出
		:w    --保存
		:wq   --保存并退出
	}
chmod -x xxx  #给xxx文件可执行权限
chmod -x xxx  #给xxx文件所有权限
```

### 6.详细操作

**注意：文件（脚本）需保存在`/etc/storage/`下，否则每次重启将会丢失**

```cmd
mkdir autologin  #新建autologin文件夹
cd autologin
vi autologin.sh
```

在vi页面

```cmd
# o --写入操作
# "Esc" + ":q" --退出
# "Esc" + ":wq"  --保存并退出

#以下为自动登录脚本代码：

#ping 的总次数
PING_SUM=3
#ping 的间隔时间，单位秒
SLEEP_SEC=10
#连续重启网卡 REBOOT_CNT 次网络都没有恢复正常，重启软路由
#时间= (SLEEP_SEC * PING_SUM + 20) * REBOOT_CNT
REBOOT_CNT=3
LOG_PATH="/etc/storage/autologin/log.txt"
cnt=0
reboot_cnt=0
while :
do
    ping -c 1 -W 1 www.baidu.com > /dev/null
    ret=$?
    
    ping -c 1 -W 1 www.bilibili.com > /dev/null
    ret2=$?
    if [[ $ret -eq 0 || $ret2 -eq 0 ]]
    then
		echo -e 'network is ok\r'
    	exit
        #cnt=0
        #reboot_cnt=0
    else
        cnt=`expr $cnt + 1`
        echo -n `date '+%Y-%m-%d %H:%M:%S'` >> $LOG_PATH
        printf '-> [%d/%d] Network maybe disconnected,checking again after %d seconds!\r\n' $cnt $PING_SUM $SLEEP_SEC >> $LOG_PATH
        printf '-> [%d/%d] Network maybe disconnected,checking again after %d seconds!\r\n' $cnt $PING_SUM $SLEEP_SEC 
        
        if [ $cnt == $PING_SUM ]
        then
            echo 'try to re curl' >> $LOG_PATH
            echo 'ifup wan!!!'
            sleep 5
            curl 'xxxxx' #填入
            
            cnt=0
            #重连后，等待10秒再进行ping检测
            sleep 8
            #网卡重启超过指定次数还没恢复正常，重启软路由
            reboot_cnt=`expr $reboot_cnt + 1`
            if [ $reboot_cnt == $REBOOT_CNT ]
            then
                echo -n `date '+%Y-%m-%d %H:%M:%S reboot!'` >> $LOG_PATH
                echo '-> Network has some problem, lets reboot' >> $LOG_PATH
                echo '-> =============== reboot!'
                reboot
            fi
        fi
    fi    
    sleep $SLEEP_SEC
done
```

最后对脚本赋权限:

```cmd
chmod 777 xxx.sh  #这里就无脑给脚本所有权限，也可以只赋予执行权限(-x)
```



且写完后，需执行：

```cmd
/sbin/mtd_storage.sh save
```

或在路由器后台“在 高级设置>系统管理>“**保存 /etc/storage/ 内容到闪存”** 点击提交。”（Padavan）



最后，在 `自定义设置 > 脚本 > 在 WAN 上行/下行启动后执行` 的内容后添加一行：`/etc/storage/xxx.sh`，并点击页面最下方的 `应用本页面设置` 即可。



## 6.重新开始

遇到无法解决的问题，可以重刷回breed控制台，然后再重新刷入一遍固件。

操作：**按住路由的“reset”键（按钮），然后通电，持续3~4s，观察到黄灯和蓝灯闪烁多下后可松手。**



## 参考

1.[小米路由4免拆机无脑刷PADAVAN固件包-小米无线路由器以及小米无线相关的设备-恩山无线论坛 (right.com.cn)](https://www.right.com.cn/forum/thread-4089487-1-1.html)

2.[【2020-11-4】小米4A百兆版 R4A-100M breed 直刷 openwrt 固件-OPENWRT专版-恩山无线论坛 (right.com.cn)](https://www.right.com.cn/forum/thread-4057583-1-1.html)

3.[【记录】小米路由4A百兆版适配Padavan - 我来时天晴，你来时风止 (yuos.top)](https://yuos.top/index.php/archives/627/)

4.[openwrt校园网自动登录且断网重连_in dreaming的博客-CSDN博客_openwrt自动登录网页认证](https://blog.csdn.net/qq248606117/article/details/125144699)

5.[使用 Padavan 路由器实现校园网自动 Web 认证 - 少数派 (sspai.com)](https://sspai.com/post/57882)

#### 不重要的

不知道是幸运还是不幸运，在刷固件的时候，就是到刷入breed的时候，路由器居然没有重启，然后。。然后它就寄了，是连指示灯都没有那种（“真”成砖了T_T）然后就去茫茫的互联网找解决办法，但好像大多是对指示灯是黄色或红色的对策。无奈就去找京东了，幸运的是我的路由器还在保！【所以说电子产品就尽量jd吧】所以重新换了一个还挺方便的。在这里感谢东哥！为我这种折腾埋单，我自己怪不好意思的~
