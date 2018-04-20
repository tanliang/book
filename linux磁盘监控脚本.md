# linux磁盘监控脚本

参考:[Linux磁盘空间监控告警](http://www.cnblogs.com/kerrycode/p/3415242.html)

~~~bash
#!/bin/bash
if [ ! -n "$1" ] ;then
  echo "missing mailto address."
  exit 1
fi

for d in `df -P | grep /dev | awk '{print $5}' | sed 's/%//g'`
 do
  if [ $d -gt 80 ]; then
    subject="disk($d%) at `hostname --fqdn`"
    echo "`df -h`" |mail -s "$subject" "$1"
    exit 0
  fi
done

#0 0 * * * /bin/bash /your path/dm.sh <mailto>
~~~