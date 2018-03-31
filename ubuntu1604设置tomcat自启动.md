# ubuntu1604设置tomcat自启动

参考:[Ubuntu下设置tomcat为服务（开机启动）](https://blog.csdn.net/wj1066/article/details/73353939)

参考:[Ubuntu配置tomcat开机自动启动](https://blog.csdn.net/gisredevelopment/article/details/51546603)

~~~bash
# 将tomcat下bin文件夹的catalina.sh文件拷贝到/etc/init.d下，并改名
$ cd /etc/init.d/
$ sudo cp /path/apache-tomcat-8.x.xx/catalina.sh tomcat

# 修改tomcat文件，增加环境变量
$ sudo vi tomcat
#!/bin/sh
CATALINA_HOME=/path/apache-tomcat-8.x.xx
JAVA_HOME=/usr/lib/jvm/java-8-oracle
...

# 增加执行权限
$ sudo chmod +x tomcat

# 在自启动文件夹下创建tomcat的软（或硬）连接，K表示不自启动，S表示自启动。
$ sudo ln -s /etc/init.d/tomcat /etc/rc2.d/S99tomcat

# 或者
$ sudo update-rc.d –f tomcat defaults

# 用如下命令查看是否设置成功
$ sudo sysv-rc-conf --list tomcat
~~~

