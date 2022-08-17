---
title: golang关于signal和context
date: 2022-04-22 17:00:00
updated: 2022-04-22 17:00:00
categories: golang
---

希望有一个程序被kill后依然能执行完执行到一半的任务，新的任务在收到signal后不继续执行，然后退出
首先完成
``` 
package main

import (
	"fmt"
	"os"
	"os/signal"
	"syscall"
	"time"
)

var c = make(chan os.Signal)

func init() {
	//监听指定信号 ctrl+c kill
	signal.Notify(c)
}
func main() {
	go func() {
		for s := range c {
			switch s {
			case syscall.SIGHUP, syscall.SIGINT, syscall.SIGTERM, syscall.SIGQUIT:
				fmt.Println("Exit...")
				os.Exit(0)
			default:
				fmt.Println("other", s)
			}
		}
	}()
	for id := 1; id <= 2; id++ {
		handler(id)
	}
}
func handler(id int) {
	for i := 1; i <= 5; i++ {
		fmt.Println("data:", id*10+i)
		time.Sleep(time.Millisecond * 500)
	}
}

```
执行后正常不退出会输出
```
data: 11
data: 12
data: 13
data: 14
data: 15
data: 21
data: 22
data: 23
data: 24
data: 25
```
执行到一半是kill 或者 ctrl+c 会输出
```
data: 11
data: 12
data: 13
^CExit...
```
程序马上就退出了，
