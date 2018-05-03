
---
title: "Golang channel 缓存与没有缓存的区别"
date: 2018-05-02T23:16:10+08:00
tags : ["golang", "channel"]
categories : ["Golang"]
---

#Golang channel 缓存与没有缓存的区别

之前一直对golang的channel有一个疑问，带不带缓存有什么样的区别？

在这里用一个简单的代码来记录下这个区别：
### 不带缓存

```
package main

import "fmt"
import "time"

func gostring(c chan []byte) {
	time.Sleep(5 * time.Second)   //注释掉这一行运行看看差别
	c <- []byte("i am done") 
}
func main() {
	c := make(chan []byte)
	go gostring(c)
	s := <-c // receive from c
	fmt.Println(string(s))
}

```

我在代码里加了time来模拟一些延时操作，运行结果可以看到，main函数在gostring函数没有返回之前一直处于阻塞的状态。只有双方都准备好了之后，程序才能继续走下去。


### 带缓存
```
package main

import "fmt"

func gonum(c chan int) {
	c <- 1

	fmt.Println("send 1")

	c <- 2
	fmt.Println("send 2")

	c <- 3
	fmt.Println("send 3")

	close(c)
}

func main() {
	c := make(chan int, 3)   // 把这里改为(chan int ,1 ) 试试
	go gonum(c)

	// for s := range c {
	// 	fmt.Println(s)
	// }

	fmt.Println("一直等着你...")
	for {

	}

}
```
简单总结： 当chan设置为1，当缓存区满了的时候，gonum会发生阻塞，此时的send 2 和 send 3 都发生了阻塞。但当你调整了chan为3的时候，可以看到，1 2 3都可以被塞进缓存区，等待接受。


