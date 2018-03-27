# 基于 AWS ELB 的 apache 获取真实 ip 的配置

参考:[apache 2.4 mod_remoteip for get real ip on AWS ELB](https://medium.com/@jiraknet/apache-2-4-mod-remoteip-for-get-real-ip-on-aws-elb-6e9f40876b06)

参考:[Apache 获取真实ip的配置的实现方法](http://www.jb51.net/article/125415.htm?utm_source=debugrun&utm_medium=referral)

1. Load Module
~~~apache
<IfModule mod_remoteip.c>
 RemoteIPHeader X-Forwarded-For
</IfModule>
~~~

2. Change Log Format

~~LogFormat "%h %l %u %t \\"%r\\" %>s %b" common~~
~~~apache
LogFormat "%a %l %u %t \"%r\" %>s %b" common
~~~

[https://httpd.apache.org/docs/2.4/mod/mod_log_config.html#formats](https://httpd.apache.org/docs/2.4/mod/mod_log_config.html#formats)