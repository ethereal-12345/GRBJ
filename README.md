# Python编程基础
## 基础语法与数据结构
1. 列表：可变、有序、可重复 → 日常存数据首选

2. 元组：不可变、有序、可重复 → 固定数据

3. 字典：键值对、查找快、key唯一 → 存对应关系

4. 集合：无序、自动去重、无重复 → 去重、对比数据

## 面向对象编程
1.类 Class：类是模板，实例是造出来的具体对象， __init__ 初始化，self指代实例

eg：class Person:

    def __init__(self, name):
    
        self.name = name
        
    def say(self):
    
        print(f"{self.name}说话")
p = Person("张三") # p就是实例对象

2.继承 lnheritance:子类抄父类的代码，不用重复写，还可以新增/重写功能

eg：class Student(Person): # Student继承Person

    def __init__(self,name,score):
    
        super().__init__(name)
        
        self.score = score

3.装饰器Decorator:不改原函数源码，扩展函数能力，@是语法糖；分清 property / staticmethod / classmethod

eg：def outer(func):

    def inner():
    
        print("执行函数前做的事")
        
        func()
        
    return inner

@outer
def hello():
    print("hello")

    
