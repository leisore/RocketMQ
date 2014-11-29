### 简介
本文档记录了在Windows平台下构建、配置和启动RocketMQ的步骤。

------

### 环境
- 操作系统：Windows XP Professional 2002 SP3
- JDK：1.6.0_20
- Maven：3.0.4
- Git：1.7.4.msysgit.0
- MinGW：1.0.11
- Eclipse：3.6.2

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

***runserver.sh***
    
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

***runbroker.sh***
    
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

***tools.sh***
    
    #===========================================================================================
    # JVM 参数配置
    #===========================================================================================
    JAVA_OPT="${JAVA_OPT} -server -Xms64M -Xmx64M"
    JAVA_OPT="${JAVA_OPT} -Djava.ext.dirs=${BASE_DIR}/lib"
    #JAVA_OPT="${JAVA_OPT} -cp ${CLASSPATH}"
    
    $JAVA ${JAVA_OPT} $@

修改地方同`runserver.sh`

------

### 运行

##### 运行mqnamesrv
    F:\rocketmq\RocketMQ\target\alibaba-rocketmq-3.2.2\alibaba-rocketmq\bin>sh
    sh-3.1$ export ROCKETMQ_HOME=../
    sh-3.1$ ./mqnamesrv &
    [1] 13452
    sh-3.1$
    sh-3.1$ The Name Server boot success.
    
    sh-3.1$ ps | grep java
        13740   12748   13452      13740  con  500 14:19:05 /d/bessystem/env/Java/jdk1.6.0_20/bin/java

查看NamingServer默认配置：

    sh-3.1$ ./mqnamesrv -p
    rocketmqHome=f:/rocketmq/RocketMQ/target/alibaba-rocketmq-3.2.2/alibaba-rocketmq
    kvConfigPath=C:\Documents and Settings\lcp\namesrv\kvConfig.json
    listenPort=9876
    serverWorkerThreads=8
    serverCallbackExecutorThreads=0
    serverSelectorThreads=3
    serverOnewaySemaphoreValue=256
    serverAsyncSemaphoreValue=64
    serverChannelMaxIdleTimeSeconds=120
    serverSocketSndBufSize=2048
    serverSocketRcvBufSize=1024
    serverPooledByteBufAllocatorEnable=false

##### 运行mqbroker
    sh-3.1$ export ROCKETMQ_HOME=../
    sh-3.1$ ./mqbroker &
    [1] 13976
    sh-3.1$ The broker[licp, 192.168.1.31:10911] boot success.

    sh-3.1$ ps | grep java
        13740   12748   13452      13740  con  500 14:19:05 /d/bessystem/env/Java/jdk1.6.0_20/bin/java
        14016   12456   13976      14016  con  500 14:21:59 /d/bessystem/env/Java/jdk1.6.0_20/bin/java

查看Broker默认配置：

    sh-3.1$ ./mqbroker -p
    rocketmqHome=f:/rocketmq/RocketMQ/target/alibaba-rocketmq-3.2.2/alibaba-rocketmq
    namesrvAddr=
    brokerIP1=192.168.1.31
    brokerIP2=192.168.1.31
    brokerName=licp
    brokerClusterName=DefaultCluster
    brokerId=0
    brokerPermission=6
    defaultTopicQueueNums=8
    autoCreateTopicEnable=true
    clusterTopicEnable=true
    brokerTopicEnable=true
    autoCreateSubscriptionGroup=true
    sendMessageThreadPoolNums=28
    pullMessageThreadPoolNums=22
    adminBrokerThreadPoolNums=16
    clientManageThreadPoolNums=16
    flushConsumerOffsetInterval=5000
    flushConsumerOffsetHistoryInterval=60000
    rejectTransactionMessage=false
    fetchNamesrvAddrByAddressServer=false
    sendThreadPoolQueueCapacity=100000
    pullThreadPoolQueueCapacity=100000
    filterServerNums=0
    longPollingEnable=true
    shortPollingTimeMills=1000
    notifyConsumerIdsChangedEnable=true
    listenPort=10911
    serverWorkerThreads=8
    serverCallbackExecutorThreads=0
    serverSelectorThreads=3
    serverOnewaySemaphoreValue=256
    serverAsyncSemaphoreValue=64
    serverChannelMaxIdleTimeSeconds=120
    serverSocketSndBufSize=131072
    serverSocketRcvBufSize=131072
    serverPooledByteBufAllocatorEnable=false
    clientWorkerThreads=4
    clientCallbackExecutorThreads=3
    clientOnewaySemaphoreValue=2048
    clientAsyncSemaphoreValue=2048
    connectTimeoutMillis=3000
    channelNotActiveInterval=60000
    clientChannelMaxIdleTimeSeconds=120
    clientSocketSndBufSize=131072
    clientSocketRcvBufSize=131072
    clientPooledByteBufAllocatorEnable=false
    storePathRootDir=C:\Documents and Settings\lcp\store
    storePathCommitLog=C:\Documents and Settings\lcp\store\commitlog
    mapedFileSizeCommitLog=1073741824
    mapedFileSizeConsumeQueue=6000000
    flushIntervalCommitLog=1000
    flushCommitLogTimed=false
    flushIntervalConsumeQueue=1000
    cleanResourceInterval=10000
    deleteCommitLogFilesInterval=100
    deleteConsumeQueueFilesInterval=100
    destroyMapedFileIntervalForcibly=120000
    redeleteHangedFileInterval=120000
    deleteWhen=04
    diskMaxUsedSpaceRatio=75
    fileReservedTime=72
    putMsgIndexHightWater=600000
    maxMessageSize=524288
    checkCRCOnRecover=true
    flushCommitLogLeastPages=4
    flushConsumeQueueLeastPages=2
    flushCommitLogThoroughInterval=10000
    flushConsumeQueueThoroughInterval=60000
    maxTransferBytesOnMessageInMemory=262144
    maxTransferCountOnMessageInMemory=32
    maxTransferBytesOnMessageInDisk=65536
    maxTransferCountOnMessageInDisk=8
    accessMessageInMemoryMaxRatio=40
    messageIndexEnable=true
    maxHashSlotNum=5000000
    maxIndexNum=20000000
    maxMsgsNumBatch=64
    messageIndexSafe=false
    haListenPort=10912
    haSendHeartbeatInterval=5000
    haHousekeepingInterval=20000
    haTransferBatchSize=32768
    haMasterAddress=
    haSlaveFallbehindMax=268435456
    brokerRole=ASYNC_MASTER
    flushDiskType=ASYNC_FLUSH
    syncFlushTimeout=5000
    messageDelayLevel=1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h
    flushDelayOffsetInterval=10000
    cleanFileForciblyEnable=true

------

### 配置

RocketMQ默认与路径有关的配置属性都与用户目录有关，在Windows下查看起来很不方便。可以通过配置文件改变这些属性。

##### 配置Logger

编辑`$ROCKETMQ_HOME/conf`下的如下文件：

    logback_broker.xml
    logback_filtersrv.xml
    logback_namesrv.xml
    logback_tools.xml

替换其中的`user.home`为`ROCKETMQ_HOME`

##### 配置NamingServer

新建`$ROCKETMQ_HOME/conf/namesrv.properties`文件，内容如下：

    listenPort=9876
    kvConfigPath=../conf/kvConfig.json

##### 配置Broker

新建`$ROCKETMQ_HOME/conf/broker.properties`文件，内容如下：

    namesrvAddr=192.168.1.31:9876
    brokerIP1=192.168.1.31
    brokerIP2=192.168.1.31
    brokerName=licp
    brokerId=0
    brokerPermission=6
    defaultTopicQueueNums=8
    autoCreateTopicEnable=true
    listenPort=10911
    storePathRootDir=../store
    storePathCommitLog=../store/commitlog
    mapedFileSizeCommitLog=3342336

重新启动NamingServer和Broker:

    sh-3.1$ ./mqnamesrv -c ../conf/namesrv.properties &
    [1] 12452
    sh-3.1$ load config properties file OK, ../conf/namesrv.properties
    The Name Server boot success.
    
    sh-3.1$ ./mqbroker -c ../conf/broker.properties &
    [2] 14904
    sh-3.1$ load config properties file OK, ../conf/broker.properties
    The broker[licp, 192.168.1.31:10911] boot success. and name server is 192.168.1.31:9876
    
    sh-3.1$ ps | grep java
         4768   13876   12452       4768  con  500 15:04:13 /d/bessystem/env/Java/jdk1.6.0_20/bin/java
        15184   15196   14904      15184  con  500 15:04:22 /d/bessystem/env/Java/jdk1.6.0_20/bin/java

查看当前的目录结构，可以看到日志和存储已经在指定位置：

    alibaba-rocketmq
    ├─benchmark
    ├─bin
    ├─conf
    │     broker.properties
    │     logback_broker.xml
    │     logback_filtersrv.xml
    │     logback_namesrv.xml
    │     logback_tools.xml
    │     namesrv.properties
    ├─lib
    ├─logs
    │  │  rmq_bk_gc.log
    │  └─rocketmqlogs
    │          broker.log
    │          broker_default.log
    │          lock.log
    │          namesrv.log
    │          namesrv_default.log
    │          remoting.log
    │          stats.log
    │          store.log
    │          storeerror.log
    │          transaction.log
    ├─store
    │  │  abort
    │  │  checkpoint
    │  │  
    │  ├─commitlog
    │  ├─config
    │  │      consumerOffset.json
    │  │      consumerOffset.json.bak
    │  │      delayOffset.json
    │  │      delayOffset.json.bak
    │  │      
    │  └─consumequeue
    └─test

------

### 客户端

运行RocketMQ自带rocketmq-example/com/alibaba/rocketmq/example/quickstart下的消息收发例子。

##### 运行Producer

做如下修改：

    DefaultMQProducer producer = new DefaultMQProducer("testProducer");
    producer.setNamesrvAddr("192.168.1.31:9876");
    producer.start();
    
    for (int i = 0; i < 10; i++) {
    ......

运行日志：

    java.lang.reflect.InvocationTargetException
    15:18:48.031 [main] DEBUG i.n.u.i.l.InternalLoggerFactory - Using SLF4J as the default logging framework
    15:18:48.046 [main] DEBUG i.n.c.MultithreadEventLoopGroup - -Dio.netty.eventLoopThreads: 6
    15:18:48.046 [main] DEBUG i.n.util.internal.PlatformDependent0 - java.nio.Buffer.address: available
    15:18:48.046 [main] DEBUG i.n.util.internal.PlatformDependent0 - sun.misc.Unsafe.theUnsafe: available
    15:18:48.046 [main] DEBUG i.n.util.internal.PlatformDependent0 - sun.misc.Unsafe.copyMemory: unavailable
    15:18:48.062 [main] DEBUG i.n.util.internal.PlatformDependent - Platform: Windows
    15:18:48.062 [main] DEBUG i.n.util.internal.PlatformDependent - Java version: 6
    15:18:48.062 [main] DEBUG i.n.util.internal.PlatformDependent - -Dio.netty.noUnsafe: false
    15:18:48.062 [main] DEBUG i.n.util.internal.PlatformDependent - sun.misc.Unsafe: unavailable
    15:18:48.062 [main] DEBUG i.n.util.internal.PlatformDependent - -Dio.netty.noJavassist: false
    15:18:48.187 [main] DEBUG i.n.util.internal.PlatformDependent - Javassist: available
    15:18:48.187 [main] DEBUG i.n.util.internal.PlatformDependent - -Dio.netty.tmpdir: C:\DOCUME~1\lcp\LOCALS~1\Temp (java.io.tmpdir)
    15:18:48.187 [main] DEBUG i.n.util.internal.PlatformDependent - -Dio.netty.bitMode: 32 (sun.arch.data.model)
    15:18:48.187 [main] DEBUG i.n.util.internal.PlatformDependent - -Dio.netty.noPreferDirect: true
    15:18:48.187 [main] INFO  i.n.util.internal.PlatformDependent - Your platform does not provide complete low-level API for accessing direct buffers reliably. Unless explicitly requested, heap buffer will always be preferred to avoid potential system unstability.
    15:18:48.203 [main] DEBUG io.netty.channel.nio.NioEventLoop - -Dio.netty.noKeySetOptimization: false
    15:18:48.203 [main] DEBUG io.netty.channel.nio.NioEventLoop - -Dio.netty.selectorAutoRebuildThreshold: 512
    15:18:48.281 [main] INFO  RocketmqClient - user specfied name server address: 192.168.1.31:9876
    15:18:48.296 [main] INFO  RocketmqClient - created a new client Instance, FactoryIndex: 0 ClinetID: 192.168.1.31@14888 ClientConfig [namesrvAddr=192.168.1.31:9876, clientIP=192.168.1.31, instanceName=14888, clientCallbackExecutorThreads=3, pollNameServerInteval=30000, heartbeatBrokerInterval=30000, persistConsumerOffsetInterval=5000] V3_2_2_SNAPSHOT
    15:18:48.343 [main] DEBUG i.n.util.internal.ThreadLocalRandom - -Dio.netty.initialSeedUniquifier: 0xffd8b012c90017c6 (took 21 ms)
    15:18:48.375 [main] DEBUG io.netty.buffer.ByteBufUtil - -Dio.netty.allocator.type: unpooled
    15:18:48.375 [main] DEBUG io.netty.buffer.ByteBufUtil - -Dio.netty.threadLocalDirectBufferSize: 65536
    15:18:48.406 [main] INFO  RocketmqRemoting - createChannel: begin to connect remote host[192.168.1.31:9876] asynchronously
    15:18:48.406 [NettyClientSelector_1] DEBUG i.n.u.i.JavassistTypeParameterMatcherGenerator - Generated: io.netty.util.internal.__matchers__.com.alibaba.rocketmq.remoting.protocol.RemotingCommandMatcher
    15:18:48.421 [NettyClientWorkerThread_1] INFO  RocketmqRemoting - NETTY CLIENT PIPELINE: CONNECT  UNKNOW => /192.168.1.31:9876
    15:18:48.437 [main] INFO  RocketmqRemoting - createChannel: connect remote host[192.168.1.31:9876] success, DefaultChannelPromise@1f78ef1(success)
    15:18:48.437 [main] DEBUG io.netty.util.Recycler - -Dio.netty.recycler.maxCapacity.default: 262144
    15:18:48.453 [NettyClientWorkerThread_1] DEBUG io.netty.util.ResourceLeakDetector - -Dio.netty.leakDetectionLevel: simple
    15:18:48.671 [PullMessageService] INFO  RocketmqClient - PullMessageService service started
    15:18:48.671 [RebalanceService] INFO  RocketmqClient - RebalanceService service started
    15:18:48.671 [main] INFO  RocketmqClient - the producer [CLIENT_INNER_PRODUCER] start OK
    15:18:48.671 [main] INFO  RocketmqClient - the client factory [192.168.1.31@14888] start OK
    15:18:48.671 [main] INFO  RocketmqClient - the producer [testProducer] start OK
    15:18:48.687 [main] WARN  RocketmqClient - get Topic [TopicTest] RouteInfoFromNameServer is not exist value
    15:18:48.687 [main] WARN  RocketmqClient - updateTopicRouteInfoFromNameServer Exception
    com.alibaba.rocketmq.client.exception.MQClientException: CODE: 17  DESC: No topic route info in name server for the topic: TopicTest
    See https://github.com/alibaba/RocketMQ/issues/55 for further details.
        at com.alibaba.rocketmq.client.impl.MQClientAPIImpl.getTopicRouteInfoFromNameServer(MQClientAPIImpl.java:1516) ~[bin/:na]
        at com.alibaba.rocketmq.client.impl.factory.MQClientInstance.updateTopicRouteInfoFromNameServer(MQClientInstance.java:593) [bin/:na]
        at com.alibaba.rocketmq.client.impl.factory.MQClientInstance.updateTopicRouteInfoFromNameServer(MQClientInstance.java:563) [bin/:na]
        at com.alibaba.rocketmq.client.impl.producer.DefaultMQProducerImpl.tryToFindTopicPublishInfo(DefaultMQProducerImpl.java:612) [bin/:na]
        at com.alibaba.rocketmq.client.impl.producer.DefaultMQProducerImpl.sendDefaultImpl(DefaultMQProducerImpl.java:498) [bin/:na]
        at com.alibaba.rocketmq.client.impl.producer.DefaultMQProducerImpl.send(DefaultMQProducerImpl.java:1026) [bin/:na]
        at com.alibaba.rocketmq.client.producer.DefaultMQProducer.send(DefaultMQProducer.java:122) [bin/:na]
        at com.alibaba.rocketmq.example.quickstart.Producer.main(Producer.java:41) [bin/:na]
    15:18:48.718 [MQClientFactoryScheduledThread] WARN  RocketmqClient - get Topic [TopicTest] RouteInfoFromNameServer is not exist value
    15:18:48.718 [MQClientFactoryScheduledThread] WARN  RocketmqClient - updateTopicRouteInfoFromNameServer Exception
    com.alibaba.rocketmq.client.exception.MQClientException: CODE: 17  DESC: No topic route info in name server for the topic: TopicTest
    See https://github.com/alibaba/RocketMQ/issues/55 for further details.
        at com.alibaba.rocketmq.client.impl.MQClientAPIImpl.getTopicRouteInfoFromNameServer(MQClientAPIImpl.java:1516) ~[bin/:na]
        at com.alibaba.rocketmq.client.impl.factory.MQClientInstance.updateTopicRouteInfoFromNameServer(MQClientInstance.java:593) [bin/:na]
        at com.alibaba.rocketmq.client.impl.factory.MQClientInstance.updateTopicRouteInfoFromNameServer(MQClientInstance.java:563) [bin/:na]
        at com.alibaba.rocketmq.client.impl.factory.MQClientInstance.updateTopicRouteInfoFromNameServer(MQClientInstance.java:557) [bin/:na]
        at com.alibaba.rocketmq.client.impl.factory.MQClientInstance$3.run(MQClientInstance.java:216) [bin/:na]
        at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:441) [na:1.6.0_20]
        at java.util.concurrent.FutureTask$Sync.innerRunAndReset(FutureTask.java:317) [na:1.6.0_20]
        at java.util.concurrent.FutureTask.runAndReset(FutureTask.java:150) [na:1.6.0_20]
        at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$101(ScheduledThreadPoolExecutor.java:98) [na:1.6.0_20]
        at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.runPeriodic(ScheduledThreadPoolExecutor.java:181) [na:1.6.0_20]
        at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:205) [na:1.6.0_20]
        at java.util.concurrent.ThreadPoolExecutor$Worker.runTask(ThreadPoolExecutor.java:886) [na:1.6.0_20]
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:908) [na:1.6.0_20]
        at java.lang.Thread.run(Thread.java:619) [na:1.6.0_20]
    15:18:48.765 [MQClientFactoryScheduledThread] INFO  RocketmqClient - the topic[TBW102] route info changed, odl[null] ,new[TopicRouteData [orderTopicConf=null, queueDatas=[QueueData [brokerName=licp, readQueueNums=8, writeQueueNums=8, perm=7, topicSynFlag=0]], brokerDatas=[BrokerData [brokerName=licp, brokerAddrs={0=192.168.1.31:10911}]], filterServerTable={}]]
    15:18:48.765 [MQClientFactoryScheduledThread] INFO  RocketmqClient - updateTopicPublishInfo prev is not null, TopicPublishInfo [orderTopic=false, messageQueueList=[], sendWhichQueue=0, haveTopicRouterInfo=false]
    15:18:48.765 [MQClientFactoryScheduledThread] INFO  RocketmqClient - updateTopicPublishInfo prev is not null, TopicPublishInfo [orderTopic=false, messageQueueList=[], sendWhichQueue=0, haveTopicRouterInfo=false]
    15:18:48.765 [MQClientFactoryScheduledThread] INFO  RocketmqClient - topicRouteTable.put TopicRouteData[TopicRouteData [orderTopicConf=null, queueDatas=[QueueData [brokerName=licp, readQueueNums=8, writeQueueNums=8, perm=7, topicSynFlag=0]], brokerDatas=[BrokerData [brokerName=licp, brokerAddrs={0=192.168.1.31:10911}]], filterServerTable={}]]
    15:18:48.781 [main] INFO  RocketmqClient - the topic[TopicTest] route info changed, odl[null] ,new[TopicRouteData [orderTopicConf=null, queueDatas=[QueueData [brokerName=licp, readQueueNums=4, writeQueueNums=4, perm=7, topicSynFlag=0]], brokerDatas=[BrokerData [brokerName=licp, brokerAddrs={0=192.168.1.31:10911}]], filterServerTable={}]]
    15:18:48.781 [main] INFO  RocketmqClient - updateTopicPublishInfo prev is not null, TopicPublishInfo [orderTopic=false, messageQueueList=[], sendWhichQueue=0, haveTopicRouterInfo=false]
    15:18:48.781 [main] INFO  RocketmqClient - topicRouteTable.put TopicRouteData[TopicRouteData [orderTopicConf=null, queueDatas=[QueueData [brokerName=licp, readQueueNums=4, writeQueueNums=4, perm=7, topicSynFlag=0]], brokerDatas=[BrokerData [brokerName=licp, brokerAddrs={0=192.168.1.31:10911}]], filterServerTable={}]]
    15:18:48.781 [main] INFO  RocketmqRemoting - createChannel: begin to connect remote host[192.168.1.31:10911] asynchronously
    15:18:48.781 [NettyClientWorkerThread_2] INFO  RocketmqRemoting - NETTY CLIENT PIPELINE: CONNECT  UNKNOW => /192.168.1.31:10911
    15:18:48.781 [main] INFO  RocketmqRemoting - createChannel: connect remote host[192.168.1.31:10911] success, DefaultChannelPromise@1b8d6f7(success)
    SendResult [sendStatus=SEND_OK, msgId=C0A8011F00002A9F0000000000000000, messageQueue=MessageQueue [topic=TopicTest, brokerName=licp, queueId=0], queueOffset=0]
    SendResult [sendStatus=SEND_OK, msgId=C0A8011F00002A9F0000000000000088, messageQueue=MessageQueue [topic=TopicTest, brokerName=licp, queueId=1], queueOffset=0]
    SendResult [sendStatus=SEND_OK, msgId=C0A8011F00002A9F0000000000000110, messageQueue=MessageQueue [topic=TopicTest, brokerName=licp, queueId=2], queueOffset=0]
    SendResult [sendStatus=SEND_OK, msgId=C0A8011F00002A9F0000000000000198, messageQueue=MessageQueue [topic=TopicTest, brokerName=licp, queueId=3], queueOffset=0]
    SendResult [sendStatus=SEND_OK, msgId=C0A8011F00002A9F0000000000000220, messageQueue=MessageQueue [topic=TopicTest, brokerName=licp, queueId=0], queueOffset=1]
    SendResult [sendStatus=SEND_OK, msgId=C0A8011F00002A9F00000000000002A8, messageQueue=MessageQueue [topic=TopicTest, brokerName=licp, queueId=1], queueOffset=1]
    SendResult [sendStatus=SEND_OK, msgId=C0A8011F00002A9F0000000000000330, messageQueue=MessageQueue [topic=TopicTest, brokerName=licp, queueId=2], queueOffset=1]
    SendResult [sendStatus=SEND_OK, msgId=C0A8011F00002A9F00000000000003B8, messageQueue=MessageQueue [topic=TopicTest, brokerName=licp, queueId=3], queueOffset=1]
    SendResult [sendStatus=SEND_OK, msgId=C0A8011F00002A9F0000000000000440, messageQueue=MessageQueue [topic=TopicTest, brokerName=licp, queueId=0], queueOffset=2]
    SendResult [sendStatus=SEND_OK, msgId=C0A8011F00002A9F00000000000004C8, messageQueue=MessageQueue [topic=TopicTest, brokerName=licp, queueId=1], queueOffset=2]
    15:18:48.953 [main] INFO  RocketmqClient - unregister client[Producer: testProducer Consumer: null] from broker[licp 0 192.168.1.31:10911] success
    15:18:48.953 [main] INFO  RocketmqClient - unregister client[Producer: CLIENT_INNER_PRODUCER Consumer: null] from broker[licp 0 192.168.1.31:10911] success
    15:18:48.968 [main] INFO  RocketmqClient - the producer [CLIENT_INNER_PRODUCER] shutdown OK
    15:18:48.968 [main] INFO  RocketmqCommon - shutdown thread PullMessageService interrupt true
    15:18:48.968 [PullMessageService] INFO  RocketmqClient - PullMessageService service end
    15:18:48.968 [main] INFO  RocketmqCommon - join thread PullMessageService eclipse time(ms) 0 90000
    15:18:48.968 [main] INFO  RocketmqRemoting - closeChannel: begin close the channel[192.168.1.31:10911] Found: true
    15:18:48.968 [main] INFO  RocketmqRemoting - closeChannel: the channel[192.168.1.31:10911] was removed from channel table
    15:18:48.968 [main] INFO  RocketmqRemoting - closeChannel: begin close the channel[192.168.1.31:9876] Found: true
    15:18:48.968 [main] INFO  RocketmqRemoting - closeChannel: the channel[192.168.1.31:9876] was removed from channel table
    15:18:48.968 [main] INFO  RocketmqRemoting - shutdown thread NettyEventExecuter interrupt false
    15:18:48.968 [main] INFO  RocketmqRemoting - join thread NettyEventExecuter eclipse time(ms) 0 90000
    15:18:48.968 [main] INFO  RocketmqCommon - shutdown thread RebalanceService interrupt false
    15:18:48.968 [RebalanceService] INFO  RocketmqClient - RebalanceService service end
    15:18:48.968 [main] INFO  RocketmqCommon - join thread RebalanceService eclipse time(ms) 0 90000
    15:18:48.968 [main] INFO  RocketmqClient - the client factory [192.168.1.31@14888] shutdown OK
    15:18:48.968 [main] INFO  RocketmqClient - the producer [testProducer] shutdown OK
    15:18:48.968 [NettyClientWorkerThread_2] INFO  RocketmqRemoting - NETTY CLIENT PIPELINE: CLOSE 192.168.1.31:10911
    15:18:48.968 [NettyClientWorkerThread_2] INFO  RocketmqRemoting - eventCloseChannel: the channel[null] has been removed from the channel table before
    15:18:48.968 [NettyClientWorkerThread_1] INFO  RocketmqRemoting - NETTY CLIENT PIPELINE: CLOSE 192.168.1.31:9876
    15:18:48.968 [NettyClientWorkerThread_1] INFO  RocketmqRemoting - eventCloseChannel: the channel[null] has been removed from the channel table before
    15:18:48.984 [NettyClientSelector_1] INFO  RocketmqRemoting - closeChannel: close the connection to remote address[192.168.1.31:10911] result: true
    15:18:48.984 [NettyClientSelector_1] INFO  RocketmqRemoting - closeChannel: close the connection to remote address[192.168.1.31:9876] result: true

##### 运行Consumer

做如下修改：

    DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("testConsumer");
    consumer.setNamesrvAddr("192.168.1.31:9876");

运行日志：

    java.lang.reflect.InvocationTargetException
    15:20:19.375 [main] INFO  RocketmqClient - the consumer [testConsumer] start beginning. messageModel=CLUSTERING, isUnitMode=false
    15:20:19.500 [main] DEBUG i.n.u.i.l.InternalLoggerFactory - Using SLF4J as the default logging framework
    15:20:19.500 [main] DEBUG i.n.c.MultithreadEventLoopGroup - -Dio.netty.eventLoopThreads: 6
    15:20:19.515 [main] DEBUG i.n.util.internal.PlatformDependent0 - java.nio.Buffer.address: available
    15:20:19.515 [main] DEBUG i.n.util.internal.PlatformDependent0 - sun.misc.Unsafe.theUnsafe: available
    15:20:19.515 [main] DEBUG i.n.util.internal.PlatformDependent0 - sun.misc.Unsafe.copyMemory: unavailable
    15:20:19.515 [main] DEBUG i.n.util.internal.PlatformDependent - Platform: Windows
    15:20:19.515 [main] DEBUG i.n.util.internal.PlatformDependent - Java version: 6
    15:20:19.515 [main] DEBUG i.n.util.internal.PlatformDependent - -Dio.netty.noUnsafe: false
    15:20:19.515 [main] DEBUG i.n.util.internal.PlatformDependent - sun.misc.Unsafe: unavailable
    15:20:19.515 [main] DEBUG i.n.util.internal.PlatformDependent - -Dio.netty.noJavassist: false
    15:20:19.640 [main] DEBUG i.n.util.internal.PlatformDependent - Javassist: available
    15:20:19.640 [main] DEBUG i.n.util.internal.PlatformDependent - -Dio.netty.tmpdir: C:\DOCUME~1\lcp\LOCALS~1\Temp (java.io.tmpdir)
    15:20:19.640 [main] DEBUG i.n.util.internal.PlatformDependent - -Dio.netty.bitMode: 32 (sun.arch.data.model)
    15:20:19.640 [main] DEBUG i.n.util.internal.PlatformDependent - -Dio.netty.noPreferDirect: true
    15:20:19.640 [main] INFO  i.n.util.internal.PlatformDependent - Your platform does not provide complete low-level API for accessing direct buffers reliably. Unless explicitly requested, heap buffer will always be preferred to avoid potential system unstability.
    15:20:19.656 [main] DEBUG io.netty.channel.nio.NioEventLoop - -Dio.netty.noKeySetOptimization: false
    15:20:19.656 [main] DEBUG io.netty.channel.nio.NioEventLoop - -Dio.netty.selectorAutoRebuildThreshold: 512
    15:20:19.734 [main] INFO  RocketmqClient - user specfied name server address: 192.168.1.31:9876
    15:20:19.750 [main] INFO  RocketmqClient - created a new client Instance, FactoryIndex: 0 ClinetID: 192.168.1.31@16212 ClientConfig [namesrvAddr=192.168.1.31:9876, clientIP=192.168.1.31, instanceName=16212, clientCallbackExecutorThreads=3, pollNameServerInteval=30000, heartbeatBrokerInterval=30000, persistConsumerOffsetInterval=5000] V3_2_2_SNAPSHOT
    15:20:19.859 [main] DEBUG i.n.util.internal.ThreadLocalRandom - -Dio.netty.initialSeedUniquifier: 0x997db08d662fdae6 (took 20 ms)
    15:20:19.906 [main] DEBUG io.netty.buffer.ByteBufUtil - -Dio.netty.allocator.type: unpooled
    15:20:19.906 [main] DEBUG io.netty.buffer.ByteBufUtil - -Dio.netty.threadLocalDirectBufferSize: 65536
    15:20:19.921 [main] INFO  RocketmqRemoting - createChannel: begin to connect remote host[192.168.1.31:9876] asynchronously
    15:20:19.937 [NettyClientSelector_1] DEBUG i.n.u.i.JavassistTypeParameterMatcherGenerator - Generated: io.netty.util.internal.__matchers__.com.alibaba.rocketmq.remoting.protocol.RemotingCommandMatcher
    15:20:19.937 [NettyClientWorkerThread_1] INFO  RocketmqRemoting - NETTY CLIENT PIPELINE: CONNECT  UNKNOW => /192.168.1.31:9876
    15:20:19.953 [main] INFO  RocketmqRemoting - createChannel: connect remote host[192.168.1.31:9876] success, DefaultChannelPromise@1081d2e(success)
    15:20:19.953 [main] DEBUG io.netty.util.Recycler - -Dio.netty.recycler.maxCapacity.default: 262144
    15:20:19.968 [NettyClientWorkerThread_1] DEBUG io.netty.util.ResourceLeakDetector - -Dio.netty.leakDetectionLevel: simple
    15:20:20.171 [PullMessageService] INFO  RocketmqClient - PullMessageService service started
    15:20:20.171 [RebalanceService] INFO  RocketmqClient - RebalanceService service started
    15:20:20.171 [main] INFO  RocketmqClient - the producer [CLIENT_INNER_PRODUCER] start OK
    15:20:20.187 [main] INFO  RocketmqClient - the client factory [192.168.1.31@16212] start OK
    15:20:20.187 [main] INFO  RocketmqClient - the consumer [testConsumer] start OK.
    15:20:20.187 [MQClientFactoryScheduledThread] WARN  RocketmqClient - get Topic [%RETRY%testConsumer] RouteInfoFromNameServer is not exist value
    15:20:20.218 [MQClientFactoryScheduledThread] INFO  RocketmqClient - the topic[TopicTest] route info changed, odl[null] ,new[TopicRouteData [orderTopicConf=null, queueDatas=[QueueData [brokerName=licp, readQueueNums=4, writeQueueNums=4, perm=6, topicSynFlag=0]], brokerDatas=[BrokerData [brokerName=licp, brokerAddrs={0=192.168.1.31:10911}]], filterServerTable={}]]
    15:20:20.218 [MQClientFactoryScheduledThread] INFO  RocketmqClient - topicRouteTable.put TopicRouteData[TopicRouteData [orderTopicConf=null, queueDatas=[QueueData [brokerName=licp, readQueueNums=4, writeQueueNums=4, perm=6, topicSynFlag=0]], brokerDatas=[BrokerData [brokerName=licp, brokerAddrs={0=192.168.1.31:10911}]], filterServerTable={}]]
    15:20:20.218 [MQClientFactoryScheduledThread] INFO  RocketmqClient - the topic[TBW102] route info changed, odl[null] ,new[TopicRouteData [orderTopicConf=null, queueDatas=[QueueData [brokerName=licp, readQueueNums=8, writeQueueNums=8, perm=7, topicSynFlag=0]], brokerDatas=[BrokerData [brokerName=licp, brokerAddrs={0=192.168.1.31:10911}]], filterServerTable={}]]
    15:20:20.218 [MQClientFactoryScheduledThread] INFO  RocketmqClient - updateTopicPublishInfo prev is not null, TopicPublishInfo [orderTopic=false, messageQueueList=[], sendWhichQueue=0, haveTopicRouterInfo=false]
    15:20:20.218 [MQClientFactoryScheduledThread] INFO  RocketmqClient - topicRouteTable.put TopicRouteData[TopicRouteData [orderTopicConf=null, queueDatas=[QueueData [brokerName=licp, readQueueNums=8, writeQueueNums=8, perm=7, topicSynFlag=0]], brokerDatas=[BrokerData [brokerName=licp, brokerAddrs={0=192.168.1.31:10911}]], filterServerTable={}]]
    15:20:20.218 [main] WARN  RocketmqClient - get Topic [%RETRY%testConsumer] RouteInfoFromNameServer is not exist value
    15:20:20.250 [main] INFO  RocketmqRemoting - createChannel: begin to connect remote host[192.168.1.31:10911] asynchronously
    15:20:20.250 [NettyClientWorkerThread_2] INFO  RocketmqRemoting - NETTY CLIENT PIPELINE: CONNECT  UNKNOW => /192.168.1.31:10911
    15:20:20.250 [main] INFO  RocketmqRemoting - createChannel: connect remote host[192.168.1.31:10911] success, DefaultChannelPromise@1fc6e42(success)
    15:20:20.390 [main] INFO  RocketmqClient - send heart beat to broker[licp 0 192.168.1.31:10911] success
    15:20:20.390 [main] INFO  RocketmqClient - HeartbeatData [clientID=192.168.1.31@16212, producerDataSet=[ProducerData [groupName=CLIENT_INNER_PRODUCER]], consumerDataSet=[ConsumerData [groupName=testConsumer, consumeType=CONSUME_PASSIVELY, messageModel=CLUSTERING, consumeFromWhere=CONSUME_FROM_FIRST_OFFSET, unitMode=false, subscriptionDataSet=[SubscriptionData [classFilterMode=false, topic=%RETRY%testConsumer, subString=*, tagsSet=[], codeSet=[], subVersion=1417245619421], SubscriptionData [classFilterMode=false, topic=TopicTest, subString=*, tagsSet=[], codeSet=[], subVersion=1417245619359]]]]]
    Consumer Started.
    15:20:20.390 [RebalanceService] INFO  RocketmqClient - the topic[%RETRY%testConsumer] route info changed, odl[null] ,new[TopicRouteData [orderTopicConf=null, queueDatas=[QueueData [brokerName=licp, readQueueNums=1, writeQueueNums=1, perm=6, topicSynFlag=0]], brokerDatas=[BrokerData [brokerName=licp, brokerAddrs={0=192.168.1.31:10911}]], filterServerTable={}]]
    15:20:20.390 [RebalanceService] INFO  RocketmqClient - topicRouteTable.put TopicRouteData[TopicRouteData [orderTopicConf=null, queueDatas=[QueueData [brokerName=licp, readQueueNums=1, writeQueueNums=1, perm=6, topicSynFlag=0]], brokerDatas=[BrokerData [brokerName=licp, brokerAddrs={0=192.168.1.31:10911}]], filterServerTable={}]]
    15:20:20.421 [NettyClientPublicExecutor_1] INFO  RocketmqClient - receive broker's notification[192.168.1.31:10911], the consumer group: testConsumer changed, rebalance immediately
    15:20:20.578 [RebalanceService] INFO  RocketmqClient - doRebalance, testConsumer, add a new mq, MessageQueue [topic=TopicTest, brokerName=licp, queueId=2]
    15:20:20.578 [RebalanceService] INFO  RocketmqClient - doRebalance, testConsumer, add a new mq, MessageQueue [topic=TopicTest, brokerName=licp, queueId=1]
    15:20:20.578 [RebalanceService] INFO  RocketmqClient - doRebalance, testConsumer, add a new mq, MessageQueue [topic=TopicTest, brokerName=licp, queueId=3]
    15:20:20.593 [RebalanceService] INFO  RocketmqClient - doRebalance, testConsumer, add a new mq, MessageQueue [topic=TopicTest, brokerName=licp, queueId=0]
    15:20:20.593 [RebalanceService] INFO  RocketmqClient - doRebalance, testConsumer, add a new pull request PullRequest [consumerGroup=testConsumer, messageQueue=MessageQueue [topic=TopicTest, brokerName=licp, queueId=2], nextOffset=0]
    15:20:20.593 [RebalanceService] INFO  RocketmqClient - doRebalance, testConsumer, add a new pull request PullRequest [consumerGroup=testConsumer, messageQueue=MessageQueue [topic=TopicTest, brokerName=licp, queueId=1], nextOffset=0]
    15:20:20.593 [RebalanceService] INFO  RocketmqClient - doRebalance, testConsumer, add a new pull request PullRequest [consumerGroup=testConsumer, messageQueue=MessageQueue [topic=TopicTest, brokerName=licp, queueId=3], nextOffset=0]
    15:20:20.593 [RebalanceService] INFO  RocketmqClient - doRebalance, testConsumer, add a new pull request PullRequest [consumerGroup=testConsumer, messageQueue=MessageQueue [topic=TopicTest, brokerName=licp, queueId=0], nextOffset=0]
    15:20:20.593 [RebalanceService] INFO  RocketmqClient - rebalanced allocate source. allocateMessageQueueStrategyName=AVG, group=testConsumer, topic=TopicTest, mqAllSize=4, cidAllSize=1, mqAll=[MessageQueue [topic=TopicTest, brokerName=licp, queueId=2], MessageQueue [topic=TopicTest, brokerName=licp, queueId=1], MessageQueue [topic=TopicTest, brokerName=licp, queueId=3], MessageQueue [topic=TopicTest, brokerName=licp, queueId=0]], cidAll=[192.168.1.31@16212]
    15:20:20.593 [RebalanceService] INFO  RocketmqClient - rebalanced result changed. allocateMessageQueueStrategyName=AVG, group=testConsumer, topic=TopicTest, ConsumerId=192.168.1.31@16212, rebalanceSize=4, rebalanceMqSet=4
    15:20:20.640 [RebalanceService] INFO  RocketmqClient - doRebalance, testConsumer, add a new mq, MessageQueue [topic=%RETRY%testConsumer, brokerName=licp, queueId=0]
    15:20:20.640 [RebalanceService] INFO  RocketmqClient - doRebalance, testConsumer, add a new pull request PullRequest [consumerGroup=testConsumer, messageQueue=MessageQueue [topic=%RETRY%testConsumer, brokerName=licp, queueId=0], nextOffset=0]
    15:20:20.640 [RebalanceService] INFO  RocketmqClient - rebalanced allocate source. allocateMessageQueueStrategyName=AVG, group=testConsumer, topic=%RETRY%testConsumer, mqAllSize=1, cidAllSize=1, mqAll=[MessageQueue [topic=%RETRY%testConsumer, brokerName=licp, queueId=0]], cidAll=[192.168.1.31@16212]
    15:20:20.640 [RebalanceService] INFO  RocketmqClient - rebalanced result changed. allocateMessageQueueStrategyName=AVG, group=testConsumer, topic=%RETRY%testConsumer, ConsumerId=192.168.1.31@16212, rebalanceSize=1, rebalanceMqSet=1
    ConsumeMessageThread_3 Receive New Messages: [MessageExt [queueId=1, storeSize=136, queueOffset=0, sysFlag=0, bornTimestamp=1417245528828, bornHost=/192.168.1.31:7135, storeTimestamp=1417245528828, storeHost=/192.168.1.31:10911, msgId=C0A8011F00002A9F0000000000000088, commitLogOffset=136, bodyCRC=1401636825, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message [topic=TopicTest, flag=0, properties={TAGS=TagA, WAIT=true, MAX_OFFSET=3, MIN_OFFSET=0}, body=16]]]
    ConsumeMessageThread_2 Receive New Messages: [MessageExt [queueId=3, storeSize=136, queueOffset=0, sysFlag=0, bornTimestamp=1417245528843, bornHost=/192.168.1.31:7135, storeTimestamp=1417245528843, storeHost=/192.168.1.31:10911, msgId=C0A8011F00002A9F0000000000000198, commitLogOffset=408, bodyCRC=1032136437, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message [topic=TopicTest, flag=0, properties={TAGS=TagA, WAIT=true, MAX_OFFSET=2, MIN_OFFSET=0}, body=16]]]
    ConsumeMessageThread_5 Receive New Messages: [MessageExt [queueId=3, storeSize=136, queueOffset=1, sysFlag=0, bornTimestamp=1417245528875, bornHost=/192.168.1.31:7135, storeTimestamp=1417245528937, storeHost=/192.168.1.31:10911, msgId=C0A8011F00002A9F00000000000003B8, commitLogOffset=952, bodyCRC=988340972, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message [topic=TopicTest, flag=0, properties={TAGS=TagA, WAIT=true, MAX_OFFSET=2, MIN_OFFSET=0}, body=16]]]
    ConsumeMessageThread_6 Receive New Messages: [MessageExt [queueId=2, storeSize=136, queueOffset=0, sysFlag=0, bornTimestamp=1417245528828, bornHost=/192.168.1.31:7135, storeTimestamp=1417245528828, storeHost=/192.168.1.31:10911, msgId=C0A8011F00002A9F0000000000000110, commitLogOffset=272, bodyCRC=1250039395, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message [topic=TopicTest, flag=0, properties={TAGS=TagA, WAIT=true, MAX_OFFSET=2, MIN_OFFSET=0}, body=16]]]
    ConsumeMessageThread_4 Receive New Messages: [MessageExt [queueId=0, storeSize=136, queueOffset=1, sysFlag=0, bornTimestamp=1417245528843, bornHost=/192.168.1.31:7135, storeTimestamp=1417245528859, storeHost=/192.168.1.31:10911, msgId=C0A8011F00002A9F0000000000000220, commitLogOffset=544, bodyCRC=601994070, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message [topic=TopicTest, flag=0, properties={TAGS=TagA, WAIT=true, MAX_OFFSET=3, MIN_OFFSET=0}, body=16]]]
    ConsumeMessageThread_8 Receive New Messages: [MessageExt [queueId=0, storeSize=136, queueOffset=2, sysFlag=0, bornTimestamp=1417245528937, bornHost=/192.168.1.31:7135, storeTimestamp=1417245528937, storeHost=/192.168.1.31:10911, msgId=C0A8011F00002A9F0000000000000440, commitLogOffset=1088, bodyCRC=710410109, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message [topic=TopicTest, flag=0, properties={TAGS=TagA, WAIT=true, MAX_OFFSET=3, MIN_OFFSET=0}, body=16]]]
    ConsumeMessageThread_1 Receive New Messages: [MessageExt [queueId=0, storeSize=136, queueOffset=0, sysFlag=0, bornTimestamp=1417245528781, bornHost=/192.168.1.31:7135, storeTimestamp=1417245528812, storeHost=/192.168.1.31:10911, msgId=C0A8011F00002A9F0000000000000000, commitLogOffset=0, bodyCRC=613185359, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message [topic=TopicTest, flag=0, properties={TAGS=TagA, WAIT=true, MAX_OFFSET=3, MIN_OFFSET=0}, body=16]]]
    ConsumeMessageThread_7 Receive New Messages: [MessageExt [queueId=1, storeSize=136, queueOffset=1, sysFlag=0, bornTimestamp=1417245528859, bornHost=/192.168.1.31:7135, storeTimestamp=1417245528859, storeHost=/192.168.1.31:10911, msgId=C0A8011F00002A9F00000000000002A8, commitLogOffset=680, bodyCRC=1424393152, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message [topic=TopicTest, flag=0, properties={TAGS=TagA, WAIT=true, MAX_OFFSET=3, MIN_OFFSET=0}, body=16]]]
    ConsumeMessageThread_9 Receive New Messages: [MessageExt [queueId=1, storeSize=136, queueOffset=2, sysFlag=0, bornTimestamp=1417245528937, bornHost=/192.168.1.31:7135, storeTimestamp=1417245528937, storeHost=/192.168.1.31:10911, msgId=C0A8011F00002A9F00000000000004C8, commitLogOffset=1224, bodyCRC=1565577195, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message [topic=TopicTest, flag=0, properties={TAGS=TagA, WAIT=true, MAX_OFFSET=3, MIN_OFFSET=0}, body=16]]]
    ConsumeMessageThread_10 Receive New Messages: [MessageExt [queueId=2, storeSize=136, queueOffset=1, sysFlag=0, bornTimestamp=1417245528875, bornHost=/192.168.1.31:7135, storeTimestamp=1417245528875, storeHost=/192.168.1.31:10911, msgId=C0A8011F00002A9F0000000000000330, commitLogOffset=816, bodyCRC=1307562618, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message [topic=TopicTest, flag=0, properties={TAGS=TagA, WAIT=true, MAX_OFFSET=2, MIN_OFFSET=0}, body=16]]]
    15:20:21.171 [MQClientFactoryScheduledThread] INFO  RocketmqClient - send heart beat to broker[licp 0 192.168.1.31:10911] success

可以看到Consumer]收到了10条消息

------

### 问题记录

##### java.lang.reflect.InvocationTargetException

Producer和Consumer运行时，总是打印这个异常，没有任何提示信息。跟踪源码问题出在`com.alibaba.rocketmq.client.log.ClientLogger`这个类
初始化客户端Logger的方法createLogger上：

   final String logback_resource_file =
                System
                    .getProperty("rocketmq.client.logback.resource.fileName", "logback_rocketmq_client.xml");
    ......
    // 如果应用没有配置，则使用jar包内置配置
    try {
        joranConfigurator = Class.forName("ch.qos.logback.classic.joran.JoranConfigurator");
        joranConfiguratoroObj = joranConfigurator.newInstance();
        ......
        URL url = ClientLogger.class.getClassLoader().getResource(logback_resource_file);
        Method doConfigure =
                joranConfiguratoroObj.getClass().getMethod("doConfigure", URL.class);
        doConfigure.invoke(joranConfiguratoroObj, url);
    catch (Exception e) {
        System.err.println(e);
    }

因为我直接在eclipse中导入源码，没有包含`logback_rocketmq_client.xml`这个配置文件，导致url为null，所以在`JoranConfigurator.doConfigure`
时发生了NPL，输入堆栈信息就可以很明白的看到如下：

    java.lang.reflect.InvocationTargetException
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:39)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:25)
        at java.lang.reflect.Method.invoke(Method.java:597)
        at com.alibaba.rocketmq.client.log.ClientLogger.createLogger(ClientLogger.java:91)
        at com.alibaba.rocketmq.client.log.ClientLogger.<clinit>(ClientLogger.java:39)
        at com.alibaba.rocketmq.client.impl.producer.DefaultMQProducerImpl.<init>(DefaultMQProducerImpl.java:61)
        at com.alibaba.rocketmq.client.producer.DefaultMQProducer.<init>(DefaultMQProducer.java:97)
        at com.alibaba.rocketmq.client.producer.DefaultMQProducer.<init>(DefaultMQProducer.java:86)
        at com.alibaba.rocketmq.example.quickstart.Producer.main(Producer.java:30)
    Caused by: java.lang.NullPointerException
        at ch.qos.logback.core.joran.GenericConfigurator.doConfigure(GenericConfigurator.java:43)
        ... 10 more

虽然不影响程序运行，但是看到第一个输出就是一个莫名其妙的InvocationTargetException，感觉挺奇怪的。

- `System.err.println(e)`这块的处理很不合理。应该作为WARN日志并说明原因
- 客户端既然使用了***slf4j***，那么用户最终使用那个日志系统以及怎么配置就完全就是用户的事了，没必要在ClientLogger中做这么多日志相关的处理

##### CODE: 17  DESC: No topic route info in name server for the topic: TopicTest

第一次运行Producer出现这个异常信息，接着运行Consumer时没有出现，第二次运行Producer也不会出现。应该是第一次运行时，NamingServer中还没有
TopicTest的路由信息导致的。