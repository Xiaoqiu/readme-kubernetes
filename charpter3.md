3.1 API server原理
- Pod，RC，Service的增删改查，watch
- 通过进程kube-apiserver进程提供服务，运行在master节点上面。
- 端口,--insecure-port=8080
- https端口, --secure-port=6443
- 可以通过kubectl与API server交互

- 案例：登录master节点，运行下面的命令调用接口
```bash
# 查询api版本
curl localhost:8080/api

# 返回结果json， kubernetes的API版本信息
{
 "kind" : "APIVersion",
 "version" : [
  "v1"
 ],
 "serverAddressByClientCIDs": [
 {
   "clientCIDR":"0.0.0.0/0",
   "serverAddress":"192.168.18.131:6443"
 }
 ]
}

# api server目前支持资源对象种类
curl localhost:8080/api/v1

# 返回集群的pod列表
curl localhost:8080/api/v1/pods

# 返回集群的service列表
curl localhost:8080/api/v1/services

# 返回集群的RC列表
curl localhost:8080/api/v1/replicationcontrollers

# 实现只暴露部分REST服务，在master节点，或者其他任何节点通过运行kubectl proxy进程启动一个内部代理来实现。
# 在8001端口启动代理，拒绝客户端访问RC的API

kubectl proxy --reject-paths="^/api/v1/replicationcontrollers" --port=8001 --v=2

# starting to serve on 127.0.0.1:8001

# 通过下面命令验证
curl localhost:8001/api/v1/replicationcontrollers

# 结果
unauthorized

# kubectl proxy 很多特性
- 简单的安全机制
- 采用白名单来限制非法客户端访问
kubectl proxy --accept-hosts="^localhost$,^127\\.0\\.0\\.1$,^\\[::1\\]$"

```
- 可以使用编程的方式调用API server的场景：
  - 1）运行在pod里的用户进程调用kubernetes api,实现分布式集群搭建的模板。
    - api server 本身也是一个service，名称为kubernetes
    - 通过命令：kubectl get service确认
    - api server 的cluster ip 地址是cluster IP地址池里面的第一个地址，
    - api server的端口是HTTPS端口443
  - 2）开发基于kubernetes的管理平台。
    - 完成Pod，service, RC等资源对象的图形化创建和管理。
    - 有各种语言的客户端

3.1.2 独特的kubernetes proxy API 接口
- 代理rest请求，即kubernetes API server 把收到的rest请求转发到某个Node上的kubelet守护进程的rest端口上，由这个kubelet进程负责响应
- kubernetes proxy API 关于node相关的接口，
```bash
# name:节点的名称或者IP地址
 /api/v1/proxy/nodes/{name}

# 列出节点的所有pod, 列出的信息来自于node不是etcd，所有又偏差。
 /api/v1/proxy/nodes/{name}/pods/

# 列出指定节点内物理资源的统计信息
 /api/v1/proxy/nodes/{name}/stats/

# 列出节点的概要信息
 /api/v1/proxy/nodes/{name}/spec/

```

- 如果kubelet进程在启动时，包含-enable-debugging-handkers=true, 代理接口会增加

```bash
# 在节点上运行某个容器
# 在节点
```

3.6 API server认证
