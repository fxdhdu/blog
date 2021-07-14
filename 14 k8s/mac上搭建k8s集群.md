kubectl 安装，可参考[在 macOS 系统上安装和设置 kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/#install-kubectl-binary-with-curl-on-macos)

要选择intel架构。不然报错./kubectl: Bad CPU type in executable 就装不下去了

```shell
kubectl version --client
```

[kubectl 文档](https://kubernetes.io/zh/docs/reference/kubectl/)



minikube安装 [Minikube - Kubernetes 本地实验环境](https://developer.aliyun.com/article/221687)

Minikube利用本地虚拟机环境部署Kubernetes

[Minikube 使用教程](https://kubernetes.io/zh/docs/tutorials/hello-minikube/)

启动

```shell
minikube start
```

打开Kubernetes控制台

```shell
minikube dashboard
```



```shell
minikube stop
```



[minikube github库](https://github.com/kubernetes/minikube?spm=a2c6h.12873639.0.0.ab202043DsffwB)



## 参考

VirtualBox https://www.virtualbox.org/

Ubuntu http://ftp.sjtu.edu.cn/ubuntu-cd/16.04/

https://www.cnblogs.com/andong2015/p/7688120.html