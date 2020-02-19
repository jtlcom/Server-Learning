# 禁止VirtualBox虚拟机同步系统时间

## 目的

可以通过这种方法，在不改变Windows本地时间的情况下，修改**VirtualBox**中**realm服**的服务器时间

## 目录

* [0. 原理](#原理)
* [1. 获取虚拟机名称](#获取虚拟机名称)
* [2. 在VirtualBox安装目录打开cmd](#在virtualbox安装目录打开cmd)
* [3. 执行关闭时间同步命令](#执行关闭时间同步命令)
* [4.	启动虚拟机并执行命令](#启动虚拟机并执行命令)
* [5.	实际应用步骤](#实际应用步骤)

## [原理](#目录)

![avatar](/res/TIM截图20200219160519.jpg)

## [获取虚拟机名称](#目录)

在虚拟机的右键菜单里面，有一个设置按钮，点开即可看到虚拟机名称

![avatar](/res/TIM截图20200219141132.jpg)

![avatar](/res/TIM截图20200219142353.jpg)

## [在VirtualBox安装目录打开cmd](#目录)

进入到**VirtualBox**的安装目录中，然后在此路径开启cmder或其他控制台软件

![avatar](/res/TIM截图20200219145954.jpg)

## [执行关闭时间同步命令](#目录)

通过控制台软件进入到刚刚的路径后，输入如下的命令以关闭同步系统时间

```bash
VBoxManage setextradata 虚拟机名称 "VBoxInternal/Devices/VMMDev/0/Config/GetHostTimeDisabled" "1"

VBoxManage guestproperty set 虚拟机名称 --timesync-set-stop
```

## [启动虚拟机并执行命令](#目录)

```bash
sudo ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

sudo echo “ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime” >> /etc/rc.local
```

## PS：以上命令只需要执行一次

## [实际应用步骤](#目录)

1. 关服
2. 修改虚拟机的系统时间
3. 开realm服

修改系统时间的命令如下：

```bash
sudo date -s '2020-10-10 23:50:00'
```

## PS：设置完成后，不能重启虚拟机！否则时间会被重置

---
