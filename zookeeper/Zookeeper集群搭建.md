# Zookeeper集群搭建

Zookeeper节点部署越多,服务的可靠性越高,建议部署奇数个节点,因为zookeeper集群是以宕机个数过半才会让整个集群宕机的.
但是随着zookeeper的集群机器增多,读请求的吞吐会提高但是写请求的吞吐会下降.  


由于主机资源有限,这里只演示四台zk主机的**伪集群**部署方式,真实的集群部署类似.  
注意:我这里增加了第四台zk服务器,这台zk服务器比较特殊,他是作为观察者存在的,不参与leader选举.

1.修改配置文件/conf    
由于这里是伪集群,所以不能使用默认的配置文件zoo.cfg,这里创建四个配置文件  
```
cp zoo.cfg zoo-1.cfg  
cp zoo.cfg zoo-2.cfg
cp zoo.cfg zoo-3.cfg
cp zoo.cfg zoo-4.cfg
``` 
分别修改四个配置文件,主要修改数据目录、端口号、集群配置  
比如```zoo-1.cfg```  
```
dataDir=ZOOKEEPER_HOME/tmp/zookeeper/1

clientPort=2182

server.1=IP:2886:3886
server.2=IP:2887:3887
server.3=IP:2888:3888
server.4=IP:2889:3889:observer
```   
其他3个配置文件类似,保证每个zk服务器的数据目录和端口号不一样,集群配置必须一致.  

这里需要关注下zk集群的配置,格式:```server.A=B:C:D[:observer]```    
A是一个数字,表示这个是第几个zk服务器    
B是这个zk服务器的IP地址,也可以设置为主机名      
C是端口号,用来集群成员的信息交换,表示这个服务器与集群中的leader服务器交换信息的端口    
D是在leader挂掉时专门用来进行选举leader所用的端口    
observer表示这台zk服务器只作为观察者存在,不参与leader选举    

2.在数据目录下创建ServerID标志    
我们需要在zk的数据目录下新建myid文件,文件内如即当前zk的A值.  
在这里对第一个zk即在数据目录```ZOOKEEPER_HOME/tmp/zookeeper/1```下新建myid文件,文件内容为1.其他几个zk服务器配置类似. 

3.启动、查看、停止zk集群  
由于是伪集群模式,我们需要告诉zk服务器使用的是哪个配置文件,默认用的是```conf/zoo.cfg```.  

```aidl
./zkServer.sh start ../conf/zoo-1.cfg

./zkServer.sh status ../conf/zoo-1.cfg  

./zkServer.sh stop ../conf/zoo-1.cfg
```  

我们分别启动zk1,zk2,zk3,zk4,查看zk集群的状态  
```aidl
./zkServer.sh status ../conf/zoo-1.cfg
ZooKeeper JMX enabled by default
Using config: ../conf/zoo-1.cfg
Mode: follower  

./zkServer.sh status ../conf/zoo-2.cfg
ZooKeeper JMX enabled by default
Using config: ../conf/zoo-2.cfg
Mode: leader  

./zkServer.sh status ../conf/zoo-3.cfg
ZooKeeper JMX enabled by default
Using config: ../conf/zoo-3.cfg
Mode: follower  

./zkServer.sh status ../conf/zoo-4.cfg
ZooKeeper JMX enabled by default
Using config: ../conf/zoo-4.cfg
Mode: observer
```  
这个结果是可预知的.  
当启动zk1的时候,通过配置文件中配的端口号D尝试与集群中的其他zk服务器通信进行leader选举.由于其他zk服务器都未启动,zk1一直在尝试连接然后被拒绝.
启动zk2,此时满足了仲裁法定人数(集群服务器数量至少需要大于所有服务器一半数量,由于zk4是观察者,不参与leader选举,这里有效的zk服务器数2大于所有服务器3的一半)
zk1和zk2可以进行leader选举.  
zookeeper的leader选举规则很简单:**优先选zxid值最大的zk服务器,如果多个zk服务器拥有最新的zxid值,则取sid值最大的zk服务器.** 注意,这里的sid即我们配置zk集群时的A值.    
基于这个leader选举规则,zk2被选举为leader,zk1成为follower.  
启动zk3,寻找到leader,跟随leader,zk3成为follower.  
启动zk4,成为observer.  
注意:简单的可以通过启动日志bin/zookeeper.out看到这个过程.

