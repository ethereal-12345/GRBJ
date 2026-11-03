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

  ## 异步编程
  1.  async / await ：语法，用来写异步函数
  
2.  asyncio ：官方库，提供事件循环、任务并发、信号量等调度能力

事件循环 event loop：异步的调度中心，负责安排协程什么时候运行

-  asyncio.run() ：python3.7+最常用，自动创建并关闭事件循环

 协程coroutine： async def  定义出来的东西

3Task任务：把协程包装成可调度任务，丢进事件循环并发执行

-  asyncio.create_task()  创建任务

-  asyncio.gather(*tasks)  ✅【高频】批量并发执行多个协程，收集所有返回结果

eg：import asyncio

async def demo():

    print("start")
    
    await asyncio.sleep(1) # 异步休眠，不会阻塞事件循环
    
    print("end")

asyncio.run(demo())

3. aiohttp ：第三方库，基于asyncio实现异步http请求

尽量复用  aiohttp.ClientSession() ，不要每次请求新建session，性能差

 session.get / session.post 前面必须加 await

 记得关闭session，推荐 async with 上下文管理器自动释放资源

eg：import asyncio

import aiohttp

async def fetch(session):

    url = "https://xxx"
    
    async with session.get(url) as resp:
    
        return await resp.text()

async def main():

    async with aiohttp.ClientSession() as session:
    
        res = await fetch(session)
        
        print(res)

asyncio.run(main())

## Pydantic类型验证与数握建備
