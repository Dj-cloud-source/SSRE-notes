



[toc]



# Python 推导式

实例演示

```python
L = [i  for i in range(11)]
print(L)

s = [x ** 2  for x in range(8)]
print(s)

even_number = [i  for i in range(30)    if i % 2 ==0]
print(even_number)
```

```python
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
[0, 1, 4, 9, 16, 25, 36, 49]
[0, 2, 4, 6, 8, 10, 12, 14, 16, 18, 20, 22, 24, 26, 28]
```



> 推导式可以让开发者以简洁且高效的方式创建列表、集合、字典等数据结构。通过推导式，我们可以减少代码量，提高代码可读性和执行效率







## 基本语法

### 列表推导式

基本语法

`L = [argumen_expression    for item in iterable   if conditon] ` 

 

### 集合推导式

基本语法和列表一样，不过自带去重功能



### 字典推导式

基本语法

 `dic = {key_expression : value_expression  for item in iterable     if condition}`



```python
even_sqrt_dic = {i:i ** 2  for i in range(10)    if i % 2 == 0}
print(even_sqrt_dic)
```

```python
{0: 0, 2: 4, 4: 16, 6: 36, 8: 64}
```









### 嵌套推导式

```python
matrix = [[1, 2, 3],
          [4, 5, 6], 
          [7, 8, 9]
         ]
flattened = [i for row in matrix for i in row]
print(flattened)
```

> 先取第一行 [1,2,3] ，再从第一行里取元素出来：1,2,3。从左往右读

```python
[1, 2, 3, 4, 5, 6, 7, 8, 9]
```





---





# 补充

[内置函数](https://ncloud.eagleslab.com/Python/Python%E9%9D%A2%E5%90%91%E5%87%BD%E6%95%B0/%E5%86%85%E7%BD%AE%E5%87%BD%E6%95%B0%E4%B8%8E%E5%8C%BF%E5%90%8D%E5%87%BD%E6%95%B0.html#%E5%86%85%E7%BD%AE%E5%87%BD%E6%95%B0)

[lambda表达式](https://ncloud.eagleslab.com/Python/Python%E9%9D%A2%E5%90%91%E5%87%BD%E6%95%B0/%E5%86%85%E7%BD%AE%E5%87%BD%E6%95%B0%E4%B8%8E%E5%8C%BF%E5%90%8D%E5%87%BD%E6%95%B0.html#%E5%8C%BF%E5%90%8D%E5%87%BD%E6%95%B0)

语法

 `lambda arguments: expression` 

```python
add = lambda x, y: x + y
# 调用匿名函数
result = add(3, 5)
print(result)  # 输出: 8
```

```python
# 使用匿名函数对列表中的元素进行转换
numbers = [1, 2, 3, 4, 5]
new_numbers = list(map(lambda x: x * 10 if x % 2 == 0 else x, numbers))
print(new_numbers)  # 输出: [1, 20, 3, 40, 5]
```

