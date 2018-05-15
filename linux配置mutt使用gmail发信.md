# linux配置mutt使用gmail发信

参考:[linux配置shadowsocks客户端](https://my.oschina.net/u/1432769/blog/619651)

参考:[Ubuntu 下使用 mutt 和 msmtp 发送 Gmail 邮件](http://www.cnblogs.com/refrag/archive/2012/11/28/2793533.html)

~~~bash
sudo apt-get install python-pip
sudo pip install shadowsocks
nohup sslocal -s server_ip -p server_port  -l 1080 -k password -t 600 -m aes-256-cfb >/dev/null 2>&1 &

sudo apt-get install mutt msmtp
sudo apt-get install gnutls-bin
sudo apt-get install ca-certificates

vi ~/.msmtprc
	defaults
    tls on
    tls_starttls on
    tls_trust_file /etc/ssl/certs/ca-certificates.crt

    account default
    host smtp.gmail.com
    port 587
    proxy_host 127.0.0.1
    proxy_port 1080
    auth on
    user who@gmail.com
    password 123456
    from who@gmail.com
    logfile ~/msmtp.log

chmod 600 ~/.msmtprc

vi ~/.muttrc
	set sendmail="/usr/bin/msmtp"
	set use_from=yes
	set from = "who@gmail.com"
	set realname = "who"
	set envelope_from=yes

echo "hello" |mutt -s "world" to@example.com
~~~

打开 who@gmail.com 收件箱，“查看被阻止的登录尝试”，按照提示“允许使用不够安全的应用”