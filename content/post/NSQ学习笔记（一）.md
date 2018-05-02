---
title: "NSQ学习笔记（一）"
date: 2018-05-02T23:16:10+08:00
tags : ["golang", "nsq","学习笔记"]
categories : ["NSQ","Golang"]
---

# NSQ学习笔记（一）
_____
最近需要设想为我的开源项目[yeetiku-react-native](https://github.com/yeelone/yeetiku-mobile-rn)架一个消息系统，因为我的后端使用的是golang [yeetiku-server](https://github.com/yeelone/yeetikuserver), 所以我自然就想到了nsq,一个基于Go语言的分布式实时消息平台。

以此文章来记录我的NSQ学习笔记。
_____

## 一 安装


[>>>>> 官方安装教程  NSQ Install <<<<<<](https://nsq.io/deployment/installing.html)

由于我使用的是MAC ，所以安装如下:

```
$ brew install nsq
```


NSQ的介绍看官网就足够，在此不多说了。具体写下自己的实践过程。

但几个重要的功能有必要记录下来：

## 一、组件

### 1、***nsqlookup*** 
  nsqlookupd就是中心管理服务，它使用tcp(默认端口4160)管理nsqd服务，使用http(默认端口4161)管理nsqadmin服务。同时为客户端提供查询功能

```
    启动方式：➜  ~ nsqlookupd 
```
### 2、***nsqd*** 
  nsqd 是一个守护进程，负责接收，排队，投递消息给客户端

主要负责message的收发，队列的维护。nsqd会默认监听一个tcp端口(4150)和一个http端口(4151)以及一个可选的https端口

```
	启动方式: ➜  ~ nsqd --lookupd-tcp-address=127.0.0.1:4160
```

### 3、***nsqadmin*** 
 很好理解，就是一个WEB UI 管理平台

```
	启动方式: ➜  ~ nsqadmin --lookupd-http-address=127.0.0.1:4161
```

## 二、概念

### 1、 topic 

一个topic就是程序发布消息的一个逻辑键，当程序第一次发布消息时就会创建topic

### 2、 channel 
channel组与消费者相关，是消费者之间的负载均衡，channel在某种意义上来说是一个“队列”。每当一个发布者发送一条消息到一个topic，消息会被复制到所有消费者连接的channel上，消费者通过这个特殊的channel读取消息，实际上，在消费者第一次订阅时就会创建channel。

## 三 测试

具体测试方式，请参考 [官方文档 Quick Start](https://nsq.io/overview/quick_start.html) 


记录一下自己的过程：

```
打开 shell 1 :  ~ nsqlookupd
打开 shell 2 :  ~ nsqd --lookupd-tcp-address=127.0.0.1:4160
打开 shell 3 :  ~ nsqadmin --lookupd-http-address=127.0.0.1:4161
```

打开 http://127.0.0.1:4171/ 

![image](https://wx4.sinaimg.cn/mw690/6547935dgy1fqxd6a7311j20mn0gmjsb.jpg)

创建topic & channel 

![image](https://wx3.sinaimg.cn/mw690/6547935dgy1fqxd8pl1ytj20f70gft9v.jpg)


```
测试发送消息
➜  curl -d 'hello nsq' 'http://127.0.0.1:4151/pub?topic=test&channel=test_channel'

我发送了6条
```

![image](https://wx4.sinaimg.cn/mw690/6547935dgy1fqxdd7iviij20rh0erabq.jpg)


## 四 测试客户端代码实现
```
   go get -v -u github.com/nsqio/go-nsq
```

```
package main

import (
	"log"

	"github.com/nsqio/go-nsq"
)

func main() {

	go startConsumer()
	for {

	}
	//startProducer()
}

// 生产者
func startProducer() {
	// cfg := nsq.NewConfig()
	// producer, err := nsq.NewProducer("127.0.0.1:4150", cfg)
	// if err != nil {
	//  log.Fatal(err)
	// }
	// // 发布消息
	// for {
	//  if err := producer.Publish("test", []byte("test message")); err != nil {
	//      log.Fatal("publish error: " + err.Error())
	//  }
	//  time.Sleep(1 * time.Second)
	// }
}

// 消费者
func startConsumer() {
	cfg := nsq.NewConfig()
	consumer, err := nsq.NewConsumer("test", "test_channel", cfg)
	if err != nil {
		log.Fatal(err)
	}
	// 设置消息处理函数
	consumer.AddHandler(nsq.HandlerFunc(func(message *nsq.Message) error {
		log.Println("message body :" + string(message.Body))
		return nil
	}))
	// 连接到单例nsqd
	if err := consumer.ConnectToNSQD("127.0.0.1:4150"); err != nil {
		log.Fatal(err)
	}
	<-consumer.StopChan
}

```

运行程序，并同时在终端中发送消息：

```
➜  nsqdata curl -d 'hello nsq ' 'http://127.0.0.1:4151/pub?topic=test&channel=test_channel'
```

可以观察到消息都被消费掉了.

***注意：如果存在消费者，则消息会马上传递给消费者，nsqd服务器不会将消息存储于内存或硬盘，而如果此时没有消息者，此消息会存储于内存或硬盘中，可以在nsqdadmin中看到数据存储于Memory + Disk中***

