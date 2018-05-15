# linux批量监控系统状态

通过 ssh 远程执行命令，对系统状态进行监控，主要3个指标，磁盘空间(df)、内存使用(free)，以及磁盘读写(iostat)。

有新机器时，只需增加对应 .ssh/config 配置即可

~~~bash
# 第一步
# 配置 ssh 自动登录
vi ~/.ssh/config
    Host ec2_prod
      User ubuntu
      Hostname xxx.xxx.xxx.xxx
      IdentityFile /path/xxx.pem

    Host ec2_test
      Hostname xxx.xxx.xxx.xxx
      User ubuntu
      IdentityFile /path/xxx.pem

＃ 第二步
＃ 读入 ssh 服务列表 ssh_cmd.txt 生成 ssh 命令待执行
vi ~/ssh_cmd.sh
    #!/bin/bash

    if [ ! -n "$1" ] ;then
      echo "missing server list."
      exit 1
    fi

    if [ ! -n "$2" ] ;then
      echo "missing command."
      exit 1
    fi

    log=/tmp/${0##*/}.tmp
    echo "" > $log
    cat $1 |while read row
    do
      if [ -n "$row" ] ;then
        echo "echo \"$row\"" >> $log
        echo "ssh $row \"$2\"" >> $log
        echo "echo" >> $log
      fi
    done

    source $log

vi ~/ssh_cmd.txt
    ec2_prod
    ec2_test

＃ 第三步
＃ 监控内存
vi ~/ssh_free.sh
    #!/bin/bash
    if [ ! -n "$1" ] ;then
      echo "missing mailto address."
      exit 1
    fi

    tmp=/tmp/free.tmp

    /path/ssh_cmd.sh /path/ssh_cmd.txt "free" > $tmp

    cat $tmp |while read row
    do
      if [ -n "$row" ] ;then
        d=`echo $row |awk '/Mem/ {print int($3/$2*100)}'`
        if [ -n "$d" ] && [ $d -gt 90 ];then
          cat $tmp |mutt -s "memory usage up $d" "$1"
          exit 0
        fi
      fi
    done

# 监控磁盘
vi ~/ssh_df.sh
    #!/bin/bash
    if [ ! -n "$1" ] ;then
      echo "missing mailto address."
      exit 1
    fi

    tmp=/tmp/df.tmp

    /path/ssh_cmd.sh /path/ssh_cmd.txt "df -P |grep /dev" > $tmp

    cat $tmp |while read row
    do
      if [ -n "$row" ] ;then
        d=`echo $row |awk '{print $5}' | sed 's/%//g'`
        if [ -n "$d" ] && [ $d -gt 80 ];then
          cat $tmp |mutt -s "disk usage up $d" "$1"
          exit 0
        fi
      fi
    done

# 监控IO，需要安装 sudo apt-get install sysstat
vi ~/ssh_iostat.sh
    #!/bin/bash
    if [ ! -n "$1" ] ;then
      echo "missing mailto address."
      exit 1
    fi

    tmp=/tmp/iostat.tmp

    /path/ssh_cmd.sh /path/ssh_cmd.txt "iostat -x" > $tmp

    cat $tmp |while read row
    do
      if [ -n "$row" ] ;then
        d=`echo $row |awk '{print int($14)}'`
        #echo $d
        if [ -n "$d" ] && [ $d -gt 60 ];then
          cat $tmp |mutt -s "memory usage up $d" "$1"
          exit 0
        fi
      fi
    done

#0 8 * * * /bin/bash /path/ssh_df.sh to@example.com
#15 8 * * * /bin/bash /path/ssh_free.sh to@example.com
#30 8 * * * /bin/bash /path/ssh_iostat.sh to@example.com
~~~
