

# Python 中的流程控制

[toc] 



## 判断语句



### 单分支判断

```python
if 条件 :
    满足条件后执行的代码
```



### 双分支判断

```python
if 条件 :
    满足条件后执行的代码
    
else :
    不满足条件执行的代码
    
```



### 多分支判断

```python
if 条件 :
    满足条件执行的代码
    
elif 条件1 :
    满足条件 1，执行的代码
   
elif 条件2 :
    满足条件 2，执行的代码
    
else :
    上面所有条件都不满足，执行此代码
```





## 循环语句



### `while` ：

```python
while 条件表达式 :
    循环体
```

>  循环条件可以直接是True/False或者1/0，也可以是某个语句（表达式）...



```python
while 条件:
    循环体
    
    if 其他条件:
        break
        
    else:
```



 `break`  语句在执行时将终止循环且不执行 `else` 子句体。  `continue` 语句在执行时将跳过子句体中的剩余部分并返回检验表达式。







### ==`for`==：

用于对序列（例如字符串、元组或列表）或其他可迭代对象中的元素进行迭代。

```python
for variable in iterable:
    执行代码
```

> for 循环会对目标列表中的变量进行赋值。 这将覆盖之前对这些变量的所有赋值，包括在 for 循环体中的赋值





*  `range`

生成指定范围数字

```python
for i in range(1,7):
    print(i)
```

> 取不到 end 的数

```python
1
2
3
4
5
6
```

规定步长

```python
for i in range(1,10,2): 
    print(i)
```





*  `enumerate` 

`enumerate()` 函数用于将一个可遍历的数据对象（如列表、元组或字符串），组合为一个索引序列，同时列出数据和数据下标。

```python
list1 = [1,2,3,6,7,8,9]
for i in enumerate(list1):
    print(i)
```




```python
(0, 1)
(1, 2)
(2, 3)
(3, 6)
(4, 7)
(5, 8)
(6, 9)
```

