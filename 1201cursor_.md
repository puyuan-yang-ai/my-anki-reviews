# 梳理深度简历优化接口逻辑
_Exported on 12/1/2025 at 15:21:06 GMT+8 from Cursor (2.1.39)_

---

**User**

@deep-resume-optimization @project_code/career/code/routers/resume_router.py 现在请根据我给你提供的这些文档，然后帮我梳理一下两个接口，以及他们的处理逻辑。一个是深度简历优化converse，另一个是深度简历优化generate，这是深度简历优化的两个不同的方面吧。

---

**Cursor**

我来帮你梳理深度简历优化的两个接口。首先让我读取你提供的文档内容。

好的，根据文档和代码，我来帮你梳理这两个接口的逻辑：

---

## 深度简历优化功能概述

这是一个针对**只有基本信息但没有完整简历的学生**设计的功能。整体流程分为两个阶段：

---

## 接口一：对话接口 (Converse)

**路由**: `POST /resume/deep-optimization/converse`

### 目的
通过**模拟面试/对话**的方式，挖掘学生的个人优点和潜力，为简历收集素材。

### 输入数据结构 (`DeepConverseRequest`)

```68:72:e:/Python Code/cv-project/project_code/career/code/routers/resume_router.py
class DeepConverseRequest(BaseModel):
    user_profile: UserProfile = Field(..., description="用户的基本档案信息")
    history: List[ChatMessage] = Field(..., description="到目前为止的对话历史")
    user_message: str = Field(..., description="用户本轮的最新回复")
```

| 字段 | 说明 |
|------|------|
| `user_profile` | 用户基本信息（姓名、学历、专业、学校、求职期望等） |
| `history` | 对话历史记录列表 |
| `user_message` | 用户本轮的最新回复 |

### 输出数据结构 (`DeepConverseResponse`)

```73:74:e:/Python Code/cv-project/project_code/career/code/routers/resume_router.py
class DeepConverseResponse(BaseModel):
    ai_message: str = Field(..., description="AI生成的新问题或回复")
```

### 处理逻辑（按文档规格）

1. **信息挖掘**：AI 扮演职业培训师/长辈角色，通过对话挖掘学生潜力
2. **问题类型**（根据 `conversation-spec.md`）：
   - 个人介绍
   - 对专业的兴趣、看法、收获和突出技能
   - 职业规划和目标
   - 对其他岗位的关注
   - 案例分析（判断思维模式）
   - 根据背景挖掘个人优势

3. **当前实现**：返回 mock 数据（随机问题）

---

## 接口二：生成接口 (Generate)

**路由**: `POST /resume/deep-optimization/generate`

### 目的
在对话充分、信息收集完毕后，将整个对话过程**提炼、总结、润色**，生成一份结构化的专业简历。

### 输入数据结构 (`DeepGenerateRequest`)

```76:78:e:/Python Code/cv-project/project_code/career/code/routers/resume_router.py
class DeepGenerateRequest(BaseModel):
    user_profile: UserProfile = Field(..., description="用户的基本档案信息，作为简历的基础")
    final_history: List[ChatMessage] = Field(..., description="最终的、完整的对话历史记录")
```

| 字段 | 说明 |
|------|------|
| `user_profile` | 用户基本档案信息（作为简历基础） |
| `final_history` | 完整的对话历史记录 |

### 输出数据结构 (`OptimizedResume`)

```87:96:e:/Python Code/cv-project/project_code/career/code/routers/resume_router.py
class OptimizedResume(BaseModel):
    profile: UserProfile
    summary: str = Field(..., description="AI生成的个人总结/职业亮点")
    experience: List[ResumeSection] = Field(..., description="工作/实习经历")
    projects: List[ResumeSection] = Field(..., description="项目经历")
    skills: Dict[str, List[str]] = Field(..., description="技能清单, 例如: {'专业技能': ['Python', 'Java'], '软技能': ['沟通能力']}")
    awards: Optional[List[str]] = Field(None, description="荣誉奖项")
```

### 处理逻辑（按文档规格）

1. **无状态单次调用**：不关心对话过程，只关心最终输入
2. **输入分析**：分析 `user_profile` + `final_history`
3. **内容生成**：
   - 个人总结 (summary)
   - 工作/实习经历 (experience)
   - 项目经历 (projects)
   - 技能清单 (skills)
   - 荣誉奖项 (awards)
4. **当前实现**：返回 mock 数据

---

## 两个接口的关系

```
┌─────────────────────────────────────────────────────────────────┐
│                      深度简历优化流程                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐     多轮调用      ┌─────────────────────────┐  │
│  │ user_profile│ ───────────────→  │    Converse 接口        │  │
│  │   (基础信息) │                   │  （对话挖掘信息）        │  │
│  └─────────────┘  ←───────────────  │   返回 AI 提问          │  │
│        │                           └─────────────────────────┘  │
│        │                                      │                 │
│        │              对话结束后               │                 │
│        │         (history 收集完毕)           ↓                 │
│        │         ┌────────────────────────────────────────────┐ │
│        └────────→│           Generate 接口                    │ │
│                  │  输入: user_profile + final_history        │ │
│                  │  输出: 结构化简历 (OptimizedResume)         │ │
│                  └────────────────────────────────────────────┘ │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**简单来说**：
- **Converse** = 信息收集阶段（多轮对话，挖掘素材）
- **Generate** = 简历生成阶段（一次性调用，汇总提炼）

这是一个典型的**两阶段设计**：先通过对话获取尽可能多的信息，再基于完整信息生成高质量简历。

```plaintext
class DeepConverseRequest(BaseModel):
    user_profile: UserProfile = Field(..., description="用户的基本档案信息")
    history: List[ChatMessage] = Field(..., description="到目前为止的对话历史")
    user_message: str = Field(..., description="用户本轮的最新回复")
```

```plaintext
class DeepConverseResponse(BaseModel):
    ai_message: str = Field(..., description="AI生成的新问题或回复")
```

```plaintext
class DeepGenerateRequest(BaseModel):
    user_profile: UserProfile = Field(..., description="用户的基本档案信息，作为简历的基础")
    final_history: List[ChatMessage] = Field(..., description="最终的、完整的对话历史记录")
```

```plaintext
class OptimizedResume(BaseModel):
    profile: UserProfile
    summary: str = Field(..., description="AI生成的个人总结/职业亮点")
    experience: List[ResumeSection] = Field(..., description="工作/实习经历")
    projects: List[ResumeSection] = Field(..., description="项目经历")
    skills: Dict[str, List[str]] = Field(..., description="技能清单, 例如: {'专业技能': ['Python', 'Java'], '软技能': ['沟通能力']}")
    awards: Optional[List[str]] = Field(None, description="荣誉奖项")
```

```plaintext
┌─────────────────────────────────────────────────────────────────┐
│                      深度简历优化流程                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐     多轮调用      ┌─────────────────────────┐  │
│  │ user_profile│ ───────────────→  │    Converse 接口        │  │
│  │   (基础信息) │                   │  （对话挖掘信息）        │  │
│  └─────────────┘  ←───────────────  │   返回 AI 提问          │  │
│        │                           └─────────────────────────┘  │
│        │                                      │                 │
│        │              对话结束后               │                 │
│        │         (history 收集完毕)           ↓                 │
│        │         ┌────────────────────────────────────────────┐ │
│        └────────→│           Generate 接口                    │ │
│                  │  输入: user_profile + final_history        │ │
│                  │  输出: 结构化简历 (OptimizedResume)         │ │
│                  └────────────────────────────────────────────┘ │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

**User**

@project_code/career/docs/specs/deep-resume-optimization/conversation-spec.md 这个是深度简历里面的对话部分，因此它是一个单独的接口。然后你看一下我的这个规格说明，@project_code/career/code/routers/interview_process_router.py 你可以参考一下virturlInterview这个接口的实现，

1 请问你明白我这个逻辑了吗？请用自己的话来复述逻辑，总结功能的核心目标。

2 用列表形式拆解为具体可实现的功能点。

3 你存在哪些疑问，向我提出具体问题来澄清这些点，确保双方对需求理解一致之后再写代码。

---

**Cursor**

让我先看一下 `virtual_interview.py` 的实现，以便更好理解参考接口的具体逻辑。

好的，我现在完全理解了。让我来回答你的三个问题：

---

## 1. 逻辑复述 & 核心目标

### 我的理解

**深度简历优化 - 对话接口 (converse)** 是一个**多轮对话式信息挖掘服务**：

- **目标用户**：只有基础个人信息（姓名、学历、专业等），但**没有完整简历**的学生
- **核心目的**：不是做求职面试评估，而是通过类似"职业培训师/长辈聊天"的方式，**挖掘学生的个人优点和潜力**，为简历收集素材
- **最终结果**：对话结束后不生成评估报告，而是交给 `generate` 接口生成一份完整简历

### 参考实现对比

参考 `virturlInterview` 接口的实现，我理解 `converse` 接口需要实现类似的模式：

| 特性 | virturlInterview（求职面试） | converse（深度简历挖掘） |
|------|-------------------------------|---------------------------|
| 响应方式 | 流式 (StreamingResponse) | 应该也是流式？ |
| 首次调用 | 生成面试计划 | 生成对话/挖掘计划 |
| 对话策略 | 评估候选人能力 | 挖掘优点和潜力 |
| 结束标志 | AI 判断面试结束 | AI 判断信息收集完毕 |
| 最终产出 | 生成评估报告 | **不生成报告**，直接调用 generate 生成简历 |

---

## 2. 功能点拆解

### 核心模块

1. **对话计划生成模块**
   - 首次调用时，根据 `user_profile`（学历、专业、求职期望）生成对话计划
   - 计划包含多个阶段的问题：个人介绍 → 专业看法 → 职业规划 → 案例分析等

2. **对话策略模块**
   - 根据当前对话历史和用户回答，决定下一步策略
   - 策略类型：追问细节 / 切换话题 / 结束对话

3. **对话执行模块**
   - 基于策略，生成具体的 AI 回复内容
   - 使用流式响应返回给前端

4. **意图/状态识别模块**
   - 判断当前对话状态：新问题 / 追问 / 确认状态 / 结束
   - 判断用户回复有效性（空回复、沉默检测）

5. **结束判断模块**
   - 判断信息是否收集完毕
   - 信号给前端：可以调用 `generate` 接口了

### 需要的 Prompt 文件

| Prompt 文件 | 用途 |
|-------------|------|
| `generate_plan_deep_resume.txt` | 根据用户档案生成对话挖掘计划 |
| `strategy_deep_resume.txt` | 对话策略生成（决定追问/换题/结束） |
| `executor_deep_resume.txt` | 执行层：生成具体的友善提问内容 |
| `judge_flow_deep_resume.txt` | 判断对话是否可以结束 |

### 数据流

```
┌────────────────────────────────────────────────────────────────┐
│  POST /resume/deep-optimization/converse                       │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  输入: user_profile + history + user_message                   │
│                           │                                    │
│                           ▼                                    │
│  ┌──────────────────────────────────────┐                      │
│  │ 首次调用?                             │                      │
│  │  YES → 生成对话计划 + 返回开场白       │                      │
│  │  NO  → 继续下面流程                   │                      │
│  └──────────────────────────────────────┘                      │
│                           │                                    │
│                           ▼                                    │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────────────┐   │
│  │ 策略层 LLM  │ → │ 执行层 LLM  │ → │ 流式返回 AI 回复     │   │
│  │ (决定追问/  │   │ (生成友善   │   │ StreamingResponse   │   │
│  │  换题/结束) │   │  的具体内容)│   │                     │   │
│  └─────────────┘   └─────────────┘   └─────────────────────┘   │
│                                               │                │
│                                               ▼                │
│                           ┌─────────────────────────────────┐  │
│                           │ 判断是否结束:                    │  │
│                           │  - 信息充足?                     │  │
│                           │  - 用户要求结束?                 │  │
│                           │  返回 over 信号                  │  │
│                           └─────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────┘
```

---

## 3. 我的疑问（需要澄清）

### 关于技术实现

1. **响应方式**：是否需要像 `virturlInterview` 一样使用流式响应 (StreamingResponse)？还是普通 JSON 响应即可？

2. **对话计划存储**：
   - `virturlInterview` 的 `interview_plan` 是存在内存中的，每次请求都是新实例
   - 深度简历的对话计划如何持久化？是前端传回来，还是后端需要维护状态？

3. **沉默检测**：是否需要类似 `virturlInterview` 的连续沉默检测（3 次无效回复强制结束）？

### 关于业务逻辑

4. **对话轮次**：是否有预设的最大对话轮次？大约多少轮合适（参考面试接口是 5-8 个问题）？

5. **问题类型**：文档提到"案例分析判断思维模式"，这类问题是否需要特殊处理（比如给出场景让学生分析）？

6. **结束触发**：
   - 谁来决定对话结束？AI 自动判断 / 用户主动点击"生成简历"？
   - 如果用户信息不充分就要求结束，是否给出提示？

7. **AI 角色定位**：文档说是"长辈或职业培训师"，语气应该是**温和鼓励型**还是**严谨专业型**？需要在 prompt 中明确。

---

请你针对以上疑问给我反馈，确认后我们就可以开始编写代码了！

```plaintext
┌────────────────────────────────────────────────────────────────┐
│  POST /resume/deep-optimization/converse                       │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  输入: user_profile + history + user_message                   │
│                           │                                    │
│                           ▼                                    │
│  ┌──────────────────────────────────────┐                      │
│  │ 首次调用?                             │                      │
│  │  YES → 生成对话计划 + 返回开场白       │                      │
│  │  NO  → 继续下面流程                   │                      │
│  └──────────────────────────────────────┘                      │
│                           │                                    │
│                           ▼                                    │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────────────┐   │
│  │ 策略层 LLM  │ → │ 执行层 LLM  │ → │ 流式返回 AI 回复     │   │
│  │ (决定追问/  │   │ (生成友善   │   │ StreamingResponse   │   │
│  │  换题/结束) │   │  的具体内容)│   │                     │   │
│  └─────────────┘   └─────────────┘   └─────────────────────┘   │
│                                               │                │
│                                               ▼                │
│                           ┌─────────────────────────────────┐  │
│                           │ 判断是否结束:                    │  │
│                           │  - 信息充足?                     │  │
│                           │  - 用户要求结束?                 │  │
│                           │  返回 over 信号                  │  │
│                           └─────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────┘
```

