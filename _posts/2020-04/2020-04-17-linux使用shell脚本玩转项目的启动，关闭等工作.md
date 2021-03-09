---
title: linux使用shell脚本玩转项目的启动，关闭等工作
categories: linux
tags: shell
date: 2020-04-17 11:14:23
---

# 背景

看到同事蛮好的脚本就琢磨了下，然后搬运下，在博客下的友链可以找到原文

2021-03-09 添加一个超清晰的linux启动web项目的脚本

# 脚本及注释

```shell
#使用方法：
#sh restart.sh start # 启动
#sh restart.sh # 重启
#sh restart.sh restart # 重启
#sh restart.sh check # 检测服务是否启动，没有启动，则启动之，否则不做其他操作
#sh restart.sh kill # kill 掉已启动的服务进程

#!/usr/bin/env bash
# 根据自己的修改对应jdk位置
export LANG=en_US.UTF-8
export JAVA_HOME=/usr/java/jdk1.8.0_144
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$CLASSPATH

# 根据自己的修改对应路径
cd `dirname $0`
CURRENT_DIR=`pwd`
LOGS_DIR=$CURRENT_DIR/logs
STDOUT_FILE=$CURRENT_DIR/logs/stdout.log
PROJECT_HOME=$CURRENT_DIR/project
JAR_FILE=$PROJECT_HOME/data-check.jar

# pid
PIDS=`ps -ef | grep java | grep -v grep | grep "$JAR_FILE" | awk '{print $2}'`

# Graceful shutdown
function gracefulShutdown(){
    if [ -z "$PIDS" ]; then
        echo "[$(date '+%Y-%m-%d %H:%M:%S')] $JAR_FILE does not started!" | tee -a $STDOUT_FILE
    else
        echo "[$(date '+%Y-%m-%d %H:%M:%S')] $JAR_FILE kill $PIDS begining" | tee -a $STDOUT_FILE
        for PID in $PIDS ; do
            kill $PID > /dev/null 2>&1
            echo "[$(date '+%Y-%m-%d %H:%M:%S')] $JAR_FILE kill $PID success" | tee -a $STDOUT_FILE
        done

        # check for graceful shutdown
        COUNT=0
        while [ $COUNT -lt 1 ]; do
            sleep 1
            COUNT=1
            for PID in $PIDS ; do
                PID_EXIST=`ps -f -p $PID | grep java`
                if [ -n "$PID_EXIST" ]; then
                    COUNT=0
                    break
                fi
            done
        done
    fi
}

function operate(){
    if [[ "$1" = "kill" ]]; then
        gracefulShutdown
    elif [[ "$1" = "start" ]] ; then
        cd $PROJECT_HOME
        # starting  指定使用的配置 是pro,local,dev
        nohup $JAVA_HOME/bin/java -server -Xms1g -Xmx1g -Xss256k -Dloader.path=$PROJECT_HOME/lib/ -Djava.io.tmpdir=$LOGS_DIR -XX:+DisableExplicitGC -XX:+UseConcMarkSweepGC -XX:+CMSParallelRemarkEnabled -XX:LargePageSizeInBytes=128m -XX:+UseFastAccessorMethods -XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=70 -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=$LOGS_DIR/$PIDS.hprof -Ddubbo.protocol.port=20882 -Dspring.profiles.active=pro -jar $JAR_FILE >> $STDOUT_FILE 2>&1 &
        PIDS=`ps -ef | grep java | grep -v grep | grep "$JAR_FILE" | awk '{print $2}'`
        echo "[$(date '+%Y-%m-%d %H:%M:%S')] $JAR_FILE started OK! pid: $PIDS" | tee -a $STDOUT_FILE
    else
        echo "[$(date '+%Y-%m-%d %H:%M:%S')] $JAR_FILE is not support $1" | tee -a $STDOUT_FILE
    fi
}

if [[ "$1" = "start" || "$1" = "check" ]]; then
    if [ -n "$PIDS" ]; then
        echo "[$(date '+%Y-%m-%d %H:%M:%S')] $JAR_FILE already started! pid: $PIDS" >> $STDOUT_FILE
        exit 1
    fi
    operate start
elif [[ "$1" = "" || "$1" = "restart" ]]; then
    operate kill
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $JAR_FILE starting" | tee -a $STDOUT_FILE
    operate start
elif [[ "$1" = "kill" ]]; then
    operate kill
else
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $JAR_FILE is not support $1" | tee -a $STDOUT_FILE
fi
```



# 脚本2及注释

```shell
#!/bin/bash 

#指定jdk
JAVA_HOME=/usr/java/jdk1.8.0_221
JAVA=$JAVA_HOME/bin/java

#指定jar
APP_NAME=/project/newctest/admin/newtTest.jar
PIDF_NAME=Admin

#指定PIDF文件，停止项目也可以直接从该文件中获取PIDF
PIDF=$PIDF_NAME\.pid

#JVM参数
#JVM="-server -Xms512m -Xmx512m -XX:PermSize=64M -XX:MaxNewSize=128m -XX:MaxPermSize=128m -Djava.awt.headless=true -XX:+CMSClassUnloadingEnabled -XX:+CMSPermGenSweepingEnabled"

#指定spring核心配置文件
#APPFILE_PATH="-Dspring.config.location=/usr/local/demo/config/application-demo1.properties"

#使用说明
usage() {
	echo "Usage: sh 执行脚本.sh [start|stop|restart|status]" 
	exit 1 
}

#检查程序是否在运行 
is_exist(){ 
	pid=`ps -ef|grep $APP_NAME|grep -v grep|awk '{print $2}' ` 
	#如果不存在返回1,存在返回0 
	if [ -z "${pid}" ]; then 
		return 1 
	else 
		return 0 
	fi 
} 

#启动方法 
start(){ 
	is_exist 
	if [ $? -eq "0" ]; then 
		echo "${APP_NAME} is already running. pid=${pid} ." 
	else 
		#指定JVM参数、spring核心配置文件启动
		#nohup java $JVM -jar $APPFILE_PATH $APP_NAME > /dev/null 2>&1 
		#直接启动
		nohup $JAVA -jar $APP_NAME >runLog.log 2>&1 &
		#PID写入文件
		echo $! > $PIDF
		echo "start ${APP_NAME} successed PID=$!" 
	fi
} 

#停止方法 
stop(){ 
	is_exist 
	if [ $? -eq "0" ]; then 
		kill $pid 
		rm -rf $PIDF
	  	sleep 5
	  	is_exist
	  	if [ $? -eq "0" ]; then 
		    echo "begin kill -9 $pid"
		    kill -9  $pid
		    sleep 2
		    echo "${APP_NAME} process stopped"  
	  	else
	    	echo "${APP_NAME} is not running"
		fi  
	else 
		echo "${APP_NAME} is not running" 
	fi 
} 

#输出运行状态 
status(){ 
	is_exist 
	if [ $? -eq "0" ]; then 
		echo "${APP_NAME} is running. Pid is ${pid}" 
	else 
		echo "${APP_NAME} is NOT running." 
	fi 
} 

#重启 
restart(){ 
	stop 
	start 
} 

#根据输入参数,选择执行对应方法,不输入则执行使用说明 
case "$1" in 
"start") 
start 
;; 
"stop") 
stop 
;; 
"status") 
status 
;; 
"restart") 
restart 
;; 
*) 
usage 
;;
esac

```