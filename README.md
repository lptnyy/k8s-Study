# k8s-Study k8s 学习以及部署总结留档
## (一)搭建k8s集群测试环境   
    准备三台服务器 or 虚拟机  
    192.168.0.105  master centos7    
    192.168.0.106  node1 centos7    
    192.168.0.107  node2 centos7  
 
## （二） 修改主机名 编辑/etc/hosts 
    设置所有服务器的主机名  
    命令 hostnamectl set-hostname master   
    命令 hostnamectl set-hostname node1   
    命令 hostnamectl set-hostname node2  
    所有服务器 /etc/hosts 添加以下内容  
    192.168.0.105  master localhost  (localhost 只需要105 配置) 
    192.168.0.106  node1 
    192.168.0.107  node2  

## （三）安装  
    升级yum版本  master node 相关服务器都执行  
    yum -y install epel-release  
    yum update  
    
#### 关闭所有服务器防火墙  
    systemctl disable firewalld  
    systemctl stop firewalld  
    setenforce 0  
#### master 执行命令  
    yum install -y etcd kubernetes-master ntp flannel  
#### node 执行命令  
    yum install -y kubernetes-node ntp flannel docker  
#### 时间设定所有服务器  
    systemctl start ntpd  
    systemctl enable ntpd  
    ntpdate ntp1.aliyun.com  
    hwclock -w  
#### 设置etcd  
    vi /etc/etcd/etcd.conf  
    ETCD_LISTEN_CLIENT_URLS="http://localhost:2379,http://192.168.0.105:2379"  
    ETCD_ADVERTISE_CLIENT_URLS="http://192.168.0.105:2379"  
    启动服务  
    systemctl start etcd  
    systemctl enable etcd  
#### 配置master k8s 服务
     vi /etc/kubernetes/config  
     KUBE_LOGTOSTDERR="--logtostderr=true"  
     KUBE_LOG_LEVEL="--v=0"  
     KUBE_ALLOW_PRIV="--allow-privileged=false"  
     KUBE_MASTER="--master=http://192.168.0.105:8080"  
     
     vi /etc/kubernetes/apiserver  
     UBE_API_ADDRESS="--insecure-bind-address=192.168.0.105" master 节点ip  
     UBE_ETCD_SERVERS="--etcd-servers=http://master:2379"  
     KUBE_ADMISSION_CONTROL="--admission-control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ServiceAccount,ResourceQuota"  
     UBE_API_ARGS=""  
     
     vi /etc/kubernetes/scheduler  
     KUBE_SCHEDULER_ARGS="--address=0.0.0.0" 
     
     vi /etc/sysconfig/flanneld  
     FLANNEL_ETCD_ENDPOINTS="http://master:2379"  
     FLANNEL_ETCD_PREFIX="/atomic.io/network"  
     
     etcd里追加 flanneld 网络配置  
     etcdctl set /atomic.io/network/config '{"Network": "172.16.0.0/16"}'
     
     启动服务   
     for i in  kube-apiserver kube-controller-manager kube-scheduler flanneld;do systemctl restart $i; systemctl enable $i; ;systemctl status $i ; done

     
#### 配置node k8s 服务 所有node 节点服务器
     vi /etc/sysconfig/flanneld   
     FLANNEL_ETCD_ENDPOINTS="http://master:2379"  
     FLANNEL_ETCD_PREFIX="/atomic.io/network"  
     
     vi  /etc/kubernetes/config 
     KUBE_LOGTOSTDERR="--logtostderr=true"  
     KUBE_LOG_LEVEL="--v=0"  
     KUBE_ALLOW_PRIV="--allow-privileged=false"  
     KUBE_MASTER="--master=http://master:8080"  
     
     vi /etc/kubernetes/proxy  
     KUBE_PROXY_ARGS="--bind=address=0.0.0.0"  
     
     vi /etc/kubernetes/kubelet  
     KUBELET_ADDRESS="--address=127.0.0.1"
     KUBELET_HOSTNAME="--hostname-override=192.168.0.106"  这里配置 每一个node节点服务器分配的Ip  
     KUBELET_API_SERVER="--api-servers=http://master:8080"
     KUBELET_POD_INFRA_CONTAINER="--pod-infra-container-image=registry.access.redhat.com/rhel7/pod-infrastructure:latest"
     KUBELET_ARGS=""  
     
     启动服务
     for i in flanneld kube-proxy kubelet docker;do systemctl restart $i;systemctl enable $i;systemctl status $i ;done  
     
 ### 验证集群结果
    root@master ~]# kubectl get nodes
    NAME            STATUS    AGE
    192.168.0.106   Ready     17m
    192.168.0.107   Ready     4s
    [root@master ~]# 