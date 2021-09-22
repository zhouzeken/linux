```
#查看任务列表
[root@localhost ~]# crontab -l


#编辑任务
[root@localhost ~]# crontab -e

```

systemctl start crond    //启动服务
systemctl stop crond     //关闭服务
systemctl restart crond  //重启服务
systemctl reload crond   //重新载入配置
systemctl status crond   //查看服务状态 