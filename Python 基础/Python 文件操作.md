



# Python 文件操作

[toc]







---

打开文件

使用 open() 函数打开文件，可以 *获得一个文件对象* ，

我们需要使用点方法才能对文件对象进行读写，

> 在 Windows 下是 gbk，在 Linux 下是 utf-8

```python
file2 = open("D:\笔记typora\测试文件.txt",'r',encoding='utf-8')
print(file2.read())
file2.close()
```

使用 open() 函数后，一定要使用 close() 方法关闭文件，释放系统资源，防止文件被占用。

> 其实，Python只是给底层提供了一个封装接口，或者说，API。

---





| 访问方式 | 说明                                     |
| -------- | ---------------------------------------- |
| r        | 读                                       |
| w        | 覆写                                     |
| a        | 在末尾添加内容                           |
| +        | 以读写的方式打开文件。可以和上面三个组合 |



## 打开文件

 ==**推荐使用 `with`  语句打开文件，可以自动关闭文件**== 

```python
with open('D:\笔记typora\测试文件.txt','r', encoding='utf-8') as file2:
    print(file2.read())
```

一般，**打开一次只能选择读或者写。不能同时读和写**



## 写入内容

使用 `write()` 方法

```python
with open('D:\笔记typora\测试文件.txt','a',encoding='utf-8') as file6:

    file6.write(input('请输入内容：')+'\n')
```

