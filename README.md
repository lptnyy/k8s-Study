# k8s-Study k8s 学习以及部署总结留档
## (一)搭建k8s集群测试环境   
    准备三台服务器 or 虚拟机  
    192.168.0.105  master centos7    
    192.168.0.106  node1 centos7    
    192.168.0.107  node2 centos7  
 
##（二）所有服务器编辑 /etc/hosts  
192.168.0.105  master  
192.168.0.106  node1   
192.168.0.107  node2   

## （三）安装  
升级yum版本  master node 相关服务器都执行  
yum -y install epel-release  
yum update  

####master 执行命令  
yum install -y etcd kubernetes-master ntp flannel  
####node 执行命令  
yum install -y kubernetes-node ntp flannel docker  
####时间设定所有服务器  
systemctl start ntpd  
systemctl enable ntpd  
ntpdate ntp1.aliyun.com  
hwclock -w  
####设置etcd  
vi /etc/etcd/etcd.conf  
ETCD_LISTEN_CLIENT_URLS="http://localhost:2379,http://master:2379"  
ETCD_ADVERTISE_CLIENT_URLS="http://master:2379"  
启动服务  
systemctl start etcd  
systemctl enable etcd  
####