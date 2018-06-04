# 一个交互式的杀死进程的shell脚本

参考:[如何设置nginx作为反向代理Apache2在Ubuntu 12.04](https://www.howtoing.com/how-to-set-up-nginx-as-a-reverse-proxy-for-apache2-on-ubuntu-12.04)

参考:[echo 显示带颜色内容的输出方法](http://blog.51cto.com/1inux/1634799)

配合[linux后台进程监控脚本](linux后台进程监控脚本)，在手机上 ssh 登录，杀死进程，自动重启。

~~~bash
#!/bin/bash
if [ $UID -ne 0 ]; then
    echo "Superuser privileges are required to run this script."
    echo "e.g. \"sudo $0\""
    exit 1
fi

name=$1
if [ ! -n "$1" ] ;then
  name="java"
fi

echo -ne "input service name for grep [\033[31m$name\033[0m]:"
read input
if [ -n "$input" ] ;then
  name=$input
fi

pids=`ps -fe |grep $name |grep -v grep |grep -v $0`
numb=`echo "$pids" |wc -l`
echo
if [ $numb -eq 0 ];then
  echo "\033[31mno named $name process found.\033[0m"
  exit
else
  echo -e "\033[32m$numb\033[0m process found below:"
fi
list=`echo "$pids" |awk '{ printf("ps_%s=\"%s ", NR, $2); for (i=8;i<=NF;i++)printf("%s ", $i); print "\""}'`
echo "$list" |awk -F "=" '{for(i=2;i<=NF;i++)printf("%s", $i); print "";}'
echo
eval $list

if [ $numb -eq 1 ]; then
  pid=`echo $ps_1 |awk '{print $1}'`
fi

echo -ne "chose pid to kill "

if [ -n "$pid" ]; then
  echo -ne "[\033[31m$pid\033[0m]"
fi

echo -n ":"

read input
if [ -n "$input" ]; then
  pid=$input
fi

if [ ! -n "$pid" ]; then
  echo -e "\033[31mno pid chosed.\033[0m"
  exit
fi

kill $pid
if [ $? -ne 0 ]; then
  kill -9 $pid
  if [ $? -ne 0 ]; then
    echo -e "\033[31mkill $pid failed.\033[0m"
    exit
  fi
fi
echo "kill $pid success."
~~~