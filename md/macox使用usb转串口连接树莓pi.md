#Macox使用USB转串口连接树莓PI


[我使用的 USB 转串口](http://item.taobao.com/item.htm?spm=a1z09.2.9.50.YOJBwG&id=38963495468&_u=4m1drce4153)
![image](http://api.drp.io/files/543243038a4f6.jpg)
下载驱动
---

选 MAC版本
![image](http://api.drp.io/files/5432419697958.jpg)

**下载地址**
[http://www.silabs.com/products/mcu/pages/usbtouartbridgevcpdrivers.aspx](http://www.silabs.com/products/mcu/pages/usbtouartbridgevcpdrivers.aspx)

1. 安装后重启树莓 PI
2. 打开 MAC terminal输入
> screen /dev/cu.SLAB_USBtoUART 115200

3. 连接好线  R--->T    T--->R
4. 最后连接电源