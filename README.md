# SmartMonitor
一个分布式监控系统，主要分为NameNode和DataNode，NameNode用来管理和收集DataNode的状态；

# HeartBeat心跳协议数据包规格
```
 0  1  2  3  4  5  6  7  0  1  2  3  4  5  6  7
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|    Magic        |Flags|S |    Exception Code  |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                                               |
|              DataNode Number                  |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|              Group Number                     |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                 Interval                      |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+

Magic: 6 bits，魔术数，用来表明这是什么协议，这里取101001
Flags: 2 bits, 1 bit 表示查询位，如果是1，表明NameNode强制查询当前DataNode的状态，是DataNode发送的；
               0 表示是DataNode主动发来的HeartBeat，正常情况是0;
               1 bit 表示响应还是请求，0表示请求request，1表示响应response
S    : 1 bit,  表示DataNode状态， 0表示异常，1表示正常
EC   : 7 bits, 表示异常码，如果S=1， 则需要对应的EC查看是什么异常
DN   : 32bits, 表示DataNode的ID标识，是一个int类型
GN   : 16bits, 表示DataNode所属的组号，是一个short int类型
I    : 16bits, 表示HeartBeat的时间间隔，是一个short int类型， 单位是秒，因此最大间隔是32768s，大概9个小时
```
# 监控过程握手
TCP连接建立之后,进行UDP端口汇报和ID分配
```sequence
DataNode->NameNode: HeartBeat UDP port 
NameNode-->DataNode: ok, udp port accepted
NameNode-->DataNode: dataNodeGroupNumber
NameNode-->DataNode: dataNodeNumber
NameNode-->DataNode: interval
DataNode->NameNode: ok, ID accepted
```
之后，DataNode启动HeartBeatUDP线程开始定时发送HeartBeatMsg，每一个DataNode主线程负责TCP连接和任务处理，HeartBeatUDP线程负责发送心跳包。NameNode主线成负责建立连接，让后每一个TCP连接会创建新的Task线程处理对应的DataNode的任务，另外还有一个HeartBeatUDP线程负责收发所有DataNode的心跳包。NameNode会维护一个DataNode的集合，收集所有连接上的DataNode的状态信息。

## 查看端口被占用并强制释放
```
kinny@kinny-Lenovo-XiaoXin:~$ sudo netstat -apn | grep 9998
[sudo] password for kinny: 
udp6       0      0 :::9998                 :::*                                21586/java      
kinny@kinny-Lenovo-XiaoXin:~$ sudo netstat -apn | grep 9999
tcp6       0      0 :::9999                 :::*                    LISTEN      21586/java      
kinny@kinny-Lenovo-XiaoXin:~$ lsof -i:9998
COMMAND   PID  USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
java    21586 kinny   78u  IPv6 910989      0t0  UDP *:9998 
kinny@kinny-Lenovo-XiaoXin:~$ lsof -i:9999
COMMAND   PID  USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
java    21586 kinny   79u  IPv6 910990      0t0  TCP *:9999 (LISTEN)
kinny@kinny-Lenovo-XiaoXin:~$ sudo kill -9 21586
kinny@kinny-Lenovo-XiaoXin:~$ sudo netstat -apn | grep 9999
kinny@kinny-Lenovo-XiaoXin:~$ 
```

## ArrayList的并发修改
对集合ArrayList进行并发修改时，虽然已经枷锁，但是跑出异常,采用 Iterator进行遍历，因为 集合在遍历的同时直接修改其结构是不允许的
```
Exception in thread "Thread-1" java.util.ConcurrentModificationException
	at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:901)
	at java.util.ArrayList$Itr.next(ArrayList.java:851)
	at sa.sse.ustc.NameNode$TimeOutChecker.run(NameNode.java:128)
```

## 参考
 - [heartbeat-protocols-algorithms-or-best-practices](http://stackoverflow.com/questions/1442189/heartbeat-protocols-algorithms-or-best-practices)
 - [java-heartbeat-design](http://stackoverflow.com/questions/33869092/java-heartbeat-design)
 - [How to add heartbeat messaging on top of this Java code](http://stackoverflow.com/questions/15604015/how-to-add-heartbeat-messaging-on-top-of-this-java-code-for-knockknockclient-se)
 - [Heartbeat Mechanism](http://docs.oracle.com/cd/E19206-01/816-4178/6madjde6e/index.html)
 - [Listen to heartbeats using JMS](http://www.javaworld.com/article/2074116/java-web-development/listen-to-heartbeats-using-jms.html)
 - [How can I guarantee a “stay alive” heartbeat is sent](http://stackoverflow.com/questions/676812/how-can-i-guarantee-a-stay-alive-heartbeat-is-sent)
 - [The HeartBeat Applet](https://www.safaribooksonline.com/library/view/learning-java/1565927184/ch11s02.html)
 - [how-to-set-a-timer-in-java](http://stackoverflow.com/questions/4044726/how-to-set-a-timer-in-java)
 - [分布式系统部署、监控与进程管理的几重境界](http://blog.csdn.net/solstice/article/details/6406944)
 - [消息队列设计精要](http://tech.meituan.com/mq-design.html)
 - [rabbitmq](https://www.rabbitmq.com/tutorials/tutorial-one-java.html)