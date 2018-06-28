# nginx配置ci的坑

参考:[当CodeIgniter遇到Nginx报404错误的解决办法](https://blog.csdn.net/yanzi1225627/article/details/49699483)

参考:[正确修改 cgi.fix_pathinfo 与 Nginx 的配置](https://www.yephy.com/pathinfo-and-nginx-conf.html)

参考:[Nginx将项目配置在子目录](https://www.cnblogs.com/skyfynn/p/5951839.html)

~~~nginx

server {
	listen 80;
	listen [::]:80;
	server_name xxx.yyy.com;
	root /path/to/project/admin-pc;
	index index.php;

	location /{
		try_files $uri $uri/ /index.php?$args;
	}

	location /v1/{
		try_files $uri $uri/ /v1/index.php?$args;
	}

	location ~ \.php?.*$ {
		# 定义 path_info
		fastcgi_split_path_info ^(.+?.php)(/.*)$;
		include snippets/fastcgi-php.conf;
		fastcgi_pass unix:/run/php/php7.1-fpm.sock;
	}

	location ^~ /.git {
		return 404;
	}
}

~~~

~~~bash

ubuntu@unknown:~/to/project$ tree -d -L 2
.
├── admin-pc
│   ├── application
│   ├── assets
│   ├── third_party
│   ├── upload
│   ├── v1 -> /path/to/project/open-api-v1
│   └── vendor
└── open-api-v1
    ├── application
    └── vendor

~~~