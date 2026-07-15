

## `hashlib` 模块



先讲讲什么是哈希函数

> 哈希函数，是一种将任意长度的输入数据，转换成固定长度输出的函数，这个输出结果通常称为哈希值。



哈希函数，通常用于：

1. 校验一致性
2. 加密处理
3. 储存和查找



### 基本写法

```python
import hashlib

a = 'hello_world'
hash_a = hashlib.sha256(a.encode()).hexdigest()
print(hash_a)
```



*  `sha256` 是更常见、更安全的算法



* 为什么要 `.encode()`  ？

因为哈希函数处理的是**字节数据**，不是普通字符串



*  `hexdigest()` 是什么？

它的作用是把结果显示成更常用的十六进制字符串

```python
hash_b = hashlib.sha256(a.encode())
hash_b1=hash_b.hexdigest()
hash_b2=hash_b.digest()

print(hash_b)
print(hash_b1)
print(hash_b2)
```

```python
<sha256 _hashlib.HASH object @ 0x0000016CBE6BD0F0>
35072c1ae546350e0bfa7ab11d49dc6f129e72ccd57ec7eb671225bbd197c8f1
b'5\x07,\x1a\xe5F5\x0e\x0b\xfaz\xb1\x1dI\xdco\x12\x9er\xcc\xd5~\xc7\xebg\x12%\xbb\xd1\x97\xc8\xf1'
```







### 加盐

加盐（salt）= 在原始密码旁，额外加一段随机数据，再一起哈希

加盐的目的不是让密码更复杂，而是让相同的密码的哈希值不同；防止撞库，增强存储安全性



常用加盐方案：`pbkdf2_hmac` 

真实项目中更多用成熟的方案： `bcrypt` 、 `argon2` 、框架自带密码工具



语法： `hashed_password = hashlib.pbkdf2_hmac(hash_algorithm,password,salt,iteration)` 



```python
import hashlib
import os

salt = os.urandom(16)   # 生成16字节随机盐
password = '123456'

hashed_password = hashlib.pbkdf2_hmac(
    'sha256',                         # 哈希算法
    password.encode(),                # 密码，必须转成字节
    salt,                             # 盐
    10                                # 迭代次数
).hex()
print(hashed_password)
```

> 诶，每次点运行生成的都不一样诶



所以注册时通常需要保存

*  `salt` 
*  `hash_algoritm` 
*  `iterations` 





### 登录验证



学习演示

```python
import hashlib
import os

# =========================
# 注册
# =========================
password = "123456"

algorithm = "sha256"
iterations = 100000
salt = os.urandom(16)

password_hash = hashlib.pbkdf2_hmac(
    algorithm,
    password.encode(),
    salt,
    iterations
)

user_record = {
    "username": "tom",
    "algorithm": algorithm,
    "iterations": iterations,
    "salt": salt.hex(),
    "password_hash": password_hash.hex()
}

print("注册时保存的数据：")
print(user_record)


# =========================
# 登录验证
# =========================
input_password = "123456"

stored_salt = bytes.fromhex(user_record["salt"])
stored_hash = bytes.fromhex(user_record["password_hash"])

check_hash = hashlib.pbkdf2_hmac(
    user_record["algorithm"],
    input_password.encode(),
    stored_salt,
    user_record["iterations"]
)

if check_hash == stored_hash:
    print("登录成功")
else:
    print("密码错误")
```

> 数据库、JSON、接口传输里，更常见是存成字符串，所以使用 `.hex()` 
>
> 登录时其实是重新算再比较，因为哈希化不可逆

 

GPT：更接近真实项目的“伪代码”

```python
def login(username, password):
    user = find_user_from_database(username)

    if user is None:
        return "用户名或密码错误"

    if not user["is_active"]:
        return "账号已被禁用"

    if verify_password(password, user["password_hash"]):
        create_session(user["id"])
        return "登录成功"

    return "用户名或密码错误"
```

>  `find_user_from_database(username)` 真实项目是：从数据库里查
>
>  `verify_password(password, user["password_hash"])` 真实项目里常常封装成一个函数，不会每次都手写细节
>
>  `create_session(user["id"])` 登录成功后，创建登录状态。还没学到