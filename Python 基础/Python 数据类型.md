# Python 数据类型

$因为Python的数据类型和JSON关联很大，而JSON又是应用非常广泛的数据格式，所以学好数据类型非常重要$

[toc] 





## 列表

列表是可变序列，通常用于存放同类项目的集合

> 可变序列 就是可以通过后面的 点方法 修改内容。可变不可变关乎能不能哈希化

### 创建列表对象

```python
fruits = ['banana','apple']
print(fruits)
```



### 添加元素

* 按索引位置添加

```python
fruits.insert(1,'watermelon')
```

* 在末尾添加

```python
fruits.append('orange')
```

* 用迭代器的方式添加（同类型的数据）

```python
fruits.extend( ['kiwi','mango'])
```



### 删除元素

* 删除指定元素

```python
fruits.remove('apple')
```

* 按索引删除

```python
fruits.pop(4)
```

* 全删

```python
del fruits[1:3] #使用切片范围删除

fruits.clear()

del fruits
```



### 修改元素

* 按索引修改

```python
fruits[1] = 'apple'
```

* 按照切片范围修改

```python
fruits[0:2] = ['kiwi','orange']
```



### 查找元素

* 以返回索引的方式查找

```python
a = fruits.index('apple')
print(a)
```

* 返回元素出现次数

```python
b = fruits.count('apple')
print(b)
```

> 虽然但是，还是不要使用这种无意义的变量名



### 切片拿出

```python
lis = [9,8,7,6,6,5,4,3]
sub_lis = [3:5]
print(sub_lis)
```



### 统计元素

```python
print(lis.count(6))
```



### 排序

```python
lis.sort()
print(lis)

lis.reverse()
print(lis)

```





---



## 元组

元组对象创建出来后，只能读取，不能修改



```python
tuple1 = (1,2,3,'a','b','5')
print(tuple1)
```





可变元组

> tuple 不可变指的是，元组对象创建出来后，内存地址就固定了。但内存里存了指向其他地址，那就可以存储可变类型

```python
tuple1 = (1,2,3,'a','6',[9,8,7,6])

print(tuple1)
print(tuple1[5][2])
```

```python
(1, 2, 3, 'a', '6', [9, 8, 7, 6])
7
```

修改数据


```python
tuple1([5][2]) = 100
print(tuple)
……
>>(1, 2, 3, 'a', '6', [9, 8, 100, 6])
```





## 字典

$前面的元组和列表，想要查找数据，就必须要记住索引。但字典不用，想查哪个数据，直接搜 键 key 就能查到。$

### 字典底层原理



字典底层不是数组，而是使用一种结构

> 哈希表 （Hash Table）

``` 
key → hash 函数 → 内存地址 → value
```

每个 *键 key* 对应一个 *哈希值* ，每个 *哈希值* 对应一个内存地址，内存中存放了 *值 value* 。

这样，查找速度就是是 `O(1)` ，所以 Python 字典用途广泛。



因为 ”**哈希函数**“  这个东西会生成一长串、不可变的 哈希值 ，所以就要求 *键 key* 也是不能变的。

1. **key 必须唯一**

```python
dic1 ={
    'name' = '小红'
    'name' = '小明'
}

ptint(dic1['name'])
```

```python
小明
```

前面的会被覆盖！



2. **key 必须是不可变类型** 

​	可以：

* str
* int
* tuple



​	不可以：

* list
* dict



因为哈希表需要 *键 key* 不会改变



3. **value 可以是任意数据类型**



---





### 增加键值

直接通过键值来添加

```python
dic1['gender']='male'
```



### 删除指定键值数据

```python
del dic1['name']
```

清空字典

```python
dic1.clear()
```





### 修改数据

```python
dic1['age'] = 19 
```





### 查找键值数据

使用 get 方法获取 value，若不存在则返回 None ，可自定义异常返回值

```python
value1 = dic1.get('name','查无此项')
print(value1)
```



使用keys()和values()方法获取键值

```python

keys = dic.keys()
print(keys)

values = dic.values()
print(values)
```



### 遍历字典

以元组的形式，同时输出键值对

```python
for i in dic1.items():
    print(i)
```



只迭代 键 key

```python
for i in dic1:
    print(i)
```







## 集合



Python 中的集合（set）具有数学中集合的性质：无序性、唯一性、确定性

基于以上性质，我们就可以知道集合是否可以哈希化：

里面的元素是可哈希的（确定性）

但集合本身不可哈希



### 定义集合

```python
set1  = {1,2,3,'a','b'}
print(set1)
```

> 因为集合具有无序性，所以每次打印，输出顺序都不一样



### 增加元素

使用 add() 方法

```python
set1.add('d')
print(set1)
```

使用 update() 方法迭代的去添加

```python
set1.update(6,'y')  xxx
```

> update接收的参数应该是可迭代的数据类型，比如字符串、元组、列表、集合、字典。这些都可以向集合中添加元素，但是整型、浮点型不可以
>
> 所以这里 6 会报错

```python
set1.update('y')
```

```python
set1.update([6,7])
```





### 删除元素

使用 remove() 方法删除元素

```python
set1.remove('a')
```



删除集合

```python
del set1
```



### 查找元素

```python
set1 = {1,2,3,'a','b','c'}
exists = "a" in set1  
print(exists)
```

  

