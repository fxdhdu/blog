[TOC]

## 知识点

- Borg：谷歌内部使用大规模集群管理系统
- Kubernetes：Borg的一个开源版本。基于Docker的大规模容器化分布式系统解决方案。
- 微服务架构：拆分、连接，有哪些基础设施。
- CNCF/Cloud Native Computing Foundation，云原生计算基金会
- Kubernetes拥有的能力：
  - 集群管理
  - 负载均衡
  - 故障恢复
  - 横向扩容/Node的扩容
- kubeadm：快速安装Kubernetes集群。
- 容器：隔离功能
- Docker
- etcd：持久化存储资源对象（资源期望状态）
- HPA（Horizontal Pod Autoscaler）
  - 缩容冷却机制（cooldown delay）：
  - Pod的metrics
- GA版本（General Availability）：https://blog.csdn.net/gnail_oug/article/details/79998154
- kubectl:  命令行工具



## 资源对象

- Service：Cluster IP对外提供服务。其他Pod通过服务发现机制（Linux环境变量）找到这个Service。
- Pod：Kubernetes管理的最小运行单元。服务进程放入容器中，容器放入Pod中。Pod运行在Node上。
- Pause容器：在每个Pod中运行，业务容器共享其网络栈和Volume挂载卷。
- 标签选择器：给Service定义标签选择器，给Pod贴上标签，就可以将Service和Pod关联了。
- Node：集群工作节点。可以是物理机、虚拟机，通常可运行几百个Pod。运行kubelet、kube-proxy，负责Pod创建、启动、监控、重启、销毁、软件模式负载均衡。
  - kubelet
- Master：集群管理节点，每个Kubernetes集群里都需要一个Master，来负责整个集群的管理和控制。独立服务器部署（高可用需要3台）。运行以下关键进程。实现集群的资源管理、Pod调度、弹性伸缩、安全控制、系统监控和纠错管理。
  - kube-apiserver：提供Http Rest接口对所有资源增、删、改、查。集群控制的入口。
  - kube-controller-manager：Kubernetes里所有资源对象的自动化控制中心
  - kubescheduler：负责资源调度（Pod 调度）的进程
  - etcd：保存资源对象的数据
- RC（Replication Controller，副本控制器）：用于服务扩容。扩容通常涉及资源分配（选择节点）、实例部署、启动。
  - Pod模版定义：用于创建新Pod
  - Pod副本数量
  - 要监控的目标Pod标签：Kubernetes用于筛选Pod并进行监控。要和Pod模版定义中的标签一致。
- apiVersion
  - v1 核心API
  - extensions/v1beta1 淘汰
  - apps/v1beta1 淘汰
  - apps/v1beta2 淘汰
  - apps/v1  1.9版本之后引入的正式扩展API组
- Annotations：类似数据库表里的备注字段。可用于实现新特性。比如Init Container一开始是在Annotations中声明的。







## 新增特性时数据库表怎么修改的最佳实践

![A42AE5B4-AED2-4921-AC3E-B0BB4CAF5713](/Users/fanxudong/IdeaProjects/blog/14 k8s/asset/A42AE5B4-AED2-4921-AC3E-B0BB4CAF5713.png)



## 第三章

