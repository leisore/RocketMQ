### ���
���ĵ���¼����Windowsƽ̨�¹��������ú�����RocketMQ�Ĳ��衣

------

### ����
- ����ϵͳ��Windows XP Professional 2002 SP3
- JDK��1.6.0_20
- Maven��3.0.4
- Git��1.7.4.msysgit.0
- MinGW��1.0.11

------

### ����

##### ����Դ��

    cd F:\rocketmq
    git clone https://github.com/alibaba/RocketMQ.git

##### ����Դ��

    set MAVEN_OPTS=-Xmx1024m`
    mvn -Dmaven.test.skip=true clean package install assembly:assembly -U

�����ַ���λ��:

    F:\rocketmq\RocketMQ\target\alibaba-rocketmq-3.2.2\alibaba-rocketmq

##### �޸Ľű�

***runserver.sh
    
    #===========================================================================================
    # JVM ��������
    #===========================================================================================
    JAVA_OPT="${JAVA_OPT} -server -Xms128M -Xmx128M -Xmn32M -XX:PermSize=32m -XX:MaxPermSize=32m"
    JAVA_OPT="${JAVA_OPT} -XX:+UseConcMarkSweepGC -XX:+UseCMSCompactAtFullCollection -XX:CMSInitiatingOccupancyFraction=70 -XX:+CMSParallelRemarkEnabled -XX:SoftRefLRUPolicyMSPerMB=0 -XX:+CMSClassUnloadingEnabled -XX:SurvivorRatio=8 -XX:+DisableExplicitGC"
    JAVA_OPT="${JAVA_OPT} -verbose:gc -Xloggc:${ROCKETMQ_HOME}/logs/rmq_srv_gc.log -XX:+PrintGCDetails"
    JAVA_OPT="${JAVA_OPT} -XX:-OmitStackTraceInFastThrow"
    JAVA_OPT="${JAVA_OPT} -Djava.ext.dirs=${BASE_DIR}/lib"
    #JAVA_OPT="${JAVA_OPT} -Xdebug -Xrunjdwp:transport=dt_socket,address=9555,server=y,suspend=n"
    #JAVA_OPT="${JAVA_OPT} -cp ${CLASSPATH}"
    
    $JAVA ${JAVA_OPT} $@

�޸����µط���
- JVM�����ã�Xms128M -Xmx128M -Xmn32M -XX:PermSize=32m -XX:MaxPermSize=32m
- GC��־λ�ã�-Xloggc:${ROCKETMQ_HOME}/logs/rmq_srv_gc.log
- ɾ��CLASSPATH����Ϊ`export CLASSPATH=.:${BASE_DIR}/conf`����`CLASSPATH`�п��ܻ�����ո񣬵��º��������ִ��ʧ��

***runbroker.sh
    
    #===========================================================================================
    # JVM ��������
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

�޸ĵط�ͬ`runserver.sh`

### ����

��������`mqnamesrv`��`mqbroker`
    
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

���Կ���NameServer��Broker�����ɹ�
