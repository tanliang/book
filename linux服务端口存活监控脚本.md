# linux服务端口存活监控脚本

参考:[shell 字符串分割与连接](https://www.cnblogs.com/gx-303841541/archive/2012/10/25/2738048.html)

参考:[自定义服务端口存活监控](https://zhouxiaowu.coding.me/2017/08/03/自定义服务端口存活监控/)

~~~bash
#!/bin/bash

exe(){
  name=$1
  ip=$2
  port=$3
  who=$4

  nmap -Pn -p$port $ip | awk "\$1 ~ /$port/ {print \$2}" | grep open >/dev/null 2>&1
  if [ $? -ne 0 ];then
    log=/tmp/$name.sm
    if [ -e $log ] && [ `cat $log |wc -l` -gt 2 ];then

      ############
      echo "`date "+%Y%m%d%H"`" > /tmp/$name.notify
      ############

      rm $log
      echo "$ip:$port" |mail -s "$name down" "$who"
    else
      echo "`date \"+%Y-%m-%d %H:%M:%S\"`" >> $log
    fi
  fi
}

run(){
  row=$1
  who=$2

  name=`echo $row |cut -d "=" -f1`
  str=`echo $row |cut -d "=" -f2`
  ip=`echo $str |cut -d ":" -f1`
  port=`echo $str |cut -d ":" -f2`
  #echo $ip
  #echo $port

  time=`date "+%Y%m%d%H"`
  file=/tmp/$name.notify
  last=0
  if [ -e $file ];then
    last=`cat $file`
  fi

  # notify once per hour
  if [ $time -ne $last ];then

    #==============
    if [ -n "$ip" ] && [ -n "$port" ];then
      exe "$name" "$ip" "$port" "$who"
    else
      echo "miss ip/port in \"$row\""
    fi
    #==============

  fi
}

if [ ! -n "$1" ] ;then
  echo "missing process list."
  exit 1
fi

if [ ! -n "$2" ] ;then
  echo "missing mailto address."
  exit 1
fi

cat $1 |while read row
do
  if [ -n "$row" ] ;then
    run "$row" "$2"
  fi
done

#sudo apt install nmap
#* * * * * /bin/bash /your path/sm.sh </you path/sm.txt> <mailto>
~~~

sm.txt
~~~
test_service=127.0.0.1:80
~~~