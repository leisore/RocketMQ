####简介####
- - -
本文档记录了在Windows平台下构建、配置和启动RocketMQ的步骤。

####环境####
- - -
1. 操作系统：Windows XP Professional 2002 SP3
2. JDK：1.6.0_20
3. Maven：3.0.4
4. Git：1.7.4.msysgit.0
5. MinGW：1.0.11

####构建####
- - -
1. 下载源码
    >cd F:\rocketmq
     git clone https://github.com/alibaba/RocketMQ.git

2. 编译源码：
    >set MAVEN_OPTS=-Xmx1024m
     mvn -Dmaven.test.skip=true clean package install assembly:assembly -U

    编译后分发包位于：
    >F:\rocketmq\RocketMQ\target\alibaba-rocketmq-3.2.2\alibaba-rocketmq

3. 运行:

