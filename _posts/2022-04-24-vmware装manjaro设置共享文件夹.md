# Mac vmware装manjaro设置共享文件夹
今天使用mac的vmware fusion装manjaro系统,无法添加共享文件夹的问题解决办法,之前一直是以为tools没装好,其实不是,虚拟机装好系统后,tools自动已经装好了,什么也别动,按以下步骤来操作,如果是根据网上其它资料导致卸载了默认安装的tools可以找资料看一下如何恢复,或者重新安装虚拟系统
1. 先设置物理系统要共享的文件夹,在虚拟机->共享设置->启动共享文件夹(✅)->添加物理机上要共享的文件夹,这时候可以看到下面提示在安装vmware tools后才能使用共享文件夹,并且在客户机中打开也是灰色的,不用管它
2. 启动虚拟系统manjaro
3. 打开终端执行 执行 vmware-hgfsclient 命令,查看物理机添加了那些文件夹可以当做共享,正常情况来讲可以看到第一步设置的共享文件夹
4. 执行命令 sudo pacman -S open-vm-tools
5. 创建共享文件夹：sudo mkdir /mnt/hgfs
6. 进入安装目录： cd /usr/bin
7. 执行命令： sudo ./vmhgfs-fuse -o allow_other -o auto_unmount /mnt/hgfs
8. 查看是否能看到共享文件夹了,进入 cd /mnt/hgfs 如果能看到了,同时也可以访问,打开文件管理器,进入到这个目录,将共享文件夹添加到常用位置即可
9. 注意这时候还没好,因为每次重启虚拟系统后,这个挂载会失效,所以还需要让系统每次启动时自动执行脚本
10. 创建一个启动service脚本 sudo vim /etc/systemd/system/rc-local.service 添加如下内容

```
[Unit]
Description="/etc/rc.local Compatibility" 

[Service]
Type=oneshot
ExecStart=/etc/rc.local start
TimeoutSec=0
StandardInput=tty
RemainAfterExit=yes
SysVStartPriority=99

[Install]
WantedBy=multi-user.target
```

11. 创建文件 sudo vim /etc/rc.local 添加如下内容

```
#!/bin/sh
# /etc/rc.local
if test -d /etc/rc.local.d; then
    for rcscript in /etc/rc.local.d/*.sh; do
        test -r "${rcscript}" && sh ${rcscript}
    done
    unset rcscript
fi
```

12. 添加执行权限 sudo chmod a+x /etc/rc.local
13. 创建文件夹 sudo mkdir /etc/rc.local.d
14. 设置开启自动启动 systemctl enable rc-local.service 
15. 可以把所有需要开启时自动启动的sh脚本放到 /etc/rc.local.d 目录下
16. 创建一个挂载共享文件夹的sh脚本 sudo vim /etc/rc.local.d/hgfs.sh 添加如下内容

```
#!/bin/sh -e

sudo vmhgfs-fuse .host:/ /mnt/hgfs -o nonempty -o allow_other
```

17. 重启系统 reboot 看共享文件夹是否正常进入
