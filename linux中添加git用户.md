1、新建用户
```
指定目录+用户组
[root@localhost ~]# useradd git_php_zzk -d /home/git_user/php/git_php_zzk -m -G 515

```

2、设置用户密码
```
[root@localhost ~]# passwd git_php_zzk
密码规则：Git_php_zzk888
```

3、删除用户,-r是和用户主目录一起删除
```
[root@localhost ~]# userdel -r test1
```

4、修改用户
```
[root@localhost ~]# usermod test1 -d /www/test1_1 -m -g ftp
```

4、查看所有用户
```
[root@localhost ~]# vi /etc/passwd
```