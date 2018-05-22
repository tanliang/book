# 基于 debian 的 docker 配置 java

参考:[Ubuntu 安装 Docker CE 社区版](https://www.getnas.com/2017/07/2620.html)

参考:[Docker学习记录(二)-Dockerfile创建镜像](https://zhuanlan.zhihu.com/p/31244172)

参考:[Docker容器启动后自运行脚本的配置](https://github.com/johnnian/Blog/issues/13)

参考:[Docker容器和主机如何互相拷贝传输文件](http://xiaorui.cc/2015/04/12/docker容器和主机如何互相拷贝传输文件/)


Dockerfile
~~~bash
# Version 0.3
# 显示该镜像是基于 debian:stretch 镜像
FROM debian:stretch
# 维护人信息
MAINTAINER tanliang tanjnr@gmail.com

# 设置debian的镜像，加快速度
#RUN echo 'deb http://mirrors.163.com/debian/ stretch main non-free contrib' > /etc/apt/sources.list \
#    && echo 'deb http://mirrors.163.com/debian/ stretch-updates main non-free contrib' >> /etc/apt/sources.list \
#    && echo 'deb http://mirrors.163.com/debian/ stretch-backports main non-free contrib' >> /etc/apt/sources.list \
#    && echo 'deb http://mirrors.163.com/debian-security/ stretch/updates main non-free contrib' >> /etc/apt/sources.list

# 设置时区 更新源
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
RUN apt-get update -yqq

RUN apt-get install tzdata
RUN dpkg-reconfigure --frontend noninteractive tzdata

# 安装 ssh 软件
RUN apt-get install -yqq openssh-server
RUN mkdir -p /var/run/sshd
RUN mkdir -p /root/.ssh
# 取消pam限制
RUN sed -ri 's/session  required   pam_loginuid.so/#session    required  pam_loginuid.so/g' /etc/pam.d/sshd
# 复制配置文件到相应位置
COPY authorized_keys /root/.ssh/authorized_keys

# 更改语言 支持中文
RUN apt-get install -yqq locales
RUN sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen && \
    sed -i -e 's/# zh_CN.UTF-8 UTF-8/zh_CN.UTF-8 UTF-8/' /etc/locale.gen && \
    echo 'LANG="zh_CN.UTF-8"'>/etc/default/locale && \
    dpkg-reconfigure --frontend=noninteractive locales && \
    update-locale LANG=zh_CN.UTF-8

# 安装java
RUN apt-get install -yqq wget
RUN \
    mkdir /mysoft && cd /mysoft && \
#    wget -q --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" https://zcc2018.oss-cn-beijing.aliyuncs.com/jdk-8u171-linux-x64.tar.gz  && \
    wget -q --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u171-b11/512cd62ec5174c3487ac17c61aaa89e8/jdk-8u171-linux-x64.tar.gz  && \
    tar -zxf $(ls) && \
    mv $(ls -d */) oracle-jdk && \
    mkdir -p /usr/local/java && \
    mv oracle-jdk /usr/local/java/oracle-jdk && \
    rm -rf /mysoft

ENV JAVA_HOME /usr/local/java/oracle-jdk
ENV PATH $PATH:$JAVA_HOME/bin
RUN echo "export JAVA_HOME=/usr/local/java/oracle-java" >> /etc/profile
RUN echo "export PATH=$PATH:$JAVA_HOME" >> /etc/profile

# 安装更多服务
RUN apt-get install -yqq cron redis-server
RUN echo "#!/bin/sh -e" > /etc/rc.local
RUN echo "service ssh start" >> /etc/rc.local
RUN echo "service cron start" >> /etc/rc.local
RUN echo "service redis-server start" >> /etc/rc.local
RUN echo "exit 0" >> /etc/rc.local
RUN chmod 755 /etc/rc.local

# 增加用户
RUN apt-get install -yqq sudo vim
RUN useradd -d /home/ubuntu -m -s /bin/bash -G sudo ubuntu
RUN echo "ubuntu:ubuntu" |chpasswd

# 开放端口
EXPOSE 22 8070
~~~

拷贝本机的 id_ras 用来免密
~~~bash
ssh-keygen
cd /path of Dockerfile
cat ~/.ssh/id_rsa.pub >authorized_keys
~~~

根据 Dockerfile 构建镜像
~~~bash
sudo docker build --no-cache -t debian_java:0.3 .
~~~

根据镜像启动容器
~~~bash
sudo docker run -itd -v /home/ubuntu:/mnt -p 8071:8070 -p 8022:22 --name niubit1 debian_java:0.3 /bin/bash -c "/etc/rc.local;/bin/bash"
~~~

