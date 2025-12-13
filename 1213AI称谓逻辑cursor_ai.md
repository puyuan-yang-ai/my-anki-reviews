# Ai称谓逻辑需求分析与澄清
_Exported on 12/13/2025 at 10:58:23 GMT+8 from Cursor (2.1.39)_

---

**User**

在和ai进行交互的时候，问他是谁的时候，要是你的职业发展顾问Comet GPT，你也可以叫我彗星老师，在这个场景中，有两个称谓，Comet GPT，和 彗星老师。



1 请问你明白我这个逻辑了吗？请用自己的话来复述逻辑，总结功能的核心目标。

2 用列表形式拆解为具体可实现的功能点。

3 你存在哪些疑问，向我提出具体问题来澄清这些点，确保双方对需求理解一致之后再写代码。

---

**Cursor**

## 1. 逻辑复述与核心目标

我理解的逻辑是：

**场景**：在你的职业发展相关项目（cv-project）中，用户会与AI进行交互。

**核心需求**：当用户询问AI的身份（比如"你是谁？"、"请自我介绍"等问题）时，AI需要以一个特定的人设/角色来回答——即 **"职业发展顾问 Comet GPT"**，用户也可以亲切地称呼它为 **"彗星老师"**。

**核心目标**：赋予AI一个明确的角色身份，让AI在职业发展咨询场景中有一个专业且亲和的人设，增强用户体验和信任感。

---

## 2. 功能点拆解

| 序号 | 功能点 | 说明 |
|------|--------|------|
| 1 | **身份识别触发** | 识别用户询问身份的意图（如"你是谁"、"你叫什么"、"介绍一下你自己"等） |
| 2 | **角色人设定义** | 定义AI的角色属性：名称、称呼、职能描述、语言风格等 |
| 3 | **自我介绍模板** | 编写标准的自我介绍话术，包含"Comet GPT"和"彗星老师"两个称谓 |
| 4 | **System Prompt配置** | 在AI的系统提示词中注入角色身份信息 |
| 5 | **称谓一致性** | 确保AI在整个对话过程中保持角色身份的一致性 |

---

## 3. 我的疑问与澄清问题

为了确保需求理解一致，我有以下问题想和你确认：

### 关于触发条件
1. **触发范围**：只有在用户明确询问"你是谁"时才触发自我介绍，还是AI在对话开始时就主动自我介绍？

### 关于角色设定
2. **角色深度**：除了名字，"Comet GPT/彗星老师"是否还需要其他人设信息？比如：
   - 专业背景描述（如"拥有多年职业规划经验"）
   - 性格特点（如"专业、耐心、鼓励性"）
   - 能力范围（如"简历优化、面试辅导、职业规划"）

3. **两个称谓的使用场景**："Comet GPT"和"彗星老师"是否有使用偏好？比如正式场合用前者，亲切互动用后者？

### 关于技术实现
4. **实现位置**：这个身份设定是要加在哪里？
   - 是在调用AI API时的 System Prompt 中配置？
   - 还是在现有代码的某个特定模块中？
   - 是否已有相关的prompt模板文件？

5. **适用范围**：这个人设是只用于某个特定功能（如面试报告对话），还是整个项目中所有AI交互都需要应用？

---

请你逐一回答这些问题，我们对齐需求后再进行代码编写。

---

**User**

1 不仅是用户明确询问"你是谁"时触发自我介绍，并且在AI在对话开始时就主动自我介绍

2 除了名字，"Comet GPT/彗星老师"暂时不需要其他人设信息

3 "Comet GPT"和"彗星老师"需要同时使用

5 实现位置不太确定，你怎么看？

5 我感觉只要是进行交互，都需要进行身份设定吧？你咋看呢？因为"Comet GPT"和"彗星老师" 是我们公司的产品

---

**Cursor**

好的，感谢你的澄清！让我先了解一下你的项目结构，以便给出合理的实现建议。

```python
{"role": "system", "content": "你的资料如下 姓名:自猎彗星,英文名:cometGPT,说明:自猎科技自研AI大模型"},
```

让我继续探索其他AI交互模块，看看项目中还有哪些地方涉及AI对话。

```python
   {"role": "system", "content": "你的资料如下 姓名:自猎彗星,英文名:cometGPT,说明:自猎科技自研AI大模型"},
```

非常好，我已经完整了解了你的项目结构。让我总结需求并给出实现建议。

---

## 需求确认总结

| 项目 | 确认内容 |
|------|----------|
| **触发时机** | ① 对话开始时主动自我介绍 ② 用户询问"你是谁"时触发 |
| **角色名称** | **Comet GPT**（英文名）+ **彗星老师**（中文称呼），同时使用 |
| **人设深度** | 暂时只需名称，不需其他背景设定 |
| **适用范围** | 所有AI交互场景都需要统一身份（公司产品） |

---

## 项目现状分析

我发现你的项目中已经有一个类似的身份设定，在 `sdk/career/intent_prompt.py` 第72行：

```71:73:E:\Python Code\cv-project\project_code\career\code\sdk\career\intent_prompt.py
            messages = [
                {"role": "system", "content": "你的资料如下 姓名:自猎彗星,英文名:cometGPT,说明:自猎科技自研AI大模型"},
                {"role": "user", "content": prompt}
```

**问题**：
1. 名称是 "自猎彗星/cometGPT"，需要改为 "Comet GPT/彗星老师"
2. 只在"闲聊"场景使用，其他模块没有这个身份设定
3. 项目中有 **28处** `system prompt` 定义，分散在不同服务模块

---

## 实现方案建议

针对"实现位置"问题，我建议采用 **集中配置方案**：

### 方案：创建统一的身份配置模块

```
project_code/career/code/
├── app/
│   ├── config.py          # 现有配置
│   └── identity.py        # 新增：身份人设配置 ⭐
```

**优点**：
- 🎯 **单点维护**：所有身份信息集中管理，后续修改只需改一处
- 🔌 **易于扩展**：未来添加更多人设信息（背景、技能、语气）方便
- 📦 **复用性强**：所有服务模块可统一引用

### 核心功能点

| 序号 | 功能 | 说明 |
|------|------|------|
| 1 | `IDENTITY_INFO` | 定义身份常量（名称、称呼） |
| 2 | `get_identity_prompt()` | 生成包含身份的 system prompt 片段 |
| 3 | `get_greeting()` | 生成开场自我介绍话术 |
| 4 | `detect_identity_query()` | 检测用户是否在询问身份（可选） |

### 示例代码结构

```python
# identity.py

# 身份常量
IDENTITY = {
    "name_en": "Comet GPT",
    "name_cn": "彗星老师",
    "company": "自猎科技"
}

def get_identity_prompt():
    """获取身份描述，用于注入 system prompt"""
    return f"你是{IDENTITY['company']}的职业发展顾问 {IDENTITY['name_en']}，用户也可以亲切地叫你{IDENTITY['name_cn']}。"

def get_greeting():
    """获取开场自我介绍"""
    return f"你好！我是你的职业发展顾问 {IDENTITY['name_en']}，你也可以叫我{IDENTITY['name_cn']}。很高兴能帮助你！"
```

---

## 我的建议

**关于问题4（实现位置）**：建议采用上述集中配置方案，创建 `identity.py` 模块。

**关于问题5（适用范围）**：你说得对！既然 "Comet GPT/彗星老师" 是公司产品，所有AI交互都应该带有这个身份。我建议：

1. **对话型交互**（如简历优化对话、面试）：对话开始时主动自我介绍
2. **生成型交互**（如简历生成、岗位推荐）：在 system prompt 中注入身份，但不需要开场白

---

## 最后确认

在我开始编写代码前，请确认：

1. ✅ 身份名称确认：**Comet GPT** + **彗星老师**
2. ❓ 开场白示例是否需要调整？（默认："你好！我是你的职业发展顾问 Comet GPT，你也可以叫我彗星老师。很高兴能帮助你！"）
3. ❓ 是否需要区分不同场景使用不同的开场白？（比如面试场景 vs 简历优化场景）

请确认或补充，我们就可以开始实现了！

```plaintext
            messages = [
                {"role": "system", "content": "你的资料如下 姓名:自猎彗星,英文名:cometGPT,说明:自猎科技自研AI大模型"},
                {"role": "user", "content": prompt}
```

```plaintext
project_code/career/code/
├── app/
│   ├── config.py          # 现有配置
│   └── identity.py        # 新增：身份人设配置 ⭐
```

```python
# identity.py

# 身份常量
IDENTITY = {
    "name_en": "Comet GPT",
    "name_cn": "彗星老师",
    "company": "自猎科技"
}

def get_identity_prompt():
    """获取身份描述，用于注入 system prompt"""
    return f"你是{IDENTITY['company']}的职业发展顾问 {IDENTITY['name_en']}，用户也可以亲切地叫你{IDENTITY['name_cn']}。"

def get_greeting():
    """获取开场自我介绍"""
    return f"你好！我是你的职业发展顾问 {IDENTITY['name_en']}，你也可以叫我{IDENTITY['name_cn']}。很高兴能帮助你！"
```

---

**User**

开场白示例需要调整，使用你提到的默认的格式
不用区分不同场景使用不同的开场白

上面是我的确认的信息，请帮我严格按照这个方案来修改相应的代码，完整follow前面修改方案的内容，不用引入不必要的额外的冗余的修改，修改之后，帮我梳理一下修改后的内容，进而方便我double check正确性
请额外注意：
1 严格遵循前面的方案的具体要求
2 不擅自添加你认为"有用"但我没有要求的功能
3 专注于我明确指出的需求
4 请不要往代码里面添加额外的md文件来总结修改内容。 


---

**Cursor**

我来按照方案严格执行修改。

```python
# 身份常量
IDENTITY = {
    "name_en": "Comet GPT",
    "name_cn": "彗星老师",
    "company": "自猎科技"
}


def get_identity_prompt():
    """获取身份描述，用于注入 system prompt"""
    return f"你是{IDENTITY['company']}的职业发展顾问 {IDENTITY['name_en']}，用户也可以亲切地叫你{IDENTITY['name_cn']}。"


def get_greeting():
    """获取开场自我介绍"""
    return f"你好！我是你的职业发展顾问 {IDENTITY['name_en']}，你也可以叫我{IDENTITY['name_cn']}。很高兴能帮助你！"

```

```python
import asyncio
import json
import logging
from datetime import datetime

from openai import OpenAI

from app.config import comet_config
from app.identity import get_identity_prompt
from sdk.career.agent.resume_optimize import resume_optimize_action

llm = OpenAI(
    api_key=comet_config['api_key'],
    base_url=comet_config['api_uri']
)


async def detect_intent(user_input: str,desc:str) -> dict:
    prompt = f"""
请根据用户对话信息判断用户意图，并以 **严格 JSON 格式** 返回结果。

可选意图：
- resume_optimize: 用户希望优化简历
- generate_job: 用户希望生成一个虚拟岗位
- chat: 闲聊或信息不足

###用户对话
{desc}
其他资料
{user_input}


### 输出json格式（必须严格遵守）：
{{
  "intent": "resume_optimize",
  "confidence": 0.95,
  "reason": "简要解释判断依据"
}}
"""
    logging.info(f"时间：[{datetime.now().strftime('%Y-%m-%d %H:%M:%S')}]--{prompt}")
    yield f"data: {json.dumps({'type': 'heartbeat', 'content': '接收请求'}, ensure_ascii=False)}\n\n"
    try:
        messages = [
            {"role": "system", "content": "你是一个意图分类器，请严格按格式输出。"},
            {"role": "user", "content": prompt}
        ]
        response = llm.chat.completions.create(
            seed=6,
            model=comet_config['api_model'],
            temperature=0.0,
            extra_body={"enable_thinking": False},
            response_format={"type": "json_object"},
            messages=messages
        )
        result = response.choices[0].message.content
        jsonobj=json.loads(result)
        logging.info(f"时间：[{datetime.now().strftime('%Y-%m-%d %H:%M:%S')}]--用户意图：{jsonobj}")
        yield f"data: {json.dumps({'type': 'heartbeat', 'content': '处理第一步请求'}, ensure_ascii=False)}\n\n"
        if jsonobj.get("intent")=="resume_optimize":
            try:
                # 获取原始生成器
                async for data in resume_optimize_action(user_input):
                    # 可以在这里加逻辑，比如：
                    # logger.info("发送一段优化结果")
                    # 或者修改 data
                    yield data  # 继续传递
            except Exception as e:
                logging.error(f"时间：[{datetime.now().strftime('%Y-%m-%d %H:%M:%S')}]--用户意图中间层错误：{str(e)}")
                yield f"data: {json.dumps({'error': '中间层错误', 'content': str(e)}, ensure_ascii=False)}\n\n"
        else:
            logging.info(f"<UNK>[{datetime.now().strftime('%Y-%m-%d %H:%M:%S')}]--<其他>{jsonobj}")
            messages = [
                {"role": "system", "content": "你的资料如下 姓名:自猎彗星,英文名:cometGPT,说明:自猎科技自研AI大模型"},
                {"role": "user", "content": prompt}
            ]
            other_response = llm.chat.completions.create(
                seed=6,
                model=comet_config['api_model'],
                temperature=0.0,
                extra_body={"chat_template_kwargs":
                                {"enable_thinking": False }},
                messages=desc,
                stream=True
            )
            print(other_response)
            for chunk in other_response:

                if chunk.choices:
                    #json_chunk= chunk.choices[0]
                    finish_reason = chunk.choices[0].finish_reason
                    if finish_reason!="stop":
                        content = chunk.choices[0].delta.content
                        yield f"data: {json.dumps({'type': 'chunk', 'content': content}, ensure_ascii=False)}\n\n"
                    else:
                        yield f"data: {json.dumps({'type': 'DONE', 'content': "[DONE]"}, ensure_ascii=False)}\n\n"

    except Exception as e:
        logging.error(f"时间：[{datetime.now().strftime('%Y-%m-%d %H:%M:%S')}]--用户意图识别错误：{str(e)}")
        yield f"data: {json.dumps({'type': 'error', 'content': str(e)}, ensure_ascii=False)}\n\n"



if __name__ == '__main__':
    str={
    "basic_info": {
        "advantages_highlights": "1. 10年50人以上搜索/推荐/nlp/AI方向团队管理经验(可以考虑转搜推/AI方向\n2. 熟练掌握搜索引擎/爬虫技术/推荐引擎相关技术栈。\n3. 熟悉掌握LangChain/llama3/ChatGLM3/prompt/fine tuning/DeepSpee d/RAG等工程落地在端测和云测相关运用 ,同时熟悉deepfm、DSSM 、NLP 、LTR、向量检索等传统机器学习模型相关技术。\n4. 对大模型预训练和sft以及LoRA微调有0-1经验,例如llama3指令集微调针对垂直领域效果突出。同时在pretraining 、RLHF中有自己的见解。\n5. 熟悉AIGC以及多模态相关技术融合以及底层GAN网络模型和特征扩散模型。\n6. 对基于事件驱动agent架构有从0-1构建经验。\n7. 丰富的0-1项目构建解决方案经验,应对系统承担千万pv访问和高并发经验。\n8. 丰富的团队梯度建设经验和强大的协作执行能力。"
    },
    "job_expectation": [
        {
            "id": 0,
            "desired_position": "技术总监",
            "expected_salary": "0-0",
            "expected_salary_begin": 0,
            "expected_salary_end": 0,
            "desired_location": "北京市",
            "job_preference": "工作时长  17年\n求职意向  技术总监\n期望城市  北京",
            "work_type": 1
        }
    ],
    "project_experience": [
        {
            "id": 123,
            "project_name": "",
            "begin_date": "",
            "end_date": "",
            "company_name": "",
            "position": "搜索",
            "job_content": "介绍:整个数据量级在 PB 级别以上(已经突破300多亿数据量),每秒平均请求2000\n多次的并发,整个 ES 集群在500台服务器的规模且都是物理机+ssd ,能够保证\n在3秒内提供相应反馈,拥有完善的负载均衡器(nginx+lvs )以及海量 redis 集群\n的缓存策略抵挡大量的热门数据的访问,并且拥有完善的 IDC 机房冷热互备机制\n保证数据的完整性,对外的 CDN 节点可以让流量均匀分布在华中,华南和华北地\n区.\n涉及到技术:spring,lucene,ElasticSearch,uwsgi,NLTK,\nnginx,lvs,nutch,redis,rabbitMQ,mongodb,htmlunit,Selenium等\n搜索开放平台包括以下模块:\n一.应用层面\n基础搜索:(1)搜索运营后台服务(用来干涉搜索排序结果,用户日志统计,广告位设置,自定义词典,商业模式等等)\n(2)搜索智能提示(拼音,简拼,混合英文,智能补全)\n(3)搜索关键词智能纠错,能够提示正确的拼写结果\n(4)精准.全文关键词搜索,自定义高亮选择段\n(5)自定义属性聚类.分组命中\n(6)动态过滤条件以及动态排序算法定制\n(7)自动 cache 用户最近最热的数据\n搜索推荐:\n(1)根据最近浏览记录推荐\n(2)购买历史记录做推荐\n(3)无上下文随机热门数据推荐\n(4)精准根据搜索关键词最相似匹配推荐\n(5)根据用户行为分析推荐\n二.底层平台技术支撑\n1、中文分词\n(1)语料标记(人工处理)\n(2)专业词典建立\n(3)分词精确度提升和优化\n(4)词库运营平台\n2、业务日志收集\n(1)错误日志收集分析\n(2)业务日志收集分析(用户行为数据和交易数据)\n3、数据挖掘分析\n(1)用户行为数据分析\n(2)商品相关数据分析\n(3)用户画像\n(4)商品画像\n三.运维支撑\n(1)消息平台\n(2)日志平台\n(3)ES 服务器监控平台\n(4)任务调度管理平台",
            "project_desc": ""
        },
        {
            "id": 456,
            "project_name": "le123全网搜索",
            "begin_date": "",
            "end_date": "",
            "company_name": "",
            "position": "",
            "job_content": "涉及到技术:lucene,hbase,spring,memcache,redis,nginx,\nxpath,htmlUnit,selenium等等\n(1)整体采用了lucene+hbase的技术架构模式,lucene的分布式全部由我们自己\n实现,并没有采用框架,底层索引使用了fastDFS文件系统存储,数据采用hbase\n保存,web框架采用springMVC\n(2)热门数据访问的缓存依然是使用了memcache+redis的模式保证\n(3)负载均衡采用统一的nginx的ip对应的hash,只保留了一个数据恢复节点,没\n有灾备措施,但是有压力节点切换的功能和基于zookeeper服务的心跳检测\n(4)对录入的新数据或者抓取来的数据,统一都是走实时搜索接口,即缓存索引\n(5)抓取服务依然是使用了,htmlunit+Selenium组合,保证浏览器能看到的,就\n一定能抓取下来,尤其是对js异步服务或延迟加载的服务\n(6)日常索引更新采用公司内部消息队列框架cbase(没有研究其原理)",
            "project_desc": ""
        },
        {
            "id": 67,
            "project_name": "旧宝网二手购物商城(B 2 C型电子商务)",
            "begin_date": "2008.01",
            "end_date": "2008.04",
            "company_name": "",
            "position": "",
            "job_content": "MyEclipse5.5+Tomcat6.0+MySql+svn 开发技术:\nStruts1.2+Hibernate3.0+JSP+JavaBean+Ajax+MySql+JS树形控\n件.+DWR+webservice 项目描述:该项目主要为旧宝网商城提供了一套网上购物\n平台。为了顺应电子商务模式的兴起,构建一个网上选择商品与购买商品的中心\n平台,将互联网应用层的各种增值应用汇接到本平台上来,依托本平台实现了网\n上购物与网上支付等多项功能 个人负责模块:后台商品管理以及购物模块 模块介\n绍: 1.后台管理模块。 2.商品浏览模块。 3.购物模块:模拟真实购物场景实\n现网上购物功能。 4.商品管理模块。 5.会员留言模块:客户对商品的反馈信\n息,以及客户要求订购的新商品信息。 6.商品销售统计模块 7.易宝支付接口对\n接",
            "project_desc": ""
        },
        {
            "id": 34,
            "project_name": "三家互联网公司项目总结",
            "begin_date": "2009.08",
            "end_date": "2016.03",
            "company_name": "",
            "position": "高级架构师",
            "job_content": "项目描述:\n1.《金山云》快盘商业版平台服务以及文件搜索引擎的设计和分布式抓取....省略\n2.《乐视网》大媒资系统和le123全网视频搜索服务.....省略\n3.《58同城》日租垂直搜索平台架构设计以及技术方案实施,整个搜索平台基于\nlucene3.6版本进行扩展,采用了分布式实时搜索框架zoie,能够处理多节点索\n引任务以及合并优化的工作。索引服务拥有读写分离,分片,rest服务一致性\nhash,负载均衡,热备份缓存索引等功能。使用zookeeper来管理服务器节点心\n跳检测来保证单点故障出现的几率降到最低。利用消息中间件和阻塞队列来保证\n多线程下数据同步。使用分布式并行运算框架TwitterStorm执行增量索引和更新\n索引任务以及事件异常扑捉,并可以实时监控服务life,来通知路由动态切换\nservice地址。前端采用分布式memcache+集群redis来缓存热门关键词数据和热\n门索引文件.搜索业务框架构建在springMVC之上,采用restful服务提供SOA服\n务,使服务与服务之间不在相互依赖,整个通信采用thrift跨语言高效的对象序列\n化模型进行交互。\n4.《58同城》分布式多线程采集和分析框架:基于浏览器内核组件\nHtmlunitDriver(linux)以及Selenium插件二次开发(windows)完美支持模拟IE,\nfirefox,chrome浏览器特性,能够很好的解决抓取javascript延迟加载,js动态\n构建DOM树,ajax请求,URL重定向,flash,MP3,视频,图片等等数据。它构\n建在多节点,多任务生产,在前台模版配置和机器自动识别的特点具有智能的判\n断和分析网页的能力。\n爬虫和搜索建立索引也息息相关,能够支持对抓取下来的数据分析完后实时推送\n给lucene构建索引。\n5.《58同城》智能图片分析推荐:根据全网抓取过来的短租图片通过感知hash算\n法建立图片指纹索引,对用户上传上来的图片提供精确地址匹配和相似度推荐,\n能够对全文检索提供额外的查询方式。\n6.《58同城》OSS数据支持平台:除了抓取的数据给自己部门使用外,还需要给58\n主站提供数据来源,整体平台采用流水作业的方式经过层层数据把控最后把合适\n的数据推送到mongodb里面去,完成状态流转最后输送到主站。\n7.《英特可》:定制客户公司OA系统",
            "project_desc": ""
        }
    ],
    "work_experience": [
        {
            "id": 10,
            "company_name": "北京字节跳动科技有限公司",
            "begin_date": "2024.03.01",
            "end_date": "至今",
            "position": "ai技术总监",
            "responsibility_performance": "核心工作:算法、工程、产品、售前4手抓,最全的mvp团队,40%精力在技术\n上底层打磨在构建技术团队、技术选型、产品技术架构、aigc和多模态端/云侧构\n建方案均下场和团队手把手参与,60%在产品和售前团队上在产品商业化变现解\n决方案、出海白皮书、对代理商和渠道商上提供最佳poc体验,对竞标、版权谈\n判都亲力亲为。\n核心业务背景:\n豆包大模型以及多模态应用在端测落地布局,配合云端火山引擎平台一站式算力\n支撑和全球领先的ai大模型套件打造ToB/ToC的领先的aigc应用。\n业务目标:\n依托豆包大模型、既梦aigc平台为基础打造数字人、数字闺蜜机、数字手办/玩具\n完整的生态链产品,为出海数字化产品业务提供一站式解决方案。\n具体工作内容:\n• 大规模对豆包大模型进行人设角色以及情感识别进行sft微调+RLHF干预。\n• 在既梦平台上针对二次元动漫(国漫、日漫、DC类型等)、游戏、影视作品等\n中人物、场景、背景、肢体行为动作素材语料进行指令集sft,训练既梦对二次元\n2d数字人在形象、肢体动作和模仿能力进行定制结合火山引擎大模型提供的\nASR+TTS以及音色克隆能力还原动漫人物真实情感表露,通过图生视频方式结合\n精调的prompt+复合型agent工作流完成无限创意的数字人、二次元数字手办/玩\n具等应用。\n• 基于豆包大模型在千机千卡大规模集群GPU上对语言模型进行预训练满足垂直领\n域大模型对教培方向、情感陪伴、家庭娱乐、LOT等领域支撑。\n• 重新对coze进行了垂直领域定制开发,加强了长短记忆模块采用上下文关联的\n智能窗口滑动,支持多个工作流穿插进行状态机切换,支持更多的基于\nlangchain协议的sdk集成,对端侧部署提供一键化支持。\n• 重新对端侧RAG能力进行加强,引入全新的Graph知识图谱对RAG理解上进行知\n识实体多路径召回融合,在bce-embedding精排模型上提供知识图谱召回权重\n加权。\n• 构建一站式aigc生成式sass平台(可私有化部署)提供各类主流ip厂商的游戏、\n动漫角色生成、声优克隆、完美人设对话模仿、换装、特色肢体动作生成和发\n布。\n最终获得成果:\n1. 团队成员一手构建拉起来的绝对超强专业的ai方向的算法和工程的同学(70人团队其中不乏有3-1/2顶尖人才)\n2. 亲自跑海外为公司争取到一线动漫/游戏顶部ip厂商授权(包括任天堂、万代南梦宫、SE、Konami等)\n3. 带领团队在8月底完成海外平台内部测试版上线"
        },
        {
            "id": 22,
            "company_name": "前程无忧(51job)",
            "begin_date": "2023.01.01",
            "end_date": "2024.03",
            "position": "ai研发总监",
            "responsibility_performance": "内容:\n是时间为1年半的技术顾问协议!\n全面负责51job 搜推团队的整体规划和建设,重新构建搜推底层算法模型、特征\n工程、NLP策略、全面接入多模态 ai 大模型来打造新一代的智能人岗匹配搜推引\n擎。\n业务目标:\n• 打造百人的顶尖搜推算法团队\n• 重构现有搜推工程、算法模型底层架构和升级\n• 基于 LLaMA 、文心千帆、通义千问200b大模型为底座打造属于51job 专属招聘\n赛道通用大模型\n具体工作内容:\n1. 经过大半年努力成功将搜推、广告、大数据团队规模提升到百人规模,覆盖搜\n广推、nlp 、知识图谱、流量变现、爬虫、ai 大模型等相关领域人才。大部分来\n源于一线大厂核心工程师!\n2. 全面重构了搜推底层架构设计从 QP 、多路召回、粗排、精排、LTR、DSSM 语\n义向量排序到动态交叉特征模型 DeepFM 引入重新定义了一整套解决方案,同时\n针对特征工程、标签生产挖掘、搜推质量评估中心等入手进行了大量系统工程升\n级和建设。\n3. 通过引入 QAnything 大模型直接对搜索体验进行了大规模升级打造属于51job\n专属 RAG 增强搜索直接对传统分词召回、语义召回进行了颠覆。基于引入\nLLaMA 大模型构建大量人岗匹配语料在此基础上实现自己的 Prompt 工程对自动\n生成简历、jd 、核心技能优势以及职场知识自动问答和针对销售语音客服都有了\n质的飞跃。\n4. 针对招聘赛道最火热的 AI评测+AI 面试上我们自研了51智能面试助手学习的是\n全行业的知识语料库从简历分析、到专业知识考察、沟通理解技巧、面部表情识\n别、声音情感分析以及性格分析全方位来帮助B端用户初筛简历、初面的全链路接\n入,同时技术上融入了3d数字人、Audio2Face、FT-w2v2-ser、AffectNet、\nTTS、ASR 等前沿的算法模型。\n最终获得成果:\n• 算法技术团队从原有30人扩充到现有的120人的规模。\n• 从新的人才和技术体系的引入全面提升了51job 的技术生产力。\n• 带领技术团队整体迈入 AI 时代、几乎所有业务中台、数据中台都在使用 AI 进行\n重构业务。\n• 5ijob 整体日活从23年平均280万提升到现如今的440万左右。\n业绩:\n1. 51job23年度优秀高管、火凤凰涅槃重生计划功勋奖人物\n2. 全面升级了51job 搜推团队打造专业技术驱动团队"
        },
        {
            "id": 34,
            "company_name": "北京华品博睿网络技术有限公司(boss直聘)",
            "begin_date": "2019.10.01",
            "end_date": "2023.01",
            "position": "搜索主任架构师",
            "responsibility_performance": "内容:\n1.负责 boss直聘 app 、小程序、pc 端整体搜索业务,承担1000多万 uv和3000\n多万 pv 搜索访问、pb 级别索引量级、强大的 nlp 处理能力辅助用户完成精准的\n搜索定位,提供4个9高可用性架构打造,近实时搜索的体验!在职期间带领团队\n在搜索意图分析优化、行业职类分词改造、LTR 、分类模型识别、NER 上做了深\n入的定制化策略!\n2.通过线上1000多个a/b测试在召回、粗排、精排、强插、补充召回、模型排序\n上不断尝试创新为 ctr 转化率大幅度提升起到关键性作用!\n3.随着用户 query 不断的增多,通过模型学习到新词发现、公司专有名词、职位\n专有名词越来越精准,为后期在卡片分类、职位识别、query 纠错上提供支撑能\n力!\n4.除了在 boss 这边带领团队负责搜索业务外,我们还打造了 boss 唯一的职类知\n识图谱平台,通过我们不断对职类体系和用户认知体系的加深以及对职类的理\n解,构建了庞大的职类知识库关联牛人、boss、job 、城市、职位、公司、学\n校、行业等实体卡片通过百川汇海的关系\n链将其通过图的方式构建出网状关系链结构经过2年多发展拥有近百亿的实体、\n500多亿的关系边、几乎覆盖整个行业、职类体系,对上10万的职位有丰富的知\n识查阅功能!\n5.全面管理搜索和 nlp 团队日常工作,团队建设,绩效考核等工作!\n业绩:\n在公司效力的期间连续3年获得B+绩效考核、优秀管理标兵、最佳面试官等荣\n誉!培养了大量的技术小迷弟,谈不上多成功的业绩但是离职的时候几十位同事\n为我送行!"
        },
        {
            "id": 66,
            "company_name": "北京思源互联科技有限公司",
            "begin_date": "2017.03.01",
            "end_date": "2019.09",
            "position": "高级研发总监",
            "responsibility_performance": "工作描述:\n一.搜索与大数据事业群\n1.负责智慧城市百亿商圈数据挖掘和搜索与推荐\n2.负责构建超大型搜索引擎分析平台支撑用户每天千万次搜索请求涉及到地理、\n酒店、旅游、美食、政务办公、教育精确搜索.对 query 意图识别、query 改写和\n精确召回有了大量的优化.对机器学习排序(LTR )提高用户点击率和转换率起到\n决定性作用.通过 HMM 对文本命名实体识别来区分关键性名词来做搜索提升体验.\n3.根据海量用户搜索日志构建用户行为画像.对用户做实时和离线推荐数据建模\n4.负责爬虫爬取海量搜索引擎以及内容平台数据构建我们基础数据仓库每天千万\n级别数据爬取\n5.基本实现自定义搜索引擎打分策略和定制行业化分词算法,对智慧城市相关行\n业做了一定深入的定制策略\n二.反垃圾邮件网关部\n1.打通\"秘邮产品\"与邮箱的消息互通,打造自己的邮箱服务与反垃圾网关服务\n2.通过基于机器学习与统计算法分析海量邮件,能够准确区分正常邮件与垃圾邮\n件并且能够对垃圾邮件做分类例如:广告、色情、政治等 (通过朴素贝叶斯+knn+k-mean 算法建模)\n3.对邮件网关产生的 ddos 、退信攻击、跨域脚本攻击防御\n4.对流行的杀毒软件集成,抵御病毒攻击"
        },
        {
            "id": 57,
            "company_name": "北京小米移动软件有限公司",
            "begin_date": "2015.10.01",
            "end_date": "2017.03",
            "position": "搜索引擎架构师",
            "responsibility_performance": "负责小米优选商城搜索业务模块,带领团队打造从0到1整体搜索架构设计到上线\n的整个过程!由于用户量不大,数据规模小,没什么亮点,很多实践需要积累数\n据量级才能付出实现,就这样吧!"
        }
    ],
    "optimize_type":"work_experience"
}
    aa=asyncio.run(detect_intent(str,"简历优化"))
    print("aa",aa)
```

```python
import asyncio
import json
import logging
from datetime import datetime

from openai import OpenAI

from app.config import comet_config
from app.identity import get_identity_prompt
from sdk.career.agent.resume_optimize import resume_optimize_action

llm = OpenAI(
    api_key=comet_config['api_key'],
    base_url=comet_config['api_uri']
)


async def detect_intent(user_input: str,desc:str) -> dict:
    prompt = f"""
请根据用户对话信息判断用户意图，并以 **严格 JSON 格式** 返回结果。

可选意图：
- resume_optimize: 用户希望优化简历
- generate_job: 用户希望生成一个虚拟岗位
- chat: 闲聊或信息不足

###用户对话
{desc}
其他资料
{user_input}


### 输出json格式（必须严格遵守）：
{{
  "intent": "resume_optimize",
  "confidence": 0.95,
  "reason": "简要解释判断依据"
}}
"""
    logging.info(f"时间：[{datetime.now().strftime('%Y-%m-%d %H:%M:%S')}]--{prompt}")
    yield f"data: {json.dumps({'type': 'heartbeat', 'content': '接收请求'}, ensure_ascii=False)}\n\n"
    try:
        messages = [
            {"role": "system", "content": "你是一个意图分类器，请严格按格式输出。"},
            {"role": "user", "content": prompt}
        ]
        response = llm.chat.completions.create(
            seed=6,
            model=comet_config['api_model'],
            temperature=0.0,
            extra_body={"enable_thinking": False},
            response_format={"type": "json_object"},
            messages=messages
        )
        result = response.choices[0].message.content
        jsonobj=json.loads(result)
        logging.info(f"时间：[{datetime.now().strftime('%Y-%m-%d %H:%M:%S')}]--用户意图：{jsonobj}")
        yield f"data: {json.dumps({'type': 'heartbeat', 'content': '处理第一步请求'}, ensure_ascii=False)}\n\n"
        if jsonobj.get("intent")=="resume_optimize":
            try:
                # 获取原始生成器
                async for data in resume_optimize_action(user_input):
                    # 可以在这里加逻辑，比如：
                    # logger.info("发送一段优化结果")
                    # 或者修改 data
                    yield data  # 继续传递
            except Exception as e:
                logging.error(f"时间：[{datetime.now().strftime('%Y-%m-%d %H:%M:%S')}]--用户意图中间层错误：{str(e)}")
                yield f"data: {json.dumps({'error': '中间层错误', 'content': str(e)}, ensure_ascii=False)}\n\n"
        else:
            logging.info(f"<UNK>[{datetime.now().strftime('%Y-%m-%d %H:%M:%S')}]--<其他>{jsonobj}")
            messages = [
                {"role": "system", "content": get_identity_prompt()},
                {"role": "user", "content": prompt}
            ]
            other_response = llm.chat.completions.create(
                seed=6,
                model=comet_config['api_model'],
                temperature=0.0,
                extra_body={"chat_template_kwargs":
                                {"enable_thinking": False }},
                messages=desc,
                stream=True
            )
            print(other_response)
            for chunk in other_response:

                if chunk.choices:
                    #json_chunk= chunk.choices[0]
                    finish_reason = chunk.choices[0].finish_reason
                    if finish_reason!="stop":
                        content = chunk.choices[0].delta.content
                        yield f"data: {json.dumps({'type': 'chunk', 'content': content}, ensure_ascii=False)}\n\n"
                    else:
                        yield f"data: {json.dumps({'type': 'DONE', 'content': "[DONE]"}, ensure_ascii=False)}\n\n"

    except Exception as e:
        logging.error(f"时间：[{datetime.now().strftime('%Y-%m-%d %H:%M:%S')}]--用户意图识别错误：{str(e)}")
        yield f"data: {json.dumps({'type': 'error', 'content': str(e)}, ensure_ascii=False)}\n\n"



if __name__ == '__main__':
    str={
    "basic_info": {
        "advantages_highlights": "1. 10年50人以上搜索/推荐/nlp/AI方向团队管理经验(可以考虑转搜推/AI方向\n2. 熟练掌握搜索引擎/爬虫技术/推荐引擎相关技术栈。\n3. 熟悉掌握LangChain/llama3/ChatGLM3/prompt/fine tuning/DeepSpee d/RAG等工程落地在端测和云测相关运用 ,同时熟悉deepfm、DSSM 、NLP 、LTR、向量检索等传统机器学习模型相关技术。\n4. 对大模型预训练和sft以及LoRA微调有0-1经验,例如llama3指令集微调针对垂直领域效果突出。同时在pretraining 、RLHF中有自己的见解。\n5. 熟悉AIGC以及多模态相关技术融合以及底层GAN网络模型和特征扩散模型。\n6. 对基于事件驱动agent架构有从0-1构建经验。\n7. 丰富的0-1项目构建解决方案经验,应对系统承担千万pv访问和高并发经验。\n8. 丰富的团队梯度建设经验和强大的协作执行能力。"
    },
    "job_expectation": [
        {
            "id": 0,
            "desired_position": "技术总监",
            "expected_salary": "0-0",
            "expected_salary_begin": 0,
            "expected_salary_end": 0,
            "desired_location": "北京市",
            "job_preference": "工作时长  17年\n求职意向  技术总监\n期望城市  北京",
            "work_type": 1
        }
    ],
    "project_experience": [
        {
            "id": 123,
            "project_name": "",
            "begin_date": "",
            "end_date": "",
            "company_name": "",
            "position": "搜索",
            "job_content": "介绍:整个数据量级在 PB 级别以上(已经突破300多亿数据量),每秒平均请求2000\n多次的并发,整个 ES 集群在500台服务器的规模且都是物理机+ssd ,能够保证\n在3秒内提供相应反馈,拥有完善的负载均衡器(nginx+lvs )以及海量 redis 集群\n的缓存策略抵挡大量的热门数据的访问,并且拥有完善的 IDC 机房冷热互备机制\n保证数据的完整性,对外的 CDN 节点可以让流量均匀分布在华中,华南和华北地\n区.\n涉及到技术:spring,lucene,ElasticSearch,uwsgi,NLTK,\nnginx,lvs,nutch,redis,rabbitMQ,mongodb,htmlunit,Selenium等\n搜索开放平台包括以下模块:\n一.应用层面\n基础搜索:(1)搜索运营后台服务(用来干涉搜索排序结果,用户日志统计,广告位设置,自定义词典,商业模式等等)\n(2)搜索智能提示(拼音,简拼,混合英文,智能补全)\n(3)搜索关键词智能纠错,能够提示正确的拼写结果\n(4)精准.全文关键词搜索,自定义高亮选择段\n(5)自定义属性聚类.分组命中\n(6)动态过滤条件以及动态排序算法定制\n(7)自动 cache 用户最近最热的数据\n搜索推荐:\n(1)根据最近浏览记录推荐\n(2)购买历史记录做推荐\n(3)无上下文随机热门数据推荐\n(4)精准根据搜索关键词最相似匹配推荐\n(5)根据用户行为分析推荐\n二.底层平台技术支撑\n1、中文分词\n(1)语料标记(人工处理)\n(2)专业词典建立\n(3)分词精确度提升和优化\n(4)词库运营平台\n2、业务日志收集\n(1)错误日志收集分析\n(2)业务日志收集分析(用户行为数据和交易数据)\n3、数据挖掘分析\n(1)用户行为数据分析\n(2)商品相关数据分析\n(3)用户画像\n(4)商品画像\n三.运维支撑\n(1)消息平台\n(2)日志平台\n(3)ES 服务器监控平台\n(4)任务调度管理平台",
            "project_desc": ""
        },
        {
            "id": 456,
            "project_name": "le123全网搜索",
            "begin_date": "",
            "end_date": "",
            "company_name": "",
            "position": "",
            "job_content": "涉及到技术:lucene,hbase,spring,memcache,redis,nginx,\nxpath,htmlUnit,selenium等等\n(1)整体采用了lucene+hbase的技术架构模式,lucene的分布式全部由我们自己\n实现,并没有采用框架,底层索引使用了fastDFS文件系统存储,数据采用hbase\n保存,web框架采用springMVC\n(2)热门数据访问的缓存依然是使用了memcache+redis的模式保证\n(3)负载均衡采用统一的nginx的ip对应的hash,只保留了一个数据恢复节点,没\n有灾备措施,但是有压力节点切换的功能和基于zookeeper服务的心跳检测\n(4)对录入的新数据或者抓取来的数据,统一都是走实时搜索接口,即缓存索引\n(5)抓取服务依然是使用了,htmlunit+Selenium组合,保证浏览器能看到的,就\n一定能抓取下来,尤其是对js异步服务或延迟加载的服务\n(6)日常索引更新采用公司内部消息队列框架cbase(没有研究其原理)",
            "project_desc": ""
        },
        {
            "id": 67,
            "project_name": "旧宝网二手购物商城(B 2 C型电子商务)",
            "begin_date": "2008.01",
            "end_date": "2008.04",
            "company_name": "",
            "position": "",
            "job_content": "MyEclipse5.5+Tomcat6.0+MySql+svn 开发技术:\nStruts1.2+Hibernate3.0+JSP+JavaBean+Ajax+MySql+JS树形控\n件.+DWR+webservice 项目描述:该项目主要为旧宝网商城提供了一套网上购物\n平台。为了顺应电子商务模式的兴起,构建一个网上选择商品与购买商品的中心\n平台,将互联网应用层的各种增值应用汇接到本平台上来,依托本平台实现了网\n上购物与网上支付等多项功能 个人负责模块:后台商品管理以及购物模块 模块介\n绍: 1.后台管理模块。 2.商品浏览模块。 3.购物模块:模拟真实购物场景实\n现网上购物功能。 4.商品管理模块。 5.会员留言模块:客户对商品的反馈信\n息,以及客户要求订购的新商品信息。 6.商品销售统计模块 7.易宝支付接口对\n接",
            "project_desc": ""
        },
        {
            "id": 34,
            "project_name": "三家互联网公司项目总结",
            "begin_date": "2009.08",
            "end_date": "2016.03",
            "company_name": "",
            "position": "高级架构师",
            "job_content": "项目描述:\n1.《金山云》快盘商业版平台服务以及文件搜索引擎的设计和分布式抓取....省略\n2.《乐视网》大媒资系统和le123全网视频搜索服务.....省略\n3.《58同城》日租垂直搜索平台架构设计以及技术方案实施,整个搜索平台基于\nlucene3.6版本进行扩展,采用了分布式实时搜索框架zoie,能够处理多节点索\n引任务以及合并优化的工作。索引服务拥有读写分离,分片,rest服务一致性\nhash,负载均衡,热备份缓存索引等功能。使用zookeeper来管理服务器节点心\n跳检测来保证单点故障出现的几率降到最低。利用消息中间件和阻塞队列来保证\n多线程下数据同步。使用分布式并行运算框架TwitterStorm执行增量索引和更新\n索引任务以及事件异常扑捉,并可以实时监控服务life,来通知路由动态切换\nservice地址。前端采用分布式memcache+集群redis来缓存热门关键词数据和热\n门索引文件.搜索业务框架构建在springMVC之上,采用restful服务提供SOA服\n务,使服务与服务之间不在相互依赖,整个通信采用thrift跨语言高效的对象序列\n化模型进行交互。\n4.《58同城》分布式多线程采集和分析框架:基于浏览器内核组件\nHtmlunitDriver(linux)以及Selenium插件二次开发(windows)完美支持模拟IE,\nfirefox,chrome浏览器特性,能够很好的解决抓取javascript延迟加载,js动态\n构建DOM树,ajax请求,URL重定向,flash,MP3,视频,图片等等数据。它构\n建在多节点,多任务生产,在前台模版配置和机器自动识别的特点具有智能的判\n断和分析网页的能力。\n爬虫和搜索建立索引也息息相关,能够支持对抓取下来的数据分析完后实时推送\n给lucene构建索引。\n5.《58同城》智能图片分析推荐:根据全网抓取过来的短租图片通过感知hash算\n法建立图片指纹索引,对用户上传上来的图片提供精确地址匹配和相似度推荐,\n能够对全文检索提供额外的查询方式。\n6.《58同城》OSS数据支持平台:除了抓取的数据给自己部门使用外,还需要给58\n主站提供数据来源,整体平台采用流水作业的方式经过层层数据把控最后把合适\n的数据推送到mongodb里面去,完成状态流转最后输送到主站。\n7.《英特可》:定制客户公司OA系统",
            "project_desc": ""
        }
    ],
    "work_experience": [
        {
            "id": 10,
            "company_name": "北京字节跳动科技有限公司",
            "begin_date": "2024.03.01",
            "end_date": "至今",
            "position": "ai技术总监",
            "responsibility_performance": "核心工作:算法、工程、产品、售前4手抓,最全的mvp团队,40%精力在技术\n上底层打磨在构建技术团队、技术选型、产品技术架构、aigc和多模态端/云侧构\n建方案均下场和团队手把手参与,60%在产品和售前团队上在产品商业化变现解\n决方案、出海白皮书、对代理商和渠道商上提供最佳poc体验,对竞标、版权谈\n判都亲力亲为。\n核心业务背景:\n豆包大模型以及多模态应用在端测落地布局,配合云端火山引擎平台一站式算力\n支撑和全球领先的ai大模型套件打造ToB/ToC的领先的aigc应用。\n业务目标:\n依托豆包大模型、既梦aigc平台为基础打造数字人、数字闺蜜机、数字手办/玩具\n完整的生态链产品,为出海数字化产品业务提供一站式解决方案。\n具体工作内容:\n• 大规模对豆包大模型进行人设角色以及情感识别进行sft微调+RLHF干预。\n• 在既梦平台上针对二次元动漫(国漫、日漫、DC类型等)、游戏、影视作品等\n中人物、场景、背景、肢体行为动作素材语料进行指令集sft,训练既梦对二次元\n2d数字人在形象、肢体动作和模仿能力进行定制结合火山引擎大模型提供的\nASR+TTS以及音色克隆能力还原动漫人物真实情感表露,通过图生视频方式结合\n精调的prompt+复合型agent工作流完成无限创意的数字人、二次元数字手办/玩\n具等应用。\n• 基于豆包大模型在千机千卡大规模集群GPU上对语言模型进行预训练满足垂直领\n域大模型对教培方向、情感陪伴、家庭娱乐、LOT等领域支撑。\n• 重新对coze进行了垂直领域定制开发,加强了长短记忆模块采用上下文关联的\n智能窗口滑动,支持多个工作流穿插进行状态机切换,支持更多的基于\nlangchain协议的sdk集成,对端侧部署提供一键化支持。\n• 重新对端侧RAG能力进行加强,引入全新的Graph知识图谱对RAG理解上进行知\n识实体多路径召回融合,在bce-embedding精排模型上提供知识图谱召回权重\n加权。\n• 构建一站式aigc生成式sass平台(可私有化部署)提供各类主流ip厂商的游戏、\n动漫角色生成、声优克隆、完美人设对话模仿、换装、特色肢体动作生成和发\n布。\n最终获得成果:\n1. 团队成员一手构建拉起来的绝对超强专业的ai方向的算法和工程的同学(70人团队其中不乏有3-1/2顶尖人才)\n2. 亲自跑海外为公司争取到一线动漫/游戏顶部ip厂商授权(包括任天堂、万代南梦宫、SE、Konami等)\n3. 带领团队在8月底完成海外平台内部测试版上线"
        },
        {
            "id": 22,
            "company_name": "前程无忧(51job)",
            "begin_date": "2023.01.01",
            "end_date": "2024.03",
            "position": "ai研发总监",
            "responsibility_performance": "内容:\n是时间为1年半的技术顾问协议!\n全面负责51job 搜推团队的整体规划和建设,重新构建搜推底层算法模型、特征\n工程、NLP策略、全面接入多模态 ai 大模型来打造新一代的智能人岗匹配搜推引\n擎。\n业务目标:\n• 打造百人的顶尖搜推算法团队\n• 重构现有搜推工程、算法模型底层架构和升级\n• 基于 LLaMA 、文心千帆、通义千问200b大模型为底座打造属于51job 专属招聘\n赛道通用大模型\n具体工作内容:\n1. 经过大半年努力成功将搜推、广告、大数据团队规模提升到百人规模,覆盖搜\n广推、nlp 、知识图谱、流量变现、爬虫、ai 大模型等相关领域人才。大部分来\n源于一线大厂核心工程师!\n2. 全面重构了搜推底层架构设计从 QP 、多路召回、粗排、精排、LTR、DSSM 语\n义向量排序到动态交叉特征模型 DeepFM 引入重新定义了一整套解决方案,同时\n针对特征工程、标签生产挖掘、搜推质量评估中心等入手进行了大量系统工程升\n级和建设。\n3. 通过引入 QAnything 大模型直接对搜索体验进行了大规模升级打造属于51job\n专属 RAG 增强搜索直接对传统分词召回、语义召回进行了颠覆。基于引入\nLLaMA 大模型构建大量人岗匹配语料在此基础上实现自己的 Prompt 工程对自动\n生成简历、jd 、核心技能优势以及职场知识自动问答和针对销售语音客服都有了\n质的飞跃。\n4. 针对招聘赛道最火热的 AI评测+AI 面试上我们自研了51智能面试助手学习的是\n全行业的知识语料库从简历分析、到专业知识考察、沟通理解技巧、面部表情识\n别、声音情感分析以及性格分析全方位来帮助B端用户初筛简历、初面的全链路接\n入,同时技术上融入了3d数字人、Audio2Face、FT-w2v2-ser、AffectNet、\nTTS、ASR 等前沿的算法模型。\n最终获得成果:\n• 算法技术团队从原有30人扩充到现有的120人的规模。\n• 从新的人才和技术体系的引入全面提升了51job 的技术生产力。\n• 带领技术团队整体迈入 AI 时代、几乎所有业务中台、数据中台都在使用 AI 进行\n重构业务。\n• 5ijob 整体日活从23年平均280万提升到现如今的440万左右。\n业绩:\n1. 51job23年度优秀高管、火凤凰涅槃重生计划功勋奖人物\n2. 全面升级了51job 搜推团队打造专业技术驱动团队"
        },
        {
            "id": 34,
            "company_name": "北京华品博睿网络技术有限公司(boss直聘)",
            "begin_date": "2019.10.01",
            "end_date": "2023.01",
            "position": "搜索主任架构师",
            "responsibility_performance": "内容:\n1.负责 boss直聘 app 、小程序、pc 端整体搜索业务,承担1000多万 uv和3000\n多万 pv 搜索访问、pb 级别索引量级、强大的 nlp 处理能力辅助用户完成精准的\n搜索定位,提供4个9高可用性架构打造,近实时搜索的体验!在职期间带领团队\n在搜索意图分析优化、行业职类分词改造、LTR 、分类模型识别、NER 上做了深\n入的定制化策略!\n2.通过线上1000多个a/b测试在召回、粗排、精排、强插、补充召回、模型排序\n上不断尝试创新为 ctr 转化率大幅度提升起到关键性作用!\n3.随着用户 query 不断的增多,通过模型学习到新词发现、公司专有名词、职位\n专有名词越来越精准,为后期在卡片分类、职位识别、query 纠错上提供支撑能\n力!\n4.除了在 boss 这边带领团队负责搜索业务外,我们还打造了 boss 唯一的职类知\n识图谱平台,通过我们不断对职类体系和用户认知体系的加深以及对职类的理\n解,构建了庞大的职类知识库关联牛人、boss、job 、城市、职位、公司、学\n校、行业等实体卡片通过百川汇海的关系\n链将其通过图的方式构建出网状关系链结构经过2年多发展拥有近百亿的实体、\n500多亿的关系边、几乎覆盖整个行业、职类体系,对上10万的职位有丰富的知\n识查阅功能!\n5.全面管理搜索和 nlp 团队日常工作,团队建设,绩效考核等工作!\n业绩:\n在公司效力的期间连续3年获得B+绩效考核、优秀管理标兵、最佳面试官等荣\n誉!培养了大量的技术小迷弟,谈不上多成功的业绩但是离职的时候几十位同事\n为我送行!"
        },
        {
            "id": 66,
            "company_name": "北京思源互联科技有限公司",
            "begin_date": "2017.03.01",
            "end_date": "2019.09",
            "position": "高级研发总监",
            "responsibility_performance": "工作描述:\n一.搜索与大数据事业群\n1.负责智慧城市百亿商圈数据挖掘和搜索与推荐\n2.负责构建超大型搜索引擎分析平台支撑用户每天千万次搜索请求涉及到地理、\n酒店、旅游、美食、政务办公、教育精确搜索.对 query 意图识别、query 改写和\n精确召回有了大量的优化.对机器学习排序(LTR )提高用户点击率和转换率起到\n决定性作用.通过 HMM 对文本命名实体识别来区分关键性名词来做搜索提升体验.\n3.根据海量用户搜索日志构建用户行为画像.对用户做实时和离线推荐数据建模\n4.负责爬虫爬取海量搜索引擎以及内容平台数据构建我们基础数据仓库每天千万\n级别数据爬取\n5.基本实现自定义搜索引擎打分策略和定制行业化分词算法,对智慧城市相关行\n业做了一定深入的定制策略\n二.反垃圾邮件网关部\n1.打通\"秘邮产品\"与邮箱的消息互通,打造自己的邮箱服务与反垃圾网关服务\n2.通过基于机器学习与统计算法分析海量邮件,能够准确区分正常邮件与垃圾邮\n件并且能够对垃圾邮件做分类例如:广告、色情、政治等 (通过朴素贝叶斯+knn+k-mean 算法建模)\n3.对邮件网关产生的 ddos 、退信攻击、跨域脚本攻击防御\n4.对流行的杀毒软件集成,抵御病毒攻击"
        },
        {
            "id": 57,
            "company_name": "北京小米移动软件有限公司",
            "begin_date": "2015.10.01",
            "end_date": "2017.03",
            "position": "搜索引擎架构师",
            "responsibility_performance": "负责小米优选商城搜索业务模块,带领团队打造从0到1整体搜索架构设计到上线\n的整个过程!由于用户量不大,数据规模小,没什么亮点,很多实践需要积累数\n据量级才能付出实现,就这样吧!"
        }
    ],
    "optimize_type":"work_experience"
}
    aa=asyncio.run(detect_intent(str,"简历优化"))
    print("aa",aa)
```

修改完成，没有 linter 错误。

---

## 修改内容梳理

### 1. 新增文件：`app/identity.py`

```1:17:E:\Python Code\cv-project\project_code\career\code\app\identity.py
# 身份常量
IDENTITY = {
    "name_en": "Comet GPT",
    "name_cn": "彗星老师",
    "company": "自猎科技"
}


def get_identity_prompt():
    """获取身份描述，用于注入 system prompt"""
    return f"你是{IDENTITY['company']}的职业发展顾问 {IDENTITY['name_en']}，用户也可以亲切地叫你{IDENTITY['name_cn']}。"


def get_greeting():
    """获取开场自我介绍"""
    return f"你好！我是你的职业发展顾问 {IDENTITY['name_en']}，你也可以叫我{IDENTITY['name_cn']}。很高兴能帮助你！"
```

---

### 2. 修改文件：`sdk/career/intent_prompt.py`

**修改点 1**：新增 import（第 9 行）

```python
# 修改前
from app.config import comet_config
from sdk.career.agent.resume_optimize import resume_optimize_action

# 修改后
from app.config import comet_config
from app.identity import get_identity_prompt
from sdk.career.agent.resume_optimize import resume_optimize_action
```

**修改点 2**：替换 system prompt 中的身份描述（第 71-74 行）

```python
# 修改前
messages = [
    {"role": "system", "content": "你的资料如下 姓名:自猎彗星,英文名:cometGPT,说明:自猎科技自研AI大模型"},
    {"role": "user", "content": prompt}
]

# 修改后
messages = [
    {"role": "system", "content": get_identity_prompt()},
    {"role": "user", "content": prompt}
]
```

---

## 修改说明

| 项目 | 说明 |
|------|------|
| **修改范围** | 2个文件（1个新增，1个修改） |
| **身份名称** | "自猎彗星/cometGPT" → "Comet GPT/彗星老师" |
| **实现方式** | 集中配置 `identity.py`，通过 `get_identity_prompt()` 函数引用 |
| **预留功能** | `get_greeting()` 函数可供后续对话开场使用 |

请检查以上修改是否符合预期！

```plaintext
# 身份常量
IDENTITY = {
    "name_en": "Comet GPT",
    "name_cn": "彗星老师",
    "company": "自猎科技"
}


def get_identity_prompt():
    """获取身份描述，用于注入 system prompt"""
    return f"你是{IDENTITY['company']}的职业发展顾问 {IDENTITY['name_en']}，用户也可以亲切地叫你{IDENTITY['name_cn']}。"


def get_greeting():
    """获取开场自我介绍"""
    return f"你好！我是你的职业发展顾问 {IDENTITY['name_en']}，你也可以叫我{IDENTITY['name_cn']}。很高兴能帮助你！"
```

```python
# 修改前
from app.config import comet_config
from sdk.career.agent.resume_optimize import resume_optimize_action

# 修改后
from app.config import comet_config
from app.identity import get_identity_prompt
from sdk.career.agent.resume_optimize import resume_optimize_action
```

```python
# 修改前
messages = [
    {"role": "system", "content": "你的资料如下 姓名:自猎彗星,英文名:cometGPT,说明:自猎科技自研AI大模型"},
    {"role": "user", "content": prompt}
]

# 修改后
messages = [
    {"role": "system", "content": get_identity_prompt()},
    {"role": "user", "content": prompt}
]
```

---

**User**

现在你告诉我应该如何去测试呢？如何去测试你刚刚应用的修改是否生效呢？ 测试哪些接口呀？以及对应的请求体是什么？我想对各个接口涉及到这个对话的接口进行测试，测试这个提示词是否生效。你帮我看一下有哪些接口需要测试。以及相应的请求体是什么？帮我梳理一下

---

**Cursor**

让我帮你分析需要测试的接口。

现在我完全了解了测试方案。让我为你梳理：

---

## 测试方案

### 需要测试的接口

根据代码分析，**只有一个接口** 直接使用了修改的身份设定：

| 接口 | 路径 | 方法 | 认证要求 |
|------|------|------|----------|
| 对话聊天 | `/chat` | POST | **无需认证** |

---

### 接口触发逻辑

```
用户发送消息 → 意图识别
  ├─ intent = "resume_optimize" → 走简历优化流程（不触发身份设定）
  └─ intent = 其他（如 "chat"）→ **使用身份设定回复** ✅
```

**关键点**：当用户问"你是谁"等闲聊问题时，意图会被识别为 `chat`，此时 AI 会以 **"Comet GPT / 彗星老师"** 的身份回复。

---

### 测试请求体

#### 测试用例 1：询问身份

```json
{
    "resume_info": "",
    "dialogHistory": [
        {"role": "user", "content": "你是谁？"}
    ]
}
```

#### 测试用例 2：询问名字

```json
{
    "resume_info": "",
    "dialogHistory": [
        {"role": "user", "content": "你叫什么名字？请介绍一下你自己"}
    ]
}
```

#### 测试用例 3：简单闲聊

```json
{
    "resume_info": "",
    "dialogHistory": [
        {"role": "user", "content": "你好"}
    ]
}
```

---

### cURL 测试命令

假设服务运行在 `localhost:9006`：

```bash
# 测试用例1：询问身份
curl -X POST http://localhost:9006/chat \
  -H "Content-Type: application/json" \
  -d '{"resume_info": "", "dialogHistory": [{"role": "user", "content": "你是谁？"}]}'
```

```bash
# 测试用例2：询问名字
curl -X POST http://localhost:9006/chat \
  -H "Content-Type: application/json" \
  -d '{"resume_info": "", "dialogHistory": [{"role": "user", "content": "你叫什么名字？请介绍一下你自己"}]}'
```

---

### 预期响应

响应为 **SSE（Server-Sent Events）** 流式格式，AI 回复中应包含：

- ✅ **"Comet GPT"**
- ✅ **"彗星老师"**
- ✅ **"职业发展顾问"**
- ✅ **"自猎科技"**

示例预期片段：
```
data: {"type": "chunk", "content": "你好！我是自猎科技的职业发展顾问 Comet GPT，你也可以叫我彗星老师..."}
```

---

### 启动服务

在测试前，需要先启动服务：

```bash
cd E:\Python Code\cv-project\project_code\career\code
python app.py
```

服务将运行在 **http://localhost:9006**

---

## 注意事项

1. **响应格式**：接口返回的是 SSE 流，如果用 Postman 测试，需要设置 `Accept: text/event-stream`
2. **日志查看**：可以查看日志文件 `logs/app_info.log` 确认身份 prompt 是否正确注入
3. **意图判断**：如果发送的内容被识别为"简历优化"意图，不会触发身份设定回复

```plaintext
用户发送消息 → 意图识别
  ├─ intent = "resume_optimize" → 走简历优化流程（不触发身份设定）
  └─ intent = 其他（如 "chat"）→ **使用身份设定回复** ✅
```

```json
{
    "resume_info": "",
    "dialogHistory": [
        {"role": "user", "content": "你是谁？"}
    ]
}
```

```json
{
    "resume_info": "",
    "dialogHistory": [
        {"role": "user", "content": "你叫什么名字？请介绍一下你自己"}
    ]
}
```

```json
{
    "resume_info": "",
    "dialogHistory": [
        {"role": "user", "content": "你好"}
    ]
}
```

```shellscript
# 测试用例1：询问身份
curl -X POST http://localhost:9006/chat \
  -H "Content-Type: application/json" \
  -d '{"resume_info": "", "dialogHistory": [{"role": "user", "content": "你是谁？"}]}'
```

```shellscript
# 测试用例2：询问名字
curl -X POST http://localhost:9006/chat \
  -H "Content-Type: application/json" \
  -d '{"resume_info": "", "dialogHistory": [{"role": "user", "content": "你叫什么名字？请介绍一下你自己"}]}'
```

```plaintext
data: {"type": "chunk", "content": "你好！我是自猎科技的职业发展顾问 Comet GPT，你也可以叫我彗星老师..."}
```

```shellscript
cd E:\Python Code\cv-project\project_code\career\code
python app.py
```

---

**User**

你好好重新看一下项目，不止这一个接口呀。 这个接口其实还没做完，并不是这个接口，而是涉及到模拟面试啊，各种对话的接口呀。你帮我看一下呀。

所以我建议你先把所有的接口都列出来，然后分析各个接口大致的一个功能，然后再去判断哪些接口需要去测试这个提示词是否全局生效。按照这个流程一步步来可以吗？

---

**Cursor**

你说得对，让我全面梳理所有接口。

现在让我查看涉及AI对话的各个service文件，分析它们的功能：

现在我有完整的信息了。让我为你整理所有接口的分析：

---

## 📋 全部接口清单

| 序号 | 接口路径 | 方法 | 功能描述 | 交互类型 |
|------|----------|------|----------|----------|
| 1 | `/chat` | POST | 通用对话聊天（含意图识别） | 对话型 ✅ |
| 2 | `/virturlInterview` | POST | 模拟面试对话 | 对话型 ✅ |
| 3 | `/interviewReport` | POST | 面试报告生成（后台异步） | 生成型 |
| 4 | `/resume/basic-optimization` | POST | 简历基础优化（一次性） | 生成型 |
| 5 | `/resume/deep-optimization/converse` | POST | 深度简历优化对话（多轮） | 对话型 ✅ |
| 6 | `/resume/deep-optimization/generate` | POST | 根据对话生成优化简历 | 生成型 |
| 7 | `/jdFillingGenerate` | POST | 虚拟岗位生成 | 生成型 |
| 8 | `/resume/virtual-generate` | POST | 虚拟简历生成 | 生成型 |
| 9 | `/resume/position-recommend` | POST | 职位推荐 | 生成型 |

---

## 🎯 涉及对话交互的接口分析

### 1️⃣ `/chat` - 通用对话聊天
| 项目 | 说明 |
|------|------|
| **Service** | `sdk/career/intent_prompt.py` → `detect_intent()` |
| **当前状态** | ✅ **已修改**（使用 `get_identity_prompt()`） |
| **触发条件** | 当用户意图被识别为 `chat`（闲聊）时 |

---

### 2️⃣ `/virturlInterview` - 模拟面试
| 项目 | 说明 |
|------|------|
| **Service** | `service/virtual_interview.py` → `CallInterview.get_ai_response()` |
| **当前角色** | AI扮演"面试官" |
| **身份设定** | ❌ **未配置** Comet GPT 身份 |
| **分析** | 面试场景中AI是面试官角色，可能不适合直接用"彗星老师"身份 |

---

### 3️⃣ `/resume/deep-optimization/converse` - 深度简历优化对话
| 项目 | 说明 |
|------|------|
| **Service** | `service/deep_resume_converse.py` → `DeepResumeConverse.converse()` |
| **当前角色** | AI扮演"培训师" |
| **身份设定** | ❌ **未配置** Comet GPT 身份 |
| **分析** | 深度对话场景，AI引导用户挖掘简历信息 |

---

## 🔍 当前身份设定覆盖情况

```
                    身份设定覆盖范围
                    
/chat (意图=chat时)  ──────────────────► ✅ 已覆盖（get_identity_prompt()）
                    
/virturlInterview   ──────────────────► ❌ 未覆盖（面试官角色）
                    
/resume/deep-optimization/converse ───► ❌ 未覆盖（培训师角色）
```

---

## ❓ 需要你确认的问题

**核心问题**：你希望 "Comet GPT / 彗星老师" 身份在哪些对话场景中生效？

| 场景 | 当前AI角色 | 是否需要加入身份设定？ |
|------|------------|------------------------|
| `/chat` 闲聊 | 职业顾问 | ✅ 已完成 |
| `/virturlInterview` 模拟面试 | 面试官 | ❓ 待确认 |
| `/resume/deep-optimization/converse` 深度对话 | 培训师 | ❓ 待确认 |

**考虑点**：
1. **模拟面试**：AI 扮演的是面试官，如果用户问"你是谁"，是回答"我是面试官"还是"我是 Comet GPT 彗星老师"？
2. **深度简历对话**：AI 扮演的是培训师，角色定位类似

请告诉我：
1. 模拟面试和深度简历对话是否也需要加入 "Comet GPT / 彗星老师" 的身份设定？
2. 如果需要，是作为 **补充身份**（"我是 Comet GPT 彗星老师，在这次面试中担任面试官"）还是 **替代身份**？

```plaintext
                    身份设定覆盖范围
                    
/chat (意图=chat时)  ──────────────────► ✅ 已覆盖（get_identity_prompt()）
                    
/virturlInterview   ──────────────────► ❌ 未覆盖（面试官角色）
                    
/resume/deep-optimization/converse ───► ❌ 未覆盖（培训师角色）
```

---

**User**

模拟面试以及深度简历对话，

都需要 加入身份的设定。然后这个身份设定我不知道更好的方式是什么呢？因为毕竟应用的场景不一样，但是身份是要一样的，都是需要包含，Comet GPT / 彗星老师，这两个要素的。然后能不能帮我看思考一下怎么加呢，怎么加比较好呢？

---

**Cursor**

```plaintext
"我是自猎科技的职业发展顾问 Comet GPT，你也可以叫我彗星老师。在本次{场景}中，我将担任{角色}。"
```

```python
# identity.py

IDENTITY = {
    "name_en": "Comet GPT",
    "name_cn": "彗星老师",
    "company": "自猎科技"
}

# 场景角色映射
SCENE_ROLES = {
    "chat": "职业发展顾问",
    "interview": "面试官",
    "deep_resume": "职业培训师"
}

def get_identity_prompt(scene: str = "chat"):
    """获取带场景角色的身份描述"""
    role = SCENE_ROLES.get(scene, "职业发展顾问")
    return f"你是{IDENTITY['company']}的{role} {IDENTITY['name_en']}，用户也可以亲切地叫你{IDENTITY['name_cn']}。"
```

这是个很好的问题！让我为你设计一个统一且灵活的方案。

---

## 💡 设计思路

**核心原则**：**身份统一，角色适配场景**

```
身份（固定）: Comet GPT / 彗星老师 / 自猎科技
    │
    ├── 闲聊场景 → 角色：职业发展顾问
    ├── 面试场景 → 角色：面试官
    └── 简历对话场景 → 角色：职业培训师
```

---

## 📐 推荐方案：扩展 `identity.py`

在现有的 `identity.py` 基础上，增加场景化支持：

```python
# app/identity.py

# 身份常量（固定不变）
IDENTITY = {
    "name_en": "Comet GPT",
    "name_cn": "彗星老师",
    "company": "自猎科技"
}

# 场景角色映射
SCENE_ROLES = {
    "chat": "职业发展顾问",
    "interview": "面试官", 
    "deep_resume": "职业培训师"
}


def get_identity_prompt(scene: str = "chat"):
    """
    获取带场景角色的身份描述，用于注入 system prompt
    
    Args:
        scene: 场景类型，可选值：chat, interview, deep_resume
    
    Returns:
        包含身份和角色的提示词
    """
    role = SCENE_ROLES.get(scene, "职业发展顾问")
    return f"你是{IDENTITY['company']}的{role} {IDENTITY['name_en']}，用户也可以亲切地叫你{IDENTITY['name_cn']}。"


def get_greeting():
    """获取开场自我介绍（通用）"""
    return f"你好！我是你的职业发展顾问 {IDENTITY['name_en']}，你也可以叫我{IDENTITY['name_cn']}。很高兴能帮助你！"
```

---

## 🎯 各场景生成效果

| 场景 | 调用方式 | 生成结果 |
|------|----------|----------|
| 闲聊 | `get_identity_prompt("chat")` | "你是自猎科技的**职业发展顾问** Comet GPT，用户也可以亲切地叫你彗星老师。" |
| 面试 | `get_identity_prompt("interview")` | "你是自猎科技的**面试官** Comet GPT，用户也可以亲切地叫你彗星老师。" |
| 简历对话 | `get_identity_prompt("deep_resume")` | "你是自猎科技的**职业培训师** Comet GPT，用户也可以亲切地叫你彗星老师。" |

---

## 📝 各接口的修改点

### 1. `/chat` - 已完成
当前调用：`get_identity_prompt()` → 默认使用 "chat" 场景 ✅

### 2. `/virturlInterview` - 需要修改
**文件**：`service/virtual_interview.py`

在 system prompt 中注入身份，需要在策略 prompt 或执行 prompt 中添加。

### 3. `/resume/deep-optimization/converse` - 需要修改
**文件**：`service/deep_resume_converse.py`

在对话的 system prompt 中注入身份。

---

## 🤔 关于注入位置的考虑

有两种注入方式：

### 方式A：在 System Prompt 开头注入（推荐）
```python
prompt = f"{get_identity_prompt('interview')}\n\n{原有prompt内容}"
```
**优点**：简单直接，身份信息在最前面
**适用**：所有场景

### 方式B：在 prompt 模板文件中添加占位符
```
# prompts/interview/executor_white_collar.txt
{identity}

你是一位专业的面试官...
```
然后代码中：
```python
prompt = template.format(identity=get_identity_prompt('interview'), ...)
```
**优点**：prompt 模板更灵活
**缺点**：需要修改多个 prompt 文件

---

## ✅ 我的建议

1. **采用方式A**：在代码中将身份 prompt 拼接到 system prompt 开头
2. **统一管理**：所有身份相关逻辑都在 `identity.py` 中
3. **最小改动**：只修改必要的地方，不改 prompt 模板文件

---

**请确认**：
1. 这个"身份统一 + 角色适配"的设计方案是否符合你的预期？
2. 角色名称是否需要调整？（职业发展顾问 / 面试官 / 职业培训师）
3. 确认后我可以帮你实现具体的代码修改

如果确认，请切换到 **Agent 模式**，我来帮你完成代码修改。

```plaintext
身份（固定）: Comet GPT / 彗星老师 / 自猎科技
    │
    ├── 闲聊场景 → 角色：职业发展顾问
    ├── 面试场景 → 角色：面试官
    └── 简历对话场景 → 角色：职业培训师
```

```python
# app/identity.py

# 身份常量（固定不变）
IDENTITY = {
    "name_en": "Comet GPT",
    "name_cn": "彗星老师",
    "company": "自猎科技"
}

# 场景角色映射
SCENE_ROLES = {
    "chat": "职业发展顾问",
    "interview": "面试官", 
    "deep_resume": "职业培训师"
}


def get_identity_prompt(scene: str = "chat"):
    """
    获取带场景角色的身份描述，用于注入 system prompt
    
    Args:
        scene: 场景类型，可选值：chat, interview, deep_resume
    
    Returns:
        包含身份和角色的提示词
    """
    role = SCENE_ROLES.get(scene, "职业发展顾问")
    return f"你是{IDENTITY['company']}的{role} {IDENTITY['name_en']}，用户也可以亲切地叫你{IDENTITY['name_cn']}。"


def get_greeting():
    """获取开场自我介绍（通用）"""
    return f"你好！我是你的职业发展顾问 {IDENTITY['name_en']}，你也可以叫我{IDENTITY['name_cn']}。很高兴能帮助你！"
```

```python
prompt = f"{get_identity_prompt('interview')}\n\n{原有prompt内容}"
```

```plaintext
# prompts/interview/executor_white_collar.txt
{identity}

你是一位专业的面试官...
```

```python
prompt = template.format(identity=get_identity_prompt('interview'), ...)
```

---

**User**

你前面的修改方案A非常好，请帮我严格按照这个方案来修改相应的代码，完整follow前面修改方案的内容，不用引入不必要的额外的冗余的修改，修改之后，帮我梳理一下修改后的内容，进而方便我double check正确性

请额外注意：

1 严格遵循前面的方案的具体要求

2 不擅自添加你认为"有用"但我没有要求的功能

3 专注于我明确指出的需求

4 请不要往代码里面添加额外的md文件来总结修改内容。 

---

**Cursor**

好的，我来严格按照方案A进行修改。

```python
# 身份常量
IDENTITY = {
    "name_en": "Comet GPT",
    "name_cn": "彗星老师",
    "company": "自猎科技"
}

# 场景角色映射
SCENE_ROLES = {
    "chat": "职业发展顾问",
    "interview": "面试官",
    "deep_resume": "职业培训师"
}


def get_identity_prompt(scene: str = "chat"):
    """
    获取带场景角色的身份描述，用于注入 system prompt
    
    Args:
        scene: 场景类型，可选值：chat, interview, deep_resume
    
    Returns:
        包含身份和角色的提示词
    """
    role = SCENE_ROLES.get(scene, "职业发展顾问")
    return f"你是{IDENTITY['company']}的{role} {IDENTITY['name_en']}，用户也可以亲切地叫你{IDENTITY['name_cn']}。"


def get_greeting():
    """获取开场自我介绍"""
    return f"你好！我是你的职业发展顾问 {IDENTITY['name_en']}，你也可以叫我{IDENTITY['name_cn']}。很高兴能帮助你！"


```

```python
self.celue_sys_prompt = celue_template.format(language=lang)
self.zhixing_sys_prompt = zhixing_template.format(language=lang)
self.lanling_sys_prompt = lanling_template.format(language=lang)
```

```python
import asyncio
import json
import logging
import time
import re
import regex
from pathlib import Path
from openai import OpenAI, AsyncOpenAI
import dashscope
import dashscope.audio.tts_v2
from app.config import ai_config
from app.identity import get_identity_prompt
from sdk.tts.punctuation_based_tts import PunctuationBasedTTS
from service.llm_interface import OpenAILLMClient


def load_prompt(filename: str) -> str:
    """
    加载prompt文件
    
    Args:
        filename: 文件名，如 "strategy_white_collar.txt"
    
    Returns:
        prompt内容
    """
    current_dir = Path(__file__).parent
    prompt_path = current_dir.parent.parent / "prompts" / "interview" / filename
    with open(prompt_path, 'r', encoding='utf-8') as f:
        return f.read()



dashscope.api_key = ai_config['api_key']
class CallInterview:
    def __init__(self, interview_id: str, llm_client=None):

        self.interview_id = interview_id
        self.ai_config = ai_config
        self.request_id = None
        self.current_question = None
        self.timeout_task = None
        self.voice_active = False
        self.jd_title = None
        self.jd_portrait = None
        self.user_portrait = None
        self.company_portrait = None
        self.company_name = None
        self.interview_type = 1
        self.speech_speed = 1.2
        self.interview_language = "zhongwen"
        self.interview_style = None
        self.TTS_model = 'longanxuan'
        self.interview_plan = None
        self.current_question_index = 0
        self.language_dict = {
            'zhongwen': ('中文', '你好，面试官！让我们用中文交流！', 'longanxuan'),
            'yingwen': ('英文', "Hello, interviewer!Let's communicate in English!", 'longxiaochun_v2'),
            'riyu': ('日语', 'こんにちは、面接官！日本語で話し合いましょう！', 'loongtomoka_v2'),
            'hanyu': ('韩语', '안녕하세요, 면접관님!한국어로 대화해요!', 'loongkyong_v2'),
            'xibanyayu': ('西班牙语', '¡Hola, entrevistador!¡Hablemos en español!', 'sambert-camila-v1'),
            'yinniyu': ('印尼语', 'Halo, pewawancara!Ayo berkomunikasi dalam bahasa Indonesia!', 'sambert-indah-v1'),
            'fayu': ('法语', 'Bonjour, intervieweur !Parlons en français !', 'sambert-clara-v1'),
            'deyu': ('德语', 'Hallo, Interviewer!Lass uns auf Deutsch sprechen!', 'sambert-hanna-v1'),
            'taiyu': ('泰语', 'สวัสดีค่ะ/ครับ, ผู้สัมภาษณ์!มาพูดคุยกันเป็นภาษาไทยกันเถอะ!', 'sambert-waan-v1')
        }

        # 如果没有传入 LLM 客户端，则使用默认的生产环境客户端
        if llm_client is None:
            llm_client = OpenAILLMClient(
                api_key=self.ai_config['api_key'],
                base_url=self.ai_config['api_uri'],
                model=self.ai_config['api_model']
            )
        self.llm_client = llm_client
        
        # 为了兼容现有代码，保留原有的客户端属性
        self.celue_client = OpenAI(
            api_key=self.ai_config['api_key'],
            base_url=self.ai_config['api_uri'],
            timeout=60.0,
        )
        self.AI_client = AsyncOpenAI(
            api_key=self.ai_config['api_key'],
            base_url=self.ai_config['api_uri'],
            timeout=30.0,
        )
        # 1. 获取当前正在运行的事件循环
        try:
            loop = asyncio.get_running_loop()
        except RuntimeError:
            loop = asyncio.new_event_loop()
            asyncio.set_event_loop(loop)

        class FutureCallback(dashscope.audio.tts_v2.ResultCallback):
            def __init__(self, future: asyncio.Future):
                self._future = future
                self._audio_buffer = bytearray()  # 用于拼接音频片段

            def on_open(self):
                pass  # 这里可以加日志

            def on_complete(self):
                # 语音合成完成，所有片段都已收到
                # 将拼接好的完整音频数据作为Future的结果
                if not self._future.done():
                    self._future.set_result(bytes(self._audio_buffer))

            def on_error(self, message: str):
                # 发生错误，将异常作为Future的结果
                if not self._future.done():
                    self._future.set_exception(Exception(f"TTS Error: {message}"))

            def on_close(self):
                # 连接关闭，如果Future还未完成（可能在on_complete前关闭），也设置结果
                if not self._future.done():
                    self._future.set_result(bytes(self._audio_buffer))

            def on_data(self, data: bytes):
                # 收到音频片段，将其拼接到缓冲区
                if data:
                    self._audio_buffer.extend(data)

        # 3. 将新的Callback类绑定到self，以便synthesis_tts方法使用
        self.FutureCallback = FutureCallback


    async def generate_interview_plan(self):
        """生成面试计划（仅首次调用）"""
        logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - 🎯 进入 generate_interview_plan 函数")
        try:
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - 步骤1: 加载 prompt 文件")
            plan_prompt = load_prompt("generate_plan_white_collar.txt")
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - prompt 文件加载成功，长度: {len(plan_prompt)}")
            
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - 步骤2: 格式化 prompt")
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - user_portrait: {self.user_portrait[:50] if self.user_portrait else 'None'}...")
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - jd_title: {self.jd_title}")
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - jd_portrait: {self.jd_portrait[:50] if self.jd_portrait else 'None'}...")
            
            plan_prompt_filled = plan_prompt.format(
                user_portrait=self.user_portrait,
                jd_title=self.jd_title,
                jd_portrait=self.jd_portrait
            )
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - prompt 格式化成功")
            
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - 步骤3: 调用 LLM 生成计划")
            response = self.llm_client.create_completion(
                messages=[
                    {"role": "system", "content": plan_prompt_filled},
                    {"role": "user", "content": "请生成面试计划"}
                ],
                temperature=0.7
            )
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - LLM 调用成功")
            
            plan_json = response.choices[0].message.content
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - LLM 返回内容（前200字符）: {plan_json[:200]}")
            
            # 清理可能的markdown标记
            plan_json = plan_json.replace("```json", "").replace("```", "").strip()
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - 步骤4: 解析 JSON")
            self.interview_plan = json.loads(plan_json)
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - JSON 解析成功")
            
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - ✅ 面试计划生成成功，共{len(self.interview_plan['questions'])}个问题")
            
            # 详细输出所有问题内容
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - " + "="*80)
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - 📋 完整面试计划内容：")
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - " + "="*80)
            for idx, question in enumerate(self.interview_plan['questions'], 1):
                logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - ")
                logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - 【问题 {idx}/{len(self.interview_plan['questions'])}】")
                logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} -   阶段: {question.get('stage', '未知')}")
                logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} -   考察维度: {question.get('dimension', '未知')}")
                logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} -   难度: {question.get('difficulty', '未知')}")
                logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} -   问题内容: {question.get('question', '未知')}")
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - " + "="*80)
            
            return self.interview_plan
            
        except Exception as e:
            logging.error(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - ❌ 生成面试计划失败: {e}", exc_info=True)
            # 失败时返回空计划，走原有逻辑
            return None


    async def _detect_interview_action(self, ai_response: str, recent_history: list) -> dict:
        """
        检测AI面试官的行动类型
        返回: {"action": "new_question" | "repeat" | "check_status" | "follow_up" | "end"}
        """
        try:
            system_prompt = """你是面试流程分析器，判断AI面试官的回复属于哪种类型。

输出严格JSON格式：
{
  "action": "new_question" | "repeat" | "check_status" | "follow_up" | "end"
}

类型定义：
- new_question: 提出了新的面试问题
- repeat: 重复之前的问题
- check_status: 询问候选人状态（如"能听到吗"）
- follow_up: 针对当前问题追问细节
- end: 结束面试

注意：如果候选人最近几轮都是空回答或无效回答（如"..."、"嗯"、""等），且AI在询问状态，则判断为 check_status"""
            
            # 构建最近对话历史的文本
            history_text = ""
            if recent_history:
                for item in recent_history[-6:]:
                    role = "候选人" if item.get("role") == "user" else "AI面试官"
                    content = item.get("content", "")
                    history_text += f"{role}: {content}\n"
            
            user_prompt = f"""最近对话：
{history_text}

AI面试官刚才的回复：
{ai_response}

请判断AI的行动类型。"""
            
            response = self.celue_client.chat.completions.create(
                model="qwen-flash",
                temperature=0.1,
                response_format={"type": "json_object"},
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ]
            )
            
            result = json.loads(response.choices[0].message.content)
            logging.info(f"-- 面试ID: {self.interview_id} - 意图识别结果: {result}")
            return result
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 意图识别失败: {e}")
            # 失败时默认返回 new_question，保持原有行为
            return {"action": "new_question"}


    async def get_ai_response(self, dialog_history):
        start_time = time.time()

        # 调试日志
        logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - get_ai_response 被调用")
        logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - dialog_history 类型: {type(dialog_history)}")
        logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - dialog_history 值: {dialog_history}")
        logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - not dialog_history = {not dialog_history}")
        logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - len(dialog_history) = {len(dialog_history) if dialog_history is not None else 'None'}")

        # 沉默检测逻辑
        if dialog_history and len(dialog_history) > 0:
            # 统计连续沉默次数
            silence_count = 0
            for msg in reversed(dialog_history):
                if msg.get("role") == "user":
                    content = msg.get("content", "").strip()
                    if not content or content in ["", "...", "嗯", "啊"]:
                        silence_count += 1
                    else:
                        break
            
            logging.info(f"-- 面试ID: {self.interview_id} - 连续沉默次数: {silence_count}/3")
            
            if silence_count >= 3:
                logging.info(f"-- 面试ID: {self.interview_id} - 🛑 连续3次沉默，强制结束")
                end_msg = "由于连续多次未能收到您的回复，本次面试将结束。如遇技术问题，请联系HR重新安排。面试到此结束。"
                yield f"data: {json.dumps({'type': 'chunk', 'content': end_msg}, ensure_ascii=False)}\n\n"
                yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
                yield f"data: {json.dumps({'type': 'over', 'content': 'over'}, ensure_ascii=False)}\n\n"
                return

        # 判断是否首次调用
        if not dialog_history or len(dialog_history) == 0:
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 进入首次调用分支")
            # 首次调用：生成面试计划
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - 准备调用 generate_interview_plan()")
            await self.generate_interview_plan()
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - generate_interview_plan() 调用完成")
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - interview_plan 是否为 None: {self.interview_plan is None}")
            
            # 返回第一个问题（开场白）
            if self.interview_plan and len(self.interview_plan['questions']) > 0:
                logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 面试计划有效，准备返回第一个问题")
                first_question = self.interview_plan['questions'][0]['question']
                self.current_question_index = 1  # 下次从第二个问题开始
                
                logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - 第一个问题: {first_question[:50]}...")
                yield f"data: {json.dumps({'type': 'chunk', 'content': first_question}, ensure_ascii=False)}\n\n"
                yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
                logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 已返回第一个问题，函数即将return")
                return
            else:
                logging.warning(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ⚠️ 面试计划为空或没有问题，继续执行原有流程")

        language, say_hi, self.TTS_model = self.language_dict[self.interview_language]
        system_message = f'''
                        本次面试参考资料如下：
                        #面试交流全程使用语种：{language}
                        面试开始时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}
                        候选人信息：{self.user_portrait}
                        岗位信息：{self.jd_title},{self.jd_portrait}
                        公司信息：{self.company_portrait}
                        #面试交流全程使用语种：{language}
                        面试正式开始！
                                        '''

        self.speech_speed = 1
        
        # 加载prompt模板
        celue_template = load_prompt("strategy_white_collar.txt")
        zhixing_template = load_prompt("executor_white_collar.txt")
        lanling_template = load_prompt("executor_blue_collar.txt")
        
        # 替换变量
        lang = self.language_dict[self.interview_language][0]
        self.celue_sys_prompt = celue_template.format(language=lang)
        self.zhixing_sys_prompt = zhixing_template.format(language=lang)
        self.lanling_sys_prompt = lanling_template.format(language=lang)
        try:
            # 准备面试计划信息
            plan_info = ""
            if self.interview_plan:
                remaining_questions = self.interview_plan['questions'][self.current_question_index:]
                # 传递完整问题信息（包括 expectedPoints 和 followUpStrategy）
                plan_info = f"""
                【面试计划参考】
                当前进度：已提问 {self.current_question_index}/{len(self.interview_plan['questions'])} 个问题
                待提问题（含期望要点和追问策略）：
                {json.dumps(remaining_questions[:3], ensure_ascii=False, indent=2)}
                
                注意：对于技术类问题，请参考 expectedPoints 和 followUpStrategy 决定是否追问（最多2次）。
                """
            
            #interview_type 1白领，0蓝领
            if self.interview_type == 1:
                self.speech_speed = 1.1
                celue_message = [{"role": "system", "content": self.celue_sys_prompt},
                                 {"role": "user",
                                  "content": f"当前面试记录如下{dialog_history}，当前时间为{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}，其他消息：{system_message}{plan_info}"}]
                logging.info(f"-- 面试ID ：{self.interview_id},当前策略prompt如下--：{celue_message}")
                for i in range(2):
                    try:
                        celue = self.llm_client.create_completion(
                            messages=celue_message,
                            temperature=0.4,
                            extra_body={"chat_template_kwargs": {"enable_thinking": True}}
                        )
                        celue_text = celue.choices[0].message.content
                        break
                    except Exception as e:
                        logging.error(f"-- 面试ID : {self.interview_id}---策略调用AI出错，第{i + 1}次尝试: {e}")
                logging.info(
                    f"-- 面试ID ：{self.interview_id},推理时间：{time.time() - start_time}s,当前策略如下--：\n{celue_text}")
                
                # 提取质量评分（如果策略中包含）
                score_match = re.search(r'(?:回答质量|质量评分)[：:]\s*(\d+)', celue_text)
                if score_match:
                    quality_score = int(score_match.group(1))
                    logging.info(f"-- 面试ID：{self.interview_id} - 📊 当前回答质量评分：{quality_score}/10")
                # 启用流式传输
                zhixing_message = [{"role": "system", "content": self.zhixing_sys_prompt},
                                   {"role": "user",
                                    "content": f"当前面试记录如下{dialog_history},当前时间为：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}，其他消息：{system_message}，当前回复策略如下{celue_text}{plan_info}"}]
            else:
                zhixing_message = [{"role": "system", "content": self.lanling_sys_prompt},
                                   {"role": "user",
                                    "content": f"当前面试记录如下{dialog_history}，当前时间为{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}，其他消息：{system_message}"}]
            logging.info(f"-- 面试ID ：{self.interview_id},当前执行prompt如下--：{zhixing_message}")
            #重试次数
            for i in range(2):
                try:
                    response = self.llm_client.create_streaming_completion(
                        messages=zhixing_message,
                        temperature=0.7,
                        seed=1
                    )
                    break
                except Exception as e:
                    logging.error(f"-- 面试ID : {self.interview_id}---流式调用AI出错，第{i + 1}次尝试: {e}")
            if not response:
                logging.error(f"-- 面试ID : {self.interview_id}---流式调用AI失败")
            ai_response = ""
            async for chunk in response:
                if chunk.choices and chunk.choices[0].delta.content:
                    content = chunk.choices[0].delta.content
                    ai_response += content

                    yield f"data: {json.dumps({'type': 'chunk', 'content': content}, ensure_ascii=False)}\n\n"

            yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"

            # 意图识别
            intent = await self._detect_interview_action(ai_response, dialog_history[-6:] if dialog_history else [])

            # 根据意图更新索引
            if self.interview_plan and intent.get("action") == "new_question":
                self.current_question_index += 1
                logging.info(f"-- 面试ID: {self.interview_id} - 📈 进入下一题，索引更新为 {self.current_question_index}")
            else:
                logging.info(f"-- 面试ID: {self.interview_id} - 🔄 保持当前问题，动作类型: {intent.get('action')}")

            end_time = time.time()
            execution_time = end_time - start_time
            logging.info(f"-- 面试ID : {self.interview_id}---执行时间为: {execution_time} 秒,AI流式返回内容: {ai_response}")
            try:
                flash_time = time.time()
                #判断面试是否结束，每个循环都有
                liucheng_prompt = load_prompt("judge_flow.txt")
                liucheng = self.celue_client.chat.completions.create(
                    model="qwen-flash",
                    temperature=0.4,
                    messages=[{"role": "system", "content": liucheng_prompt},
                              {"role": "user", "content": f"面试官回复内容如下{ai_response}"}],
                )
                liucheng_flag = liucheng.choices[0].message.content
                logging.info(
                    f"-- 面试ID : {self.interview_id}---流程判断内容:  {liucheng_flag}  ,耗时{time.time() - flash_time}秒")
                if "结束" in liucheng_flag:
                    yield f"data: {json.dumps({'type': 'over', 'content': 'over'}, ensure_ascii=False)}\n\n"
            except Exception as e:
                logging.error(f"-- 面试ID : {self.interview_id}---流程判断出错: {e}")
                yield f"data: {json.dumps({'type': 'error', 'content': str(e)}, ensure_ascii=False)}\n\n"

        except Exception as e:
            logging.error(f"-- 面试ID : {self.interview_id}---AI流式响应获取错误: {e}")
            yield f"data: {json.dumps({'type': 'error', 'content': '模型检测到非法输入，请重新开始本次面试'+str(e)}, ensure_ascii=False)}\n\n"






```

```python
self.celue_sys_prompt = celue_template.format(language=lang)
self.zhixing_sys_prompt = zhixing_template.format(language=lang)
self.lanling_sys_prompt = lanling_template.format(language=lang)
```

```python
import asyncio
import json
import logging
import time
import re
import regex
from pathlib import Path
from openai import OpenAI, AsyncOpenAI
import dashscope
import dashscope.audio.tts_v2
from app.config import ai_config
from app.identity import get_identity_prompt
from sdk.tts.punctuation_based_tts import PunctuationBasedTTS
from service.llm_interface import OpenAILLMClient


def load_prompt(filename: str) -> str:
    """
    加载prompt文件
    
    Args:
        filename: 文件名，如 "strategy_white_collar.txt"
    
    Returns:
        prompt内容
    """
    current_dir = Path(__file__).parent
    prompt_path = current_dir.parent.parent / "prompts" / "interview" / filename
    with open(prompt_path, 'r', encoding='utf-8') as f:
        return f.read()



dashscope.api_key = ai_config['api_key']
class CallInterview:
    def __init__(self, interview_id: str, llm_client=None):

        self.interview_id = interview_id
        self.ai_config = ai_config
        self.request_id = None
        self.current_question = None
        self.timeout_task = None
        self.voice_active = False
        self.jd_title = None
        self.jd_portrait = None
        self.user_portrait = None
        self.company_portrait = None
        self.company_name = None
        self.interview_type = 1
        self.speech_speed = 1.2
        self.interview_language = "zhongwen"
        self.interview_style = None
        self.TTS_model = 'longanxuan'
        self.interview_plan = None
        self.current_question_index = 0
        self.language_dict = {
            'zhongwen': ('中文', '你好，面试官！让我们用中文交流！', 'longanxuan'),
            'yingwen': ('英文', "Hello, interviewer!Let's communicate in English!", 'longxiaochun_v2'),
            'riyu': ('日语', 'こんにちは、面接官！日本語で話し合いましょう！', 'loongtomoka_v2'),
            'hanyu': ('韩语', '안녕하세요, 면접관님!한국어로 대화해요!', 'loongkyong_v2'),
            'xibanyayu': ('西班牙语', '¡Hola, entrevistador!¡Hablemos en español!', 'sambert-camila-v1'),
            'yinniyu': ('印尼语', 'Halo, pewawancara!Ayo berkomunikasi dalam bahasa Indonesia!', 'sambert-indah-v1'),
            'fayu': ('法语', 'Bonjour, intervieweur !Parlons en français !', 'sambert-clara-v1'),
            'deyu': ('德语', 'Hallo, Interviewer!Lass uns auf Deutsch sprechen!', 'sambert-hanna-v1'),
            'taiyu': ('泰语', 'สวัสดีค่ะ/ครับ, ผู้สัมภาษณ์!มาพูดคุยกันเป็นภาษาไทยกันเถอะ!', 'sambert-waan-v1')
        }

        # 如果没有传入 LLM 客户端，则使用默认的生产环境客户端
        if llm_client is None:
            llm_client = OpenAILLMClient(
                api_key=self.ai_config['api_key'],
                base_url=self.ai_config['api_uri'],
                model=self.ai_config['api_model']
            )
        self.llm_client = llm_client
        
        # 为了兼容现有代码，保留原有的客户端属性
        self.celue_client = OpenAI(
            api_key=self.ai_config['api_key'],
            base_url=self.ai_config['api_uri'],
            timeout=60.0,
        )
        self.AI_client = AsyncOpenAI(
            api_key=self.ai_config['api_key'],
            base_url=self.ai_config['api_uri'],
            timeout=30.0,
        )
        # 1. 获取当前正在运行的事件循环
        try:
            loop = asyncio.get_running_loop()
        except RuntimeError:
            loop = asyncio.new_event_loop()
            asyncio.set_event_loop(loop)

        class FutureCallback(dashscope.audio.tts_v2.ResultCallback):
            def __init__(self, future: asyncio.Future):
                self._future = future
                self._audio_buffer = bytearray()  # 用于拼接音频片段

            def on_open(self):
                pass  # 这里可以加日志

            def on_complete(self):
                # 语音合成完成，所有片段都已收到
                # 将拼接好的完整音频数据作为Future的结果
                if not self._future.done():
                    self._future.set_result(bytes(self._audio_buffer))

            def on_error(self, message: str):
                # 发生错误，将异常作为Future的结果
                if not self._future.done():
                    self._future.set_exception(Exception(f"TTS Error: {message}"))

            def on_close(self):
                # 连接关闭，如果Future还未完成（可能在on_complete前关闭），也设置结果
                if not self._future.done():
                    self._future.set_result(bytes(self._audio_buffer))

            def on_data(self, data: bytes):
                # 收到音频片段，将其拼接到缓冲区
                if data:
                    self._audio_buffer.extend(data)

        # 3. 将新的Callback类绑定到self，以便synthesis_tts方法使用
        self.FutureCallback = FutureCallback


    async def generate_interview_plan(self):
        """生成面试计划（仅首次调用）"""
        logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - 🎯 进入 generate_interview_plan 函数")
        try:
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - 步骤1: 加载 prompt 文件")
            plan_prompt = load_prompt("generate_plan_white_collar.txt")
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - prompt 文件加载成功，长度: {len(plan_prompt)}")
            
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - 步骤2: 格式化 prompt")
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - user_portrait: {self.user_portrait[:50] if self.user_portrait else 'None'}...")
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - jd_title: {self.jd_title}")
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - jd_portrait: {self.jd_portrait[:50] if self.jd_portrait else 'None'}...")
            
            plan_prompt_filled = plan_prompt.format(
                user_portrait=self.user_portrait,
                jd_title=self.jd_title,
                jd_portrait=self.jd_portrait
            )
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - prompt 格式化成功")
            
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - 步骤3: 调用 LLM 生成计划")
            response = self.llm_client.create_completion(
                messages=[
                    {"role": "system", "content": plan_prompt_filled},
                    {"role": "user", "content": "请生成面试计划"}
                ],
                temperature=0.7
            )
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - LLM 调用成功")
            
            plan_json = response.choices[0].message.content
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - LLM 返回内容（前200字符）: {plan_json[:200]}")
            
            # 清理可能的markdown标记
            plan_json = plan_json.replace("```json", "").replace("```", "").strip()
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - 步骤4: 解析 JSON")
            self.interview_plan = json.loads(plan_json)
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - JSON 解析成功")
            
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - ✅ 面试计划生成成功，共{len(self.interview_plan['questions'])}个问题")
            
            # 详细输出所有问题内容
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - " + "="*80)
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - 📋 完整面试计划内容：")
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - " + "="*80)
            for idx, question in enumerate(self.interview_plan['questions'], 1):
                logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - ")
                logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - 【问题 {idx}/{len(self.interview_plan['questions'])}】")
                logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} -   阶段: {question.get('stage', '未知')}")
                logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} -   考察维度: {question.get('dimension', '未知')}")
                logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} -   难度: {question.get('difficulty', '未知')}")
                logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} -   问题内容: {question.get('question', '未知')}")
            logging.info(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - " + "="*80)
            
            return self.interview_plan
            
        except Exception as e:
            logging.error(f"[DEBUG-生成计划] 面试ID: {self.interview_id} - ❌ 生成面试计划失败: {e}", exc_info=True)
            # 失败时返回空计划，走原有逻辑
            return None


    async def _detect_interview_action(self, ai_response: str, recent_history: list) -> dict:
        """
        检测AI面试官的行动类型
        返回: {"action": "new_question" | "repeat" | "check_status" | "follow_up" | "end"}
        """
        try:
            system_prompt = """你是面试流程分析器，判断AI面试官的回复属于哪种类型。

输出严格JSON格式：
{
  "action": "new_question" | "repeat" | "check_status" | "follow_up" | "end"
}

类型定义：
- new_question: 提出了新的面试问题
- repeat: 重复之前的问题
- check_status: 询问候选人状态（如"能听到吗"）
- follow_up: 针对当前问题追问细节
- end: 结束面试

注意：如果候选人最近几轮都是空回答或无效回答（如"..."、"嗯"、""等），且AI在询问状态，则判断为 check_status"""
            
            # 构建最近对话历史的文本
            history_text = ""
            if recent_history:
                for item in recent_history[-6:]:
                    role = "候选人" if item.get("role") == "user" else "AI面试官"
                    content = item.get("content", "")
                    history_text += f"{role}: {content}\n"
            
            user_prompt = f"""最近对话：
{history_text}

AI面试官刚才的回复：
{ai_response}

请判断AI的行动类型。"""
            
            response = self.celue_client.chat.completions.create(
                model="qwen-flash",
                temperature=0.1,
                response_format={"type": "json_object"},
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ]
            )
            
            result = json.loads(response.choices[0].message.content)
            logging.info(f"-- 面试ID: {self.interview_id} - 意图识别结果: {result}")
            return result
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 意图识别失败: {e}")
            # 失败时默认返回 new_question，保持原有行为
            return {"action": "new_question"}


    async def get_ai_response(self, dialog_history):
        start_time = time.time()

        # 调试日志
        logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - get_ai_response 被调用")
        logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - dialog_history 类型: {type(dialog_history)}")
        logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - dialog_history 值: {dialog_history}")
        logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - not dialog_history = {not dialog_history}")
        logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - len(dialog_history) = {len(dialog_history) if dialog_history is not None else 'None'}")

        # 沉默检测逻辑
        if dialog_history and len(dialog_history) > 0:
            # 统计连续沉默次数
            silence_count = 0
            for msg in reversed(dialog_history):
                if msg.get("role") == "user":
                    content = msg.get("content", "").strip()
                    if not content or content in ["", "...", "嗯", "啊"]:
                        silence_count += 1
                    else:
                        break
            
            logging.info(f"-- 面试ID: {self.interview_id} - 连续沉默次数: {silence_count}/3")
            
            if silence_count >= 3:
                logging.info(f"-- 面试ID: {self.interview_id} - 🛑 连续3次沉默，强制结束")
                end_msg = "由于连续多次未能收到您的回复，本次面试将结束。如遇技术问题，请联系HR重新安排。面试到此结束。"
                yield f"data: {json.dumps({'type': 'chunk', 'content': end_msg}, ensure_ascii=False)}\n\n"
                yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
                yield f"data: {json.dumps({'type': 'over', 'content': 'over'}, ensure_ascii=False)}\n\n"
                return

        # 判断是否首次调用
        if not dialog_history or len(dialog_history) == 0:
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 进入首次调用分支")
            # 首次调用：生成面试计划
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - 准备调用 generate_interview_plan()")
            await self.generate_interview_plan()
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - generate_interview_plan() 调用完成")
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - interview_plan 是否为 None: {self.interview_plan is None}")
            
            # 返回第一个问题（开场白）
            if self.interview_plan and len(self.interview_plan['questions']) > 0:
                logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 面试计划有效，准备返回第一个问题")
                first_question = self.interview_plan['questions'][0]['question']
                self.current_question_index = 1  # 下次从第二个问题开始
                
                logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - 第一个问题: {first_question[:50]}...")
                yield f"data: {json.dumps({'type': 'chunk', 'content': first_question}, ensure_ascii=False)}\n\n"
                yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
                logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 已返回第一个问题，函数即将return")
                return
            else:
                logging.warning(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ⚠️ 面试计划为空或没有问题，继续执行原有流程")

        language, say_hi, self.TTS_model = self.language_dict[self.interview_language]
        system_message = f'''
                        本次面试参考资料如下：
                        #面试交流全程使用语种：{language}
                        面试开始时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}
                        候选人信息：{self.user_portrait}
                        岗位信息：{self.jd_title},{self.jd_portrait}
                        公司信息：{self.company_portrait}
                        #面试交流全程使用语种：{language}
                        面试正式开始！
                                        '''

        self.speech_speed = 1
        
        # 加载prompt模板
        celue_template = load_prompt("strategy_white_collar.txt")
        zhixing_template = load_prompt("executor_white_collar.txt")
        lanling_template = load_prompt("executor_blue_collar.txt")
        
        # 替换变量
        lang = self.language_dict[self.interview_language][0]
        identity_prompt = get_identity_prompt("interview")
        self.celue_sys_prompt = f"{identity_prompt}\n\n{celue_template.format(language=lang)}"
        self.zhixing_sys_prompt = f"{identity_prompt}\n\n{zhixing_template.format(language=lang)}"
        self.lanling_sys_prompt = f"{identity_prompt}\n\n{lanling_template.format(language=lang)}"
        try:
            # 准备面试计划信息
            plan_info = ""
            if self.interview_plan:
                remaining_questions = self.interview_plan['questions'][self.current_question_index:]
                # 传递完整问题信息（包括 expectedPoints 和 followUpStrategy）
                plan_info = f"""
                【面试计划参考】
                当前进度：已提问 {self.current_question_index}/{len(self.interview_plan['questions'])} 个问题
                待提问题（含期望要点和追问策略）：
                {json.dumps(remaining_questions[:3], ensure_ascii=False, indent=2)}
                
                注意：对于技术类问题，请参考 expectedPoints 和 followUpStrategy 决定是否追问（最多2次）。
                """
            
            #interview_type 1白领，0蓝领
            if self.interview_type == 1:
                self.speech_speed = 1.1
                celue_message = [{"role": "system", "content": self.celue_sys_prompt},
                                 {"role": "user",
                                  "content": f"当前面试记录如下{dialog_history}，当前时间为{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}，其他消息：{system_message}{plan_info}"}]
                logging.info(f"-- 面试ID ：{self.interview_id},当前策略prompt如下--：{celue_message}")
                for i in range(2):
                    try:
                        celue = self.llm_client.create_completion(
                            messages=celue_message,
                            temperature=0.4,
                            extra_body={"chat_template_kwargs": {"enable_thinking": True}}
                        )
                        celue_text = celue.choices[0].message.content
                        break
                    except Exception as e:
                        logging.error(f"-- 面试ID : {self.interview_id}---策略调用AI出错，第{i + 1}次尝试: {e}")
                logging.info(
                    f"-- 面试ID ：{self.interview_id},推理时间：{time.time() - start_time}s,当前策略如下--：\n{celue_text}")
                
                # 提取质量评分（如果策略中包含）
                score_match = re.search(r'(?:回答质量|质量评分)[：:]\s*(\d+)', celue_text)
                if score_match:
                    quality_score = int(score_match.group(1))
                    logging.info(f"-- 面试ID：{self.interview_id} - 📊 当前回答质量评分：{quality_score}/10")
                # 启用流式传输
                zhixing_message = [{"role": "system", "content": self.zhixing_sys_prompt},
                                   {"role": "user",
                                    "content": f"当前面试记录如下{dialog_history},当前时间为：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}，其他消息：{system_message}，当前回复策略如下{celue_text}{plan_info}"}]
            else:
                zhixing_message = [{"role": "system", "content": self.lanling_sys_prompt},
                                   {"role": "user",
                                    "content": f"当前面试记录如下{dialog_history}，当前时间为{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}，其他消息：{system_message}"}]
            logging.info(f"-- 面试ID ：{self.interview_id},当前执行prompt如下--：{zhixing_message}")
            #重试次数
            for i in range(2):
                try:
                    response = self.llm_client.create_streaming_completion(
                        messages=zhixing_message,
                        temperature=0.7,
                        seed=1
                    )
                    break
                except Exception as e:
                    logging.error(f"-- 面试ID : {self.interview_id}---流式调用AI出错，第{i + 1}次尝试: {e}")
            if not response:
                logging.error(f"-- 面试ID : {self.interview_id}---流式调用AI失败")
            ai_response = ""
            async for chunk in response:
                if chunk.choices and chunk.choices[0].delta.content:
                    content = chunk.choices[0].delta.content
                    ai_response += content

                    yield f"data: {json.dumps({'type': 'chunk', 'content': content}, ensure_ascii=False)}\n\n"

            yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"

            # 意图识别
            intent = await self._detect_interview_action(ai_response, dialog_history[-6:] if dialog_history else [])

            # 根据意图更新索引
            if self.interview_plan and intent.get("action") == "new_question":
                self.current_question_index += 1
                logging.info(f"-- 面试ID: {self.interview_id} - 📈 进入下一题，索引更新为 {self.current_question_index}")
            else:
                logging.info(f"-- 面试ID: {self.interview_id} - 🔄 保持当前问题，动作类型: {intent.get('action')}")

            end_time = time.time()
            execution_time = end_time - start_time
            logging.info(f"-- 面试ID : {self.interview_id}---执行时间为: {execution_time} 秒,AI流式返回内容: {ai_response}")
            try:
                flash_time = time.time()
                #判断面试是否结束，每个循环都有
                liucheng_prompt = load_prompt("judge_flow.txt")
                liucheng = self.celue_client.chat.completions.create(
                    model="qwen-flash",
                    temperature=0.4,
                    messages=[{"role": "system", "content": liucheng_prompt},
                              {"role": "user", "content": f"面试官回复内容如下{ai_response}"}],
                )
                liucheng_flag = liucheng.choices[0].message.content
                logging.info(
                    f"-- 面试ID : {self.interview_id}---流程判断内容:  {liucheng_flag}  ,耗时{time.time() - flash_time}秒")
                if "结束" in liucheng_flag:
                    yield f"data: {json.dumps({'type': 'over', 'content': 'over'}, ensure_ascii=False)}\n\n"
            except Exception as e:
                logging.error(f"-- 面试ID : {self.interview_id}---流程判断出错: {e}")
                yield f"data: {json.dumps({'type': 'error', 'content': str(e)}, ensure_ascii=False)}\n\n"

        except Exception as e:
            logging.error(f"-- 面试ID : {self.interview_id}---AI流式响应获取错误: {e}")
            yield f"data: {json.dumps({'type': 'error', 'content': '模型检测到非法输入，请重新开始本次面试'+str(e)}, ensure_ascii=False)}\n\n"






```

```python
import json
import logging
import time
from pathlib import Path
from openai import OpenAI
from app.config import ai_config
from app.identity import get_identity_prompt
from service.llm_interface import OpenAILLMClient


def load_prompt(filename: str) -> str:
    """
    加载prompt文件
    
    Args:
        filename: 文件名，如 "generate_plan.txt"
    
    Returns:
        prompt内容
    """
    current_dir = Path(__file__).parent
    prompt_path = current_dir.parent.parent / "prompts" / "deep_resume" / filename
    with open(prompt_path, 'r', encoding='utf-8') as f:
        return f.read()


class DeepResumeConverse:
    """深度简历优化对话服务"""
    
    def __init__(self, llm_client=None):
        self.ai_config = ai_config
        
        # 如果没有传入 LLM 客户端，则使用默认的生产环境客户端
        if llm_client is None:
            llm_client = OpenAILLMClient(
                api_key=self.ai_config['api_key'],
                base_url=self.ai_config['api_uri'],
                model=self.ai_config['api_model']
            )
        self.llm_client = llm_client
        
        # 用于快速模型调用（意图识别、结束判断）
        self.fast_client = OpenAI(
            api_key=self.ai_config['api_key'],
            base_url=self.ai_config['api_uri'],
            timeout=60.0,
        )
    
    def _check_silence(self, history: list) -> int:
        """
        检测连续沉默次数
        
        Args:
            history: 对话历史
            
        Returns:
            连续沉默次数
        """
        if not history:
            return 0
            
        silence_count = 0
        invalid_messages = ["", "...", "嗯", "啊", "哦", "额"]
        
        for msg in reversed(history):
            if msg.get("role") == "user":
                content = msg.get("content", "").strip()
                if not content or content in invalid_messages:
                    silence_count += 1
                else:
                    break
        
        return silence_count
    
    def _format_history(self, history: list) -> str:
        """格式化对话历史为字符串"""
        if not history:
            return "暂无对话历史"
        
        formatted = []
        for msg in history:
            role = "学生" if msg.get("role") == "user" else "培训师"
            content = msg.get("content", "")
            formatted.append(f"{role}: {content}")
        
        return "\n".join(formatted)
    
    def _format_user_profile(self, user_profile: dict) -> str:
        """格式化用户档案为字符串，适配前端传入的数据结构"""
        base_info = user_profile.get("base_info", {})
        edu_exp_list = user_profile.get("edu_exp", [])
        job_exp_list = user_profile.get("job_exp", [])
        work_exp_list = user_profile.get("work_exp", [])
        
        # 基本信息
        result = f"""
【基本信息】
姓名: {base_info.get('name', '未知')}
性别: {base_info.get('sex', '未知')}
出生日期: {base_info.get('birth_year_month', '未知')}
学历: {base_info.get('degree', '未知')}
工作年限: {base_info.get('work_year', '未知')}年
自我评价: {base_info.get('advantages_highlights', '无')}
"""
        
        # 教育经历
        if edu_exp_list:
            result += "\n【教育经历】\n"
            for edu in edu_exp_list:
                result += f"- {edu.get('school', '未知')} | {edu.get('specialty', '未知')} | {edu.get('education_background', '未知')} | {edu.get('begin_date', '')} 至 {edu.get('end_date', '')}\n"
        
        # 求职期望
        if job_exp_list:
            result += "\n【求职期望】\n"
            for job in job_exp_list:
                result += f"- 期望职位: {job.get('desired_position', '未知')} | 期望地点: {job.get('desired_location', '未知')} | 期望薪资: {job.get('expected_salary', '未知')}千/月\n"
        
        # 工作经历
        if work_exp_list:
            result += "\n【工作经历】\n"
            for work in work_exp_list:
                result += f"- {work.get('company_name', '未知')} | {work.get('position', '未知')} | {work.get('begin_date', '')} 至 {work.get('end_date', '')}\n"
                result += f"  职责: {work.get('responsibility_performance', '无')}\n"
        
        return result
    
    def _format_plan_info(self, conversation_plan: dict) -> str:
        """
        格式化对话计划为参考信息
        
        Args:
            conversation_plan: 对话计划（可能为空）
            
        Returns:
            格式化的计划信息，如果为空则返回空字符串
        """
        if not conversation_plan:
            return ""
        
        stages = conversation_plan.get("stages", [])
        if not stages:
            return ""
        
        stage_info = []
        for stage in stages:
            stage_info.append(f"- {stage.get('stage', '')}: {stage.get('goal', '')}")
        
        return f"""
【对话计划参考】
{chr(10).join(stage_info)}
"""
    
    async def generate_plan(self, user_profile: dict):
        """
        生成对话计划（仅首次调用）
        
        Args:
            user_profile: 用户档案
            
        Returns:
            (opening_message, plan) 开场白和计划
        """
        logging.info("[DeepResumeConverse] 生成对话计划")
        
        try:
            prompt_template = load_prompt("generate_plan.txt")
            prompt = prompt_template.format(
                user_profile=self._format_user_profile(user_profile)
            )
            
            response = self.llm_client.create_completion(
                messages=[
                    {"role": "system", "content": prompt},
                    {"role": "user", "content": "请生成对话计划"}
                ],
                temperature=0.7
            )
            
            result_text = response.choices[0].message.content
            # 清理可能的markdown标记
            result_text = result_text.replace("```json", "").replace("```", "").strip()
            
            plan_data = json.loads(result_text)
            opening_message = plan_data.get("opening_message", "你好！让我们来聊聊你的经历吧。")
            stages = plan_data.get("stages", [])
            
            plan = {"stages": stages}
            
            logging.info(f"[DeepResumeConverse] 对话计划生成成功，共{len(stages)}个阶段")
            return opening_message, plan
            
        except Exception as e:
            logging.error(f"[DeepResumeConverse] 生成对话计划失败: {e}")
            # 降级：返回默认开场白和空计划
            return "你好！让我们来聊聊你的经历吧。你最引以为傲的项目或经历是什么？", None
    
    async def generate_strategy(self, user_profile: dict, history: list, 
                                 user_message: str, conversation_plan: dict) -> str:
        """
        生成对话策略
        
        Args:
            user_profile: 用户档案
            history: 对话历史
            user_message: 用户最新回复
            conversation_plan: 对话计划（可选）
            
        Returns:
            策略文本
        """
        logging.info("[DeepResumeConverse] 生成对话策略")
        
        try:
            prompt_template = load_prompt("strategy.txt")
            plan_info = self._format_plan_info(conversation_plan)
            
            prompt = prompt_template.format(
                user_profile=self._format_user_profile(user_profile),
                history=self._format_history(history),
                user_message=user_message,
                plan_info=plan_info
            )
            
            response = self.llm_client.create_completion(
                messages=[
                    {"role": "system", "content": prompt},
                    {"role": "user", "content": "请制定下一步对话策略"}
                ],
                temperature=0.4
            )
            
            strategy = response.choices[0].message.content
            logging.info(f"[DeepResumeConverse] 策略生成成功")
            return strategy
            
        except Exception as e:
            logging.error(f"[DeepResumeConverse] 生成策略失败: {e}")
            return "继续追问细节，挖掘更多信息"
    
    async def execute_response(self, user_profile: dict, history: list,
                                user_message: str, strategy: str):
        """
        执行对话，流式生成AI回复
        
        Args:
            user_profile: 用户档案
            history: 对话历史
            user_message: 用户最新回复
            strategy: 对话策略
            
        Yields:
            AI回复的文本片段
        """
        logging.info("[DeepResumeConverse] 执行对话生成")
        
        try:
            prompt_template = load_prompt("executor.txt")
            prompt = prompt_template.format(
                user_profile=self._format_user_profile(user_profile),
                history=self._format_history(history),
                user_message=user_message,
                strategy=strategy
            )
            
            response = self.llm_client.create_streaming_completion(
                messages=[
                    {"role": "system", "content": prompt},
                    {"role": "user", "content": "请生成下一轮对话内容"}
                ],
                temperature=0.7
            )
            
            async for chunk in response:
                if chunk.choices and chunk.choices[0].delta.content:
                    yield chunk.choices[0].delta.content
                    
        except Exception as e:
            logging.error(f"[DeepResumeConverse] 执行对话失败: {e}")
            yield "抱歉，我需要整理一下思路。你能再说一遍吗？"
    
    def detect_intent(self, ai_response: str, recent_history: list) -> dict:
        """
        检测AI回复的意图类型
        
        Args:
            ai_response: AI的回复内容
            recent_history: 最近的对话历史
            
        Returns:
            {"action": "new_question" | "follow_up" | "case_analysis" | "check_status" | "end"}
        """
        try:
            system_prompt = """你是对话意图分析器，判断AI培训师的回复属于哪种类型。

输出严格JSON格式：
{
  "action": "new_question" | "follow_up" | "case_analysis" | "check_status" | "end"
}

类型定义：
- new_question: 提出了新的话题或问题
- follow_up: 针对当前话题追问细节
- case_analysis: 给出案例场景让学生分析
- check_status: 询问学生状态
- end: 表示对话可以结束"""
            
            response = self.fast_client.chat.completions.create(
                model="qwen-flash",
                temperature=0.1,
                response_format={"type": "json_object"},
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": f"AI回复：{ai_response}"}
                ]
            )
            
            result = json.loads(response.choices[0].message.content)
            logging.info(f"[DeepResumeConverse] 意图识别结果: {result}")
            return result
            
        except Exception as e:
            logging.error(f"[DeepResumeConverse] 意图识别失败: {e}")
            return {"action": "follow_up"}
    
    def judge_end(self, ai_response: str) -> bool:
        """
        判断对话是否结束
        
        Args:
            ai_response: AI的回复内容
            
        Returns:
            是否结束
        """
        try:
            prompt_template = load_prompt("judge_end.txt")
            prompt = prompt_template.format(ai_response=ai_response)
            
            response = self.fast_client.chat.completions.create(
                model="qwen-flash",
                temperature=0.1,
                messages=[
                    {"role": "system", "content": prompt},
                    {"role": "user", "content": "请判断"}
                ]
            )
            
            result = response.choices[0].message.content.strip()
            is_end = "结束" in result
            logging.info(f"[DeepResumeConverse] 结束判断: {result}, is_end={is_end}")
            return is_end
            
        except Exception as e:
            logging.error(f"[DeepResumeConverse] 结束判断失败: {e}")
            return False
    
    async def converse(self, user_profile: dict, history: list, 
                       user_message: str, conversation_plan: dict = None):
        """
        主对话流程
        
        Args:
            user_profile: 用户档案
            history: 对话历史
            user_message: 用户最新回复
            conversation_plan: 对话计划（可选增强项）
            
        Yields:
            SSE格式的响应数据
        """
        start_time = time.time()
        
        # 步骤1：沉默检测
        silence_count = self._check_silence(history)
        logging.info(f"[DeepResumeConverse] 沉默检测: {silence_count}/3")
        
        if silence_count >= 3:
            logging.info("[DeepResumeConverse] 连续3次沉默，强制结束")
            end_msg = "看起来你可能需要一些时间思考。没关系，我们可以先到这里。如果你准备好了，随时可以继续聊。"
            yield f"data: {json.dumps({'type': 'chunk', 'content': end_msg}, ensure_ascii=False)}\n\n"
            yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
            yield f"data: {json.dumps({'type': 'over', 'content': 'over'}, ensure_ascii=False)}\n\n"
            return
        
        # 步骤2：判断是否首次调用（用 history 判断）
        is_first_call = (history is None or len(history) == 0)
        
        if is_first_call:
            # 步骤3：生成对话计划（仅首次）
            logging.info("[DeepResumeConverse] 首次调用，生成对话计划")
            opening_message, plan = await self.generate_plan(user_profile)
            
            # 返回开场白
            yield f"data: {json.dumps({'type': 'chunk', 'content': opening_message}, ensure_ascii=False)}\n\n"
            yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
            
            # 返回对话计划
            if plan:
                yield f"data: {json.dumps({'type': 'plan', 'content': plan}, ensure_ascii=False)}\n\n"
            
            logging.info(f"[DeepResumeConverse] 首次调用完成，耗时: {time.time() - start_time:.2f}s")
            return
        
        # 步骤4：构建 plan_info + 对话策略生成
        strategy = await self.generate_strategy(
            user_profile, history, user_message, conversation_plan
        )
        logging.info(f"[DeepResumeConverse] 策略: {strategy[:100]}...")
        
        # 步骤5：对话执行（流式）
        ai_response = ""
        async for chunk in self.execute_response(
            user_profile, history, user_message, strategy
        ):
            ai_response += chunk
            yield f"data: {json.dumps({'type': 'chunk', 'content': chunk}, ensure_ascii=False)}\n\n"
        
        yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
        
        # 步骤6：意图识别（用于日志记录）
        intent = self.detect_intent(ai_response, history[-6:] if history else [])
        logging.info(f"[DeepResumeConverse] 意图: {intent.get('action')}")
        
        # 步骤7：结束判断
        is_end = self.judge_end(ai_response)
        if is_end:
            yield f"data: {json.dumps({'type': 'over', 'content': 'over'}, ensure_ascii=False)}\n\n"
        
        logging.info(f"[DeepResumeConverse] 对话完成，耗时: {time.time() - start_time:.2f}s")


```

```python
import json
import logging
import time
from pathlib import Path
from openai import OpenAI
from app.config import ai_config
from app.identity import get_identity_prompt
from service.llm_interface import OpenAILLMClient


def load_prompt(filename: str) -> str:
    """
    加载prompt文件
    
    Args:
        filename: 文件名，如 "generate_plan.txt"
    
    Returns:
        prompt内容
    """
    current_dir = Path(__file__).parent
    prompt_path = current_dir.parent.parent / "prompts" / "deep_resume" / filename
    with open(prompt_path, 'r', encoding='utf-8') as f:
        return f.read()


class DeepResumeConverse:
    """深度简历优化对话服务"""
    
    def __init__(self, llm_client=None):
        self.ai_config = ai_config
        
        # 如果没有传入 LLM 客户端，则使用默认的生产环境客户端
        if llm_client is None:
            llm_client = OpenAILLMClient(
                api_key=self.ai_config['api_key'],
                base_url=self.ai_config['api_uri'],
                model=self.ai_config['api_model']
            )
        self.llm_client = llm_client
        
        # 用于快速模型调用（意图识别、结束判断）
        self.fast_client = OpenAI(
            api_key=self.ai_config['api_key'],
            base_url=self.ai_config['api_uri'],
            timeout=60.0,
        )
    
    def _check_silence(self, history: list) -> int:
        """
        检测连续沉默次数
        
        Args:
            history: 对话历史
            
        Returns:
            连续沉默次数
        """
        if not history:
            return 0
            
        silence_count = 0
        invalid_messages = ["", "...", "嗯", "啊", "哦", "额"]
        
        for msg in reversed(history):
            if msg.get("role") == "user":
                content = msg.get("content", "").strip()
                if not content or content in invalid_messages:
                    silence_count += 1
                else:
                    break
        
        return silence_count
    
    def _format_history(self, history: list) -> str:
        """格式化对话历史为字符串"""
        if not history:
            return "暂无对话历史"
        
        formatted = []
        for msg in history:
            role = "学生" if msg.get("role") == "user" else "培训师"
            content = msg.get("content", "")
            formatted.append(f"{role}: {content}")
        
        return "\n".join(formatted)
    
    def _format_user_profile(self, user_profile: dict) -> str:
        """格式化用户档案为字符串，适配前端传入的数据结构"""
        base_info = user_profile.get("base_info", {})
        edu_exp_list = user_profile.get("edu_exp", [])
        job_exp_list = user_profile.get("job_exp", [])
        work_exp_list = user_profile.get("work_exp", [])
        
        # 基本信息
        result = f"""
【基本信息】
姓名: {base_info.get('name', '未知')}
性别: {base_info.get('sex', '未知')}
出生日期: {base_info.get('birth_year_month', '未知')}
学历: {base_info.get('degree', '未知')}
工作年限: {base_info.get('work_year', '未知')}年
自我评价: {base_info.get('advantages_highlights', '无')}
"""
        
        # 教育经历
        if edu_exp_list:
            result += "\n【教育经历】\n"
            for edu in edu_exp_list:
                result += f"- {edu.get('school', '未知')} | {edu.get('specialty', '未知')} | {edu.get('education_background', '未知')} | {edu.get('begin_date', '')} 至 {edu.get('end_date', '')}\n"
        
        # 求职期望
        if job_exp_list:
            result += "\n【求职期望】\n"
            for job in job_exp_list:
                result += f"- 期望职位: {job.get('desired_position', '未知')} | 期望地点: {job.get('desired_location', '未知')} | 期望薪资: {job.get('expected_salary', '未知')}千/月\n"
        
        # 工作经历
        if work_exp_list:
            result += "\n【工作经历】\n"
            for work in work_exp_list:
                result += f"- {work.get('company_name', '未知')} | {work.get('position', '未知')} | {work.get('begin_date', '')} 至 {work.get('end_date', '')}\n"
                result += f"  职责: {work.get('responsibility_performance', '无')}\n"
        
        return result
    
    def _format_plan_info(self, conversation_plan: dict) -> str:
        """
        格式化对话计划为参考信息
        
        Args:
            conversation_plan: 对话计划（可能为空）
            
        Returns:
            格式化的计划信息，如果为空则返回空字符串
        """
        if not conversation_plan:
            return ""
        
        stages = conversation_plan.get("stages", [])
        if not stages:
            return ""
        
        stage_info = []
        for stage in stages:
            stage_info.append(f"- {stage.get('stage', '')}: {stage.get('goal', '')}")
        
        return f"""
【对话计划参考】
{chr(10).join(stage_info)}
"""
    
    async def generate_plan(self, user_profile: dict):
        """
        生成对话计划（仅首次调用）
        
        Args:
            user_profile: 用户档案
            
        Returns:
            (opening_message, plan) 开场白和计划
        """
        logging.info("[DeepResumeConverse] 生成对话计划")
        
        try:
            prompt_template = load_prompt("generate_plan.txt")
            prompt = prompt_template.format(
                user_profile=self._format_user_profile(user_profile)
            )
            
            identity_prompt = get_identity_prompt("deep_resume")
            response = self.llm_client.create_completion(
                messages=[
                    {"role": "system", "content": f"{identity_prompt}\n\n{prompt}"},
                    {"role": "user", "content": "请生成对话计划"}
                ],
                temperature=0.7
            )
            
            result_text = response.choices[0].message.content
            # 清理可能的markdown标记
            result_text = result_text.replace("```json", "").replace("```", "").strip()
            
            plan_data = json.loads(result_text)
            opening_message = plan_data.get("opening_message", "你好！让我们来聊聊你的经历吧。")
            stages = plan_data.get("stages", [])
            
            plan = {"stages": stages}
            
            logging.info(f"[DeepResumeConverse] 对话计划生成成功，共{len(stages)}个阶段")
            return opening_message, plan
            
        except Exception as e:
            logging.error(f"[DeepResumeConverse] 生成对话计划失败: {e}")
            # 降级：返回默认开场白和空计划
            return "你好！让我们来聊聊你的经历吧。你最引以为傲的项目或经历是什么？", None
    
    async def generate_strategy(self, user_profile: dict, history: list, 
                                 user_message: str, conversation_plan: dict) -> str:
        """
        生成对话策略
        
        Args:
            user_profile: 用户档案
            history: 对话历史
            user_message: 用户最新回复
            conversation_plan: 对话计划（可选）
            
        Returns:
            策略文本
        """
        logging.info("[DeepResumeConverse] 生成对话策略")
        
        try:
            prompt_template = load_prompt("strategy.txt")
            plan_info = self._format_plan_info(conversation_plan)
            
            prompt = prompt_template.format(
                user_profile=self._format_user_profile(user_profile),
                history=self._format_history(history),
                user_message=user_message,
                plan_info=plan_info
            )
            
            response = self.llm_client.create_completion(
                messages=[
                    {"role": "system", "content": prompt},
                    {"role": "user", "content": "请制定下一步对话策略"}
                ],
                temperature=0.4
            )
            
            strategy = response.choices[0].message.content
            logging.info(f"[DeepResumeConverse] 策略生成成功")
            return strategy
            
        except Exception as e:
            logging.error(f"[DeepResumeConverse] 生成策略失败: {e}")
            return "继续追问细节，挖掘更多信息"
    
    async def execute_response(self, user_profile: dict, history: list,
                                user_message: str, strategy: str):
        """
        执行对话，流式生成AI回复
        
        Args:
            user_profile: 用户档案
            history: 对话历史
            user_message: 用户最新回复
            strategy: 对话策略
            
        Yields:
            AI回复的文本片段
        """
        logging.info("[DeepResumeConverse] 执行对话生成")
        
        try:
            prompt_template = load_prompt("executor.txt")
            prompt = prompt_template.format(
                user_profile=self._format_user_profile(user_profile),
                history=self._format_history(history),
                user_message=user_message,
                strategy=strategy
            )
            
            response = self.llm_client.create_streaming_completion(
                messages=[
                    {"role": "system", "content": prompt},
                    {"role": "user", "content": "请生成下一轮对话内容"}
                ],
                temperature=0.7
            )
            
            async for chunk in response:
                if chunk.choices and chunk.choices[0].delta.content:
                    yield chunk.choices[0].delta.content
                    
        except Exception as e:
            logging.error(f"[DeepResumeConverse] 执行对话失败: {e}")
            yield "抱歉，我需要整理一下思路。你能再说一遍吗？"
    
    def detect_intent(self, ai_response: str, recent_history: list) -> dict:
        """
        检测AI回复的意图类型
        
        Args:
            ai_response: AI的回复内容
            recent_history: 最近的对话历史
            
        Returns:
            {"action": "new_question" | "follow_up" | "case_analysis" | "check_status" | "end"}
        """
        try:
            system_prompt = """你是对话意图分析器，判断AI培训师的回复属于哪种类型。

输出严格JSON格式：
{
  "action": "new_question" | "follow_up" | "case_analysis" | "check_status" | "end"
}

类型定义：
- new_question: 提出了新的话题或问题
- follow_up: 针对当前话题追问细节
- case_analysis: 给出案例场景让学生分析
- check_status: 询问学生状态
- end: 表示对话可以结束"""
            
            response = self.fast_client.chat.completions.create(
                model="qwen-flash",
                temperature=0.1,
                response_format={"type": "json_object"},
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": f"AI回复：{ai_response}"}
                ]
            )
            
            result = json.loads(response.choices[0].message.content)
            logging.info(f"[DeepResumeConverse] 意图识别结果: {result}")
            return result
            
        except Exception as e:
            logging.error(f"[DeepResumeConverse] 意图识别失败: {e}")
            return {"action": "follow_up"}
    
    def judge_end(self, ai_response: str) -> bool:
        """
        判断对话是否结束
        
        Args:
            ai_response: AI的回复内容
            
        Returns:
            是否结束
        """
        try:
            prompt_template = load_prompt("judge_end.txt")
            prompt = prompt_template.format(ai_response=ai_response)
            
            response = self.fast_client.chat.completions.create(
                model="qwen-flash",
                temperature=0.1,
                messages=[
                    {"role": "system", "content": prompt},
                    {"role": "user", "content": "请判断"}
                ]
            )
            
            result = response.choices[0].message.content.strip()
            is_end = "结束" in result
            logging.info(f"[DeepResumeConverse] 结束判断: {result}, is_end={is_end}")
            return is_end
            
        except Exception as e:
            logging.error(f"[DeepResumeConverse] 结束判断失败: {e}")
            return False
    
    async def converse(self, user_profile: dict, history: list, 
                       user_message: str, conversation_plan: dict = None):
        """
        主对话流程
        
        Args:
            user_profile: 用户档案
            history: 对话历史
            user_message: 用户最新回复
            conversation_plan: 对话计划（可选增强项）
            
        Yields:
            SSE格式的响应数据
        """
        start_time = time.time()
        
        # 步骤1：沉默检测
        silence_count = self._check_silence(history)
        logging.info(f"[DeepResumeConverse] 沉默检测: {silence_count}/3")
        
        if silence_count >= 3:
            logging.info("[DeepResumeConverse] 连续3次沉默，强制结束")
            end_msg = "看起来你可能需要一些时间思考。没关系，我们可以先到这里。如果你准备好了，随时可以继续聊。"
            yield f"data: {json.dumps({'type': 'chunk', 'content': end_msg}, ensure_ascii=False)}\n\n"
            yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
            yield f"data: {json.dumps({'type': 'over', 'content': 'over'}, ensure_ascii=False)}\n\n"
            return
        
        # 步骤2：判断是否首次调用（用 history 判断）
        is_first_call = (history is None or len(history) == 0)
        
        if is_first_call:
            # 步骤3：生成对话计划（仅首次）
            logging.info("[DeepResumeConverse] 首次调用，生成对话计划")
            opening_message, plan = await self.generate_plan(user_profile)
            
            # 返回开场白
            yield f"data: {json.dumps({'type': 'chunk', 'content': opening_message}, ensure_ascii=False)}\n\n"
            yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
            
            # 返回对话计划
            if plan:
                yield f"data: {json.dumps({'type': 'plan', 'content': plan}, ensure_ascii=False)}\n\n"
            
            logging.info(f"[DeepResumeConverse] 首次调用完成，耗时: {time.time() - start_time:.2f}s")
            return
        
        # 步骤4：构建 plan_info + 对话策略生成
        strategy = await self.generate_strategy(
            user_profile, history, user_message, conversation_plan
        )
        logging.info(f"[DeepResumeConverse] 策略: {strategy[:100]}...")
        
        # 步骤5：对话执行（流式）
        ai_response = ""
        async for chunk in self.execute_response(
            user_profile, history, user_message, strategy
        ):
            ai_response += chunk
            yield f"data: {json.dumps({'type': 'chunk', 'content': chunk}, ensure_ascii=False)}\n\n"
        
        yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
        
        # 步骤6：意图识别（用于日志记录）
        intent = self.detect_intent(ai_response, history[-6:] if history else [])
        logging.info(f"[DeepResumeConverse] 意图: {intent.get('action')}")
        
        # 步骤7：结束判断
        is_end = self.judge_end(ai_response)
        if is_end:
            yield f"data: {json.dumps({'type': 'over', 'content': 'over'}, ensure_ascii=False)}\n\n"
        
        logging.info(f"[DeepResumeConverse] 对话完成，耗时: {time.time() - start_time:.2f}s")


```

```python
import json
import logging
import time
from pathlib import Path
from openai import OpenAI
from app.config import ai_config
from app.identity import get_identity_prompt
from service.llm_interface import OpenAILLMClient


def load_prompt(filename: str) -> str:
    """
    加载prompt文件
    
    Args:
        filename: 文件名，如 "generate_plan.txt"
    
    Returns:
        prompt内容
    """
    current_dir = Path(__file__).parent
    prompt_path = current_dir.parent.parent / "prompts" / "deep_resume" / filename
    with open(prompt_path, 'r', encoding='utf-8') as f:
        return f.read()


class DeepResumeConverse:
    """深度简历优化对话服务"""
    
    def __init__(self, llm_client=None):
        self.ai_config = ai_config
        
        # 如果没有传入 LLM 客户端，则使用默认的生产环境客户端
        if llm_client is None:
            llm_client = OpenAILLMClient(
                api_key=self.ai_config['api_key'],
                base_url=self.ai_config['api_uri'],
                model=self.ai_config['api_model']
            )
        self.llm_client = llm_client
        
        # 用于快速模型调用（意图识别、结束判断）
        self.fast_client = OpenAI(
            api_key=self.ai_config['api_key'],
            base_url=self.ai_config['api_uri'],
            timeout=60.0,
        )
    
    def _check_silence(self, history: list) -> int:
        """
        检测连续沉默次数
        
        Args:
            history: 对话历史
            
        Returns:
            连续沉默次数
        """
        if not history:
            return 0
            
        silence_count = 0
        invalid_messages = ["", "...", "嗯", "啊", "哦", "额"]
        
        for msg in reversed(history):
            if msg.get("role") == "user":
                content = msg.get("content", "").strip()
                if not content or content in invalid_messages:
                    silence_count += 1
                else:
                    break
        
        return silence_count
    
    def _format_history(self, history: list) -> str:
        """格式化对话历史为字符串"""
        if not history:
            return "暂无对话历史"
        
        formatted = []
        for msg in history:
            role = "学生" if msg.get("role") == "user" else "培训师"
            content = msg.get("content", "")
            formatted.append(f"{role}: {content}")
        
        return "\n".join(formatted)
    
    def _format_user_profile(self, user_profile: dict) -> str:
        """格式化用户档案为字符串，适配前端传入的数据结构"""
        base_info = user_profile.get("base_info", {})
        edu_exp_list = user_profile.get("edu_exp", [])
        job_exp_list = user_profile.get("job_exp", [])
        work_exp_list = user_profile.get("work_exp", [])
        
        # 基本信息
        result = f"""
【基本信息】
姓名: {base_info.get('name', '未知')}
性别: {base_info.get('sex', '未知')}
出生日期: {base_info.get('birth_year_month', '未知')}
学历: {base_info.get('degree', '未知')}
工作年限: {base_info.get('work_year', '未知')}年
自我评价: {base_info.get('advantages_highlights', '无')}
"""
        
        # 教育经历
        if edu_exp_list:
            result += "\n【教育经历】\n"
            for edu in edu_exp_list:
                result += f"- {edu.get('school', '未知')} | {edu.get('specialty', '未知')} | {edu.get('education_background', '未知')} | {edu.get('begin_date', '')} 至 {edu.get('end_date', '')}\n"
        
        # 求职期望
        if job_exp_list:
            result += "\n【求职期望】\n"
            for job in job_exp_list:
                result += f"- 期望职位: {job.get('desired_position', '未知')} | 期望地点: {job.get('desired_location', '未知')} | 期望薪资: {job.get('expected_salary', '未知')}千/月\n"
        
        # 工作经历
        if work_exp_list:
            result += "\n【工作经历】\n"
            for work in work_exp_list:
                result += f"- {work.get('company_name', '未知')} | {work.get('position', '未知')} | {work.get('begin_date', '')} 至 {work.get('end_date', '')}\n"
                result += f"  职责: {work.get('responsibility_performance', '无')}\n"
        
        return result
    
    def _format_plan_info(self, conversation_plan: dict) -> str:
        """
        格式化对话计划为参考信息
        
        Args:
            conversation_plan: 对话计划（可能为空）
            
        Returns:
            格式化的计划信息，如果为空则返回空字符串
        """
        if not conversation_plan:
            return ""
        
        stages = conversation_plan.get("stages", [])
        if not stages:
            return ""
        
        stage_info = []
        for stage in stages:
            stage_info.append(f"- {stage.get('stage', '')}: {stage.get('goal', '')}")
        
        return f"""
【对话计划参考】
{chr(10).join(stage_info)}
"""
    
    async def generate_plan(self, user_profile: dict):
        """
        生成对话计划（仅首次调用）
        
        Args:
            user_profile: 用户档案
            
        Returns:
            (opening_message, plan) 开场白和计划
        """
        logging.info("[DeepResumeConverse] 生成对话计划")
        
        try:
            prompt_template = load_prompt("generate_plan.txt")
            prompt = prompt_template.format(
                user_profile=self._format_user_profile(user_profile)
            )
            
            identity_prompt = get_identity_prompt("deep_resume")
            response = self.llm_client.create_completion(
                messages=[
                    {"role": "system", "content": f"{identity_prompt}\n\n{prompt}"},
                    {"role": "user", "content": "请生成对话计划"}
                ],
                temperature=0.7
            )
            
            result_text = response.choices[0].message.content
            # 清理可能的markdown标记
            result_text = result_text.replace("```json", "").replace("```", "").strip()
            
            plan_data = json.loads(result_text)
            opening_message = plan_data.get("opening_message", "你好！让我们来聊聊你的经历吧。")
            stages = plan_data.get("stages", [])
            
            plan = {"stages": stages}
            
            logging.info(f"[DeepResumeConverse] 对话计划生成成功，共{len(stages)}个阶段")
            return opening_message, plan
            
        except Exception as e:
            logging.error(f"[DeepResumeConverse] 生成对话计划失败: {e}")
            # 降级：返回默认开场白和空计划
            return "你好！让我们来聊聊你的经历吧。你最引以为傲的项目或经历是什么？", None
    
    async def generate_strategy(self, user_profile: dict, history: list, 
                                 user_message: str, conversation_plan: dict) -> str:
        """
        生成对话策略
        
        Args:
            user_profile: 用户档案
            history: 对话历史
            user_message: 用户最新回复
            conversation_plan: 对话计划（可选）
            
        Returns:
            策略文本
        """
        logging.info("[DeepResumeConverse] 生成对话策略")
        
        try:
            prompt_template = load_prompt("strategy.txt")
            plan_info = self._format_plan_info(conversation_plan)
            
            prompt = prompt_template.format(
                user_profile=self._format_user_profile(user_profile),
                history=self._format_history(history),
                user_message=user_message,
                plan_info=plan_info
            )
            
            identity_prompt = get_identity_prompt("deep_resume")
            response = self.llm_client.create_completion(
                messages=[
                    {"role": "system", "content": f"{identity_prompt}\n\n{prompt}"},
                    {"role": "user", "content": "请制定下一步对话策略"}
                ],
                temperature=0.4
            )
            
            strategy = response.choices[0].message.content
            logging.info(f"[DeepResumeConverse] 策略生成成功")
            return strategy
            
        except Exception as e:
            logging.error(f"[DeepResumeConverse] 生成策略失败: {e}")
            return "继续追问细节，挖掘更多信息"
    
    async def execute_response(self, user_profile: dict, history: list,
                                user_message: str, strategy: str):
        """
        执行对话，流式生成AI回复
        
        Args:
            user_profile: 用户档案
            history: 对话历史
            user_message: 用户最新回复
            strategy: 对话策略
            
        Yields:
            AI回复的文本片段
        """
        logging.info("[DeepResumeConverse] 执行对话生成")
        
        try:
            prompt_template = load_prompt("executor.txt")
            prompt = prompt_template.format(
                user_profile=self._format_user_profile(user_profile),
                history=self._format_history(history),
                user_message=user_message,
                strategy=strategy
            )
            
            response = self.llm_client.create_streaming_completion(
                messages=[
                    {"role": "system", "content": prompt},
                    {"role": "user", "content": "请生成下一轮对话内容"}
                ],
                temperature=0.7
            )
            
            async for chunk in response:
                if chunk.choices and chunk.choices[0].delta.content:
                    yield chunk.choices[0].delta.content
                    
        except Exception as e:
            logging.error(f"[DeepResumeConverse] 执行对话失败: {e}")
            yield "抱歉，我需要整理一下思路。你能再说一遍吗？"
    
    def detect_intent(self, ai_response: str, recent_history: list) -> dict:
        """
        检测AI回复的意图类型
        
        Args:
            ai_response: AI的回复内容
            recent_history: 最近的对话历史
            
        Returns:
            {"action": "new_question" | "follow_up" | "case_analysis" | "check_status" | "end"}
        """
        try:
            system_prompt = """你是对话意图分析器，判断AI培训师的回复属于哪种类型。

输出严格JSON格式：
{
  "action": "new_question" | "follow_up" | "case_analysis" | "check_status" | "end"
}

类型定义：
- new_question: 提出了新的话题或问题
- follow_up: 针对当前话题追问细节
- case_analysis: 给出案例场景让学生分析
- check_status: 询问学生状态
- end: 表示对话可以结束"""
            
            response = self.fast_client.chat.completions.create(
                model="qwen-flash",
                temperature=0.1,
                response_format={"type": "json_object"},
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": f"AI回复：{ai_response}"}
                ]
            )
            
            result = json.loads(response.choices[0].message.content)
            logging.info(f"[DeepResumeConverse] 意图识别结果: {result}")
            return result
            
        except Exception as e:
            logging.error(f"[DeepResumeConverse] 意图识别失败: {e}")
            return {"action": "follow_up"}
    
    def judge_end(self, ai_response: str) -> bool:
        """
        判断对话是否结束
        
        Args:
            ai_response: AI的回复内容
            
        Returns:
            是否结束
        """
        try:
            prompt_template = load_prompt("judge_end.txt")
            prompt = prompt_template.format(ai_response=ai_response)
            
            response = self.fast_client.chat.completions.create(
                model="qwen-flash",
                temperature=0.1,
                messages=[
                    {"role": "system", "content": prompt},
                    {"role": "user", "content": "请判断"}
                ]
            )
            
            result = response.choices[0].message.content.strip()
            is_end = "结束" in result
            logging.info(f"[DeepResumeConverse] 结束判断: {result}, is_end={is_end}")
            return is_end
            
        except Exception as e:
            logging.error(f"[DeepResumeConverse] 结束判断失败: {e}")
            return False
    
    async def converse(self, user_profile: dict, history: list, 
                       user_message: str, conversation_plan: dict = None):
        """
        主对话流程
        
        Args:
            user_profile: 用户档案
            history: 对话历史
            user_message: 用户最新回复
            conversation_plan: 对话计划（可选增强项）
            
        Yields:
            SSE格式的响应数据
        """
        start_time = time.time()
        
        # 步骤1：沉默检测
        silence_count = self._check_silence(history)
        logging.info(f"[DeepResumeConverse] 沉默检测: {silence_count}/3")
        
        if silence_count >= 3:
            logging.info("[DeepResumeConverse] 连续3次沉默，强制结束")
            end_msg = "看起来你可能需要一些时间思考。没关系，我们可以先到这里。如果你准备好了，随时可以继续聊。"
            yield f"data: {json.dumps({'type': 'chunk', 'content': end_msg}, ensure_ascii=False)}\n\n"
            yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
            yield f"data: {json.dumps({'type': 'over', 'content': 'over'}, ensure_ascii=False)}\n\n"
            return
        
        # 步骤2：判断是否首次调用（用 history 判断）
        is_first_call = (history is None or len(history) == 0)
        
        if is_first_call:
            # 步骤3：生成对话计划（仅首次）
            logging.info("[DeepResumeConverse] 首次调用，生成对话计划")
            opening_message, plan = await self.generate_plan(user_profile)
            
            # 返回开场白
            yield f"data: {json.dumps({'type': 'chunk', 'content': opening_message}, ensure_ascii=False)}\n\n"
            yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
            
            # 返回对话计划
            if plan:
                yield f"data: {json.dumps({'type': 'plan', 'content': plan}, ensure_ascii=False)}\n\n"
            
            logging.info(f"[DeepResumeConverse] 首次调用完成，耗时: {time.time() - start_time:.2f}s")
            return
        
        # 步骤4：构建 plan_info + 对话策略生成
        strategy = await self.generate_strategy(
            user_profile, history, user_message, conversation_plan
        )
        logging.info(f"[DeepResumeConverse] 策略: {strategy[:100]}...")
        
        # 步骤5：对话执行（流式）
        ai_response = ""
        async for chunk in self.execute_response(
            user_profile, history, user_message, strategy
        ):
            ai_response += chunk
            yield f"data: {json.dumps({'type': 'chunk', 'content': chunk}, ensure_ascii=False)}\n\n"
        
        yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
        
        # 步骤6：意图识别（用于日志记录）
        intent = self.detect_intent(ai_response, history[-6:] if history else [])
        logging.info(f"[DeepResumeConverse] 意图: {intent.get('action')}")
        
        # 步骤7：结束判断
        is_end = self.judge_end(ai_response)
        if is_end:
            yield f"data: {json.dumps({'type': 'over', 'content': 'over'}, ensure_ascii=False)}\n\n"
        
        logging.info(f"[DeepResumeConverse] 对话完成，耗时: {time.time() - start_time:.2f}s")


```

```python
import json
import logging
import time
from pathlib import Path
from openai import OpenAI
from app.config import ai_config
from app.identity import get_identity_prompt
from service.llm_interface import OpenAILLMClient


def load_prompt(filename: str) -> str:
    """
    加载prompt文件
    
    Args:
        filename: 文件名，如 "generate_plan.txt"
    
    Returns:
        prompt内容
    """
    current_dir = Path(__file__).parent
    prompt_path = current_dir.parent.parent / "prompts" / "deep_resume" / filename
    with open(prompt_path, 'r', encoding='utf-8') as f:
        return f.read()


class DeepResumeConverse:
    """深度简历优化对话服务"""
    
    def __init__(self, llm_client=None):
        self.ai_config = ai_config
        
        # 如果没有传入 LLM 客户端，则使用默认的生产环境客户端
        if llm_client is None:
            llm_client = OpenAILLMClient(
                api_key=self.ai_config['api_key'],
                base_url=self.ai_config['api_uri'],
                model=self.ai_config['api_model']
            )
        self.llm_client = llm_client
        
        # 用于快速模型调用（意图识别、结束判断）
        self.fast_client = OpenAI(
            api_key=self.ai_config['api_key'],
            base_url=self.ai_config['api_uri'],
            timeout=60.0,
        )
    
    def _check_silence(self, history: list) -> int:
        """
        检测连续沉默次数
        
        Args:
            history: 对话历史
            
        Returns:
            连续沉默次数
        """
        if not history:
            return 0
            
        silence_count = 0
        invalid_messages = ["", "...", "嗯", "啊", "哦", "额"]
        
        for msg in reversed(history):
            if msg.get("role") == "user":
                content = msg.get("content", "").strip()
                if not content or content in invalid_messages:
                    silence_count += 1
                else:
                    break
        
        return silence_count
    
    def _format_history(self, history: list) -> str:
        """格式化对话历史为字符串"""
        if not history:
            return "暂无对话历史"
        
        formatted = []
        for msg in history:
            role = "学生" if msg.get("role") == "user" else "培训师"
            content = msg.get("content", "")
            formatted.append(f"{role}: {content}")
        
        return "\n".join(formatted)
    
    def _format_user_profile(self, user_profile: dict) -> str:
        """格式化用户档案为字符串，适配前端传入的数据结构"""
        base_info = user_profile.get("base_info", {})
        edu_exp_list = user_profile.get("edu_exp", [])
        job_exp_list = user_profile.get("job_exp", [])
        work_exp_list = user_profile.get("work_exp", [])
        
        # 基本信息
        result = f"""
【基本信息】
姓名: {base_info.get('name', '未知')}
性别: {base_info.get('sex', '未知')}
出生日期: {base_info.get('birth_year_month', '未知')}
学历: {base_info.get('degree', '未知')}
工作年限: {base_info.get('work_year', '未知')}年
自我评价: {base_info.get('advantages_highlights', '无')}
"""
        
        # 教育经历
        if edu_exp_list:
            result += "\n【教育经历】\n"
            for edu in edu_exp_list:
                result += f"- {edu.get('school', '未知')} | {edu.get('specialty', '未知')} | {edu.get('education_background', '未知')} | {edu.get('begin_date', '')} 至 {edu.get('end_date', '')}\n"
        
        # 求职期望
        if job_exp_list:
            result += "\n【求职期望】\n"
            for job in job_exp_list:
                result += f"- 期望职位: {job.get('desired_position', '未知')} | 期望地点: {job.get('desired_location', '未知')} | 期望薪资: {job.get('expected_salary', '未知')}千/月\n"
        
        # 工作经历
        if work_exp_list:
            result += "\n【工作经历】\n"
            for work in work_exp_list:
                result += f"- {work.get('company_name', '未知')} | {work.get('position', '未知')} | {work.get('begin_date', '')} 至 {work.get('end_date', '')}\n"
                result += f"  职责: {work.get('responsibility_performance', '无')}\n"
        
        return result
    
    def _format_plan_info(self, conversation_plan: dict) -> str:
        """
        格式化对话计划为参考信息
        
        Args:
            conversation_plan: 对话计划（可能为空）
            
        Returns:
            格式化的计划信息，如果为空则返回空字符串
        """
        if not conversation_plan:
            return ""
        
        stages = conversation_plan.get("stages", [])
        if not stages:
            return ""
        
        stage_info = []
        for stage in stages:
            stage_info.append(f"- {stage.get('stage', '')}: {stage.get('goal', '')}")
        
        return f"""
【对话计划参考】
{chr(10).join(stage_info)}
"""
    
    async def generate_plan(self, user_profile: dict):
        """
        生成对话计划（仅首次调用）
        
        Args:
            user_profile: 用户档案
            
        Returns:
            (opening_message, plan) 开场白和计划
        """
        logging.info("[DeepResumeConverse] 生成对话计划")
        
        try:
            prompt_template = load_prompt("generate_plan.txt")
            prompt = prompt_template.format(
                user_profile=self._format_user_profile(user_profile)
            )
            
            identity_prompt = get_identity_prompt("deep_resume")
            response = self.llm_client.create_completion(
                messages=[
                    {"role": "system", "content": f"{identity_prompt}\n\n{prompt}"},
                    {"role": "user", "content": "请生成对话计划"}
                ],
                temperature=0.7
            )
            
            result_text = response.choices[0].message.content
            # 清理可能的markdown标记
            result_text = result_text.replace("```json", "").replace("```", "").strip()
            
            plan_data = json.loads(result_text)
            opening_message = plan_data.get("opening_message", "你好！让我们来聊聊你的经历吧。")
            stages = plan_data.get("stages", [])
            
            plan = {"stages": stages}
            
            logging.info(f"[DeepResumeConverse] 对话计划生成成功，共{len(stages)}个阶段")
            return opening_message, plan
            
        except Exception as e:
            logging.error(f"[DeepResumeConverse] 生成对话计划失败: {e}")
            # 降级：返回默认开场白和空计划
            return "你好！让我们来聊聊你的经历吧。你最引以为傲的项目或经历是什么？", None
    
    async def generate_strategy(self, user_profile: dict, history: list, 
                                 user_message: str, conversation_plan: dict) -> str:
        """
        生成对话策略
        
        Args:
            user_profile: 用户档案
            history: 对话历史
            user_message: 用户最新回复
            conversation_plan: 对话计划（可选）
            
        Returns:
            策略文本
        """
        logging.info("[DeepResumeConverse] 生成对话策略")
        
        try:
            prompt_template = load_prompt("strategy.txt")
            plan_info = self._format_plan_info(conversation_plan)
            
            prompt = prompt_template.format(
                user_profile=self._format_user_profile(user_profile),
                history=self._format_history(history),
                user_message=user_message,
                plan_info=plan_info
            )
            
            identity_prompt = get_identity_prompt("deep_resume")
            response = self.llm_client.create_completion(
                messages=[
                    {"role": "system", "content": f"{identity_prompt}\n\n{prompt}"},
                    {"role": "user", "content": "请制定下一步对话策略"}
                ],
                temperature=0.4
            )
            
            strategy = response.choices[0].message.content
            logging.info(f"[DeepResumeConverse] 策略生成成功")
            return strategy
            
        except Exception as e:
            logging.error(f"[DeepResumeConverse] 生成策略失败: {e}")
            return "继续追问细节，挖掘更多信息"
    
    async def execute_response(self, user_profile: dict, history: list,
                                user_message: str, strategy: str):
        """
        执行对话，流式生成AI回复
        
        Args:
            user_profile: 用户档案
            history: 对话历史
            user_message: 用户最新回复
            strategy: 对话策略
            
        Yields:
            AI回复的文本片段
        """
        logging.info("[DeepResumeConverse] 执行对话生成")
        
        try:
            prompt_template = load_prompt("executor.txt")
            prompt = prompt_template.format(
                user_profile=self._format_user_profile(user_profile),
                history=self._format_history(history),
                user_message=user_message,
                strategy=strategy
            )
            
            identity_prompt = get_identity_prompt("deep_resume")
            response = self.llm_client.create_streaming_completion(
                messages=[
                    {"role": "system", "content": f"{identity_prompt}\n\n{prompt}"},
                    {"role": "user", "content": "请生成下一轮对话内容"}
                ],
                temperature=0.7
            )
            
            async for chunk in response:
                if chunk.choices and chunk.choices[0].delta.content:
                    yield chunk.choices[0].delta.content
                    
        except Exception as e:
            logging.error(f"[DeepResumeConverse] 执行对话失败: {e}")
            yield "抱歉，我需要整理一下思路。你能再说一遍吗？"
    
    def detect_intent(self, ai_response: str, recent_history: list) -> dict:
        """
        检测AI回复的意图类型
        
        Args:
            ai_response: AI的回复内容
            recent_history: 最近的对话历史
            
        Returns:
            {"action": "new_question" | "follow_up" | "case_analysis" | "check_status" | "end"}
        """
        try:
            system_prompt = """你是对话意图分析器，判断AI培训师的回复属于哪种类型。

输出严格JSON格式：
{
  "action": "new_question" | "follow_up" | "case_analysis" | "check_status" | "end"
}

类型定义：
- new_question: 提出了新的话题或问题
- follow_up: 针对当前话题追问细节
- case_analysis: 给出案例场景让学生分析
- check_status: 询问学生状态
- end: 表示对话可以结束"""
            
            response = self.fast_client.chat.completions.create(
                model="qwen-flash",
                temperature=0.1,
                response_format={"type": "json_object"},
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": f"AI回复：{ai_response}"}
                ]
            )
            
            result = json.loads(response.choices[0].message.content)
            logging.info(f"[DeepResumeConverse] 意图识别结果: {result}")
            return result
            
        except Exception as e:
            logging.error(f"[DeepResumeConverse] 意图识别失败: {e}")
            return {"action": "follow_up"}
    
    def judge_end(self, ai_response: str) -> bool:
        """
        判断对话是否结束
        
        Args:
            ai_response: AI的回复内容
            
        Returns:
            是否结束
        """
        try:
            prompt_template = load_prompt("judge_end.txt")
            prompt = prompt_template.format(ai_response=ai_response)
            
            response = self.fast_client.chat.completions.create(
                model="qwen-flash",
                temperature=0.1,
                messages=[
                    {"role": "system", "content": prompt},
                    {"role": "user", "content": "请判断"}
                ]
            )
            
            result = response.choices[0].message.content.strip()
            is_end = "结束" in result
            logging.info(f"[DeepResumeConverse] 结束判断: {result}, is_end={is_end}")
            return is_end
            
        except Exception as e:
            logging.error(f"[DeepResumeConverse] 结束判断失败: {e}")
            return False
    
    async def converse(self, user_profile: dict, history: list, 
                       user_message: str, conversation_plan: dict = None):
        """
        主对话流程
        
        Args:
            user_profile: 用户档案
            history: 对话历史
            user_message: 用户最新回复
            conversation_plan: 对话计划（可选增强项）
            
        Yields:
            SSE格式的响应数据
        """
        start_time = time.time()
        
        # 步骤1：沉默检测
        silence_count = self._check_silence(history)
        logging.info(f"[DeepResumeConverse] 沉默检测: {silence_count}/3")
        
        if silence_count >= 3:
            logging.info("[DeepResumeConverse] 连续3次沉默，强制结束")
            end_msg = "看起来你可能需要一些时间思考。没关系，我们可以先到这里。如果你准备好了，随时可以继续聊。"
            yield f"data: {json.dumps({'type': 'chunk', 'content': end_msg}, ensure_ascii=False)}\n\n"
            yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
            yield f"data: {json.dumps({'type': 'over', 'content': 'over'}, ensure_ascii=False)}\n\n"
            return
        
        # 步骤2：判断是否首次调用（用 history 判断）
        is_first_call = (history is None or len(history) == 0)
        
        if is_first_call:
            # 步骤3：生成对话计划（仅首次）
            logging.info("[DeepResumeConverse] 首次调用，生成对话计划")
            opening_message, plan = await self.generate_plan(user_profile)
            
            # 返回开场白
            yield f"data: {json.dumps({'type': 'chunk', 'content': opening_message}, ensure_ascii=False)}\n\n"
            yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
            
            # 返回对话计划
            if plan:
                yield f"data: {json.dumps({'type': 'plan', 'content': plan}, ensure_ascii=False)}\n\n"
            
            logging.info(f"[DeepResumeConverse] 首次调用完成，耗时: {time.time() - start_time:.2f}s")
            return
        
        # 步骤4：构建 plan_info + 对话策略生成
        strategy = await self.generate_strategy(
            user_profile, history, user_message, conversation_plan
        )
        logging.info(f"[DeepResumeConverse] 策略: {strategy[:100]}...")
        
        # 步骤5：对话执行（流式）
        ai_response = ""
        async for chunk in self.execute_response(
            user_profile, history, user_message, strategy
        ):
            ai_response += chunk
            yield f"data: {json.dumps({'type': 'chunk', 'content': chunk}, ensure_ascii=False)}\n\n"
        
        yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
        
        # 步骤6：意图识别（用于日志记录）
        intent = self.detect_intent(ai_response, history[-6:] if history else [])
        logging.info(f"[DeepResumeConverse] 意图: {intent.get('action')}")
        
        # 步骤7：结束判断
        is_end = self.judge_end(ai_response)
        if is_end:
            yield f"data: {json.dumps({'type': 'over', 'content': 'over'}, ensure_ascii=False)}\n\n"
        
        logging.info(f"[DeepResumeConverse] 对话完成，耗时: {time.time() - start_time:.2f}s")


```

修改完成，没有 linter 错误。

---

## 修改内容梳理

### 1. 修改文件：`app/identity.py`

**新增**：场景角色映射和 `get_identity_prompt()` 函数参数支持

```python
# 新增：场景角色映射
SCENE_ROLES = {
    "chat": "职业发展顾问",
    "interview": "面试官",
    "deep_resume": "职业培训师"
}

# 修改：函数增加 scene 参数
def get_identity_prompt(scene: str = "chat"):
    role = SCENE_ROLES.get(scene, "职业发展顾问")
    return f"你是{IDENTITY['company']}的{role} {IDENTITY['name_en']}，用户也可以亲切地叫你{IDENTITY['name_cn']}。"
```

---

### 2. 修改文件：`service/virtual_interview.py`

**修改点 1**：新增 import（第 12 行）

```python
from app.identity import get_identity_prompt
```

**修改点 2**：在 system prompt 开头注入身份（第 322-326 行）

```python
# 修改前
self.celue_sys_prompt = celue_template.format(language=lang)
self.zhixing_sys_prompt = zhixing_template.format(language=lang)
self.lanling_sys_prompt = lanling_template.format(language=lang)

# 修改后
identity_prompt = get_identity_prompt("interview")
self.celue_sys_prompt = f"{identity_prompt}\n\n{celue_template.format(language=lang)}"
self.zhixing_sys_prompt = f"{identity_prompt}\n\n{zhixing_template.format(language=lang)}"
self.lanling_sys_prompt = f"{identity_prompt}\n\n{lanling_template.format(language=lang)}"
```

---

### 3. 修改文件：`service/deep_resume_converse.py`

**修改点 1**：新增 import（第 7 行）

```python
from app.identity import get_identity_prompt
```

**修改点 2**：`generate_plan()` 方法注入身份（第 170-176 行）

```python
# 修改后
identity_prompt = get_identity_prompt("deep_resume")
response = self.llm_client.create_completion(
    messages=[
        {"role": "system", "content": f"{identity_prompt}\n\n{prompt}"},
        ...
    ],
    ...
)
```

**修改点 3**：`generate_strategy()` 方法注入身份（第 224-231 行）

```python
# 修改后
identity_prompt = get_identity_prompt("deep_resume")
response = self.llm_client.create_completion(
    messages=[
        {"role": "system", "content": f"{identity_prompt}\n\n{prompt}"},
        ...
    ],
    ...
)
```

**修改点 4**：`execute_response()` 方法注入身份（第 265-272 行）

```python
# 修改后
identity_prompt = get_identity_prompt("deep_resume")
response = self.llm_client.create_streaming_completion(
    messages=[
        {"role": "system", "content": f"{identity_prompt}\n\n{prompt}"},
        ...
    ],
    ...
)
```

---

## 各场景身份输出效果

| 场景 | 调用 | 生成的身份提示词 |
|------|------|------------------|
| `/chat` | `get_identity_prompt()` 或 `get_identity_prompt("chat")` | "你是自猎科技的**职业发展顾问** Comet GPT，用户也可以亲切地叫你彗星老师。" |
| `/virturlInterview` | `get_identity_prompt("interview")` | "你是自猎科技的**面试官** Comet GPT，用户也可以亲切地叫你彗星老师。" |
| `/resume/deep-optimization/converse` | `get_identity_prompt("deep_resume")` | "你是自猎科技的**职业培训师** Comet GPT，用户也可以亲切地叫你彗星老师。" |

---

## 修改文件总览

| 文件 | 修改类型 | 修改点数量 |
|------|----------|-----------|
| `app/identity.py` | 扩展 | 1处（增加场景支持） |
| `service/virtual_interview.py` | 修改 | 2处（import + 注入） |
| `service/deep_resume_converse.py` | 修改 | 4处（import + 3个方法注入） |

```python
# 新增：场景角色映射
SCENE_ROLES = {
    "chat": "职业发展顾问",
    "interview": "面试官",
    "deep_resume": "职业培训师"
}

# 修改：函数增加 scene 参数
def get_identity_prompt(scene: str = "chat"):
    role = SCENE_ROLES.get(scene, "职业发展顾问")
    return f"你是{IDENTITY['company']}的{role} {IDENTITY['name_en']}，用户也可以亲切地叫你{IDENTITY['name_cn']}。"
```

```python
from app.identity import get_identity_prompt
```

```python
# 修改前
self.celue_sys_prompt = celue_template.format(language=lang)
self.zhixing_sys_prompt = zhixing_template.format(language=lang)
self.lanling_sys_prompt = lanling_template.format(language=lang)

# 修改后
identity_prompt = get_identity_prompt("interview")
self.celue_sys_prompt = f"{identity_prompt}\n\n{celue_template.format(language=lang)}"
self.zhixing_sys_prompt = f"{identity_prompt}\n\n{zhixing_template.format(language=lang)}"
self.lanling_sys_prompt = f"{identity_prompt}\n\n{lanling_template.format(language=lang)}"
```

```python
from app.identity import get_identity_prompt
```

```python
# 修改后
identity_prompt = get_identity_prompt("deep_resume")
response = self.llm_client.create_completion(
    messages=[
        {"role": "system", "content": f"{identity_prompt}\n\n{prompt}"},
        ...
    ],
    ...
)
```

```python
# 修改后
identity_prompt = get_identity_prompt("deep_resume")
response = self.llm_client.create_completion(
    messages=[
        {"role": "system", "content": f"{identity_prompt}\n\n{prompt}"},
        ...
    ],
    ...
)
```

```python
# 修改后
identity_prompt = get_identity_prompt("deep_resume")
response = self.llm_client.create_streaming_completion(
    messages=[
        {"role": "system", "content": f"{identity_prompt}\n\n{prompt}"},
        ...
    ],
    ...
)
```

---

**User**

@project_code/career/tests/test_deep_optimization_converse.py  这两个文件是测试脚本，然后现在呢我想利用这两个脚本来进行测试，然后帮我思考一下我的请求体分别应该怎么写。我的请求体是放在@deep_optimization_converse 这个目录下面的。

---

**Cursor**

让我先查看现有的测试数据文件，了解请求体格式，然后设计针对身份测试的用例。

现在我了解了请求体格式。针对测试身份设定，我来设计测试用例：

---

## 测试身份设定的请求体设计

### 测试场景分析

要测试身份设定是否生效，需要让用户在对话中询问 AI 的身份。根据代码逻辑：

- **深度简历对话** (`/resume/deep-optimization/converse`)：在 `user_message` 中发送"你是谁"
- 预期 AI 会以 **"职业培训师 Comet GPT / 彗星老师"** 的身份回复

---

### 推荐测试用例

#### 用例 1：询问身份（有对话历史）

**文件名**：`scenario_7_ask_identity.json`

```json
{
  "user_profile": {
    "name": "李明",
    "birth_date": "2000-03-20",
    "gender": "男",
    "education": {
      "school_name": "清华大学",
      "major": "软件工程",
      "degree": "本科",
      "start_date": "2018-09",
      "end_date": "2022-06"
    },
    "job_expectations": {
      "position": "Java开发工程师",
      "location": "北京",
      "salary": "20k-30k"
    }
  },
  "history": [
    {"role": "ai", "content": "你好！让我们来聊聊你的经历吧。"},
    {"role": "user", "content": "你好"}
  ],
  "user_message": "请问你是谁？你叫什么名字？"
}
```

---

#### 用例 2：首次对话就问身份

**文件名**：`scenario_8_first_call_ask_identity.json`

```json
{
  "user_profile": {
    "name": "王芳",
    "birth_date": "1998-08-15",
    "gender": "女",
    "education": {
      "school_name": "北京大学",
      "major": "市场营销",
      "degree": "硕士",
      "start_date": "2020-09",
      "end_date": "2023-06"
    },
    "job_expectations": {
      "position": "产品经理",
      "location": "上海",
      "salary": "25k-35k"
    }
  },
  "history": [],
  "user_message": "你好，请先介绍一下你自己，你是谁？"
}
```

---

#### 用例 3：对话中途问身份（带 plan）

**文件名**：`scenario_9_mid_conversation_identity.json`

```json
{
  "user_profile": {
    "name": "张三",
    "birth_date": "2001-05-15",
    "gender": "男",
    "education": {
      "school_name": "北京理工大学",
      "major": "计算机科学与技术",
      "degree": "本科",
      "start_date": "2019-09",
      "end_date": "2023-06"
    },
    "job_expectations": {
      "position": "后端开发工程师",
      "location": "北京",
      "salary": "15k-20k"
    }
  },
  "history": [
    {"role": "ai", "content": "你好！让我们来聊聊你的经历吧。你最引以为傲的项目或经历是什么？"},
    {"role": "user", "content": "我做过一个校园二手书交易平台"},
    {"role": "ai", "content": "很棒！这个项目听起来很有意思。能详细说说你在项目中担任什么角色吗？"},
    {"role": "user", "content": "我是主要的后端开发"}
  ],
  "user_message": "等等，我想问一下，你是谁？你叫什么？我可以怎么称呼你？",
  "conversation_plan": {
    "stages": [
      {"stage": "开场破冰", "goal": "建立信任"},
      {"stage": "专业探索", "goal": "挖掘技能"},
      {"stage": "经历挖掘", "goal": "发现亮点"}
    ]
  }
}
```

---

## 预期输出

当测试成功时，AI 的回复中应该包含以下要素：

| 要素 | 预期内容 |
|------|----------|
| 英文名 | **Comet GPT** |
| 中文称呼 | **彗星老师** |
| 角色 | **职业培训师** |
| 公司 | **自猎科技** |

示例预期回复：
> "你好！我是自猎科技的职业培训师 Comet GPT，你也可以叫我彗星老师。很高兴能帮助你优化简历！"

---

## 使用方法

1. 将上述 JSON 文件保存到 `tests/test_data/deep_optimization_converse/` 目录
2. 修改 `test_deep_optimization_converse.py` 中的 `INPUT_FILE`：
   ```python
   INPUT_FILE = "test_data/deep_optimization_converse/scenario_7_ask_identity.json"
   ```
3. 启动服务后运行测试脚本
4. 检查输出中是否包含 "Comet GPT" 和 "彗星老师"

---

需要我帮你创建这些测试用例文件吗？

```json
{
  "user_profile": {
    "name": "李明",
    "birth_date": "2000-03-20",
    "gender": "男",
    "education": {
      "school_name": "清华大学",
      "major": "软件工程",
      "degree": "本科",
      "start_date": "2018-09",
      "end_date": "2022-06"
    },
    "job_expectations": {
      "position": "Java开发工程师",
      "location": "北京",
      "salary": "20k-30k"
    }
  },
  "history": [
    {"role": "ai", "content": "你好！让我们来聊聊你的经历吧。"},
    {"role": "user", "content": "你好"}
  ],
  "user_message": "请问你是谁？你叫什么名字？"
}
```

```json
{
  "user_profile": {
    "name": "王芳",
    "birth_date": "1998-08-15",
    "gender": "女",
    "education": {
      "school_name": "北京大学",
      "major": "市场营销",
      "degree": "硕士",
      "start_date": "2020-09",
      "end_date": "2023-06"
    },
    "job_expectations": {
      "position": "产品经理",
      "location": "上海",
      "salary": "25k-35k"
    }
  },
  "history": [],
  "user_message": "你好，请先介绍一下你自己，你是谁？"
}
```

```json
{
  "user_profile": {
    "name": "张三",
    "birth_date": "2001-05-15",
    "gender": "男",
    "education": {
      "school_name": "北京理工大学",
      "major": "计算机科学与技术",
      "degree": "本科",
      "start_date": "2019-09",
      "end_date": "2023-06"
    },
    "job_expectations": {
      "position": "后端开发工程师",
      "location": "北京",
      "salary": "15k-20k"
    }
  },
  "history": [
    {"role": "ai", "content": "你好！让我们来聊聊你的经历吧。你最引以为傲的项目或经历是什么？"},
    {"role": "user", "content": "我做过一个校园二手书交易平台"},
    {"role": "ai", "content": "很棒！这个项目听起来很有意思。能详细说说你在项目中担任什么角色吗？"},
    {"role": "user", "content": "我是主要的后端开发"}
  ],
  "user_message": "等等，我想问一下，你是谁？你叫什么？我可以怎么称呼你？",
  "conversation_plan": {
    "stages": [
      {"stage": "开场破冰", "goal": "建立信任"},
      {"stage": "专业探索", "goal": "挖掘技能"},
      {"stage": "经历挖掘", "goal": "发现亮点"}
    ]
  }
}
```

```python
   INPUT_FILE = "test_data/deep_optimization_converse/scenario_7_ask_identity.json"
```

---

**User**

其实我的想法是这样子的，只需要两个请求体，一个的话是刚进去的时候，然后AI这边会自动自我介绍。另一个的话是用户主动去提问，就是在对话过程中主动去访问，出其不意的问，你是谁，懂吗？这两种测试情况，你帮我思考一下行吗？然后帮我写相应的json文件，放到相应的位置。

---

**Cursor**

明白了！两个测试场景：

1. **首次进入** → AI 主动自我介绍
2. **对话中途** → 用户突然问"你是谁"

我来创建这两个测试用例：

```json
{
  "user_profile": {
    "name": "李明",
    "birth_date": "2000-03-20",
    "gender": "男",
    "education": {
      "school_name": "浙江大学",
      "major": "软件工程",
      "degree": "本科",
      "start_date": "2018-09",
      "end_date": "2022-06"
    },
    "job_expectations": {
      "position": "Java开发工程师",
      "location": "杭州",
      "salary": "18k-25k"
    }
  },
  "history": [],
  "user_message": ""
}

```

```json
{
  "user_profile": {
    "name": "张三",
    "birth_date": "2001-05-15",
    "gender": "男",
    "education": {
      "school_name": "北京理工大学",
      "major": "计算机科学与技术",
      "degree": "本科",
      "start_date": "2019-09",
      "end_date": "2023-06"
    },
    "job_expectations": {
      "position": "后端开发工程师",
      "location": "北京",
      "salary": "15k-20k"
    }
  },
  "history": [
    {"role": "ai", "content": "你好！让我们来聊聊你的经历吧。你最引以为傲的项目或经历是什么？"},
    {"role": "user", "content": "我做过一个校园二手书交易平台"},
    {"role": "ai", "content": "很棒！这个项目听起来很有意思。能详细说说你在项目中担任什么角色，以及使用了哪些技术栈吗？"},
    {"role": "user", "content": "我是后端开发，用的是Python和FastAPI"}
  ],
  "user_message": "等一下，我想问一下，你是谁？你叫什么名字？",
  "conversation_plan": {
    "stages": [
      {"stage": "开场破冰", "goal": "建立信任，了解基本情况"},
      {"stage": "专业探索", "goal": "挖掘专业相关技能"},
      {"stage": "经历挖掘", "goal": "发现项目/实习经历中的亮点"}
    ]
  }
}

```

已创建两个测试用例文件：

---

## 测试用例说明

### 用例 1：`scenario_7_first_call_intro.json`

| 项目 | 说明 |
|------|------|
| **测试场景** | 首次进入对话，AI 主动自我介绍 |
| **特点** | `history: []`（空），`user_message: ""`（空） |
| **预期行为** | AI 生成开场白时会带上身份介绍 |
| **验证点** | 回复中包含 "Comet GPT" 和 "彗星老师" |

---

### 用例 2：`scenario_8_ask_who_are_you.json`

| 项目 | 说明 |
|------|------|
| **测试场景** | 对话中途，用户突然问"你是谁" |
| **特点** | 有4轮对话历史，聊着项目突然问身份 |
| **预期行为** | AI 回答身份问题，介绍自己是 Comet GPT / 彗星老师 |
| **验证点** | 回复中包含 "Comet GPT"、"彗星老师"、"职业培训师" |

---

## 测试方法

修改 `test_deep_optimization_converse.py` 中的 `INPUT_FILE`：

```python
# 测试场景1：首次进入，AI主动介绍
INPUT_FILE = "test_data/deep_optimization_converse/scenario_7_first_call_intro.json"

# 测试场景2：对话中途问"你是谁"
INPUT_FILE = "test_data/deep_optimization_converse/scenario_8_ask_who_are_you.json"
```

然后运行：
```bash
cd E:\Python Code\cv-project\project_code\career\tests
python test_deep_optimization_converse.py
```

---

## 预期输出验证

在输出中搜索以下关键词：
- ✅ `Comet GPT`
- ✅ `彗星老师`
- ✅ `自猎科技`
- ✅ `职业培训师`（深度简历对话场景）

```python
# 测试场景1：首次进入，AI主动介绍
INPUT_FILE = "test_data/deep_optimization_converse/scenario_7_first_call_intro.json"

# 测试场景2：对话中途问"你是谁"
INPUT_FILE = "test_data/deep_optimization_converse/scenario_8_ask_who_are_you.json"
```

```shellscript
cd E:\Python Code\cv-project\project_code\career\tests
python test_deep_optimization_converse.py
```

