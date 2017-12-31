# HeartBeat心跳协议数据包规格
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