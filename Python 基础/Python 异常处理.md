# 异常处理

<!--感觉这个不常用。GPT：脚本、爬虫、后端、自动化，还是读文件、请求接口、连数据库 用到 -->

## 前言



异常处理能代替 `if` 逻辑判断？

能提前判断的，就先判断

不好判断的、可能出错的地方，再用异常处理



异常处理就是：程序出错后，不让它直接崩溃，而是按规则处理



## 常见异常类型

1.  `ZeroDivisionError` ：除以 0
2.  `ValueError` ：类型转换失败
3.  `NameError` ：变量没定义
4.  `TypeError` ：类型不匹配 
5.  `IndexError` ：下标越界
6.  `KeyError` ：字典里没有这个键
7.  `FilrNotFoundError` ：文件不存在





## 异常处理语法

```python
try:
    ...
except 某种异常:
    ...
```



## 多个 `except` 

```python
try:
    ...
except 某种异常:
    ...
    
except ValueError:
    
except ZeroDivisionError:
```



 ## `else` ：没出错时执行

```python
try:
    num = int(input("请输入一个数字："))
except ValueError:
    print("输入错误")
else:
    print("你输入的是：", num)
```



`try` 出错了 → 进 `except`

`try` 没出错 → 执行 `else`







##  `raise` ：主动抛出异常

```python
age = -1
if age < 0:
    raise ValueError("年龄不能是负数")
```

“这里的意思是：

> 我自己判断这个值不合法，所以主动报错

这个以后写函数时很有用。“