# 记一次 ubuntu 的 autoremove 恢复历程

同事在公司内网服务器执行了 sudo apt autoremove 把系统搞挂了，ssh 无法连接，系统直连只能进入 recovery 模式。

服务器装了 3 个 kvm 虚拟机，几个重要服务，以及若干重要数据，重装是不可能重装了，这辈子都不可能重装的。

网上 google 了半天，试了各种方法，最后用个同版本的 ubuntu 16.04 的 livecd 启动系统，然后

参考:[Ubuntu 备份与恢复](https://www.jianshu.com/p/d1dfe3104ffe)

~~~bash
cd / 
tar -cvpzf backup.tar.gz --exclude=/backup.tar.gz --one-file-system / 
~~~

把 livecd 的系统备份

~~~bash
sudo mount /dev/sda2 /mnt
sudo tar -xvpzf /path/to/backup.tar.gz -C /mnt --numeric-owner
~~~

再恢复到挂了的系统

结果是完美启动，然后需要重新配置下网络 /etc/network/interfaces，重新安装下 apt install openssh-server，重装下 samba。

启动 kvm 失败，systemctl status libvirtd 查看状态，根据提示

~~~bash
groupadd libvirtd
useradd libvirt-qemu
useradd libvirt-dnsmaq
adduser libvirt-qemu libvirtd
adduser libvirt-dnsmaq libvirtd
.
.
.
~~~


