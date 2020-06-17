---
title: Centos 7 运行Java程序
date: 2020-06-04 18:53:22
tags: Linux
---
* 在当前窗口运行，会锁定 ssh 窗口，Ctrl + C 或者 关闭窗口可以中断运行
```
java -jar test.jar 
```
* 在后台运行，不锁定 ssh 窗口,关闭窗口和退出账号时会中断运行
```
java -jar test.jar &
```
* 后台运行，不锁定 ssh 窗口，关闭窗口和退出账号都不会中断运行
```
nohup java -jar test.jar &
```
* 后台运行,且将日志输出到指定文件
```
nohup java -jar test.jar & > spring.out
```

* 如何关闭后台运行的 Java 程序
``` 
--查看后台运行的进程
ps -aux
--关闭指定进程
kill [pid]
```


