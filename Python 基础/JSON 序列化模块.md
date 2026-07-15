## `json` 模块

JSON(JavaScript Object Notation) 是一种用于前后端数据传输、交换的数据格式。



`json` 模块提供了处理 JSON 数据的功能。我们可以使用 JSON 模块，使 Python 数据类型和 JSON 字符串相互转换。

 

## 语法

*  ==`json.dumps(数据对象)` ：将 Python 数据对象转换为 JSON 字符串==
*  ==`json.loads(JSON字符串)`== ：将 JSON 字符串转换为数据对象



```python
import json

data ={
    'name':'zhangsan',
    'age':'18',
    'course':['python','linux','mysql']

}

json_data=json.dumps(data)
print(type(json_data))
print(json_data,'\n---')

json_data1=json.loads(json_data)
print(type(json_data1))
print(json_data1['name'])
```

```python
<class 'str'>
{"name": "zhangsan", "age": "18", "course": ["python", "linux", "mysql"]} 
---
<class 'dict'>
zhangsan
```



还有配合文件操作的，这里简单演示

```python
import json

with open("data.json", "r", encoding="utf-8") as f:
    data = json.load(f)

print(data)
```



Python 数据类型和 JSON 对应关系（只列重点）

| Python      | JSON   |
| ----------- | ------ |
| int / float | number |
| None        | null   |
| tuple       | array  |

> JSON 没有 `set` 类型。想转只能把 `set` 转成 `list` 
>
> data = {1, 2, 3}
> json_str = json.dumps(list(data))



---



