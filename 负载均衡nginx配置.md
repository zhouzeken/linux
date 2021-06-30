1、在nginx.conf配置文件的http体内添加
```
upstream vm.test.xxx{
    server 192.168.145.122:4001;
    server 192.168.145.123:4001;
}

server {
    listen       80;
    server_name  localhost;

    location / {
        #root   html;
        index  index.php index.html index.htm;
        #使用test分配规则，即刚刚自定义添加的upstream节点
        proxy_pass http://vm.test.xxx;
    }
}
```

2、开放服务器的4001端口号

3、192.168.145.122:4001配置
```
server {
        listen        4001;
        server_name  _;
        root   "/www/test";
        location / {
            index index.php index.html error/index.html;
        }
        location ~ \.php(.*)$ {
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_split_path_info  ^((?U).+\.php)(/?.+)$;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            fastcgi_param  PATH_INFO  $fastcgi_path_info;
            fastcgi_param  PATH_TRANSLATED  $document_root$fastcgi_path_info;
            include        fastcgi_params;
        }
}
```

4、192.168.145.123:4001配置
```
server {
        listen        4001;
        server_name  _;
        root   "/www/test";
        location / {
            index index.php index.html error/index.html;
        }
        location ~ \.php(.*)$ {
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_split_path_info  ^((?U).+\.php)(/?.+)$;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            fastcgi_param  PATH_INFO  $fastcgi_path_info;
            fastcgi_param  PATH_TRANSLATED  $document_root$fastcgi_path_info;
            include        fastcgi_params;
        }
}


```