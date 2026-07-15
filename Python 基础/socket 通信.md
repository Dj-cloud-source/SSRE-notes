



# socket 通信

> socket 的基础知识就不多赘述了



## socket 通信流程：

Server ：

1. 创建 socket
2. 绑定 IP 和端口 port 
3. 开始监听
4. 等待客户端连接
5. 收发数据
6. 关闭连接



Client ：

1. 创建 socket 
2. 连接服务器
3. 收发数据
4. 关闭连接



## 单次通信

服务器

```python
import socket

server = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
server.bind(('127.0.0.1',9000))

server.listen(6) #?6?
conn,addr = server.accept()
print(conn,addr)

data = conn.recv(1024)
print(data.decode('utf-8'))

conn.send('你好，用户'.encode('utf-8'))

conn.close()
server.close()
```

先开启服务器



再开启客户端

```python
import socket

client =socket.socket(socket.AF_INET,socket.SOCK_STREAM)
client.connect(('127.0.0.1',9000))

client.send('你好，服务器'.encode('utf-8'))
#client.send(input(':'))


data = client.recv(1024)
print(data.decode('utf-8'))

client.close()
```



```python
#服务器收到
<socket.socket fd=388, family=2, type=1, proto=0, laddr=('127.0.0.1', 9000), raddr=('127.0.0.1', 63347)> ('127.0.0.1', 63347)
你好，服务器
```



```python
#用户收到
你好，用户
```







## 小疑惑

为什么send 和recv 要encode 和 decode ？

因为 socket 传输的是 字节，不是字符串



listen里面的数字？

官方叫 backlog ，表示在服务器还没有进行 accept() 接收操作之前，允许等待的未连接数量，“连接请求的排队区大小”





[^1]:break 是结束循环，return 是结束整个函数。这里是 while 循环，但没出现函数





## 多次发送

想多次收发怎么办？

* 进行循环处理

```python
……
#先开服务器
while True:
    data = conn.recv(1024)

    if not data:
        print('客户端已断开')
        break
        #reture '已断开'

    msg = data.decode('utf-8')
    print('客户端发送了一条消息：',msg)

    if msg.lower() == 'quit':
        print('聊天结束')
        break

    reply = input('回复：')
    conn.send(reply.encode('utf-8'))

    if reply.lower() == 'quit':
        print('聊天结束')
        break
```



```python
……
#客户端
while True:
    msg = input('请输入要发送的内容：')
    client.send(msg.encode('utf-8'))

    if msg.lower() == 'quit':
        print('聊天结束')
        break

    data = client.recv(1024)

    if not data:
        print('服务器已断开')
        break

    reply = data.decode('utf-8')
    print('客户端发来：',reply)

    if reply.lower() == 'quit':
        print('聊天结束')
        break
```



这样就可以实现简单的聊天了。当然只能进行可怜的几次聊天😅,还得轮流说

想像微信那种 “双方随时发信息”，还得学

* 多线程（线程并发）
* 多进程（进程并发）
* 异步 I/O



不用 return 而用 break[^1]



---

### 粘包问题

出现不足：

1. 前面有提到，TCP 传输的是字节流，不是消息队列，没有天然的消息边界。它只管 **可靠到达** 对吧，它不管你“原来发的时候分了几次”

​	 `recv(1024)` 不是“收一条消息”，而是“最多收 1024 个字节”



2. 双方收发都有缓冲区

​	`send()` 的数据不是瞬间凭空到对方的程序里，而是会经过 OS 的缓冲区。所以多个小数据，可能会在缓冲区里被合并，也就是平时常说的“粘包”





粘包解决方法

固定包头：先发一个固定长度的包头，里面带有正文长度。先收包头，在按长度读正文。

所以你要自己定义消息边界

那 Python 里常用 `struct` 模块，它的作用是：把整数等数据打包成固定字节格式，或者再解包回来。那配合 `len()` 函数，就能完成这些想要的功能



发送

```python
import struct

msg = 'hello'.encode('utf-8')
header = struct.pack('i',len(msg))

conn.send(header)
conn.send(msg)
```

接收

```python
import struct

header = conn.recv(4)
msg_len = struct.unpack('i',header)[0]

data = conn.recv(msg_len)
print(data.decode('utf-8'))
```



####  自定义协议

但是 recv “很懒”，所以要循环收，直到收满你要的长度

写一个接收函数,重写接收

```python
def recv_exact(conn,size):
    data = b''
    while len(data) < size:
        chunk = conn.recv(size - len(data))
        if not chunk :
            raise ConnectionError('连接中断')
        
        data += chunk
    return data
```

> chunk ：数据块

```python
import struct
header = recv_exact(conn,4)
msg_len = struct,unpack('i',header)[0]

msg_body = recv_exact(conn,msg_len)
print(body.decode('utf-8'))
```

