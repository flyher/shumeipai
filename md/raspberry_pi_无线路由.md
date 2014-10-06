
#使用树莓raspberry pi 搭建路由器
2014年10月4日19:19

参考文章
---
* [http://zhainan.org/post-1661.html](http://zhainan.org/post-1661.html)
* [http://wwssllabcd.github.io/blog/2013/04/22/how-to-setup-wireless-in-raspberry-pi/](http://wwssllabcd.github.io/blog/2013/04/22/how-to-setup-wireless-in-raspberry-pi/)
* [http://wangye.org/blog/archives/845/](http://wangye.org/blog/archives/845/)
* [http://blog.alexchi.me/archives/751](http://blog.alexchi.me/archives/751)
* [http://blog.alexchi.me/archives/751](http://blog.alexchi.me/archives/751)
* [http://www.mendrugox.net/2013/08/tp-link-tl-wn725n-v2-working-on-raspberry-raspbian/](http://www.mendrugox.net/2013/08/tp-link-tl-wn725n-v2-working-on-raspberry-raspbian/)
> hostapd实现的是无线共享，但需要注意的是官方提供的程序不兼容RTL8188CUS芯片的无线网卡，不过Edimax团队为我们专门编译了兼容的版本，下面的操作需要替换hostapd为兼容版本


无线网卡驱动安装
---
![image](http://api.drp.io/files/543244a3c4507.jpg)
[http://item.jd.com/618066.html](http://item.jd.com/618066.html)

参考[http://www.mendrugox.net/2013/08/tp-link-tl-wn725n-v2-working-on-raspberry-raspbian/](http://www.mendrugox.net/2013/08/tp-link-tl-wn725n-v2-working-on-raspberry-raspbian/) 下载对应的驱动

**把 .ko文件拷贝到下面的目录**

```
/lib/modules/(your-kernel-version)/kernel/drivers/net/wireless
例如: /lib/modules/3.6.11+/kernel/drivers/net/wireless
```

> 最新的`rtl8188eu`驱动包含 `rtl8188eufw.bin` 固件




**把固件拷到下面目录**

```
/lib/firmware/rtlwifi/
```

**运行**

```
    depmod -a
    modprobe 8188eu
```

`ifconfig`就能看到`wlan0`了

无线路由部分
---

**首先，我们需要安装必须软件和环境，一个是hostap，一个是dhcp**

```
sudo apt-get install hostapd
sudo apt-get install dhcp3-server
```

**安装完毕后编辑/etc/hostapd/hostapd.conf文件，如果没有，就自行新建**

```
sudo nano /etc/hostapd/hostapd.conf
```

```
interface=wlan0
#driver是wlan的驱动模块名称
driver=rtl871xdrv
#ssid名称，随意改
ssid=RasPi
channel=6
hw_mode=g
wpa=2
#密码，随意改
wpa_passphrase=1234567890
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```

**保存之~然后再修改/etc/dhcp/dhcpd.conf 文件进行子网配置**

```
sudo nano /etc/dhcp/dhcpd.conf
```
```
subnet 192.168.0.0 netmask 255.255.255.0
 {
 range 192.168.0.2 192.168.0.10;
 #option routers要和wlan0 IP一致，注意后文有提到改命令
 option routers 192.168.0.1;
 #domain-name-servers是可访问internet的eth0所使用的DNS，可以使用公共DNS，如8.8.8.8等
 option domain-name-servers 8.8.8.8;
 }
```


**替换的命令如下：**

```
wget http://www.daveconroy.com/wp3/wp-content/uploads/2013/07/hostapd.zip
unzip hostapd.zip 
sudo mv /usr/sbin/hostapd /usr/sbin/hostapd.bak
sudo mv hostapd /usr/sbin/hostapd.edimax 
sudo ln -sf /usr/sbin/hostapd.edimax /usr/sbin/hostapd 
sudo chown root.root /usr/sbin/hostapd 
sudo chmod 755 /usr/sbin/hostapd
```


**启动**

```
#下行命令为手动设置无线网卡IP地址，和上述option routers一致
sudo ifconfig wlan0 192.168.0.1 netmask 255.255.255.0
#启动DHCP服务，否则连上无线之后分配不到IP
service isc-dhcp-server start
#启动hostapd
sudo hostapd -B /etc/hostapd/hostapd.conf
#打开IP转发
sudo bash -c "echo 1 >/proc/sys/net/ipv4/ip_forward"
#设置NAT
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

开机启动
---
```
我添加了dhcp开机自启，你也可以在/etc/init.d/rc.local中添加
sudo ifconfig wlan0 192.168.100.1 netmask 255.255.255.0
sudo service isc-dhcp-server start
sudo bash -c "echo 1 >/proc/sys/net/ipv4/ip_forward"
sudo hostapd -B /etc/hostapd/hostapd.conf 
这样就可以实现路由功能开机自启了
```


> 以后给树莓 PI接上电源,没有显示器,没有路由器下(不知道 PI 的 IP 地址情况下也能 SSH 登陆了)直接 SSH 输入192.168.0.1,实现快速连接了.

![image](http://api.drp.io/files/54324e4e3d0c2.jpg)
![image](http://api.drp.io/files/54324e4e4490e.jpg)
![image](http://api.drp.io/files/54324e4e30ed3.jpg)
