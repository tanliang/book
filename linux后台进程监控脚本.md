# linux后台进程监控脚本

参考:[Bash Shell字符串操作小结](https://my.oschina.net/aiguozhe/blog/41557)

参考:[Shell 函数](http://www.runoob.com/linux/linux-shell-func.html)

参考:[shell判断输入变量或者参数是否为空](http://blog.sina.com.cn/s/blog_a5b3ccfd01016n9z.html)

~~~bash
#!/bin/bash
log(){
  type=$1
  info=$2
  echo "[`date \"+%Y-%m-%d %H:%M:%S\"`] ($type) $info" >> /tmp/pm.log
}

exe(){
  key=$1
  cmd=$2
  log=$3

  ps=`ps -fe|grep $key |grep -v grep |wc -l`
  if [ $ps -eq 0 ];then
    to=/tmp/$key.log
    if [ ! -n "$log" ] ;then
      to=/dev/null
    fi
    nohup $cmd >$to 2>&1 &
    log "start" "$cmd >$to"
  fi
}

run(){
  row=$1
  log=$2

  # get [key] string
  b=`expr index "$row" "["`
  e=`expr index "$row" "]"`
  i=`expr $b + 1`
  l=`expr $e - $i`
  k=`expr substr "$row" $i $l`

  # get cmd
  s=${row/\[/}
  s=${s/\]/}
  #echo $k
  #echo $s
  if [ $b -ne 0 ] && [ $e -ne 0 ];then
    exe "$k" "$s" "$log"
  else
    log "error" "miss [key] string in \"$row\""
  fi
}

if [ ! -n "$1" ] ;then
  log "error" "missing process list."
  exit 1
fi

cat $1 |while read row
do
  if [ -n "$row" ] ;then
    run "$row" "$2"
  fi
done

#*/3 * * * * /bin/bash /your path/pm.sh </you path/process.txt> [log]
~~~

process.txt 用[key]标示进程关键字
~~~
java -jar /your path/[one-service-1.0]-SNAPSHOT.jar
java -jar /your path/[two-service-1.0]-SNAPSHOT.jar
~~~