### 简介
本文档记录了在Windows平台下构建、配置和启动RocketMQ的步骤。

------

### 环境
- 操作系统：Windows XP Professional 2002 SP3
- JDK：1.6.0_20
- Maven：3.0.4
- Git：1.7.4.msysgit.0
- MinGW：1.0.11

------

### 构建

##### 下载源码

    cd F:\rocketmq
    git clone https://github.com/alibaba/RocketMQ.git

##### 编译源码

    set MAVEN_OPTS=-Xmx1024m`
    mvn -Dmaven.test.skip=true clean package install assembly:assembly -U

编译后分发包位于:

    F:\rocketmq\RocketMQ\target\alibaba-rocketmq-3.2.2\alibaba-rocketmq

##### 修改脚本

***runserver.sh
    
    #===========================================================================================
    # JVM 参数配置
    #===========================================================================================
    JAVA_OPT="${JAVA_OPT} -server -Xms128M -Xmx128M -Xmn32M -XX:PermSize=32m -XX:MaxPermSize=32m"
    JAVA_OPT="${JAVA_OPT} -XX:+UseConcMarkSweepGC -XX:+UseCMSCompactAtFullCollection -XX:CMSInitiatingOccupancyFraction=70 -XX:+CMSParallelRemarkEnabled -XX:SoftRefLRUPolicyMSPerMB=0 -XX:+CMSClassUnloadingEnabled -XX:SurvivorRatio=8 -XX:+DisableExplicitGC"
    JAVA_OPT="${JAVA_OPT} -verbose:gc -Xloggc:${ROCKETMQ_HOME}/logs/rmq_srv_gc.log -XX:+PrintGCDetails"
    JAVA_OPT="${JAVA_OPT} -XX:-OmitStackTraceInFastThrow"
    JAVA_OPT="${JAVA_OPT} -Djava.ext.dirs=${BASE_DIR}/lib"
    #JAVA_OPT="${JAVA_OPT} -Xdebug -Xrunjdwp:transport=dt_socket,address=9555,server=y,suspend=n"
    #JAVA_OPT="${JAVA_OPT} -cp ${CLASSPATH}"
    
    $JAVA ${JAVA_OPT} $@

修改如下地方：
- JVM堆配置：Xms128M -Xmx128M -Xmn32M -XX:PermSize=32m -XX:MaxPermSize=32m
- GC日志位置：-Xloggc:${ROCKETMQ_HOME}/logs/rmq_srv_gc.log
- 删除CLASSPATH：因为`export CLASSPATH=.:${BASE_DIR}/conf`导致`CLASSPATH`中可能会引入空格，导致后面的命令执行失败

***runbroker.sh
    
    #===========================================================================================
    # JVM 参数配置
    #===========================================================================================
    JAVA_OPT="${JAVA_OPT} -server -Xms256M -Xmx256M -Xmn128M -XX:PermSize=32m -XX:MaxPermSize=32M"
    JAVA_OPT="${JAVA_OPT} -XX:+UseConcMarkSweepGC -XX:+UseCMSCompactAtFullCollection -XX:CMSInitiatingOccupancyFraction=70 -XX:+CMSParallelRemarkEnabled -XX:SoftRefLRUPolicyMSPerMB=0 -XX:+CMSClassUnloadingEnabled -XX:SurvivorRatio=8 -XX:+DisableExplicitGC"
    JAVA_OPT="${JAVA_OPT} -verbose:gc -Xloggc:${ROCKETMQ_HOME}/logs/rmq_bk_gc.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps"
    JAVA_OPT="${JAVA_OPT} -XX:-OmitStackTraceInFastThrow"
    JAVA_OPT="${JAVA_OPT} -Djava.ext.dirs=${BASE_DIR}/lib"
    #JAVA_OPT="${JAVA_OPT} -Xdebug -Xrunjdwp:transport=dt_socket,address=9555,server=y,suspend=n"
    #JAVA_OPT="${JAVA_OPT} -cp ${CLASSPATH}"
    
    numactl --interleave=all pwd > /dev/null 2>&1
    if [ $? -eq 0 ]
    then
        numactl --interleave=all $JAVA ${JAVA_OPT} $@
    else
        $JAVA ${JAVA_OPT} $@
    fi

修改地方同`runserver.sh`

### 运行

依次运行`mqnamesrv`和`mqbroker`
    
    F:\rocketmq\RocketMQ\target\alibaba-rocketmq-3.2.2\alibaba-rocketmq\bin>sh
    sh-3.1$ ./mqnamesrv &
    [1] 11424
    sh-3.1$ The Name Server boot success.

    sh-3.1$ ./mqbroker &
    [2] 12032
    sh-3.1$ The broker[licp, 192.168.1.31:10911] boot success.

    sh-3.1$ ps
          PID    PPID    PGID     WINPID  TTY  UID    STIME COMMAND
    I    9496       1    9496       9496  con  500 12:13:16 /bin/sh
        10536       1   10536      10536  con  500 12:23:36 /bin/sh
        11424   10536   11424      11504  con  500 12:23:41 /bin/sh
        11784   11424   11424      11772  con  500 12:23:42 /bin/sh
        12028   11784   11424      12028  con  500 12:23:42 /d/bessystem/env/Java/jdk1.6.0_20/bin/java
        12032   10536   12032      12240  con  500 12:23:56 /bin/sh
         9600   12032   12032       6884  con  500 12:23:57 /bin/sh
        10344    9600   12032      10344  con  500 12:23:57 /d/bessystem/env/Java/jdk1.6.0_20/bin/java
        11568   10536   11568      11348  con  500 12:24:02 /bin/ps

可以看到NameServer和Broker启动成功
