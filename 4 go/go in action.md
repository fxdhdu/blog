[TOC]

# Go 基础

## 基本概念

### 并发

#### goroutine

goroutine是可以与其他goroutine并行执行的函数，也和主程序并行执行。很像线程，但占用内存少于线程，使用更简单。

```go
func log(msg String) {

}

go log
```

#### 通道

通道是一种数据结构。用于在几个运行的goroutine之间发送数据。保证同一时刻只有一个goroutine修改数据。不提供跨goroutine的数据访问保护机制。

### 类型系统

### 内存管理