# Zookeeper:开源的分布式应用程序协调服务  

#### 1.Zookeeper的特性  
**一致性**：数据一致性,最终一致性,数据按照顺序分批入库  
**原子性**：事物要么成功要么失败,不会局部化  
**单一视图**：客户端连接集群中的任一zk节点,数据都是一致的  
**可靠性**：每次对zk的操作状态都会保存在服务端  
**实时性**：某段特定时间内客户端可以读取到zk服务端的最新数据  


#### 2.Zookeeper的基本数据模型  
树形结构,节点称为**znode**,可以包含数据也可以包含子节点    
zk为了保证高吞吐和低延迟,在内存中维护了这个树状的目录结构  
节点存储的数据不宜过大,几K即可,默认上限1M  
节点分为临时节点和永久节点,临时节点在客户端断开后消失    
节点有版本号,当节点数据发生变化,该节点的版本号就会累加(乐观锁)    
删除、修改过时节点,版本号不匹配则会报错       
节点可以设置权限acl,通过权限来限制用户的访问    


#### 3.Zookeeper的应用场景    
a.master节点选举    
b.统一配置文件管理,部署一套服务器即可把相同配置文件同步到其他所有服务器  
c.发布与订阅      
d.提供**分布式锁**,分布式环境中不同进程之间争夺资源,类似于多线程的锁        
e.集群管理,集群中保证数据的强一致性  
f.负载均衡  
g.命名服务  
h.分布式协调、通知  
i.分布式队列  
  
  
#### 4.Zookeeper的常用命令  
```./zkCli.sh```    
默认连接本机端口号为2181的zk server：```localhost:2181```
如果要指定连接到其他的zk server,可以设置具体的连接zk地址：```./zkCli.sh -server ip:port```  

```ls 、ls2 、 stat 、 get```   
```ls```列出节点下的子节点   
```ls2```展示节点的状态信息并列出节点下的子节点  
```stat```展示节点的状态信息  
```get```获取节点的数据信息并展示节点的状态信息   
 
```create [-s] [-e] path data acl``` 
```-s```这个参数表示创建有序节点    
```-e```这个参数表示创建临时节点,否则创建永久节点  

```set path data [version]```  
```delete path [version]```    
```version```节点的数据版本  
  
更多命令  
```aidl
[zk: ip:port(CONNECTED) 1] help
ZooKeeper -server host:port cmd args
        connect host:port
        get path [watch]
        ls path [watch]
        set path data [version]
        rmr path
        delquota [-n|-b] path
        quit 
        printwatches on|off
        create [-s] [-e] path data acl
        stat path [watch]
        close 
        ls2 path [watch]
        history 
        listquota path
        setAcl path acl
        getAcl path
        sync path
        redo cmdno
        addauth scheme auth
        delete path [version]
        setquota -n|-b val path
```  

详见```org.apache.zookeeper.ZooKeeperMain```
 
 
#### 5.Zookeeper的状态信息  
```aidl
[zk: ip:port(CONNECTED) 0] stat /
cZxid = 0x0
ctime = Thu Jan 01 08:00:00 CST 1970
mZxid = 0x0
mtime = Thu Jan 01 08:00:00 CST 1970
pZxid = 0x46e0a0
cversion = 1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 3
``` 
```cZxid```创建节点的事务ID  
```ctime```节点创建时间  
```mZxid```节点修改的事务ID  
```mtime```节点修改时间  
```pZxid```子节点列表最后一次被修改的事务ID  
```cversion```节点版本,子节点变化才会变更这个节点版本号    
```dataVersion```节点数据版本  
```aclVersion```节点权限版本  
```ephemeralOwner```用于判断节点是临时节点还是永久节点,永久节点为0x0    
```dataLength```节点数据长度  
```numChildren```子节点数量  


#### 6.Zookeeper的watcher机制  
Zookeeper中的watcher是**一次性**的,触发后立即销毁    
对于不同类型的操作触发的watcher事件是不同的   
  
**Watcher事件类型**    
NodeCreated  创建节点事件    
NodeDataChanged  修改节点数据事件    
NodeDeleted  删除节点事件    
NodeChildrenChanged 创建、删除子节点事件  

**Zookeeper的Watcher的使用场景**  
统一资源配置   
   

#### 7.ACL(access control lists)权限控制  
权限可以指定不同的权限范围以及角色  
 
ACL的构成:```scheme:id:permissions```    
```scheme```:代表采用的某种权限机制    
```id```:代表允许访问的用户    
```permissions```:权限组合字符串    
  
```Schema```权限机制类型:    
- world:```world:anyone:[permissions]```    
- auth:代表认证登录,```auth:user:password:[permissions]```  
- digest:需要对密码加密才能访问,```digest:username:BASE64(SHA1(password)):[permissions]```  
- ip:限制ip访问,```ip:192.168.1.1:[permissions]```  
- super:超级管理员,拥有所有权限    

注意：auth与digest的区别即前者明文后者密文。  
setAcl /path auth:test:test:cdrwa 等价于
setAcl /path digest:test:BASE64(SHA1(test)):cdrwa  
在通过addauth digest test:test后都能操作指定节点的权限  
  
加密方法```org.apache.zookeeper.server.auth.DigestAuthenticationProvider#generateDigest```

permissions：crdwa  
c:create  创建子节点    
r:read  获取节点/子节点    
d:delete  删除子节点  
w:write  设置节点数据    
a:admin  设置权限  
 
创建一个节点,默认的ACL为：  
```
[zk: ip:port(CONNECTED) 0] getAcl /
'world,'anyone
: cdrwa
```  

Demo:  
```aidl
[zk: ip:port(CONNECTED) 0] create -e /study ""
Created /study
[zk: ip:port(CONNECTED) 1] setAcl /study auth:test:test:cdrwa
Acl is not valid : /study
[zk: ip:port(CONNECTED) 2] addauth digest test:test
[zk: ip:port(CONNECTED) 3] getAcl /study                     
'world,'anyone
: cdrwa
[zk: ip:port(CONNECTED) 4] setAcl /study auth:test:test:cdrwa
cZxid = 0x1600000071
ctime = Wed Oct 17 16:56:13 HKT 2018
mZxid = 0x1600000071
mtime = Wed Oct 17 16:56:13 HKT 2018
pZxid = 0x1600000071
cversion = 0
dataVersion = 0
aclVersion = 1
ephemeralOwner = 0x100c3363cdc001b
dataLength = 0
numChildren = 0
[zk: ip:port(CONNECTED) 5] getAcl /study                     
'digest,'test:V28q/NynI4JI3Rk54h0r8O5kMug=
: cdrwa

[zk: ip:port(CONNECTED) 6] setAcl /study world:anyone:cdrwa  
cZxid = 0x1600000071
ctime = Wed Oct 17 16:56:13 HKT 2018
mZxid = 0x1600000071
mtime = Wed Oct 17 16:56:13 HKT 2018
pZxid = 0x1600000071
cversion = 0
dataVersion = 0
aclVersion = 2
ephemeralOwner = 0x100c3363cdc001b
dataLength = 0
numChildren = 0
[zk: ip:port(CONNECTED) 7] getAcl /study                   
'world,'anyone
: cdrwa

[zk: ip:port(CONNECTED) 8] setAcl /study digest:test:V28q/NynI4JI3Rk54h0r8O5kMug=:cdrwa
cZxid = 0x1600000071
ctime = Wed Oct 17 16:56:13 HKT 2018
mZxid = 0x1600000071
mtime = Wed Oct 17 16:56:13 HKT 2018
pZxid = 0x1600000071
cversion = 0
dataVersion = 0
aclVersion = 3
ephemeralOwner = 0x100c3363cdc001b
dataLength = 0
numChildren = 0
[zk: ip:port(CONNECTED) 9] getAcl /study
'digest,'test:V28q/NynI4JI3Rk54h0r8O5kMug=
: cdrwa

[zk: ip:port(CONNECTED) 1] setAcl /study ip:10.45.81.181:crdwa
cZxid = 0x160000008a
ctime = Wed Oct 17 18:37:53 HKT 2018
mZxid = 0x160000008a
mtime = Wed Oct 17 18:37:53 HKT 2018
pZxid = 0x160000008a
cversion = 0
dataVersion = 0
aclVersion = 1
ephemeralOwner = 0x100c3363cdc0024
dataLength = 0
numChildren = 0
[zk: ip:port(CONNECTED) 2] getAcl /study
'ip,'10.45.81.181
: cdrwa
```  

对于超级管理员的方式有点特殊,需要修改zkServer.sh添加系统参数   
```-Dzookeeper.DigestAuthenticationProvider.superDigest=super:Wx8GhnKMUA5//T7rz6vCLnIl1jA=```  
这里配置系统参数值为:super:11    
具体可以查看代码```org.apache.zookeeper.server.auth.DigestAuthenticationProvider```  
```aidl
    /** specify a command line property with key of 
     * "zookeeper.DigestAuthenticationProvider.superDigest"
     * and value of "super:<base64encoded(SHA1(password))>" to enable
     * super user access (i.e. acls disabled)
     */
    private final static String superDigest = System.getProperty(
        "zookeeper.DigestAuthenticationProvider.superDigest");
```
像上面几种方式如果权限设置错误导致节点不可用的场景,就可以通过超级管理员来修改权限.  
```
[zk: ip:port(CONNECTED) 1] ls /study
Authentication is not valid : /study
[zk: ip:port(CONNECTED) 2] addauth digest super:11
[zk: ip:port(CONNECTED) 3] ls /study
[]
```


#### 8.Zookeeper的四字命令  
zk提供四字命令与服务器进行交互  
注意:需要用到nc命令,安装:yum install nc  
使用命令: ```echo [commond]|nc [ip] [port]```  

支持的四字命令：  
```ruok``` 查看zk服务器是否启动,返回imok  
```stat``` 查看zk状态信息以及mode(单机/集群)  
```dump``` 列出未经处理的会话和临时节点  
```conf``` 查看服务器配置  
```cons``` 展示连接到zk服务器的客户端信息  
```envi``` 环境变量  
```mntr``` 监控zk健康信息  
```wchs``` 展示watch的信息  
```wchc``` session与watch的信息  
```wchp``` path与watch的信息  

注意:3.4.10版本开始,有些四字命令默认是不开启的.  
```zoo.cfg```中加入配置```4lw.commands.whitelist=*```


#### 9.Zookeeper集群  
集群角色:Leader、Follower、Observer    
[集群搭建](https://github.com/luoluocaihong/notes/blob/master/zookeeper/Zookeeper%E9%9B%86%E7%BE%A4%E6%90%AD%E5%BB%BA.md)  

