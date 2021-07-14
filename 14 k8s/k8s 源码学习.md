## GoLang 导入 k8s 源码

下载源码


```shell
mkdir -p $GOPATH/src/k8s.io
cd $GOPATH/src/k8s.io
git clone https://github.com/kubernetes/kubernetes
git clone git@github.com:kubernetes/api.git
```

创建工程

1. 打开 GoLand
2. 选择 New Project
3. 将目标文件夹指向 $GOPATH
4. 确认之后会提示文件夹不为空，是否继续，点击确定就行
5. 慢慢等待 IDE 完成对源代码的索引。
6. 设置导入源码的 Project GOPATH 为当前路径 $GOPATH/src。路径为preferences->Go->GOPATH。



## 参考：

[Golang-goland导入k8s源码](https://blog.csdn.net/yilan1993/article/details/106575852)

[github](https://github.com/kubernetes/kubernetes)
