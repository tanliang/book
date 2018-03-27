# ubuntu1604安装kvm

公司用*8核32g内存单网卡*台式做服务器，装的ubuntu16.04，为了方便开发测试，计划在其部署2个kvm实例，步骤如下。

- [安装kvm](#install_kvm)
- [设置桥接](#bridge_set)
- [安装镜像](#install_img)
- [复制镜像](#clone_img)
- [启动镜像](#start_img)

## 安装kvm<a name="install_kvm"></a>

参考:[Ubuntu 16.04 安装使用 KVM](http://blog.topspeedsnail.com/archives/8573)

~~~bash
#在开始安装KVM之前，我们需要确保CPU支持VMX/SVM硬件虚拟化。
$ egrep -c '(vmx|svm)' /proc/cpuinfo
#如果输出大于等于1，表示系统可安装KVM。如果输出为0，表示系统不支持KVM。

#安装KVM
$ sudo apt-get install qemu-kvm libvirt-bin virtinst bridge-utils cpu-checker

#安装确认
$ kvm-ok
INFO: /dev/kvm exists
KVM acceleration can be used
#如果不成功，一般会提示进入 bios 开启 VT 支持
~~~


## 设置桥接<a name="bridge_set"></a>

参考:[Ubuntu14.04+KVM配置虚拟机桥接（bridge）](https://blog.csdn.net/fieldoffier/article/details/48497833)
参考:[Ubuntu开发环境搭建](https://www.leolan.top/index.php/posts/193.html)

Bridge方式即虚拟网桥的网络连接方式，是客户机和子网里面的机器能够互相通信。可以使虚拟机成为网络中具有独立IP的主机。
桥接网络（也叫物理设备共享）被用作把一个物理设备复制到一台虚拟机。网桥多用作高级设置，特别是主机多个网络接口的情况。
![bridge方式](resource/ubuntu1604/bridge.png)
~~~bash
vim /etc/network/interfaces

# interfaces(5) file used by ifup(8) and ifdown(8)
auto lo
iface lo inet loopback

auto enp2s0
iface enp2s0 inet manual

auto br0
iface br0 inet static
    address 192.168.199.122
    netmask 255.255.255.0
    gateway 192.168.199.1
    bridge_ports enp2s0
~~~

## 安装镜像<a name="install_img"></a>

参考:[Create Virtual Machine#1](https://www.server-world.info/en/note?os=Ubuntu_16.04&p=kvm&f=2)

字符模式下安装虚拟机，并用终端登录，因为是网络安装，速度有些慢
~~~bash
root@dlp:~# apt-get -y install libosinfo-bin libguestfs-tools virt-top

# create a storage pool
root@dlp:~# mkdir -p /var/kvm/images 
root@dlp:~# virt-install \
--name template \
--ram 4096 \
--disk path=/var/kvm/images/template.img,size=30 \
--vcpus 2 \
--os-type linux \
--os-variant ubuntu16.04 \
--network bridge=br0 \
--graphics none \
--console pty,target_type=serial \
--location 'http://jp.archive.ubuntu.com/ubuntu/dists/xenial/main/installer-amd64/' \
--extra-args 'console=ttyS0,115200n8 serial'

Starting install...     # installation starts

# after installation, back to KVM host and shutdown the guest like follows
root@dlp:~# virsh shutdown template 
Domain template is being shutdown

# mount guest's disk and enable a service like follows
root@dlp:~# guestmount -d template -i /mnt 
root@dlp:~# ln -s /mnt/lib/systemd/system/getty@.service /mnt/etc/systemd/system/getty.target.wants/getty@ttyS0.service 
root@dlp:~# umount /mnt

# start guest again, if it's possible to connect to the guest's console, it's OK all
root@dlp:~# virsh start template --console 
Domain template started
Connected to domain template
Escape character is ^]

Ubuntu 16.04 LTS ubuntu ttyS0

ubuntu login:
~~~

## 复制镜像<a name="clone_img"></a>

参考:[kvm 虚拟化 virt-clone 克隆虚拟机](https://blog.csdn.net/wanglei_storage/article/details/51106096)

~~~bash
＃另起一个 console 关闭 template
$ sudo virsh shutdown template
$ sudo virt-clone --connect=qemu:///system -o template -n ubuntu_kvm_1 -f /var/kvm/images/ubuntu_kvm_1

~~~

### 启动镜像<a name="start_img"></a>

~~~bash
# 设置自动启动
$ sudo virsh autostart ubuntu_kvm_1

# 关闭自动启动
$ sudo virsh autostart --disable ubuntu_kvm_1

# 控制台登录 修改 ip 为静态设置 参考 桥接
$ sudo virsh start ubuntu_kvm_1 --console
# 结束直接关闭窗口
~~~
