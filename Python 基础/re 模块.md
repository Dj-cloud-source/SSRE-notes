

[toc] 



#  `re` 模块

 $RE正则表达式有多重要，就不过多赘述$ 



## 常用函数




### 1. 找出所有符合规则的内容

 `re.findall(pattern)` 

```python
import re

txt = 'h34ggf6网飞3几日'
result = re.findall(r'\d+',txt)

print(result)
```

```python
['34', '6', '3']
```



>  `r' '` 表示原始字符串，因为反斜杠 `\` 在Python字符串里有特殊含义，“写正则时加 `r` 更安全”，写正则时，前面优先加 `r` 。
>
>  `\d` ：数字
>
>  `+` ：一个或多个





### 2. 搜索第一个符合规则的内容

 `re.rearch()` 

```python
result1 = re.search(r'\d+',txt)
print(result1)
print(result1.group())
```

```python
<re.Match object; span=(1, 3), match='34'>
34
```

>  `search()` 返回的是一个对象
>
> 真正的内容要用 `.group()` 取出来



### 3. 检查开头

 `re.match()` 

```python
a = 'b677'
result2 = re.match(r'\d+',a)
print(result2)
```

```python
None
```



### 4. 替换符合规则的内容

 `re.sub()` 

```python
txt = 'h34ggf6网飞3几日'
result3 = re.sub(r'\d+','&&*',txt)
print(result3)
```

```python
h&&*ggf&&*网飞&&*几日
```

> 这个常用，可以替换电话号码之类的







## 基础的正则符号



### 1.  `\d` ：匹配数字

```python
r'\d'  #表示匹配一个数字
r'\d+'  #表示匹配一个或多个数字
```



### 2.  `\w` ：匹配字母、数字、下划线 

> 大写是取反，“非”



### 3.  `\s` ：匹配空白字符（空格、换行、tab）

```python
r'\s+'  #表示一个或多个空白字符
```



### 4.  `.` ：匹配任意字符（除了空白）

```python
result5 = re.findall(r'网.3',txt)
print(result5)
```

```python
['网飞3']
```







## 重要的数量符号



### 1.  `+` ：前面的内容出现一次或多次

```python
result6 = re.findall(r'g+',txt)
print(result6)
```

```python
['gg']
```



### 2.  `*` ：前面的内容出现0次或多次

```python
result6 = re.findall(r'g*',txt)
print(result6)
```

```python
['', '', '', 'gg', '', '', '', '', '', '', '', '']
```

> 估计要是没有就显示 空



### 3.  `?` ：前面的内容出现0次或1次

```python
result7 = re.findall(r'g?',txt)
print(result7)
```

```python
['', '', '', 'g', 'g', '', '', '', '', '', '', '', '']
```



### 4.  `{n}` ：前面的内容恰好出现 n 次

```python
result8 = re.findall(r'g{2}',txt)
print(result8)
```

```python
['gg']
```



### 5.  `{m,n}` ：前面的内容出现 m 到 n 次

```python
result9 = re.findall(r'g{3,4}',txt)
print(result9)
```

```python
[]
```





## 范围匹配




```python
txt = 's32d25fh25ja4325gt3thd64sfas34635jn3as3k894722'
```




### 1.  `[abc]` ：表示匹配其中的任意字符

```python
r1 = re.findall(r'[abd]',txt)
print(r1)
```

```python
['d', 'a', 'd', 'a', 'a']
```



### 2.  `[5-9]` ：匹配范围内的任意数字

```python
r2 = re.findall(r'[6-9]',txt)
r22 = re.findall(r'[6-9]+',txt)
print(r2,r22)
```

```python
['6', '6', '8', '9', '7']
['6', '6', '89', '7']
```





### 3.  `[a-z]` ：匹配小写字母

```python
r3 = re.findall(r'[a-g]',txt)
print(r3)
```

```python
['d', 'f', 'a', 'g', 'd', 'f', 'a', 'a']
```

 

### 4.  `[A-Za-z0-9]` ：匹配大小写字母和数字





## 开始和结束符号

重要



### 1.  `^` ：表示开头

```python
r4 = re.findall(r'^s32',txt)
print(r4)
```

```python
['s32']   #表示字符串必须以 `s32` 开头 
```





### 2.  `$` ：表示结尾

```python
r5 = re.findall(r's32$',txt)
print(r5)
```

```python
[]   #表示字符串必须以 `s32` 结尾
```





> [贪婪、命名分组、HTML提取](https://ncloud.eagleslab.com/Python/Python%E9%9D%A2%E5%90%91%E5%87%BD%E6%95%B0/%E5%B8%B8%E7%94%A8%E6%A8%A1%E5%9D%97.html#%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F ' 更详细请看课件')









# 案例



判断手机号格式

```python
import re

phone = "13812345678"

result = re.match(r"^\d{11}$", phone)
if result:
    print("格式正确")
else:
    print("格式错误")
```

* 从开头到结尾

* 必须是 11 位数字

* 不能多也不能少



去除 HTML 标签

```python
import re
 
html = '<p>这是一个 <b>HTML</b> 段落。</p>'
pattern = r'<[^>]+>'
 
clean_text = re.sub(pattern, '', html)
print("去除标签后的文本:", clean_text)
```

```python
这是一个 HTML 段落。
```

