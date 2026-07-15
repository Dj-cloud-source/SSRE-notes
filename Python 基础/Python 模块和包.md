[toc]



# 模块和包

> 诶，我突然发现，API调用，用的就是模块的知识



## 前言

还记得面向对象的的特点吗？

* 封装
* 继承
* 多态



这里的模块和包，就是封装的最佳注脚

开发者把代码写好，把它封装起来，在写其它代码时就无需重复，直接调用

> 它允许开发者将代码分割成可管理、可复用的单元，提高代码的可读性和可维护性。

这又是符合 Python 易维护的特点！



## 模块

模块（module），其实就是 Python 文件。Python 文件通常以 `.py` 结尾。模块里可以定义函数、类、变量、也可以包含可执行的代码。



### 模块的作用

* 代码复用：可以在不同程序中多次使用同一个模块的代码，避免重复编写
* 代码管理：将相关的代码放在同一个模块中，使代码结构更清晰，便于管理和维护
* 命名空间隔离





### 使用方法



* 导入模块

1.  `import` + `模块名` 
2.  `from` + `模块名` + `import` + `函数` 



* 使用模块

 ==**模块名点变量、函数**== 



```python
#test1.py
print('This is a module')

def greet(name):
    print(f'hello {name}')

PI = 3
```

```python
#test2.py
import test1

print(test1.PI)
test1.greet('hangman')
```

```python
This is a module
3
hello zhangsan
```



> **`if __name__ == "__main__"`**：在模块中使用 `if __name__ == "__main__"` 来区分模块作为脚本直接运行和作为模块被导入的情况。











## 包

包，是一种以使用目录来组织模块的形式。一个包里包含了多个模块

### 层级关系、目录结构

```
模块 → 包 → 子包 → 项目 → 库 / 框架

my_package/
    __init__.py
    math_tools.py
    string_tools.py
```



### 导入方式

使用 `import` 导入包

```python
import my_package
```

在导入 `my_package` 时，会执行  `my_package` 下的 `__init__.py` 中的代码



绝对导入相对导入

其实就是绝对路径和相对路径

`..` 是上一级文件夹

 `.` 是 本级文件夹



 ### `__init__.py` 说明

`init.py` 就是用来标记，“这是一个包文件”

`init.py` 里可以同一导出常用模块，简化导入路径

```python
#init.py

from .math_tools import add
from .string_tools import upper_text
```

这样就可以直接写

```python
from my_package import add,upper_text

print(add(2,3))
```

而不是

```python
from my_package.math_tools import add
from my_package.string_tools import upper_text
```

