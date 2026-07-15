

# 线程 



## 前言

前面说过，进程创建和切换开销大
那自然而然的引发需求：能不能让”房子里的的人“共享一套资源。
这就是线程：
**想多个任务同时推进，但又不想像进程那样开销这么大**

还记得之前的比喻
进程像房子，线程像在房子里装修的人
既然一起装修，就是“共享了这套房子”，能一起合作，也会“因为自己的进度而拖累别人”

所以说，成也共享，败也共享



假设写一个程序，要同时
* 下载文件
* 显示进度条

如果按时间顺序执行，就很奇怪
使用两个线程

* 一个线程负责下载
* 一个线程负责显示进度条
会更自然







## Python 线程

```python
import threading,time

start = time.time()
def task():
    print('子进程开始')
    time.sleep(5)
    print('子进程结束')

print('主进程开始')

t = threading.Thread(target = task)
t.start()
t.join()

end = time.time()
print(f'主进程结束，耗时{end-start}')
```

```python
主进程开始
子进程开始
子进程结束
主进程结束，耗时5.002299547195435
```

















## 为什么线程适合网络编程？

因为**当一个线程在等的时候，另一个线程还能继续推进**；

因为网络编程里经常会有大量等待

如：

* 网络请求
* 等服务器响应
* 等数据库返回



程序很多时候不是在计算，而是在等，这就叫 I/O 密集型



适合进程的场景：

* 很吃 CPU 的场景
* 大量数值计算
* 图像、视频处理

因为进程之间更独立，更容易利用多核 CPU 来并行处理任务







既然线程共享资源，那肯定很方便？

前面说，成也共享，败也共享。共享是方便，但也容易乱。

因为多个线程一起改同一份数据时，很容易出问题

所以后面还有  **锁** 



---

## 全局解释器锁

这是系统编译器自带的东西，而且不稳定不可靠，

解释说明

GIL 是“同一时刻只能有一个线程执行操作”，但是，线程A是可能随时被暂停，然后换线程B来执行

通俗来讲，就是线程A操作到一半，就被GIL拖出来，

也就是说 **GIL 释放时机是不可控的** 

所以开发者通常不能用业务逻辑安全来赌 GIL 的切换时机

---

## 锁

`Lock` 

锁：同一时刻只允许一个线程访问共享资源



演示

```python
import threading
import time

def work():
    global count
    with lock:
        for _ in range(1000):
            tmp = count
            tmp = count + 1
            time.sleep(0.001)
            count = tmp

if __name__ =='__main__':
    count = 0
    lock = threading.Lock()

    threads = [threading.Thread(target= work) for i in range(100)]

    for t in threads:
        t.start()

    for t in threads:
        t.join()

    print('预期是：100000')
    print(f'实际结果：{count}')
```

```python
预期是：100000
实际结果：100000
```

> 虽然运行了很久，但总归没错



看看没加锁的结果

```python
def work():
    global count
    #with lock:
    for _ in range(1000):
        tmp = count
        tmp = count + 1
        time.sleep(0.001)
        count = tmp

if __name__ =='__main__':
    count = 0
    #lock = threading.Lock()

    threads = [threading.Thread(target= work) for i in range(100)]

    for t in threads:
        t.start()

    for t in threads:
        t.join()

    print('预期是：100000')
    print(f'实际结果：{count}')
```

```python
预期是：100000
实际结果：1019
```

> 确实会发生竞态，数据错乱。这就是不加锁的下场







## 可重入锁

如果多个线程相互等待对方释放资源（循环、递归），导致大家都被卡住的情况，就称为死锁

可重入锁就是用来解决这种情况的

`threading.RLock()` 

> 代码只要把  `threading.Lock()` 换成 `threading.RLock()` 就行了





## 事件 `Event` 

 `Event` 就像一个信号开关

比如，有一个线程 A 先准备数据，其他线程 必须等线程 A 发送“准备好”的通知，才能继续执行



## 信号量 Semaphore 

信号量：同一时刻最多 N 个线程



适合场景：

最多允许 3 个线程同时下载

最多允许 5 个线程同时访问某个资源





## 消息队列 Queue

安全又好用

==现实开发建议：**尽量用 Queue 传数据**==，别手搓共享变量 + 锁