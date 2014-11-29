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

#### 下载源码

    cd F:\rocketmq
    git clone https://github.com/alibaba/RocketMQ.git

#### 编译源码

    set MAVEN_OPTS=-Xmx1024m`
    mvn -Dmaven.test.skip=true clean package install assembly:assembly -U

编译后分发包位于:

    F:\rocketmq\RocketMQ\target\alibaba-rocketmq-3.2.2\alibaba-rocketmq

#### 修改默认运行参数

***runserver.sh***
    
    #===========================================================================================
    # JVM 参数配置
    #===========================================================================================
    JAVA_OPT="${JAVA_OPT} -server -Xms128M -Xmx128M -Xmn32M -XX:PermSize=32M -XX:MaxPermSize=32M"
    JAVA_OPT="${JAVA_OPT} \XX:+UseConcMarkSweepGC -XX:+UseCMSCompactAtFullCollection -XX:CMSInitiatingOccupancyFraction=70 -XX:+CMSParallelRemarkEnabled -XX:SoftRefLRUPolicyMSPerMB=0 -XX:+CMSClassUnloadingEnabled -XX:SurvivorRatio=8 -XX:+DisableExplicitGC"
    JAVA_OPT="${JAVA_OPT} -verbose:gc -Xloggc:${ROCKETMQ_HOME}/logs/rmq_srv_gc.log -XX:+PrintGCDetails"
    JAVA_OPT="${JAVA_OPT} -XX:-OmitStackTraceInFastThrow"
    JAVA_OPT="${JAVA_OPT} -Djava.ext.dirs=${BASE_DIR}/lib"
    #JAVA_OPT="${JAVA_OPT} -Xdebug -Xrunjdwp:transport=dt_socket,address=9555,server=y,suspend=n"
    #JAVA_OPT="${JAVA_OPT} -cp ${CLASSPATH}"

    $JAVA ${JAVA_OPT} $@


