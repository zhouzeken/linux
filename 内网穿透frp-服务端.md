重启
1.查看进程id：ps -aux|grep frp
2.杀死进程：kill 100
3.重启frp：nohup ./frps -c ./frps.ini &