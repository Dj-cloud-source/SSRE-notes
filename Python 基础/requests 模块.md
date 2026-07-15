



# requests 模块

[toc]





 `requests` 模块是Python中非常常用的第三方 http 库，用来向网站或接口发送请求，即让 Python 去访问网址、调用接口



## 安装第三方模块

```bash
#命令行
python -m pip install requests
```

```python
#python
import requests
```





## 基本用法

###  `get` 请求

用于“获取数据”

```python
response = requests.get('https://httpbin.org/')
```



```python
print(response)
```

```python
<Response [200]>
```

> Requests 的主要方法都会返回 `Response` 对象
>
> 200 是状态码。200 表示成功



```python
print(response.status_code)
```

```python
200
```





查看返回的文本内容

```python
print(response.text)
```

```python
{
  "args": {}, 
  "headers": {
    "Accept": "*/*", 
    "Accept-Encoding": "gzip, deflate, zstd", 
    "Host": "httpbin.org", 
    "User-Agent": "python-requests/2.33.1", 
    "X-Amzn-Trace-Id": "Root=1-69cca5da-19fe3d1e140d5cf430b2913f"
  }, 
  "origin": "218.33.111.3", 
  "url": "https://httpbin.org/get"
}
```







### `post` 请求

用于“提交数据”

> 如果接口要求发 JSON ，应该用 `json=` 

```python
r = requests.post('https://httpbin.org/post',json = load)
print(r.text)
```

```python
{
  "args": {}, 
  "data": "{\"name\": \"Alice\", \"age\": 20}", 
  "files": {}, 
  "form": {}, 
  "headers": {
    "Accept": "*/*", 
    "Accept-Encoding": "gzip, deflate, zstd", 
    "Content-Length": "28", 
    "Content-Type": "application/json", 
    "Host": "httpbin.org", 
    "User-Agent": "python-requests/2.33.1", 
    "X-Amzn-Trace-Id": "Root=1-69cca97a-169f15fd51493b3e0ee09141"
  }, 
  "json": {
    "age": 20, 
    "name": "Alice"
  }, 
  "origin": "218.33.111.3", 
  "url": "https://httpbin.org/post"
}
```



响应头

```python
print(r.headers)  
```

```python
{'Date': 'Wed, 01 Apr 2026 05:15:02 GMT', 'Content-Type': 'application/json', 'Content-Length': '517', 'Connection': 'keep-alive', 'Server': 'gunicorn/19.9.0', 'Access-Control-Allow-Origin': '*', 'Access-Control-Allow-Credentials': 'true'}
```



JSON 转 Python 对象

```python
print(r.json())
```

```python
{'args': {}, 'data': '{"name": "Alice", "age": 20}', 'files': {}, 'form': {}, 'headers': {'Accept': '*/*', 'Accept-Encoding': 'gzip, deflate, zstd', 'Content-Length': '28', 'Content-Type': 'application/json', 'Host': 'httpbin.org', 'User-Agent': 'python-requests/2.33.1', 'X-Amzn-Trace-Id': 'Root=1-69ccaa21-5d3d1fab44990f7a105b802d'}, 'json': {'age': 20, 'name': 'Alice'}, 'origin': '218.33.111.3', 'url': 'https://httpbin.org/post'}
```





## 实际开发

1. 加超时（秒）

```python
r = requests.post('https://httpbin.org/post',json = load,timeout=5)
```

> 实际开发里，不加超时很容易让程序一直等



2. 异常处理

```python
try:
    r = requests.get("https://httpbin.org/get", timeout=5)
    print(r.status_code)
except requests.exceptions.RequestException as e:
    print("请求失败：", e)
```





### 连续请求同一个网站

可以用 `Session` 

```python
s = requests.Session()
r1 = s.get("https://httpbin.org/cookies/set/[1,2,3],666")
r2 = s.get("https://httpbin.org/cookies")
print(r2.text)
print(r2)
```

```python
# r2.txt
{
  "cookies": {
    "a": "[1,2,3],666"
  }
}
```

```python
# r2
b'{\n  "cookies": {\n    "a": "[1,2,3],666"\n  }\n}\n'
```




> `Session` 可以跨请求保留某些参数和 cookies，并复用连接



