---
title: Go语言锁初探(一)
date: 2019-06-21 16:36:15
tags: Go基础知识
categories: Go
---

主要介绍Go语言中，协程同步与Go语言中的锁

<!-- more -->

# go语言锁机制

1. Cond  
    - Singal: 仅唤醒一个等待中的协程
    - NewCond: 返回一个新的条件
    - Broadcast: 广播，通知所有等待此条件的协程
    - Wait: 等待被Broadcast或Signal唤醒
2. Locker  
    - Lock: 锁对象
    - Unlock: 解锁
3. Map(安全字典)
    - Delete: 删除key
    - Load: 获取key
    - LoadOrStore: 如果有就返回该key值，没有就设置并返回传入值
    - Range: 遍历
    - Store: 存储
4. Mutex(互斥锁)
    - Lock
    - UnLock
5. Once
    - Do(f func()): 仅执行一次，即只有第一次会被唤醒，如果执行过程中出错，后续执行不会有任何返回值
6. Pool(队列)  
    - Get
    - Put
7. RWMutex(读写锁)  
    - Lock
    - Rlock
    - RLocker
    - RUnlock
    - Unlock
8. WaitGroup(协程等待队列)
    - Add
    - Done
    - Wait