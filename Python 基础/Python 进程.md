

## 进程

[toc]







### 详细介绍

程序是静态的，躺在桌面的微信是程序

双击打开的微信，运行起来，就成了进程



每次打开微信，系统都要把它从硬盘里拿出来，放到内存，分配一套完整、独立的运行环境 
**所以说进程的创建和切换开销大！**




### 进程三个特点

1. 进程是独立的

	* 一个进程通常有自己独立的内存空间。比如QQ卡死并不会把微信一起带崩




2. 进程是 OS 调度的对象



3. 进程之间彼此独立



### 进程和线程的区别

进程就像一个完整的房子

线程就像房子里装修干活的人

（这么说进程实际上并不干活（！），只是负责申请一个地方，然后把活丢给下面的线程干）





### Python 进程



使用 `multiprocessing` 模块创建进程

```python
from multiprocessing import Process
import os

def work():
    print('子进程正在运行')
    print(f'子进程PID：{os.getpid()}')

if __name__ == '__main__' :
    p = Process(target=work)
    p.start()
    p.join()

    print('主进程结束')
    print(f'主进程PID：{os.getpid()}')

#work()
```

```python
子进程正在运行
子进程PID：21900
主进程结束
主进程PID：5848
```





1.  `Process` ：创建一个进程对象

```python
p = Process(target = work)
```

> **创建一个子进程** ，让它去执行 `work` 函数



2. `start()` ：启动这个子进程

```python
p.start()
```



3. `join()` ：等待子进程结束

```python
p.join()
```

>  join 会阻塞主进程，等待子进程结束后，主进程才能继续执行



4. 主进程

```python
if __name__ = '__main__':
```

> 主进程是这下面的所有代码

**写进程时，必须加上[这段代码]( '在Windows操作系统中由于没有fork(linux操作系统中创建进程的机制)，在创建子进程的时候会自动 import 启动它的这个文件，而在 import 的时候又执行了整个文件。因此如果将 process() 直接写在文件中就会无限递归创建子进程报错。所以必须把创建子进程的部分使用 `if __name__ =='__main__'` 判断保护起来，import 的时候，就不会递归运行了。')。当成多进程代码的标准写法**



---









### 进程间通信



IPC  (Inter-Process-Communication) 进程间通信  





为什么要搞进程间通信？

我们在前面提到，OS 将各个进程的内存空间严格隔离开。

举个例子，你把工厂的工人关在不同且完全封闭的空间，虽然这样很安全，但如果他们需要合作，就无法直接把东西递给对方。

使用进程间通信，就能实现共享数据，协作、高效地完成任务。



IPC 三种思路

1. Queue 队列
2. Pipe 管道
3. Shared Memory 共享内存





#### Queue



```python
from multiprocessing import Process,Queue

def worker(arg):
    arg.put('111我是子进程')

if __name__ == '__main__':
    arg = Queue()

    p = Process(target = worker,args = (arg,))
    p.start()

    msg = arg.get()
    print(f'主进程收到：{msg}')
    p.join()
```

```python
主进程收到：111我是子进程
```



1.  `arg = Queue()` ：创建进程共享队列
	* 可以想象成一个跨进程通道



2.  `args=(arg,)` ：把队列传给子进程



3.  `arg.put()` ：子进程把数据**放**进队列里
4.  `msg = arg.get()` ：主进程从队列里把数据**取**出来



Queue 是较简单的 IPC 工具，

常用于：

子进程把结果返回主进程

多个”消费者“放任务

多个“消费者”取任务











#### Pipe 

```python
from multiprocessing import Process,Pipe

def woker(arg):
    arg.send('666')
    #print(arg.recv())  #如果在这里加了recv就会造成阻塞🤔🤔
    arg.close()

if __name__ == '__main__':
    parent_conn,child_conn = Pipe()

    p = Process(target = woker,args =(child_conn,))
    p.start()
    
    msg = parent_conn.recv()
    print(msg)

    p.join()
```



Pipe 是两个端点直接通信

`Pipe()` 直接连接了`parent_conn` 和 `child_conn` 两端 

子进程 send ，主进程 recv 就完成了一次通信











共享内存（需求场景）

* 大数组
* 大量二进制数据
* 想高性能







#### 进程池

进程池就是把进程重复拿来干活。

任务来了，就分给它们干，谁干完了就继续接着干下一个任务

普通进程需要手动创建和管理

而进程池，可以让系统帮你批量管理一堆进程，你只负责给任务





进程池示例

```python
import os,time
from multiprocessing import Process,Pool

def task(n):
    print(f'任务{n}在进程{os.getpid()}中运行')
    time.sleep(1)
    return n * n


if __name__ == '__main__':
    pool =Pool(3)
    result = pool.map(task,[1,3,5,7,9])

    pool.close()
    pool.join()
    print(result)
```

```python
任务1在进程10984中运行
任务3在进程23244中运行
任务5在进程22744中运行
任务7在进程10984中运行
任务9在进程23244中运行
[1, 9, 25, 49, 81]
```







```python
pool = pool(3)
```

> 创建一个进程池，里面放 3 个进程
>
> 同一时刻让 3 个进程并发干活





```python
result = pool.map(task,[1,3,5,7,9])
```

> map()：把列表里的元素，交给 task 处理（给进程池并发处理）



```python
pool.close()
```

> 不再往进程池添加新任务





```python
pool.join()
```

> 等进程池里的任务执行完









#### 异步提交任务

> 因为 同步提交 太垃圾了，现实开发几乎不用，不讲了



异步： **主进程给完任务就接着执行代码，不等**，主进程的代码会瞬间往下执行到底



````python
if __name__ == '__main__':
    pool =Pool(3)

    result = []
    for i in range(7):
        r = pool.apply_async(task,args=(i,))
        result.append(r)

    pool.close()
    pool.join()

    for r in result:
        #print(result)
        print(r.get())

````

```python
D:\Python_workplace\Python_coding\.venv\Scripts\python.exe D:\Python_workplace\Python_coding\清明\test5.py 
任务0在进程22676中运行
任务1在进程13264中运行
任务2在进程24244中运行
任务3在进程22676中运行
任务4在进程13264中运行
任务5在进程24244中运行
任务6在进程22676中运行
0
1
4
9
16
25
36
```







当然，异步提交任务，只管提交，不管收集结果

所以还要 回调函数 来收集结果



#### 回调函数



回调函数就是，当子进程完成任务后，**自动**触发收尾的函数

回调函数是在主进程里执行的，不是在子进程里

回调函数接收的是目标函数的返回值





为什么要用回调函数？

前面不用回调，得手写，而且还要等执行完才能处理。而回调可以实时处理结果，只要处理好了，它就自动拿走，不用等活全部干完了才处理



演示

```python
if __name__='__name__':
    pool = Pool(3)
    
    for i in range(4):
        pool.apply_async(task,args(i,),callback = 想执行的函数)
```







#### 解释一些概念

阻塞和非阻塞：

* 阻塞：这一步没结果，就卡在这不走
* 非阻塞：这一步没结果，我先不等，我先去干别的

阻塞常在IO操作出现（和异步区别开）

前面学 socket 时

`recv` 和 `accept` 就是阻塞





课件案例动手