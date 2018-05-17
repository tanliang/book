# linux监控java日志重启服务脚本

参考:[Linux Find 命令精通指南](http://www.oracle.com/technetwork/cn/topics/calish-find-096463-zhs.html)

参考:[bash的数组](http://www.cnblogs.com/hopeworld/archive/2011/04/12/2013425.html)

参考:[linux批量监控系统状态](linux批量监控系统状态.md)

通过监控 java 的日志文件修改时间，判断 java 服务是否宕机，以及重启。条件是 jar 包，及 日志文件都统一放置。

~~~bash
# 第一步 声明数组，日志文件名＝java服务关键字
vi xpm.arr
  declare -A arr_ps
  arr_ps["zuul"]=xxx_gateway-1.0
  arr_ps["deamon"]=xxx-deamon-service-1.0
  arr_ps["couponManage"]=xxx-coupon-service-1.0

# 第二部 加载数组，并判断执行
vi xpm.sh
  #!/bin/bash

  log(){
    type=$1
    info=$2
    echo "[`date \"+%Y-%m-%d %H:%M:%S\"`] ($type) $info" >> /tmp/xpm.log
  }

  if [ ! -n "$1" ] ;then
    log "error" "missing arr file."
    exit 1
  fi

  if [ ! -n "$2" ] ;then
    log "error" "missing log dir."
    exit 1
  fi

  if [ ! -n "$3" ] ;then
    log "error" "missing cmd dir."
    exit 1
  fi

  source $1
  dir=$2
  cmd=$3

  cd $dir

  for key in `ls |sed "s:^:$dir/:" |find -name "*.log" -mmin 6 -exec basename -s .log {} \;`
  do
    val=${arr_ps[${key}]}
    if [ -n "$val" ]; then
      pid=`ps aux |grep $val |grep -v grep |awk '{print $2}'`
      log "$key" "$val is offline."

      res=$?
      if [ -n "$pid" ];then
        kill $pid
        res=$?
        if [ $res -ne 0 ];then
          kill -9 $pid
          res=$?
        fi
      fi

      if [ $res -eq 0 ];then
        nohup java -jar $cmd/${val}-SNAPSHOT.jar >/dev/null 2>&1 &
        log "$key" "$val is online."
      else
        log "$key" "$val is failed to restart."
      fi
    fi
  done

#* * * * * /bin/bash /path/xpm.sh /path/xpm.arr <log path> <jar path>
~~~
