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

#### ����Դ��

    cd F:\rocketmq
    git clone https://github.com/alibaba/RocketMQ.git

#### ����Դ��

    set MAVEN_OPTS=-Xmx1024m`
    mvn -Dmaven.test.skip=true clean package install assembly:assembly -U

�����ַ���λ��:

    F:\rocketmq\RocketMQ\target\alibaba-rocketmq-3.2.2\alibaba-rocketmq

#### �޸�Ĭ�����в���

***runserver.sh***
    
    #===========================================================================================
    # JVM ��������
    #===========================================================================================
    JAVA_OPT="${JAVA_OPT} -server -Xms128M -Xmx128M -Xmn32M -XX:PermSize=32M -XX:MaxPermSize=32M"
    JAVA_OPT="${JAVA_OPT} \XX:+UseConcMarkSweepGC -XX:+UseCMSCompactAtFullCollection -XX:CMSInitiatingOccupancyFraction=70 -XX:+CMSParallelRemarkEnabled -XX:SoftRefLRUPolicyMSPerMB=0 -XX:+CMSClassUnloadingEnabled -XX:SurvivorRatio=8 -XX:+DisableExplicitGC"
    JAVA_OPT="${JAVA_OPT} -verbose:gc -Xloggc:${ROCKETMQ_HOME}/logs/rmq_srv_gc.log -XX:+PrintGCDetails"
    JAVA_OPT="${JAVA_OPT} -XX:-OmitStackTraceInFastThrow"
    JAVA_OPT="${JAVA_OPT} -Djava.ext.dirs=${BASE_DIR}/lib"
    #JAVA_OPT="${JAVA_OPT} -Xdebug -Xrunjdwp:transport=dt_socket,address=9555,server=y,suspend=n"
    #JAVA_OPT="${JAVA_OPT} -cp ${CLASSPATH}"

    $JAVA ${JAVA_OPT} $@


