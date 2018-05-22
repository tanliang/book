# 给不熟linux的同事写代理服务切换脚本

参考:[如何设置nginx作为反向代理Apache2在Ubuntu 12.04](https://www.howtoing.com/how-to-set-up-nginx-as-a-reverse-proxy-for-apache2-on-ubuntu-12.04)

参考:[echo 显示带颜色内容的输出方法](http://blog.51cto.com/1inux/1634799)

用 docker 部署 2 套 java 服务，通过 nginx 的反向代理提供出口，实现 java 升级时无需停止服务。

~~~bash
#!/bin/bash

if [ ! -n "$1" ] ;then
  echo "error, missing port."
  exit 1
fi

conf=/etc/nginx/sites-enabled/default
curr=`cat $conf |grep proxy_pass |awk -F":" '{print $3}'`
echo -e "current proxy is \033[31m$curr\033[0m"
echo -e "sure to use port \033[32m$1\033[0m [enter]?"
read -p "or use other port [input]:" port

if [ ! -n "$port" ] ;then
  port=$1
fi

port=`echo $port |awk '{print int($0)}'`
if [ $port -le 0 ];then
  echo "error, invalid port."
  exit
fi

sed -i "s/$curr/${port};/" $conf
service nginx reload
res=$?
if [ $res -eq 0 ];then
  echo "service reload ok with $port."
fi

＃sudo nginx_switch.sh 8071
~~~