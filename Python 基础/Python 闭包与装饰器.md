[toc]



# 闭包函数与装饰器



在函数部分，由函数名引出闭包的概念

当 **一个函数内部定义了另外一个函数** ，并且 **内部函数引用外层函数的变量** ，同时， **外部函数返回了内部函数** ，就形成一个闭包。







```python
def counter():

    count = 0

    def increment():

        nonlocal count
        count += 1

        return count

    return increment

add = counter()
print(add)
print(add())
print(add())
```

```python
<function counter.<locals>.increment at 0x0000018BE6F57270>
1
2
```

> 1. add 接收了返回值 count
>
> 2. 可以发现，使用闭包函数，可以“保持状态”





使用闭包函数访问网址信息，不用每次都去访问

```python
from urllib.request import urlopen

def func():
    content = urlopen('https://myip.ipip.net').read().decode('utf-8')
    def inner():
        print(content)
    return inner

f = func()
for i in range(5):
    f()

print(f.__code__.co_freevars)
```











## 装饰器

接下来是，Python 装饰器（Decorate）

装饰器可以让我们在不修改原有函数代码的情况，为函数**添加新的功能**。

> 这也是符合Python易维护的特点的，装饰器报错也不影响原有功能



```python
def fun1():
    print('in fun1')


fun1()
print('\n')

def fun2(god):

    def inner():
        print('in inner')
        god()
        print('hellp python')

    return inner

fun1 = fun2(fun1)
fun1()
```

```python
in fun1


in inner
in fun1
hellp python

```





这样写确实可以增加新功能，但每次想加新功能，都要写一遍

`fun1 = fun2(fun1)` 

所以为了更方便，更偷懒，Python提供了一种更简洁的语法来使用装饰器，即语法糖



## 语法糖

语法糖：使用“  `@`  ” 符号。



当在某个函数上方紧挨着加上 `@my_decorate` 的时候 ，Python 会将下面定义的函数作为参数，传递给  `my_decorate()` 。

```python
import time
def timer(decor):

    def inner():
        start  = time.time()
        decor()
        time.sleep(5)
        print(time.time() - start)

    return inner

@timer
def fun1():
    print('in fun1')
```

```python
in fun1
5.00016713142395
```





> 如果原函数有多个参数，或动态参数，装饰器也需要设计成，有相同数量的参数



---

 

## `warps` 装饰器

我们知道，经过装饰器 装饰的函数，已经不是原来的函数了，毕竟内存地址都不一样。

这里引入 `warps` 装饰器。 `warps` 装饰器用于保留原函数的元数据

```python
from functools import wraps
import time

def func1():
    print('in func1')

print(func1)

def timer(god):
    @wraps(god)
    def inner():
        start  =time.time()
        god()
        print(time.time()- start)

    return inner

func66 = timer(func1)
print(func66)
print(func66.__name__)
```

```python
<function func1 at 0x0000020D9CA3F1C0>
<function func1 at 0x0000020D9CABDC70>
func1
```

> 意料之中，内存地址变了；但函数名因为 wraps 没变。“这样有助于调试和文档生成。”





## 带参装饰器

灵活使用的装饰器：带参数的装饰器

让装饰器接受一个参数，用于指定是否启用装饰器

```python
import time
from functools import wraps

def timing_decorator(print_time=True):
    def decorator(god):
        @wraps(god)
        def wrapper(*args,**kwargs):
            start_time = time.time()
            result = god(*args,**kwargs)
            end_time =time.time()

            if print_time:
                exec_time = end_time- start_time
                print(f'{god.__name__} 执行时间：{exec_time:.4f}秒')
            return result
        return wrapper
    return decorator


@timing_decorator(print_time=True)
def add(x,y):
    time.sleep(2)
    return x+y

@timing_decorator(print_time=False)
def multiply(x,y):
    time.sleep(1)
    return x * y

result_add = add(5,3)
print(f'加法结果：{result_add}')
result_multiply = multiply(5,3)
print(f'乘法结果{result_multiply}')
```

```python
add 执行时间：2.0005秒
加法结果：8
乘法结果15
```

> 注意第 20 行和 25 行，这里的参数决定了是否启用装饰器。
>
> “需要使用嵌套函数。”







## 多个装饰器装饰一个函数

此时，装饰器会按照从内到外的顺序依次调用

```python
def wrapper1(god):
    def inner1():
        print('inner1前 ')
        god()
        print('inner1后')

    return inner1

def wrapper2(god):
    def inner2():
        print('inner2前')
        god()
        print('inner后')

    return inner2

@wrapper2
@wrapper1
def  f():
    print('真正的我')

f()
```

```python
inner2前
inner1前 
真正的我
inner1后
inner后
```

> 有种嵌套的感觉



## 案例



使用装饰器模拟登录状态

```python
def authenticate(original_function):

    def wrapper(*args,**kwargs):
        is_authenticate = False
        if is_authenticate:
            result=original_function(*args,**kwargs)

            return result

        else:
            print('Authentication failed')
            return None
    return wrapper

@authenticate
def protected_function():
    print('This is protected function')

protected_function()
```

> 当然，这段代码写死了，所以只是用来巩固装饰器的知识



现实项目开发

```python
from functools import wraps

# 模拟当前请求中的用户信息
request = {
    "user": {
        "username": "wenju",
        "is_authenticated": True
    }
}

def authenticate(original_function):
    @wraps(original_function)
    def wrapper(*args, **kwargs):
        user = request.get("user")

        if user and user.get("is_authenticated"):
            return original_function(*args, **kwargs)
        else:
            return "Authentication failed"
    return wrapper

@authenticate
def protected_function():
    return "This is protected function"

print(protected_function())
```

