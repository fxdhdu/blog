[Kubernetes kubectl 命令表](http://docs.kubernetes.org.cn/683.html)

[kubectl 命令表](http://kubernetes.kansea.com/docs/user-guide/kubectl-overview/)



mirror pods

DaemonSet-managed pods



## kubectl run	

创建容器镜像

| 命令                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| kubectl run kubernetes-bootcamp --image=docker.io/jocatalin/kubernetes-bootcamp:v1 --port=8080 | 部署名字为 kubernetes-bootcamp 的应用，指定镜像和对外提供服务的端口 |
|                                                              |                                                              |
|                                                              |                                                              |
|                                                              |                                                              |
|                                                              |                                                              |
|                                                              |                                                              |
|                                                              |                                                              |



## kubectl create  

通过配置文件名或stdin创建一个集群资源对象。支持JSON和YAML格式的文件。

## kubectl expose 

将资源暴露为新的Kubernetes Service

| 命令                                                         | 说明                           |
| ------------------------------------------------------------ | ------------------------------ |
| kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080 | 将容器的 8080 端口映射到节点上 |
|                                                              |                                |
|                                                              |                                |
|                                                              |                                |
|                                                              |                                |
|                                                              |                                |
|                                                              |                                |



## kubectl get

获取列出一个或多个资源的信息。

### 语法

具体语法可以通过 kubectl get --help 查看，不同版本会有不同。

```tex
$ get 
[(-o|--output=)json|yaml|wide|custom-columns=...|custom-columns-file=...|go-template=...|go-template-file=...|jsonpath=...|jsonpath-file=...] 
(TYPE [NAME | -l label] | TYPE/NAME ...) 
[flags]
```

| 命令                              | 说明                                                         |
| --------------------------------- | ------------------------------------------------------------ |
| kubectl get pods                  | 列出所有运行的Pod信息                                        |
| kubectl get pods pod_name         |                                                              |
| kubectl get pods/pod_name         |                                                              |
| Kubectl get pods -o wide          | 新增 IP NODE 等字段                                          |
| kubectl get pods pod_name -o json | 以 json 格式输出指定 pod_name 的详细描述                     |
| kubectl get pods -l key=value     | 过滤出 label 中，带有 key: value 的 pods。label 信息可以通过 -o json 查看。 |
| kubectl get endpoints             |                                                              |
|                                   |                                                              |
|                                   |                                                              |

## kubectl drain

驱逐准备进行维护的node

指定的node 会被标记为 unschedulable，防止新的pods调度到这个node上。（kubectl uncordon 把node 标记为 schedulable）

然后会删除所有的pods，除了mirror pods，这种pods不能通过 api 删除。

| 命令                                                         | 说明                 |
| ------------------------------------------------------------ | -------------------- |
| Kubectl --kubeconfig=xxxx/kubectl.kubeconfig drain 10.0.0.11 --ignore-daemonsets --delete-local-data --force | --force 强制删除pods |
|                                                              |                      |
|                                                              |                      |

## kubectl scale

## kubectl rollout undo