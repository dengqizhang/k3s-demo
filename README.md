# k3s-demo

#### 介绍
关于k3s集群的搭建，和Kubernetes的语法demo。
#### 使用说明

# k3s的使用笔记
k3s的一些命令：
- `kubectl get node` 查看node
- `kubectl create -f [文件名].yaml` 运行配置文件
- `kubectl get pod -o wide` 查看运行的pod  
- `kubectl get service` 查看service
- `kubectl get deploy` 查看deploy
- `kubectl get svc -o wide` 查看端口
- `kubectl apply -f [文件名].yaml` 更新配置文件
- `kubectl delete pod [pod名称]` 删除pod
**访问pod：**  
node ,pod,service都有不同的虚拟ip,启动了3个pod后，我们可以通过service的端口进行访问，service会在三个pod做轮询操作，service寻找pod是动态的，删除所有pod再新建依然可以使用。    

使用deploy可以弹性的管理pod，扩容或缩容。   
此demo的使用步骤：
- 1、将springboot项目通过docker制作成镜像，上传到aliyun容器镜像仓库
- 2、准备两台服务器，master服务器，和Agent服务器。  
在master服务器里下载k3s的脚本
```
curl -sfL https://rancher-mirror.rancher.cn/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn K3S_URL=https://myserver:6443 K3S_TOKEN=mynodetoken sh -
```
然后通过`cat /var/lib/rancher/k3s/server/node-token`命令查看master服务器里的k3s token，把该token保留。   
在Agent服务器里输入
```
curl -sfL https://rancher-mirror.rancher.cn/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn K3S_URL=https://[master服务器的内网ip地址]:6443 K3S_TOKEN=[master的token] sh -
```
到这时我们就完成了集群的搭建。
我们可以在master上用`kubectl get node`来查看有哪些node。

3、接下来我们使用yaml文件的配置方式来操作pod，deployment,service。
pod.yaml:
```
apiVersion: v1
kind: Pod
metadata:
  name: pod1 //pod名称
  labels:
    name: pod1 //pod的标签名称
spec:
  containers:
  - name: pod1
    image: nginx //镜像名称
    resources:
      limits:
        memory: "128Mi"
        cpu: "300m"
    ports:
      - containerPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: pod2
  labels:
    name: pod1
spec:
  containers:
  - name: pod2
    image: nginx
    resources:
      limits:
        memory: "128Mi"
        cpu: "300m"
    ports:
      - containerPort: 80 //镜像内的启动端口
```
deploy.yaml:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deploy //deploy的名称
spec:
  replicas: 1 //弹性的pod数量
  selector:
    matchLabels:
      name: pod1
  template:
    metadata:
      labels:
        name: pod1
    spec:
      containers:
      - name: myapp
        image: nginx
        resources:
          limits:
            memory: "128Mi"
            cpu: "300m"
        ports:
          - containerPort: 80
```
service.yaml:
```
apiVersion: v1
kind: Service
metadata:
  name: myapp-mvc
spec:
  type: NodePort
  selector:
    name: pod1
  ports:
  - port: 2000 //service端口
    targetPort: 80 //镜像端口
    nodePort: 30001 //noPort端口
```
使用`kubectl create -f [文件名].yaml`运行配置文件   
4、按照pod，service，deploy的顺序运行下来输入`kubectl get pod`会发现3个pod成功运行，此时删除掉一个，deploy会自动补充一个上去。

5、我们此时的项目只能在内网内进行访问，如果想通过外网进行访问服务，则需要阿里云的CLB负载均衡，反向代理ip和端口，完成外网的访问。

6、配置完负载均衡后，我们在页面访问发现线程池和pod一直在轮询变化，完成了对外网访问和负载均衡。




#### 特技

1.  使用 Readme\_XXX.md 来支持不同的语言，例如 Readme\_en.md, Readme\_zh.md
2.  Gitee 官方博客 [blog.gitee.com](https://blog.gitee.com)
3.  你可以 [https://gitee.com/explore](https://gitee.com/explore) 这个地址来了解 Gitee 上的优秀开源项目
4.  [GVP](https://gitee.com/gvp) 全称是 Gitee 最有价值开源项目，是综合评定出的优秀开源项目
5.  Gitee 官方提供的使用手册 [https://gitee.com/help](https://gitee.com/help)
6.  Gitee 封面人物是一档用来展示 Gitee 会员风采的栏目 [https://gitee.com/gitee-stars/](https://gitee.com/gitee-stars/)
