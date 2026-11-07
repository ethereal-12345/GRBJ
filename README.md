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
### 核心可以做什么
1. 类型校验：传进来的数据不对直接抛异常，不用自己手写一堆if判断

2. 自动类型转换：能转就帮你转，比如字符串"123" → int 123（符合规则前提下）

3. 数据建模：定义字段、默认值、字段限制（长度、大小、正则）

4. 导出字典/JSON：model_dump() 非常方便，对接大模型API、FastAPI必备

#### 基础写法
from pydantic import BaseModel

class User(BaseModel):

    id: int
    
    name: str
    
    age: int | None = None # 可选字段，不传有默认值

    ### 常用校验器
    Field：设置字段约束，最大最小、长度、描述，生成json schema（function calling非常依赖schema）
    eg：from pydantic import Field
    
name: str = Field(min_length=1, 
max_length=50, description="用户姓名")

## 环境管理（Conda/Poetry)
### Conda可以做什么
1. 创建隔离虚拟环境，不同项目Python版本随便切

2. 不光能装Python包，还能装C/C++、CUDA等非Python依赖（深度学习非常好用）

3. 解决有些包pip装不上、编译麻烦的痛点

#### 创建环境
conda create -n myenv python=3.11
#### 激活环境
conda activate myenv
#### 安装包
conda install xxx
#### 导出环境配置
conda env export > environment.yml
#### 根据yml还原环境
conda env create -f environment.yml

### Poetry可以做什么
1. 虚拟环境管理 + 精准锁定依赖版本，生成 pyproject.toml  +  poetry.lock 

2. 统一管理依赖、项目元信息，一键打包、发布Python库

3. 解决经典痛点： requirements.txt  版本混乱、分不清直接依赖和间接依赖

#### 初始化项目，生成pyproject.toml
poetry init
#### 创建虚拟环境并进入
poetry shell
#### 添加依赖（自动更新lock文件）
poetry add xxx
#### 安装全部依赖（从lock文件，保证团队环境一致）
poetry install
#### 导出成requirements.txt（兼容老项目）
poetry export -f requirements.txt --output requirements.txt

### Conda和 Poetry如何选择
1.选 Conda：深度学习、需要CUDA、底层非python库、想随意切换Python大版本

2. 选 Poetry：业务后端、LLM应用开发、写SDK包、多人协作，追求依赖可复现

# Web Apl开发
## FastAPI框架（路由/依赖注入/中间件)
1.路由：告诉程序，访问哪个网址、用什么请求方式，该执行哪段代码
2.依赖注入：把多个接口重复要用的逻辑抽出来，需要的时候直接“注入”进去，不用每个接口重复写代码
3.中间件：所有请求的必经关卡，请求进来到接口之前、接口返回结果出去之后都能做统一处理

## RESTful AP设计与现实
RESTful就是把所有东西看成资源，URL写资源名，用HTTP方法表示做什么操作

### 五个常用的http动作
GET：拿数据（查）

 POST：新建一条数据（增）

 PUT：全部覆盖更新

 PATCH：只改一部分字段

DELETE：删掉资源

### 状态代码的理解
200：正常搞定

 201：创建成功

 204：删除成功，不用返回内容

 400：参数不对

 401：没登录 / token失效

 403：登录了但是不让你操作

 404：找不到这个资源

 500：服务器崩了

## 异常处理与日志记录
### 异常是什么
代码运行报错就是异常。不处理的话，程序直接崩、接口直接500。

### try-except核心逻辑
- 可能报错的代码放  try 

- 报错了就进  except ，不崩程序

-  else ：没报错才执行

-  finally ：不管错没错，必定执行

  ###  开发最关键原则
 尽量精准捕获异常，不要随便写万能捕获

 不要吞异常：捕获了不打印、不记录，线上出问题完全查不出来

 能修复的异常处理，修复不了的友好返回，不让程序炸

 ## 日志记录

 ### 日志作用
 print 只能控制台看、不存文件、没时间、没级别。
日志就是正规版print，专门用来线上排查bug。

### 五个日志级别
DEBUG：开发调试用，细节信息

 INFO：正常运行关键记录

WARNING：有小问题，但不影响主功能

ERROR：功能报错、接口异常

CRITICAL：严重错误，程序快挂了

 ## 数据库基础（MySQL CRUD/索引优化）
 ### CRUD四大操作
 #### 1. C 增（INSERT）
往表里加新数据，直接填字段和值就行。

 #### 2. R 查（SELECT）
用得最多！查数据、筛选、排序、分页全靠它。
查询是最需要优化的操作。

#### 3. U 改（UPDATE）
改表里已有数据。

#### 4. D 删（DELETE）
删除数据。

### 常用索引类型
1. 主键索引
一张表只能一个，最快、不能为空、不能重复。

2. 唯一索引
字段不能重复，可以为空，用来保证数据唯一（手机号、账号）。

3. 普通索引
单纯加速查询，最常用，字段可以重复。

4. 联合索引
多个字段合在一起建一个索引。
死规矩：最左匹配原则
比如索引  (a,b,c) 
 
- 查 a、ab、abc 能命中索引

- 直接查 b、c、bc 直接失效

  ## Redis（缓存与消息队列)
  Redis 是跑在内存里的键值数据库，读写速度贼快，最常拿来做缓存，简单场景下也能充当消息队列。

  # Docker与部署
  
  ## Dockerfile写与镜像构建
Dockerfile 就是一份打包说明书，写清楚需要什么环境、拷什么文件、装哪些依赖；执行构建命令之后，就能生成镜像。
镜像 = 打好包的软件模板；容器 = 镜像跑起来之后运行中的程序

