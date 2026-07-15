



# 面向对象



毕竟已经对面向对象有了很多了解，就不多赘述



[toc]





## 核心概念



*  **类 ** ：是模板和蓝图

*  **对象** ：是根据类创建出来的具体的东西



* **属性** ：是类或对象的特征，用于储存数据（就像身份卡）
* **方法** ：定义在类中的函数



**可以使用点号 `.` 来访问、操作对象的属性和调用对象方法**  

> 但点方法会造成麻烦：外部可以胡乱修改数据[^1]。这个等到 封装 时再说





###  `__init__`

这是“初始化方法”

创建对象时会自动执行

专门用来给对象“出厂设置”

```python
class Student:
    def __init__(self  ,name,age):
         self.name=name
         self.age = age
        
s1 = Student('tom',18)
print(s1)
print(s1.name)
```

```python
<__main__.Student object at 0x0000021AEC32F620>
tom
```

> 这里给对象“出厂设置”了一个名字
>
> 属性要记得加  `self` 
>
> 类名遵循驼峰命名法







###  `self`

就相当于形参，表示对象自己

> 还是上面例子

 `self` 就是 `s1` 

 `self.name = name` 就是在给 `s1.name` 赋值





### 方法

方法，就是“写在 类 里面的函数”

通过“点方法”调用

```python
class Dog:
    def __init__(self, name, age):
        self.name = name

    def bark(self):
        print(f"{self.name}在叫")
        
d1 = Dog("旺财")
d1.bark()
```

```python
旺财在叫
```










[^1]:点方法有点无敌了😥



动态添加属性

 **对象属性** 

* 定义添加
* 动态添加

```python
class MyClass:
      def __init__(self, name):
        #  定义添加
        self.name = name
        
    def add_instance_attr(self, age):
        #  通过方法动态添加
        self.age = age
```

```python
#  对象属性：外部添加
obj1.gender = "Male"  # 动态给 obj1 添加 gender 属性。仅 obj1 拥有，其他对象没有
print(obj1.gender)    
```

```python
male
```



 **类属性** 

* 定义添加
* 动态添加

```python
#  类属性：外部动态添加
Student.attribute = 'a attribute'
print(Student.attribute) 
```

```python
a attribute
```



---













# 封装



封装：把把数据和方法装进对象里，尽量不让外部乱改

比如，将年龄乱改为负数，所以就有了封装的需求

* 不让外部随便乱改数据
* 修改时要经过逻辑判断



## 代码演示

```python
class Student :
    def __init__(self,name,age):
        self.name = name
        self.__age = age

    def get_age(self):
            return self.__age

    def set_age(self,age):
        if age >= 0:
                self.__age = age

        else :
                print('年龄不能是负数')
                
s1 = Student('tom',18)
```



```python
print(s1.age)
```

```bash
#直接报错了,说明已经不能直接调用了
    print(s1.age)
          ^^^^^^
AttributeError: 'Student' object has no attribute 'age'
```







```python
print(s1.get_age())
```

```python
18
```

> 这样可以正常访问



想修改呢，也不能随意修改了

```python
s1.set_age(-8)
s1.set_age(7)
print(s1.get_age())
```

```python
年龄不能是负数
7
```



## 类的私有属性和私有方法

私有属性，在属性面前添加双下划线，可以将该属性标记为私有。在类的外部无法直接访问，只能在类的内部访问

 `__age` 

私有属性

前面两个下划线，表示：这是一个“不希望外部直接操作”的属性。

虽然上面不能直接访问，但是还是有方法能访问，所以 Python 的封装并不是“绝对不能操作”，而是“不建议直接操作”。

不直接操作，非要 `get` / `set` ，是因为函数方法里能添加规则，通过逻辑来判断是否能操作





当然，也有比 `get` 、`set` 更自然的写法

用 `@property` 

```python
class Student:
    def __init__(self, name, age):
        self.name = name
        self.__age = age

    @property
    def age(self):
        return self.__age

    @age.setter
    def age(self,new_age):
        if new_age >= 0:
            self.__age = new_age

        else:
            print('年龄不能是负数')

s1 =Student('alice',19)

print(s1.age)
s1.age = -1
```

```python
19
年龄不能是负数
```

> 这样更自然，像普通属性























# 继承



## 继承

一个类沿用另一个类的属性和方法，还能在此基础上扩展和重写。被继承的成为父类、基类，继承的叫做子类、派生类

```python
class Animal:
    def __init__(self,name):
        self.name = name

    def eat(self):
        return f'{self.name} is eating'

    
class Cat(Animal):
    def neow(self):
        return f'{self.name} says neow!'

    
cat1 = Cat('whisker')

print(cat1.eat())
print(cat1.neow())
```

> 这里 `Cat` 继承自 `Animal` 类，所以可以使用 `Animal` 类中的方法；
>
> 同时还有自己的 `neow` 方法



继承符合 Python 减少重复代码的理念，但是增加了代码的关联性（耦合性）





## 重写

改写父类方法。子类可以用自己的方法，覆盖父类同名方法

```python
class Animal:
    def speak(self):
        print('动物发出声音')

class  Dog(Animal):
    def speak(self):
        print('狗叫')

d1 = Dog()
d1.speak()
```

```python
狗叫
```







## 扩展

调用父类的方法

 `super()` 

子类扩展父类，子类想在父类的基础上继续补充内容。先用父类原来的逻辑，再补一点自己的逻辑

```python
class Animal:
    def __init__(self,name):
        self.name=name

class  Dog(Animal):
    def __init__(self,name,age):
        super().__init__(name)
        self.age = age


d1 = Dog('wang',16)
print(d1.name)
print(d1.age)
```

> 已忽略 `speak` 

```python
wang
16
```



这里使用 `super.__init__(name)` 继承父类内容，表示先调用父类的 `__init__` ，把父类内容的先初始化

又在下面添加了自己的内容， `self.age = age` 。

子类重写、添加了 `__init__` 后，想保留父类的初始化逻辑，就要主动调用 `super.__init__()` 。

如果子类自己写了 `__init__` ，父类的 `__init__` 不会自动执行。



```python
class Animal:
    def speak(self):
        print('动物发出声音')

class  Dog(Animal):
    def woof(self):
        super().speak()
        print('狗叫')

d1 = Dog()
d1.woof()
```

```python
动物发出声音
狗叫
```



















# 多态



前面已经学了：

* 继承

* 方法重写

* super()



多态就是再进一步

多态：调用同样的方法，传入不同对象，会有不同表现

1. 

```python
class Animal:
    def speak(self):
        print('动物发出声音')

class Dog(Animal):
    def speak(self):
        print('汪')

class Cat(Animal):
    def speak(self):
        print('喵')
```



2. 然后定义一个统一调用的函数

```python
def make_speak(arg):
    arg.speak()
```



3. 调用

```python
d = Dog()
c = Cat()

make_speak(d)
make_speak(c)
```

```python
汪
喵
```



这就是多态，**调用函数名都一样，但对象不同，结果不同**。

”同一个接口，不同实现“





在 Python 中，“只要你有这个方法，我就能调用。”

被称作 “鸭子”类型

“Python 并不关心是不是继承，只关心你有没有我要的函数、方法‘’





---



抽象类

父类给出一个抽象方法，子类必须给出抽象方法的具体实现

```python
import abc
class Payment(metaclass=abc.ABCMeta):
    @abc.abstractmethod
    def pay(self,money):
        pass


class Alipay(Payment):
    def pay(self,money):
        print(f'使用支付宝支付了 {money} 元')

class Wechatpay(Payment):
    def charge(self):
        print('微信支付')

def pay(arg,money):
    arg.pay(money)
```

```python
a = Alipay()
pay(a,100)

w = Wechatpay()
pay(w)
```

```python
使用支付宝支付了 100 元
# Wecahtpay 不遵循设计规范，已经报错了
```





----





# 双下方法

双下方法：让你的对象支持 Python 自带的行为




| 方法名     | 作用                                       |
| :--------- | :----------------------------------------- |
| `__init__` | 初始化对象属性                             |
| `__del__`  | 对象被从内存中销毁前，会被自动调用         |
| `__str__`  | 控制 `print(对象)` 时显示什么              |
| `__dir__`  | 查看对象内所有属性以及方法                 |
| `__len__`  | 输出对象长度                               |
| `__call__` | 对象后面可以加括号。让对象能像函数一样调用 |







案例

```python
class Student:
    def __init__(self,name):
        self.name = name
```

1. 


```python
    def __str__(self):
        return f'Student(name={self.name})'
    
    
s = Student('tom')
print(s)
```

```python
Student(name=tom)
```

2. 

```python
    def __len__(self):
        return len(self.name)
    
print(len(s))
```

```python
3
```

3. 

```python
    def __call__(self):
        print('这是 call 方法')
        
s()
```

```python
这是 call 方法
```







---


 内置方法



`.__dict__` 

查看类或对象中的内容

```python
class Human(object):    # 默认继承自 object
    """
    此类用来构造人类
    """
    mind = "思考问题.."
    # 在__init__中，通过self给对象封装属性
    def __init__(self,name,age,):
        self.name = name
        self.age = age

    def run(self):
        print('跑步')

print(Human.__dict__)
```

```python
{'__module__': '__main__', '__firstlineno__': 26, '__doc__': '\n此类用来构造人类\n', 'mind': '思考问题..', '__init__': <function Human.__init__ at 0x000001C4E7953270>, 'run': <function Human.run at 0x000001C4E795DC70>, 'eat': <function Human.eat at 0x000001C4E7E8F270>, '__static_attributes__': ('age', 'height', 'name'), '__dict__': <attribute '__dict__' of 'Human' objects>, '__weakref__': <attribute '__weakref__' of 'Human' objects>}
```



```python
xiaoming = Human('小明',18)
print(xiaoming.__dict__)
```

```python
{'name': '小明', 'age': 18}
```

---







# 反射

> 只看最有用的吧

动态调用

```python
class Student:
    def say_hello(self):
        print('你好')

        
s = Student()

arg = getattr(s,'say_hello')

arg()
```

```python
你好
```

根据字符串，动态调用对象的方法

是不是有点 “只要在函数名后面加括号就能调用” 的感觉？🤓







------------

<!--OK呀，也是终于把面向对象补完了-->



