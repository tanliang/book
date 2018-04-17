# linux后台进程监控脚本

参考:[ps命令--排序](https://blog.csdn.net/lmy4710/article/details/8680763)

~~~bash
#!/bin/bash
if [ ! -n "$1" ] ;then
  echo "missing mailto address."
  exit 1
fi

time=`date "+%Y%m%d"`
file=/tmp/${0##*/}.tmp
last=0
if [ -e $file ];then
  last=`cat $file`
fi

# notify once per day
if [ $time -eq $last ];then
  exit 1
fi

cpus=`cat /proc/cpuinfo |grep processor |wc -l`
load=`uptime |awk -F " " '{ print $NF }'`
warn=$(echo "$load > 0.60*$cpus" |bc)

if [ $warn = 1 ];then
  echo $time > $file
  subject="load=$load(cpus=$cpus) at `hostname --fqdn`"
  content="`uptime && ps -eo pid,pcpu,command |sort -k 2 -r -n |head`"
  echo "$content" |mail -s "$subject" "$1"
fi

#sudo apt install bc
#* * * * * /bin/bash /your path/lm.sh <mailto>
~~~