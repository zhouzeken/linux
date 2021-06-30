第一台服务器(192.168.145.80)

a、安装rsync
```
[root@nginx src]# tar zxvf rsync-3.0.9.tar.gz 
[root@nginx src]# cd rsync-3.0.9 
[root@nginx rsync-3.0.9]# ./configure --prefix=/usr/local/rsync 
[root@nginx rsync-3.0.9]# make 
[root@nginx rsync-3.0.9]# make install

```
b、创建密码认证文件(注意：第一台服务器的密码文件，只需要密码，而第二台服务器则需要用户:密码)
```
[root@nginx rsync-3.0.9]# cd /usr/local/rsync/
[root@nginx rsync]# echo "rsync-pwd" >/usr/local/rsync/rsync1.passwd
```
c、给密码文件赋予600权限
```
[root@nginx rsync]# chmod 600 rsync1.passwd
```

d、安装inotify
```
[root@nginx src]# tar zxvf inotify-tools-3.14.tar.gz 
[root@nginx src]# cd inotify-tools-3.14 
[root@nginx inotify-tools-3.14]# ./configure --prefix=/usr/local/inotify 
[root@nginx inotify-tools-3.14]# make 
[root@nginx inotify-tools-3.14]# make install

```

e、创建监控脚本.sh文件
```
#!/bin/bash
host=192.168.145.90
src=/www/test/
des=test_web
user=webuser
/usr/local/inotify/bin/inotifywait -mrq --timefmt '%d/%m/%y %H:%M' --format '%T %w%f%e' -e modify,delete,create,attrib $src \ 
| while read files
do
/usr/bin/rsync -vzrtopg --delete --progress --password-file=/usr/local/rsync/rsync1.passwd $src $user@$host::$des
echo "${files} was rsynced" >>/tmp/rsync.log 2>&1
done
```
注意：host为第二台服务器IP，src为要本服务器监控的路径，test_web是认证模块名称(对应第二台服务器的rsync配置)，最后把监控脚本命名为rsync1.sh放在要监控的路径下。此处应是/www/

 f、给监控脚本赋予764权限
 ```
 [root@nginx tmp]# chmod 764 /www/rsync1.sh
 ```


第二台服务器(192.168.145.90)
a、安装rsync
```
[root@nginx src]# tar zxvf rsync-3.0.9.tar.gz 
[root@nginx src]# cd rsync-3.0.9 
[root@nginx rsync-3.0.9]# ./configure --prefix=/usr/local/rsync 
[root@nginx rsync-3.0.9]# make 
[root@nginx rsync-3.0.9]# make install

```
b、创建密码认证文件(注意：第二台服务器需要用户:密码,两台服务器的密码保持一致)
```
[root@nginx rsync-3.0.9]# cd /usr/local/rsync/
[root@nginx rsync]# echo "webuser:rsync-pwd" >/usr/local/rsync/rsync2.passwd
```
c、给密码文件赋予600权限
```
[root@nginx rsync]# chmod 600 rsync1.passwd
```

d、创建rsync配置文件
```
uid = root 
gid = root 
use chroot = no 
max connections = 10 
strict modes = yes
pid file = /var/run/rsyncd.pid 
lock file = /var/run/rsync.lock 
log file = /var/log/rsyncd.log 
[test_web] 
path = /www/test/
ignore errors
read only = no
write only = no
hosts allow = 192.168.145.80
hosts deny = *
list = false
uid = root
gid = root
auth users = webuser
secrets file = /usr/local/rsync/rsync2.passwd
```

h、启动该配置文件
```
[root@nginx-backup rsync]# /usr/local/rsync/bin/rsync --daemon --config=/usr/local/rsync/rsync2.conf
```
需要开机启动的话：
```
[root@nginx-backup rsync]# echo "/usr/local/rsync/bin/rsync --daemon --config=/usr/local/rsync/rsync2.conf" >> /etc/rc.local
```


最后启动第一台服务器的监控脚本
在80服务器
```
[root@nginx tmp]# sh /www/rsync1.sh &
```

sh需要开机启动的话，在服务器/etc/init.d/创建执行文件
```
vi /etc/init.d/rsyncsh
```

里面内容为
```
#!/bin/sh
# chkconfig: 3 88 88
sh /www/rsync1.sh &
```

然后添加到开机启动
```
chkconfig rsyncsh on
```

注意：如果连接不上，防火墙需要开放873端口

转: https://blog.csdn.net/hyh9401/article/details/52043134

参: https://www.cnblogs.com/gpfeisoft/p/6112847.html