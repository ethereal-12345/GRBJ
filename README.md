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

### 开发最关键原则
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

## Docker Compose多容器编排
Dockerfile搞定一个容器，Docker Compose搞定一堆互相配合的容器

### Docker Compose常用命令
#### 后台启动全部服务（开发最常用）
docker-compose up -d
#### 实时看日志
docker-compose logs -f
#### 查看当前编排下的容器状态
docker-compose ps
#### 停止容器，保留容器
docker-compose stop
#### 停止并删除容器（卷默认保留，加 -v 才会删掉数据卷）
docker-compose down
#### 修改yml之后重新构建启动
docker-compose up -d --build

## Linux常用命令与进程管理
Linux命令就是在服务器上敲指令干活，进程就是正在跑起来的程序；进程管理就是查看程序、杀卡死的程序、后台跑服务。

## Git版本控制
Git就是代码的时光机+多人协作工具，可以保存每一次代码改动快照，可以回退到过去版本，还能多个人同时改一份代码、合并代码。

## CI/CD基础概念
CI/CD就是一套自动化流水线，不想手动打包、手动传服务器、手动点上线，交给机器自动完成，降低人为失误。


# 大模型基础认知
## Transformer架构入门（自注意力/Token/上下文窗口)
Transformer就是现在所有大模型的核心骨架，靠自注意力机制，读一段话的时候，能判断哪些词之间关联性强、要重点关注，不再像老模型那样顺着文本一个挨一个往下读。

### Token
模型自带分词器切出来的最小的单元

### 上下文窗口 context window
模型一次性最多能看见的token总量

### 自注意力
大模型自带的找关联，分轻重

## 主流模型对比（GPT/Claude/Gemini/DeepSeek/Qwen)
GPT:综合全能型选手，生态最成熟
Claude：长文档专家，幻觉控制做得很好
Gemini ：多模态特长生，图片、音频、视频解析是强项
DeepSeek：推理、代码很强，性价比高
Qwen：国产全能工具箱，开源生态完善，中文友好

## 大模型API调用(OpenAl/DeepSeek 结构化输出)
调用大模型API，就是不用本地跑大模型，通过网络发请求给别人已经部署好的服务，把问题传过去，模型返回结果；结构化输出就是强制大模型不要自由写一大段文字，必须返回JSON格式的数据，方便程序直接解析。

## Token计算与成本优化
大模型计费不是按汉字，是按 Token 算钱；成本优化本质就是：减少进出模型的token数量，同样效果少花钱、速度还更快。

## Streaming流式响应处理
非流式：模型全部生成完，一次性返回整段文字，要等很久页面才出结果。

流式（streaming）：模型生成一点，立刻吐出一点，边生成边返回，前端可以逐字展示，用户体验像打字一样。

# Prompt Engineering
## 结构化提示词   
结构化提示词：不给模型自由发挥的空间，规定格式、规定步骤、规定输出结构，让模型稳定、统一、可被代码解析。 

普通提示词 = 聊天

结构化提示词 = 干活、输出数据、给程序读

##  Few-shot Learning(少样本学习)
不用喂成千上万条标注好的数据，只给很少几条示范案例，大模型看完例子，照着里面的模式去完成新任务。不用喂成千上万条标注好的数据，只给很少几条示范案例，大模型看完例子，照着里面的模式去完成新任务。

## Chain-of-Thought（思维链)
不让大模型直接蹦最终答案，引导它一步步写出推理过程，再得出结果，就像人做题先写演算步骤。

## ReAct框架（推理+行动）
ReAct 就是让模型先想一想（推理Thought），再动手调用工具去做事（Action），拿到工具返回结果之后，接着再思考、再行动，循环往复，直到拿到最终答案。

## 高级策略（Self-Consistency / Reflexion)
1. Self-Consistency（自一致性）
同一个问题，让模型多跑几条不一样的思考路线，最后投票选多数认同的答案。

2.Reflexion（反思） 
模型做完任务、拿到反馈之后，回头复盘自己错在哪，改自己的思路，重新再来一遍。

# RAG检索增强生成
## 完整流程（文档加载一分割→向量化→检索→重排序→生成)
1.文档加载：把文件读进程序里，提取里面的文字

2.分割：文档太长需要切成一小段一小段的

3.向量化：相当于给每张小纸条打上一个内容指纹

4.检索：相当于拿着问题的指纹去资料库找匹配的纸条

5.重排序：相当于初选后，又进行一次筛选，留下最有用的

6.生产：把用户问题 + 筛选好的相关上下文，一起打包丢给大模型，
大模型参考这些资料，输出答案

## Embedding向量化技术
把文字、图片这类人类看得懂的内容，转成一串数字（向量），语义相近的内容，数字向量距离就很近，方便机器算相似度。

## 向量数据库（ChromaDB/ Milvus/ Qdrant)

专门用来存Embedding向量并且快速查找相似向量的数据库。MySQL、Redis不擅长干这个。

1. ChromaDB
轻量、简单，开箱即用，Python一行代码就能跑，自带持久化。
适合：本地测试、demo、小体量知识库，开发学习用。

2. Qdrant
主打高性能，资源占用比较友好，支持丰富的过滤、分片。
适合：中小型生产项目，RAG业务很常用，上手难度适中。

3. Milvus
功能最全、扩展性最强，支持超大百万/亿级向量，集群部署。普通文本大模型只能读文字；Vision模型多了看图的能力。
它会先把图片里的画面、文字、图表、物体转换成模型能识别的特征，再结合你的文字问题一起推理。

适合：大规模线上业务、高并发、海量知识库；

##  混合检索（语义+关键词）
同时用两种方式找资料，一边靠意思匹配（语义检索），一边靠文字关键词匹配，最后把两边结果合并打分，挑最好的片段。

## 重排序模型(Reranker)
前面检索先捞出来一批候选段落，相当于海选；Reranker 就是复审，拿着用户的问题，一条一条仔细判断这段内容到底跟问题有没有关系，重新排顺序，把没用的往后放甚至丢掉。

## 上下文压缩技术
RAG检索出来的chunk，经常是一整段原文，里面夹杂无关描述、铺垫、重复句子。

上下文压缩就是给这段文字“精简提炼”。

不是自己瞎编内容，只删、摘、浓缩原文里本来就有的信息。


# 多模态Al能力
## Vision模型（GPT-4V/ DeepSeek Vision)
普通文本大模型只能读文字；Vision模型多了看图的能力。

它会先把图片里的画面、文字、图表、物体转换成模型能识别的特征，再结合你的文字问题一起推理。

### GPT-4V 和 DeepSeek Vision简单区别

GPT-4V：能力稳，识图、读手写文字、理解复杂图表、推理都很强；闭源，走API，价格偏高。

DeepSeek Vision：国产，开源版本也有，识图速度不错，适合私有化部署、本地跑；复杂图表、精细推理会弱一点。

## 图像Embedding与CLIP
普通文本Embedding只转文字；图像Embedding就是把图片变成一串数字向量，代表这张图讲了啥。

CLIP最特别：图片和文字放到同一个向量空间。

## 语音识别（Whisper ASR)
普通文字模型只能处理文字；Whisper接收音频，先把声音信号拆解，识别里面说的话，输出文字。

不光能识别，还自带语种识别、断句、标点。

## 多模态RAG
普通RAG只能处理文字，多模态RAG就是既能读文字，也能读图片、表格、截图、PDF里的图、公式的检索增强。

### 多模态RAG的完整流程
文档解析 → 文字分块、图片抽出来 → 文字Embedding、图像Embedding → 全部存入向量库 → 用户提问（文字/图片）→ 检索匹配文字+图片 → 把检索到的图文一起喂给VLM → 输出答案。


# Agent核心概念
## 任务规划
任务规划就是让Agent学会把一个复杂大活儿，拆成一个个能直接干的小步骤，不会一口吃个胖子。

### 两种常见的规划方式
静态规划：一次性全部拆完，拿到完整步骤再挨个执行。适合流程固定的任务，比如整理一份调研报告。缺点：中间出意外，不会改计划。

动态规划：走一步看一步，做完一步，根据结果再重新规划下一步。遇到失败、新信息可以随时调整，ReAct、LangGraph基本都是这种。

## 工具调用(Function Calling/ Tool Use)
大模型自己知道“这个问题我答不了，得叫外面工具帮忙查、算、操作”，按规定格式调用工具，拿到结果再接着回答你。

## 记忆管理（短时记忆+长时记忆)
### 短时记忆
就是当前这一轮对话的上下文，相当于你当下脑子里正在想的东西。
 
内容：咱们这次聊天的全部对话、刚刚调用工具返回的结果、当前任务进度

 特点：跟着会话走，一关对话就没；有token上限，装多了会溢出、丢前面内容

 作用：保证Agent连贯，知道刚才在干啥

 ### 长时记忆
相当于笔记本、档案柜，把重要信息单独存到数据库/向量库里面，不占对话上下文。
 
内容：用户偏好、历史资料、之前完成的任务、重要结论

特点：跨会话保存，这次聊完，下次打开还能检索出来；不是全部都放，只存关键信息

读取方式：用户提问的时候，先检索相关记忆，再喂给大模型

## ReAct工作流
一边思考、一边动手，循环反复，直到把事情做完，就是ReAct。

ReAct = Reason（思考） + Act（行动）

# Agent开发框架
## LangChain生态(Chains/Tools/Context Managemen）
LangChain就是一堆可以拼起来的零件，Chains是流水线，Tools是它能用的工具，Context Management专门管上下文记忆。

## LangGraph（基于状态机的复杂Agent)
LangGraph 是给LangChain用的流程图工具，用状态机把Agent的每一步画成节点，自己控制怎么走、要不要循环、分支往哪边拐。

## CrewAl(角色扮演式多智能体协作)
组建一个AI小团队，给每个AI分配不同岗位，大家分工配合、一起完成一个大任务。

##  Dify（开源知识库)
不用自己写一堆RAG、Agent代码，可视化拖拽，快速搭知识库问答、AI应用的开源工具。

## Hermes Agent(函数调用驱动的轻量级Agent执行器)
一个很轻、主打纯函数调用的Agent运行器，不搞复杂的图、不搞重框架，模型输出函数调用就直接执行。

工作流程
用户提问 → 大模型输出函数调用指令 → Hermes解析这个指令 → 执行对应的工具函数 → 拿到返回结果塞回模型 → 模型继续思考，直到不用调用工具、给出最终答案。

## 了解框架（Llanmalndex/AutoGen)
两个框架，侧重点完全不一样。LlamaIndex主打RAG检索；AutoGen主打多Agent互相聊天协作。
### LlamaIndex 
1. 核心定位：专门为知识库、检索增强而生的框架。

2. 能做什么：文档加载、文本分割、Embedding、向量入库、检索、重排、RAG问答。它的所有组件都是围绕“怎么把外部资料喂给大模型”设计的。

### AutoGen 
1. 核心定位：多智能体对话框架，让多个AI互相聊天、讨论、辩论。

2. 可以理解为：开好几个Agent，它们可以互相发消息、来回商量，一起啃难题。

比如：一个写代码Agent，一个查bug Agent，一个评审Agent，它们互相对话

# MCP协议与工具集成(深入理解)

## Model Context Protocol核心概念（Tool/Resource)

### Tool（工具） 
就是能执行动作、干活的能力，会产生变化、发起请求。
简单来说就是调用之后要做一件事。

### Resource（资源）
就是只读的信息源，只拿来读数据，不会改动任何东西。
简单说就是给模型看的资料、文件、数据流。

## JSON-RPC 2.0通信协议
RPC = 远程过程调用。简单来说就是你在本地，调用另一台机器上的一个方法。

JSON-RPC，就是请求和返回全部用JSON字符串，简单好解析。

## FastMCP SDK构建MCP Server
FastMCP 是帮你快速写 MCP 服务端的工具，不用手动写 JSON-RPC 那些底层消息。

## Skills编排与MCP集成
Skills就是一组工具的编排流程，MCP负责统一接入工具，两者搭配，让Agent可以按顺序、按条件调用一堆外部能力。
