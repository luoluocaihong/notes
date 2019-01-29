[官网](https://www.consul.io/)下载合适的版本,这里下载当前最新的1.4.2版本  
```aidl
//1.下载压缩包
wget https://releases.hashicorp.com/consul/1.4.2/consul_1.4.2_linux_amd64.zip  
//2.解压
unzip consul_1.4.2_linux_amd64.zip  
//3.检查安装
./consul -v
Consul v1.4.2
Protocol 2 spoken by default, understands 2 to 3 (agent will automatically use protocol >2 when speaking to compatible agents)  
``` 

如果是开发者模式,单节点部署的话,只需要通过以下启动方式来启动即可:  
```nohup ./consul agent -dev &```  

这里部署consul集群:    
172.16.22.2    
172.16.22.51  
10.45.82.76       

启动命令:    
```aidl
172.16.22.2  
[root@jobserver consul]# nohup ./consul agent -server -bind=172.16.22.2 -client=0.0.0.0 -bootstrap-expect=3 -data-dir=/home/consul/data -node=server-xh-1 -ui &  

172.16.22.51
[iotspark@iotsparknode1 consul]$nohup ./consul agent -server -bind=172.16.22.51 -client=0.0.0.0 -bootstrap-expect=3 -data-dir=/usr/iotspark/consul/data -node=server-xh-2 -ui &  

10.45.82.76  
iotcmp@CentOS7.3[/ztesoft/iotcmp/tools/consul]$nohup ./consul agent -server -bind=10.45.82.76 -client=0.0.0.0 -bootstrap-expect=3 -data-dir=/ztesoft/iotcmp/tools/consul/data -node=server-xh-3 -ui &  
```     

启动命令参数可参考[Command-line Options](https://www.consul.io/docs/agent/options.html)  

这个时候查看下consul服务端,会发现每个consul服务端下都只有各自自身,即每个consul服务端都是相互独立的.   
```aidl  
172.16.22.2 
[root@jobserver consul]# ./consul members
Node         Address           Status  Type    Build  Protocol  DC   Segment
server-xh-1  172.16.22.2:8301  alive   server  1.4.1  2         dc1  <all>  

172.16.22.51  
[iotspark@iotsparknode1 consul]$./consul members
Node         Address            Status  Type    Build  Protocol  DC   Segment
server-xh-2  172.16.22.51:8301  alive   server  1.4.2  2         dc1  <all>

10.45.82.76   
iotcmp@CentOS7.3[/ztesoft/iotcmp/tools/consul]$./consul members
Node         Address           Status  Type    Build  Protocol  DC   Segment
server-xh-3  10.45.82.76:8301  alive   server  1.4.2  2         dc1  <all>
```  

通过将某个consul server加入其它consul server来实现集群部署  
```aidl
[root@jobserver consul]# ./consul join 10.45.82.76
Successfully joined cluster by contacting 1 nodes.
[root@jobserver consul]# ./consul members
Node       Address            Status  Type    Build  Protocol  DC   Segment
server-01  172.16.22.2:8301   alive   server  1.4.1  2         dc1  <all>
server-03  10.45.82.76:8301   alive   server  1.4.2  2         dc1  <all> 
```

```aidl
[iotspark@iotsparknode1 consul]$./consul join 10.45.82.76
Successfully joined cluster by contacting 1 nodes.
[iotspark@iotsparknode1 consul]$./consul members
Node       Address            Status  Type    Build  Protocol  DC   Segment
server-01  172.16.22.2:8301   alive   server  1.4.1  2         dc1  <all>
server-02  172.16.22.51:8301  alive   server  1.4.2  2         dc1  <all>
server-03  10.45.82.76:8301   alive   server  1.4.2  2         dc1  <all>
```
至此,包含三个节点的consul cluster搭建好了  

我们可以通过web UI查看集群信息  
注意,启动consul server的时候带参数```-ui```即可启动consul自带的web管理界面,默认端口号8500  
http://10.45.82.76:8500   
![consul web ui](https://github.com/luoluocaihong/notes/blob/master/springcloud/pic/consul-webUI.png)   
 
注意:集群部署的时候可以看下启动日志,能比较清楚的看到Leader选举的流程等    





