[toc] 



# 迭代器

诶！我们之前学了 Python 的数据类型，有个“不可变数据类型”，可能会和这里混淆，所以说一下

* ”可变“：能不能改内容
* “可迭代”：能不能一个一个地把元素取出来

像 **`字符串、元组`** ，就是可迭代，数据可以一个一个拿出来，但是不可以改变其中内容。

---



## 可迭代对象

迭代：使用 `for i in xxx` 进行循环取值的过程称为迭代，也叫遍历。

可迭代对象，是指可以使用 `for` 循环遍历其元素的对象。

可迭代对象有：字符串，列表、元组、字典、集合、序列。

```python
li = [1,2,3,'a','b']
for i in li:
    print(i)

str1 = 'hello'
for j in str1:
    print(j)

dic = {'name':'wen_ju','age':18,'gender':'female'}
for k in dic.items():
    print(k)
```

```python
1
2
3
a
b
h
e
l
l
o
('name', 'wen_ju')
('age', 18)
('gender', 'female')
```



## 迭代器

上面讲了可迭代对象，它们是 *能被 for 遍历的东西*

下面讲迭代器。

迭代器才是实现 遍历 的东西

> 迭代器里面有个记录索引的方法 `next` ，每次取完，就让索引指向下一个，下次读取，就可以读到下一个



可迭代对象不能直接一个一个取，必须先变成迭代器

> GPT的类比：
>
> “Iterable = 一本书 📖
>
> Iterator = 你翻书的手 ✋
>
> 书本身可以被读（iterable）
>
> 但真正一页一页翻的是“手”（iterator）

 `Iterable --(iter)--> Iterator --(next)--> 元素` 







## 迭代器协议

虽然可迭代对象就这么几种类型

我们不需要知道什么协议，但要知道，可迭代 ≠ 类型，而是一种能力

一言以蔽之，什么是协议：**如果一个对象能提供 “一个一个取值的方法”，它就能被 for** 





---

## 生成器 `yield` 

```python
def fun():
    print('hello')
    yield 1
    print('python')
    yield 2

y=fun()
next(y)
print('---')
next(y)
```

```python
hello
---
python
```



 `yield` 可以暂停函数，下次从这里继续。函数不是重新执行，而是接着上次继续



生成器很实用的一点是，只在你需要的时候生成元素，节省内存。

在处理大量数据是有用。

```python
def infinite():
    i = 1
    while True:
        yield i
        i += 1
        print(i)

n = infinite()
next(n)
next(n)
next(n)
```

```python
2
3
```



迭代器的惰性求值

迭代器的惰性求值意味着它只在需要时才计算和返回元素，而不是一次性计算所有元素。这在处理大量数据时非常有用，可以节省内存。
