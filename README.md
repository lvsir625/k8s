# k8s
--------
前言：
kubernetes 高可用集群（以下简称 k8s ha cluster）的常见的大致分为3种：

1. 手工部署为以“进程”方式运行
2. 用kubeadm部署为以“容器”方式运行
3. 手工部署为以“容器”方式运行

以容器方式运行有很多的优点，也是推荐的方式。
如果对以“进程”方式运行感兴趣，可以访问以下地址了解etcd cluster的部署方式
https://github.com/zhangjiongdev/k8s/blob/master/etcd_setup_for_centos7.md

在以“容器”方式运行的2种方式中，用 kubeadm 部署是比较简单的一种，也是本次介绍的内容。
安装部署的顺序和我后面介绍的顺序需保持一致，在实际的测试中不要轻易打乱文档步骤的顺序，以避免部署问题。

第一部分
简要介绍部署 k8s ha cluster 所需要用到的虚机的准备工作
我用的部署环境是台式电脑：
CPU：Intel i7
内存：16GB
操作系统：64位 Windows 10 专业版
网络：可以无限制的访问国内互联网

虚机软件：VirtualBox 6.0.8
虚机操作系统镜像：CentOS-7-x86_64-DVD-1810.iso

注：某些公司网络下的上网行为管理，可能会影响安装部署过程的顺利进行

部署步骤如下：
下载VirtualBox and CentOS
安装VirtualBox
创建 VirtualBox NAT 网络
https://github.com/zhangjiongdev/k8s/blob/master/lesson1/part1.md
创建虚拟机
安装CentOS
https://github.com/zhangjiongdev/k8s/blob/master/lesson1/part2.md

以上part1.md  part2.md 是 k8s ha cluster 准备环境的安装步骤，请花1分钟左右简单浏览一下

part1.md中，“创建 VirtualBox NAT 网络”是必须的，用来让虚拟机可以互访并访问公网，同时也能通过“端口转发”来直连ssh
part2.md中 处理器数量：2 （大于1）是必须的，否则会失败
part2.md中“修改主机名”中的 hostnamectl set-hostname master1，
4台服务器需要对应的修改为（master1,master2,master3,node1）
part2.md中“设置静态IP地址”中的hip=30.0.2.11，4台服务器对应修改为（30.0.2.11,30.0.2.12,30.0.2.13,30.0.2.14）

其他步骤请遵照文档步骤进行

按照上述 part1.md、part2.md 文档部署4台虚拟机用于k8s ha 集群
虚拟服务器IP和主机名列表如下：
30.0.2.11 master1
30.0.2.12 master2
30.0.2.13 master3
30.0.2.14 node1
以上步骤完成后，4台的CentOS7的安装配置就算完成了。


第二部分
4台虚机，安装docker
容器的安装是k8s的基础，我们用的容器是docker
安装步骤对应的说明如下：
https://github.com/zhangjiongdev/k8s/blob/master/lesson1/part3.md
请花1分钟左右简单浏览一下

目前我们安装完后的版本是 Docker version 18.09.6, build 481bc77156


第三部分
Keepalived+HAProxy安装配置
这是k8s ha cluster的最后一步准备工作

第1台（master1）控制平面安装配置Keepalived+HAProxy
部署步骤如下：
在master1 安装 Keepalived+HAProxy
https://github.com/zhangjiongdev/k8s/blob/master/lesson1/part4.md
在master1 配置启动 Keepalived+HAProxy
https://github.com/zhangjiongdev/k8s/blob/master/lesson1/part5.md
请花1分钟左右简单浏览一下

第2、3台（master2、master3）控制平面安装配置Keepalived+HAProxy
在master2、master3 安装 Keepalived+HAProxy
https://github.com/zhangjiongdev/k8s/blob/master/lesson1/part4.md
在master2、master3 配置启动 Keepalived+HAProxy
https://github.com/zhangjiongdev/k8s/blob/master/lesson1/part6.md


我们有3台master组成的k8s控制平面集群
请注第2、3台控制平面用的是part4.md+part6.md，而第一台用的是part4.md+part5.md，是不同的。
可以留意一下part6.md和part5.md区别之处
如果第2、3台也用了part4.md+part5.md，会失败
准备工作全部完成，下面进入k8s部分的安装


第四部分
部署k8s ha cluster 控制平面

第1台控制平面（master1）安装配置k8s ha 集群
https://github.com/zhangjiongdev/k8s/blob/master/lesson1/part7.md
docker pull 会需要一些时间，请确保需要的images全部pull下来
初始化 kubeadm init --config kubeadm-init.yaml 需要等待几分钟，因为容器需要花一些时间去自动创建
kubectl get pod --all-namespaces -o wide
除了前两行coredns 是pending之外，其他的都要running
kubectl apply -f kube-flannel.yml 执行完后，同样要等待一会
部署完后，会显示类似以下的命令行，第一段命令是第2、3台master加入k8s集群用的，第二段是node节点加入k8s集群用的，两者的差异在于master多了一个--experimental-control-plane参数

类似这样的

kubeadm join 30.0.2.10:6443 --token abcdef.0123456789abcdef     --discovery-token-ca-cert-hash sha256:7b12dba65658b9082ee9025f5f0eb3252c517b167317bfcbe1888bf004658d20     --experimental-control-plane
kubeadm join 30.0.2.10:6443 --token abcdef.0123456789abcdef     --discovery-token-ca-cert-hash sha256:7b12dba65658b9082ee9025f5f0eb3252c517b167317bfcbe1888bf004658d20


第2、3台控制平面（master2、master3）安装配置加入 k8s ha 集群
https://github.com/zhangjiongdev/k8s/blob/master/lesson1/part8.md
scp root@master1 这种命令不要多行一起执行，因为中间会需要输入master1的root密码


第五部分
部署k8s ha cluster 数据平面
数据平面(node1)安装配置加入 k8s ha 集群
https://github.com/zhangjiongdev/k8s/blob/master/lesson1/part9.md
部署完后运行master1 输出的kubeadm join 命令行
类似这样：
kubeadm join 30.0.2.10:6443 --token abcdef.0123456789abcdef     --discovery-token-ca-cert-hash sha256:7b12dba65658b9082ee9025f5f0eb3252c517b167317bfcbe1888bf004658d20

如果在部署的过程中需要重试 kubeadm join，可以使用以下命令后重新执行部署命令
kubeadm reset


第6部分
测试一个nginx无状态pod
https://github.com/zhangjiongdev/k8s/blob/master/lesson1/part10.md
用 curl http://10.245.130.246 可以测试集群是否正常运行
