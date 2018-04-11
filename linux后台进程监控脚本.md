# linux后台进程监控脚本

参考:[Bash Shell字符串操作小结](https://my.oschina.net/aiguozhe/blog/41557)

参考:[Shell 函数](http://www.runoob.com/linux/linux-shell-func.html)

~~~bash
#!/bin/bash
log(){
  type=$1
  info=$2
  echo "[`date \"+%Y-%m-%d %H:%M:%S\"`] ($type) $info" >> /tmp/pm.log
}

_pm(){
  ps=`ps -fe|grep $1 |grep -v grep |wc -l`
  if [ $ps -eq 0 ];then
    nohup $2 >/dev/null 2>&1 &
    log "start" "$2"
  fi
}

cat $1 |while read LINE
do
  # get [key] string
  b=`expr index "$LINE" "["`
  e=`expr index "$LINE" "]"`
  i=`expr $b + 1`
  l=`expr $e - $i`
  k=`expr substr "$LINE" $i $l`

  # get cmd
  s=${LINE/\[/}
  s=${s/\]/}
  #echo $k
  #echo $s
  if [ $b -ne 0 ] && [ $e -ne 0 ];then
    _pm "$k" "$s"
  else
    echo "[error] $LINE"
    log "error" "miss [key] string in \"$LINE\""
  fi
done

#*/3 * * * * /bin/bash /your path/pm.sh /you path/process.txt
~~~

process.txt 用[key]标示进程关键字
~~~
java -jar /your path/[one-service-1.0]-SNAPSHOT.jar
java -jar /your path/[two-service-1.0]-SNAPSHOT.jar
~~~