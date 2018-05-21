# 基于 debian 的 docker 配置 java

参考:[Docker学习记录(二)-Dockerfile创建镜像](https://zhuanlan.zhihu.com/p/31244172)

~~~bash
# Dockerfile
# Version 0.3
#显示该镜像是基于 debian:stretch 镜像
FROM debian:stretch
#维护人信息
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

# 开启 crontab 服务
RUN apt-get install -yqq cron
RUN service cron start

# 增加用户
RUN apt-get install -yqq sudo vim
RUN useradd -G sudo ubuntu
RUN echo "ubuntu:ubuntu" |chpasswd

# 开放端口
EXPOSE 22 8070
#设置启动命令
CMD ["/usr/sbin/sshd","-D"]
~~~
