[toc]



# 常用模块



**重点模块已独立记录，详细移步至各模块笔记**





## `json` 模块

JSON(JavaScript Object Notation) 是一种用于前后端数据传输、交换的数据格式。



`json` 模块提供了处理 JSON 数据的功能。我们可以使用 JSON 模块，使 Python 数据类型和 JSON 字符串相互转换。

 


---







## `hashlib` 模块



先讲讲什么是哈希函数

> 哈希函数，是一种将任意长度的输入数据，转换成固定长度输出的函数，这个输出结果通常称为哈希值。



哈希函数，通常用于：

1. 校验一致性
2. 加密处理
3. 储存和查找





---



##  `re` 模块

































##  `collections` 模块

1.  `counter` 
2.  `defaultdict` 
3.  `deque` 



虽然普通的数据类型已经够用了，但 `collections` 可以

* 写起来更方便
* 某些操作更高效



### `counter` 

 统计频率方便

```python
from collections import Counter

text = "banana"
result = Counter(text)
print(result)
```

```python
Counter({'a': 3, 'n': 2, 'b': 1})
```



### `defaultdict`

字典里 key 不存在时，自动给默认值

适合分组、累加、建立“一个 key 对应多个值”的结构

```python
from collections import defaultdict

data = defaultdict(list)
data["a"].append(1)
data["a"].append(5)
data['b'].append(2)

print(data)
```

```python
defaultdict(<class 'list'>, {'a': [1, 5], 'b': [2]})
```



###  `deque` 

适合两端快速添加/删除元素的队列

常用于BFS、消息队列、滑动窗口

```python
from collections import deque

q = deque([1, 2, 3])
q.appendleft(8)
q.append(0)

print(q)
```

```python
deque([8, 1, 2, 3, 0])
```







##  `sys` 模块

 `sys` 是和 Python 运行环境相关的模块

`sys` 主要是让你访问：

- Python 解释器本身的一些信息
- 程序运行时环境
- 命令行参数
- 退出程序
- 模块搜索路径



当你开始写：

- 命令行工具
- 自动化脚本
- 脚本参数处理
- 调试导入问题

`sys` 就会经常出现。

尤其是：

 `sys.argv`

这是很多命令行脚本的基础。



### 1.  `sys.argv` 
获取命令行参数

```python
import sys

print(sys.argv)
```

```bash
#使用 bash 运行
python test.py hello 123
```

```python
# python 输出
['test.py', 'hello', '123']
#脚本文件名  第一个参数  第二个参数
```



### 2.  `sys.exit()` 
结束程序
```python
import sys

print("开始")
sys.exit()
print("这句不会执行")
```

```python
开始
```



### 3.  `sys.path` 

模块搜索路径

```python
import sys

print(sys.path)
```

> 会出来一大长串路径





---















## `OS` 模块

`os` 模块是用来和操作系统交互的模块

* 文件和目录

* 路径

* 当前工作目录

* 创建/删除目录

* 重命名

* 环境变量



### 文件/目录操作



1.  `os.getcwd()`

获取当前工作目录（Current Working Director），查看程序现在在哪个文件夹里

```python
import os
print(os.getcwd())
```

```python
D:\Python_workplace\code_exercise\0329
```



2.  `os.chdir('Path_name')`

切换当前工作目录



3.  `os.listdir('path')`

列出某个目录下的内容



4.  `os.mkdir('dir_name')` 

创建一个目录



5.  `os.remove('file_name')` 

删除一个**文件**



6.  `os.rmdir('dir_name')` 

删除一个**空目录**



7.  `os.rename('old_name','new_name')` 

重命名文件或目录



### 路径相关

”路径“需要重点学习



1.  `os.path.join(path)` 

拼接路径

```python
path1=os.path.join(os.getcwd(), 'test2.py')
print(path1)
```

```python
D:\Python_workplace\code_exercise\0329\test2.py
```

> 为什么不手写？
>
> 因为不同系统，路径分隔符可能不一样





2.  `os.path.exists('file_or_path')` 

判断路径是否存在

```python
print(os.path.exists('test6.py'))
```

```python
True
```



3.  `os.path.isfile('name')`
4.  `os.path.isdir('name')` 

判断是不是文件，是不是目录





---



##  `shutil` 模块

和 `os` 区别是什么

 `os` 更底层，偏“系统接口”

 `shutil` 更像高级文件操作的工具箱



1. 复制文件

```python
import shutil

shutil.copy("data.txt", "data_backup.txt")
```



2. 复制目录

 `shutil.copytree("project", "project_backup")` 



3. 移动文件或文件夹（剪切）

 `shutil.move("report.txt", "archive/report.txt")` 



4. 删除整个目录

 `shutil.rmtree("test_folder")`  

> 这个很危险，因为它会把目录和里面的内容全部删掉



5. 打包、压缩文件

 `shutil.make_archive("backup", "zip", "my_folder")` 



6. 解压文件

 `shutil.make_archive("backup", "zip", "my_folder")` 





---



## 时间模块

*  `time` 模块
*  `datetime` 模块





---





## `random` 模块



 `.random()` 

随机生成（1，0）之间的小数

```python
import random

print(random.random())
```



 `.randint()` 

随机生成【a，b】之间的整数

```python
print(random.randint(1,10))
```



 `.randrange()` 

在【a，b）随机选一个，可加步长

```python
print(random.randrange(0,100,3))
```



 `.choice()` 

在可变数据类型（？）中随机抽取一个

```python
profile1 = ['文菊','汤姆','杰克','alice']
print(random.choice(profile1))
```

```python
alice
```

> 可用于：
>
> * 抽奖
> * 随机点名
> * 随机出题





 `.sample()` 

随机抽取任意数量、不重复的元素

```python
s = [1,2,3,4,5,6,7,8,9,10,11,12,13,14,15]
print(random.sample(s,6))
```

```python
[12, 8, 5, 7, 11, 4]
```





打乱顺序

```python
item = [16,7,8,8,94,9,4]
random.shuffle(item)
print(item)
```

```python
[8, 16, 4, 7, 94, 8, 9]
```











* [x] JSON
* [x] 哈希
* [x] collections
* [x] time 模块
* [x] random 模块
* [x] os 模块
* [x] sys 模块
* [x] re 模块
* [x] shutil 模块

* [ ] request



数据处理

* [ ] numpy



web

* [ ] flask

