## Rancher环境搭建
本例仅安装单个rancher节点，找一台能够与已创建的k8s集群联通的服务器，用来安装rancher服务

### 镜像下载
docker pull rancher/rancher:latest

### 开启转发参数
sysctl -w net.ipv4.ip_forward=1

### 启动rancher容器
docker run -d --restart=unless-stopped -p 8088:80 -p 8443:443 rancher/rancher:latest

### 浏览器访问
10.10.10.100:8088  
页面自动跳转到10.10.10.100:8443

调整rancher界面语言设置  

### 导入集群
导入集群时选择import```导入现有的Kubernetes集群```

在已创建的k8s集群的master节点上执行：

```
kubectl apply -f https://10.238.18.129:8443/v3/import/8wmpj86prxr7x9fncctkr769qnq477nzfxtfpv2mrdxlxlwcswbqss.yaml
```

如果出现ssl认证错误则执行:

```
curl --insecure -sfL https://10.238.18.129:8443/v3/import/8wmpj86prxr7x9fncctkr769qnq477nzfxtfpv2mrdxlxlwcswbqss.yaml | kubectl apply -f -
```

等响应的组建安装完成后，可以看到rancher中集群更新的信息包含了已创建k8s集群的相应节点(master节点加入后，rancher也会通过master发现其他节点，无需重复加入)

