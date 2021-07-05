注意：必须先关闭SELinux服务
```
[root@localhost ~]# vi /etc/selinux/config
#找到并修改配置
SELINUX=disabled
```
重启机器或者用下面的临时关闭setenforce 0
```
[root@localhost ~]# setenforce 0
```

1、先安装vsftpd服务
```
[root@localhost ~]# yum -y install vsftpd
#开机启动
[root@localhost ~]# chkconfig vsftpd on
```

2、添加虚拟用户
```
[root@localhost ~]# vi /etc/vsftpd/vsftp_users.conf
ftp_test1
12345678
ftp_test2
12345678
```
里面添加用户，奇数行说用户名，偶数行是密码


3、生成vsftpd的认证文件
```
[root@localhost ~]# db_load -T -t hash -f /etc/vsftpd/vsftp_users.conf /etc/vsftpd/vsftp_users.db
```
注：如果没有db_load命令，请安装db4*相关rpm包

设置认证文件只对用户可读可写
```
[root@localhost ~]# chmod 600 /etc/vsftpd/vsftp_users.db
```

4、建立虚拟用户所需的PAM配置文件

把里面原来的内容清除，然后写入下面两句

auth required pam_userdb.so db=/etc/vsftpd/vsftp_users # 和上面的db文件名保持一致

account required pam_userdb.so db=/etc/vsftpd/vsftp_users

```
[root@localhost ~]# vi /etc/pam.d/vsftpd
auth required pam_userdb.so db=/etc/vsftpd/vsftp_users
account required pam_userdb.so db=/etc/vsftpd/vsftp_users
```

5、添加本地用户对应虚拟用户并指定目录
```
#添加用户并指定目录
[root@localhost ~]# useradd ftp_test1 -g ftp -s /sbin/nologin -d /www/test1 -m
[root@localhost ~]# chmod 755 /www/test1  #开放权限
[root@localhost ~]# chown -Rf ftp /www/test1  #指定ftp组

```

6、修改vsftpd配置文件
```
[root@localhost ~]# vi /etc/vsftpd/vsftpd.conf
找到以下配置并修改，配置里面缺少的就加上
chroot_local_user=YES
chroot_list_enable=YES
chroot_list_file=/etc/vsftpd/chroot_list
anonymous_enable=NO
local_enable=YES
write_enable=YES
anon_upload_enable=YES
anon_mkdir_write_enable=YES
dirmessage_enable=YES
ascii_upload_enable=YES
ascii_download_enable=YES
pam_service_name=vsftpd
userlist_enable=YES
tcp_wrappers=YES
guest_enable=YES
guest_username=ftp
user_config_dir=/etc/vsftpd/vuser_user
local_root=/www
allow_writeable_chroot=YES

```

7、虚拟用户建立单独的配置文件
```
[root@localhost ~]# cd /etc/vsftpd
[root@localhost vsftpd]# mkdir vuser_user  #对应配置文件的user_config_dir

#设置目录权限
[root@localhost vsftpd]# chmod 700 vuser_user
```

8、虚拟用户独立配置
```
#添加虚拟用户独立配置，文件名与用户名一致
[root@localhost ~]# vi /etc/vsftpd/vuser_user/ftp_test1
#加入以下配置
local_root=/www/test8  #指定用户主目录
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES
write_enable=YES
```


8、重启vsftpd
```
[root@localhost ~]# service vsftpd restart
```

```
有可能会提示下面的错误，但是不影响使用
Starting vsftpd for vsftp_users: 500 OOPS: missing value in config file for: zhouzeken
                                                           [FAILED]
```

再次添加用户则重复：第2、3、5、8、9步骤