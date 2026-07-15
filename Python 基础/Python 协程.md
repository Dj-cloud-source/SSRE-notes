









# 协程



## 协程出现背景

虽然线程已经比进程轻了，但还是会有成本

* 共享资源竞争
* 锁



而且很多网络程序都在等数据返回。

“既然大部分时间都在等，那我不想开这么多线程，而是一个线程灵活调用”

所以，协程更像是个 **时间管理大师**

协程不是“多个人同时干活”，而是“一个人，**把等待时间利用得很好**”



## 区别

普通函数可能一路执行到底

而协程能执行到一半，暂停挂起，以后还能从暂停的地方继续



## 适用场景

协程适合 I/O 密集型任务，这些任务最经典的特点是：任务经常在等

因为协程的特点是一个任务在等时，立刻切去进行别的任务，而IO密集型经常需要等，所以协程适合IO密集型任务







## 协程基本语法

1.  `async` ：定义协程函数

```python
async def task():
    print('我是协程')
```

>  `async def` 定义出来的，不是普通函数，而是协程函数。
>
> 不会像普通函数一样调用、直接执行。这只是创建了一个待执行的协程对象，还没有真正跑起来。！协程函数调用后，不会像普通函数一样立刻执行！



2. `await` ：先让出去，让别的协程先跑，等时间到了再回来继续

```python
	await async.sleep(1)
```



> 这个和普通的 sleep 不一样。
>
> 且只能出现在 async def 定义的函数体里面；普通函数不能写 await 。
>
> 即：async 和 await 经常一起出现



既然协程函数直接调用，不会自动执行，那就需要一个东西让它”动起来“



3.  `async.run()`

协程的启动方式



完整例子

```python
import async

async def task():
    print('1')
    await async.sleep(1)
    print('2')
    
async.run(task())
```





好，一个协程的完整案例学会了，那多协程呢？那就需要专人来负责协程的调度了



## 事件循环

事件循环是专门负责调度协程的“调度员”，

由它来决定“哪个协程先跑、哪个暂停、哪个恢复”





上面的 await 是主动让出执行权，而事件循环则是负责接管执行权，再重新分配



案例

```python
import asyncio
async def task1():
    print('task1 start')
    await async.sleep(1)
    print('task1 end')
    
async def task2():
    print('task2 start')
    await async.sleep(1)
    print('task2 end')
    
async def main():
    await asyncio.gather(task1(),task2())
    
if __name__='__main__':
	asyncio.run(main())

```











