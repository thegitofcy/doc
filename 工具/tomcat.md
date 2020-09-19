# 1.启动

## Service Tomcat start

需要`jsvc` 文件.

1. 目录`tomcat/bin/commons-daemon-native.tar.gz`, 解压此文件
2. 进入 `解压目录/unix`, 分别执行 `./configure` 和 `make`两个命令
3. 以上步骤完成, 就会生成 `jsvc` 文件, 将此文件拷贝值 `tomcat/bin`下.



在 `/etc/init.d` 下创建 `tomcat(启动时的名称)` 文件.

```shell
#!/bin/bash
#
# Tomcat daemon.sh   Startup script for the Tomcat Server
#
# chkconfig: 2345 10 90
# description: Tomcat 8 server
# processname: java
# Source function library.
. /etc/rc.d/init.d/functions
. /etc/profile
#Tomcat Daemon path
daemon_path=/usr/local/tomcat7/bin
# See how we were called.
run(){
    echo "Start Tomcat without detaching from console..."
    $daemon_path/daemon.sh run
    touch /var/lock/subsys/tomcat
}
start(){
     echo "Start Tomcat..."
     $daemon_path/daemon.sh start
     touch /var/lock/subsys/tomcat
}
stop(){
     echo "Stop Tomcat..."
     $daemon_path/daemon.sh stop
     rm -f /var/lock/subsys/tomcat.pid echo
}
restart(){
     stop
     start
}
version(){
     echo "What version of commons daemon and Tomcat are you running?"
     $daemon_path/daemon.sh version
}
status(){
    ps ax --width=1000 | grep "[o]rg.apache.catalina.startup.Bootstrap" | awk '{printf $1 " "}' | wc | awk '{print $2}' > /tmp/tomcat_process_count.txt
    read line < /tmp/tomcat_process_count.txt
    if [ $line -gt 0 ]; then
        echo -n "tomcat ( pid "
        ps ax --width=1000 | grep "org.apache.catalina.startup.Bootstrap start" | awk '{printf $1 " "}'
        echo -n ") is running..."
        echo
    else
        echo "Tomcat is stopped"
    fi
}
case "$1" in
     run)
         run
         ;;
     start)
         start
         ;;
     stop)
         stop
         ;;
     restart)
         restart
         ;;
     status)
         status
         ;;
     version)
         version
         ;;
     *)
         echo "Usage: $0 {run|start|stop|restart|status|version}"
         exit 1
         ;;
esac
exit 0
```



命令:

```shell
service tomcat start
service tomcat stop
service tomcat restart
service tomcat status
// tomcat 为 /etc/init.d 下创建的文件名.
```

