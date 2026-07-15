







# Python 函数



[toc] 





## 定义函数



### 函数主体

```python
def function_name(argument):
    
    body
    
    return 

```





不用变量来承接函数返回值时，没有返回值（好像有点废话，但这是对装饰器说的）

```python
def function1():
    print('函数内部打印起作用了')
    return '1'

function1()
```

```python
函数内部打印起作用了
```

> return 很重要，在装饰器部分会用到





### 没有返回值

```python
def greet():
    print('hello python')
    
result1 = greet()
print(f'打印了返回值：{result1}')
```

```python
hello python
打印了返回值：None
```



### 有返回值

```python
def calculate_square(number):
    return number ** 2

result2 = calculate_square(int(input('请输入数字：')))
print(f'返回了：{result2}')
```

```python
请输入数字：9
返回了：81
```



### 有多返回值

用单个变量接收，会变成元组

```python
def get_user_info():
    name = '张三'
    age  = '18'
    gender = 'male'

    return name,age,gender

a=get_user_info()
print(a)
```

```python
('张三', '18', 'male') 
```





## 函数参数

讲一下函数参数



* 形参：就是临时变量，用来在函数体中”占位置。
* 实参：在函数调用时，输入的参数。
* 位置参数：在设计函数时，可能有多个变量，需要传递多个参数。而在函数调时，要按照相同的顺序，从左到右，输入参数。
* 默认参数。
* 关键字参数：在参数前面加上 *关键字* ，可以不用关心参数传递顺序。



### 关键字参数实例

```python
def greet3(name,gender,site='njupt'):
    print(f'hello,{site}性别为{gender}的{name}')

greet3(gender='性别',name='名字')
```

```python
hello,njupt性别为性别的名字
```





### 动态参数（可变数量参数）

 `*args` 可以使函数接受任意数量参数。收集到的参数，会形成一个元组

```python
def sum_number(*number):
    total = 0
    for i in number:
        total += i

    return total


print(sum_number(1,9,6,66))
```





### 动态关键字参数

和前面的一样，可以接收任意数量的关键字。这些被接收后，关键字作为 键key，参数作为 值value，组成一个字典

```python
def info(**kwargs):
    print(kwargs)
    
    for key, value in kwargs.items():
        print(f'{key}:{value}')
        
info(name='Aceli',age=18,city='New York')
```

```python
{'name': 'Aceli', 'age': 18, 'city': 'New York'}
name:Aceli
age:18
city:New York
```

> 可以同时使用 `*args` 和 `**kwargs` ，但 `*args` 要在 `**kwargs` 前







## 命名空间与作用域



* 全局命名空间
* 局部命名空间：在函数调用时创建，执行完毕后销毁。



* 局部作用域：函数内部的变量，只能在函数内部访问。
* 全局作用域
* 嵌套作用域：内层函数可以访问上层函数的变量.







### `globals` 和 `noloacl` 关键字

在变量前面加上` globals` ，表明这个变量是全局变量

 `nolocal` ：声明使用外层变量。表明这个变量与上层变量关联。

 *“对父级作用域（或者更外层作用域非全局作用域）的变量进行引用和修改，并且引用的哪层，从那层及以下此变量全部发生改变。”*

> `nonlocal` 仅在嵌套函数中有效，不能用于声明全局变量。



最佳实践：

* 避免使用全局变量。这与Python 模块化、易维护背道而驰。全局变量会增加代码的耦合度（关联性，牵一发而动全身），降低代码的可维护性。尽量使用局部变量和函数参数来传递数据。

* 使用封装思想。将代码逻辑封装在函数中，使用局部作用域来管理变量，提高代码的可读性和可维护性。





## 函数名



函数名，就是函数的内存地址；也是函数对象的引用。

只要函数名后面没有加上括号，就没有被调用；加了括号 ()  才算调用函数，所以有各种花式方法将函数名和括号组合起来。

比如：将函数名作为返回值。用一个变量来承接返回值，这个变量加上括号后，也可以调用



由此引出 **闭包函数** 

 





