####���####
- - -
���ĵ���¼����Windowsƽ̨�¹��������ú�����RocketMQ�Ĳ��衣

####����####
- - -
1. ����ϵͳ��Windows XP Professional 2002 SP3
2. JDK��1.6.0_20
3. Maven��3.0.4
4. Git��1.7.4.msysgit.0
5. MinGW��1.0.11

####����####
- - -
1. ����Դ��
    >cd F:\rocketmq
     git clone https://github.com/alibaba/RocketMQ.git

2. ����Դ�룺
    >set MAVEN_OPTS=-Xmx1024m
     mvn -Dmaven.test.skip=true clean package install assembly:assembly -U

    �����ַ���λ�ڣ�
    >F:\rocketmq\RocketMQ\target\alibaba-rocketmq-3.2.2\alibaba-rocketmq

3. ����:

