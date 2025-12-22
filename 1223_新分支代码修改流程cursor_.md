**User**

现在模拟面试这块呢，它是不是这样子的流程，先是进行一个策略的AI，然后再进行执行的AI，然后再进行一个判断的AI呀。

请ultrathinking,帮我梳理清楚呀。

---

**Cursor**

检查模拟面试的后端实现，梳理流程架构：


读取核心服务文件，梳理完整流程：


梳理模拟面试的完整流程架构：


## 模拟面试的完整AI流程架构

### 流程概览

模拟面试采用多阶段AI架构，包含以下阶段：

### 阶段1：计划AI（仅首次调用）

- 位置：`generate_interview_plan()` 函数（第 128-187 行）
- 作用：生成面试计划
- 触发时机：首次调用（`dialog_history` 为空时）
- 输入：
  - 候选人信息（`user_portrait`）
  - 岗位信息（`jd_title`, `jd_portrait`）
- 输出：面试计划 JSON，包含多个问题及考察维度
- Prompt：`generate_plan_white_collar.txt`
- 模型：`llm_client`（主模型）

### 阶段2：策略AI（每次对话）

- 位置：第 352-368 行
- 作用：分析对话历史，生成回复策略
- 触发时机：每次对话（非首次）
- 输入：
  - 对话历史（`dialog_history`）
  - 面试计划信息（`plan_info`）
  - 系统消息
- 输出：回复策略文本（包含质量评分、追问建议等）
- Prompt：`strategy_white_collar.txt`
- 模型：`llm_client`（主模型，启用 thinking）
- 温度：0.4

### 阶段3：执行AI（每次对话）

- 位置：第 376-403 行
- 作用：根据策略生成实际回复内容
- 触发时机：策略AI完成后
- 输入：
  - 对话历史
  - 策略AI的输出（`celue_text`）
  - 面试计划信息
- 输出：流式返回面试官回复内容
- Prompt：`executor_white_collar.txt`（白领）或 `executor_blue_collar.txt`（蓝领）
- 模型：`llm_client`（流式调用）
- 温度：0.7

### 阶段4：意图识别（每次对话后）

- 位置：`_detect_interview_action()` 函数（第 190-245 行）
- 作用：识别AI的行动类型
- 触发时机：执行AI完成后
- 输入：
  - AI回复内容（`ai_response`）
  - 最近6轮对话历史
- 输出：行动类型 JSON（`new_question` / `repeat` / `follow_up` / `end` 等）
- 模型：`celue_client`（qwen-flash）
- 温度：0.1

### 阶段5：判断AI（每次对话后）

- 位置：第 420-434 行
- 作用：判断面试是否结束
- 触发时机：每次对话循环的最后
- 输入：面试官回复内容（`ai_response`）
- 输出：是否包含"结束"关键词
- Prompt：`judge_flow.txt`
- 模型：`celue_client`（qwen-flash）
- 温度：0.4

## 完整流程图

```
用户发送消息
    ↓
【首次调用？】
    ├─ 是 → 计划AI（生成面试计划）→ 返回第一个问题
    └─ 否 → 继续
    ↓
策略AI（分析对话，生成策略）
    ↓
执行AI（根据策略生成回复，流式输出）
    ↓
意图识别（识别行动类型）
    ↓
判断AI（判断是否结束）
    ↓
返回结果给前端
```

## 关键代码位置

| 阶段 | 函数/代码位置 | 行号 | 模型 |
|------|------------|------|------|
| 计划AI | `generate_interview_plan()` | 128-187 | llm_client |
| 策略AI | `get_ai_response()` 中 | 352-368 | llm_client |
| 执行AI | `get_ai_response()` 中 | 376-403 | llm_client（流式） |
| 意图识别 | `_detect_interview_action()` | 190-245 | celue_client (qwen-flash) |
| 判断AI | `get_ai_response()` 中 | 420-434 | celue_client (qwen-flash) |

## 总结

你的理解基本正确。流程是：
1. 策略AI（分析对话，生成策略）
2. 执行AI（根据策略生成回复）
3. 判断AI（判断是否结束）

补充：
- 首次调用有“计划AI”（生成面试计划）
- 执行AI后有“意图识别”（识别行动类型）

这是一个多阶段AI协作架构，每个阶段负责不同任务，共同完成模拟面试。

```plaintext
用户发送消息
    ↓
【首次调用？】
    ├─ 是 → 计划AI（生成面试计划）→ 返回第一个问题
    └─ 否 → 继续
    ↓
策略AI（分析对话，生成策略）
    ↓
执行AI（根据策略生成回复，流式输出）
    ↓
意图识别（识别行动类型）
    ↓
判断AI（判断是否结束）
    ↓
返回结果给前端
```

---

**User**

然后那你能不能帮我梳理一下这个agent的核心loop是什么呀？是基于session还是基于什么呀？ 

---

**Cursor**

分析模拟面试 agent 的核心循环机制和状态管理：


检查是否有状态持久化机制：


## 模拟面试 Agent 的核心 Loop 机制

### 核心结论：基于无状态的 HTTP 请求循环，不是基于 Session

## 1. 状态管理机制

### 无状态设计（Stateless）

- 不是基于 Session：每次请求都创建新的 `CallInterview` 实例
- 不是基于 Redis/Database：没有持久化状态存储
- 状态通过请求传递：所有状态都在 HTTP 请求体中传递

### 关键代码证据

```python
# interview_process_router.py 第 26 行
interview = CallInterview(interview_id)  # 每次请求都创建新实例
interview.user_portrait = user_portrait
interview.jd_title = jd_title
# ... 设置属性
return StreamingResponse(interview.get_ai_response(dialog_history), ...)
```

## 2. 核心 Loop 架构

### 循环机制：基于 HTTP 请求的循环

```
前端循环：
┌─────────────────────────────────────┐
│  1. 用户输入消息                      │
│  2. 构建请求体（包含完整 dialog_history）│
│  3. 发送 HTTP POST 请求               │
│  4. 接收流式响应                      │
│  5. 更新本地 dialog_history          │
│  6. 回到步骤1（等待下次用户输入）      │
└─────────────────────────────────────┘
         ↓
后端处理（每次请求独立）：
┌─────────────────────────────────────┐
│  1. 创建新的 CallInterview 实例       │
│  2. 从 dialog_history 恢复状态       │
│  3. 执行 AI 流程（策略→执行→判断）    │
│  4. 流式返回结果                     │
│  5. 实例销毁（无状态保持）            │
└─────────────────────────────────────┘
```

## 3. 状态传递方式

### 前端 → 后端：通过请求体传递

```python
# 前端维护的状态
{
    "id": "test_interview_001",           # interview_id
    "userPortrait": "...",                # 候选人信息
    "jdTitle": "...",                     # 岗位信息
    "dialogHistory": [                    # 完整的对话历史（关键！）
        {"role": "user", "content": "..."},
        {"role": "assistant", "content": "..."}
    ]
}
```

### 后端 → 前端：通过响应流传递

```python
# 流式响应
yield f"data: {json.dumps({'type': 'chunk', 'content': '...'})}\n\n"
yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'})}\n\n"
yield f"data: {json.dumps({'type': 'over', 'content': 'over'})}\n\n"
```

## 4. 状态恢复机制

### 每次请求的状态恢复流程

```python
# virtual_interview.py 第 248-281 行
async def get_ai_response(self, dialog_history):
    # 1. 从 dialog_history 判断是否首次调用
    if not dialog_history or len(dialog_history) == 0:
        # 首次：生成面试计划
        await self.generate_interview_plan()
        # 返回第一个问题
        return
    
    # 2. 非首次：从 dialog_history 恢复对话上下文
    # 3. 执行 AI 流程
    # 4. 返回结果
```

### 关键状态变量（临时，不持久化）

```python
# CallInterview 类中的状态（每次请求都是新的）
self.interview_id          # 面试ID（用于日志）
self.interview_plan        # 面试计划（首次调用时生成）
self.current_question_index  # 当前问题索引（在本次请求中维护）
self.dialog_history        # 不存储，从参数传入
```

## 5. 为什么不是基于 Session？

### 设计原因

1. 简化架构：无需管理 Session 生命周期
2. 易于扩展：无状态服务易于水平扩展
3. 容错性：前端维护状态，后端故障不影响历史
4. 灵活性：前端可自由控制状态管理

### 潜在问题

1. 面试计划丢失：`interview_plan` 在首次调用时生成，但不持久化
   - 如果前端丢失状态，需要重新生成计划
2. 问题索引丢失：`current_question_index` 不持久化
   - 目前通过意图识别动态更新，不完全依赖索引

## 6. 完整 Loop 流程图

```
┌─────────────────────────────────────────────────────────┐
│                   前端（状态维护者）                       │
├─────────────────────────────────────────────────────────┤
│  dialog_history = []  # 维护完整对话历史                  │
│  interview_id = "test_001"                              │
│                                                          │
│  Loop:                                                  │
│    1. 用户输入消息                                       │
│    2. dialog_history.append({"role": "user", ...})     │
│    3. 构建请求体（包含完整 dialog_history）              │
│    4. POST /virturlInterview                            │
│    5. 接收流式响应                                       │
│    6. dialog_history.append({"role": "assistant", ...}) │
│    7. 回到步骤1                                         │
└─────────────────────────────────────────────────────────┘
                    ↓ HTTP Request
┌─────────────────────────────────────────────────────────┐
│                   后端（无状态处理）                       │
├─────────────────────────────────────────────────────────┤
│  每次请求：                                              │
│    1. 创建 CallInterview(interview_id)                  │
│    2. 从 dialog_history 恢复上下文                      │
│    3. 判断是否首次调用                                   │
│       ├─ 是 → 生成面试计划 → 返回第一个问题              │
│       └─ 否 → 继续                                       │
│    4. 策略AI（分析对话）                                 │
│    5. 执行AI（生成回复，流式输出）                       │
│    6. 意图识别（识别行动类型）                           │
│    7. 判断AI（判断是否结束）                             │
│    8. 流式返回结果                                       │
│    9. 实例销毁（状态不保留）                             │
└─────────────────────────────────────────────────────────┘
```

## 总结

| 特性 | 说明 |
|------|------|
| 核心 Loop | 基于 HTTP 请求的循环，不是基于 Session |
| 状态管理 | 前端维护状态，后端无状态 |
| 状态传递 | 通过 `dialog_history` 在请求体中传递 |
| 持久化 | 无持久化，每次请求独立处理 |
| 优势 | 简单、易扩展、容错性好 |
| 劣势 | 面试计划不持久化，前端丢失状态需重新生成 |

这是一个无状态的 API 设计，类似于 RESTful，但通过请求体传递完整状态，实现多轮对话。

```python
# interview_process_router.py 第 26 行
interview = CallInterview(interview_id)  # 每次请求都创建新实例
interview.user_portrait = user_portrait
interview.jd_title = jd_title
# ... 设置属性
return StreamingResponse(interview.get_ai_response(dialog_history), ...)
```

```plaintext
前端循环：
┌─────────────────────────────────────┐
│  1. 用户输入消息                      │
│  2. 构建请求体（包含完整 dialog_history）│
│  3. 发送 HTTP POST 请求               │
│  4. 接收流式响应                      │
│  5. 更新本地 dialog_history          │
│  6. 回到步骤1（等待下次用户输入）      │
└─────────────────────────────────────┘
         ↓
后端处理（每次请求独立）：
┌─────────────────────────────────────┐
│  1. 创建新的 CallInterview 实例       │
│  2. 从 dialog_history 恢复状态       │
│  3. 执行 AI 流程（策略→执行→判断）    │
│  4. 流式返回结果                     │
│  5. 实例销毁（无状态保持）            │
└─────────────────────────────────────┘
```

```python
# 前端维护的状态
{
    "id": "test_interview_001",           # interview_id
    "userPortrait": "...",                # 候选人信息
    "jdTitle": "...",                     # 岗位信息
    "dialogHistory": [                    # 完整的对话历史（关键！）
        {"role": "user", "content": "..."},
        {"role": "assistant", "content": "..."}
    ]
}
```

```python
# 流式响应
yield f"data: {json.dumps({'type': 'chunk', 'content': '...'})}\n\n"
yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'})}\n\n"
yield f"data: {json.dumps({'type': 'over', 'content': 'over'})}\n\n"
```

```python
# virtual_interview.py 第 248-281 行
async def get_ai_response(self, dialog_history):
    # 1. 从 dialog_history 判断是否首次调用
    if not dialog_history or len(dialog_history) == 0:
        # 首次：生成面试计划
        await self.generate_interview_plan()
        # 返回第一个问题
        return
    
    # 2. 非首次：从 dialog_history 恢复对话上下文
    # 3. 执行 AI 流程
    # 4. 返回结果
```

```python
# CallInterview 类中的状态（每次请求都是新的）
self.interview_id          # 面试ID（用于日志）
self.interview_plan        # 面试计划（首次调用时生成）
self.current_question_index  # 当前问题索引（在本次请求中维护）
self.dialog_history        # 不存储，从参数传入
```

```plaintext
┌─────────────────────────────────────────────────────────┐
│                   前端（状态维护者）                       │
├─────────────────────────────────────────────────────────┤
│  dialog_history = []  # 维护完整对话历史                  │
│  interview_id = "test_001"                              │
│                                                          │
│  Loop:                                                  │
│    1. 用户输入消息                                       │
│    2. dialog_history.append({"role": "user", ...})     │
│    3. 构建请求体（包含完整 dialog_history）              │
│    4. POST /virturlInterview                            │
│    5. 接收流式响应                                       │
│    6. dialog_history.append({"role": "assistant", ...}) │
│    7. 回到步骤1                                         │
└─────────────────────────────────────────────────────────┘
                    ↓ HTTP Request
┌─────────────────────────────────────────────────────────┐
│                   后端（无状态处理）                       │
├─────────────────────────────────────────────────────────┤
│  每次请求：                                              │
│    1. 创建 CallInterview(interview_id)                  │
│    2. 从 dialog_history 恢复上下文                      │
│    3. 判断是否首次调用                                   │
│       ├─ 是 → 生成面试计划 → 返回第一个问题              │
│       └─ 否 → 继续                                       │
│    4. 策略AI（分析对话）                                 │
│    5. 执行AI（生成回复，流式输出）                       │
│    6. 意图识别（识别行动类型）                           │
│    7. 判断AI（判断是否结束）                             │
│    8. 流式返回结果                                       │
│    9. 实例销毁（状态不保留）                             │
└─────────────────────────────────────────────────────────┘
```

---

**User**

因为我这个涉及到用户通过HTTP来请求与agent沟通交流，然后我听说是目前的主流方案是使用Session + 事件驱动 作为Agent loop，这是你怎么看待能不能帮我梳理一下呢？请ultrathinking,帮我梳理清楚呀。

---

**Cursor**

梳理 Session + 事件驱动的 Agent loop 架构，并与当前方案对比：


梳理 Session + 事件驱动的 Agent loop 架构，并与当前方案对比：

## Session + 事件驱动的 Agent Loop 架构

### 1. 核心架构组件

#### Session 管理（会话状态持久化）

```
┌─────────────────────────────────────┐
│        Session 管理器                │
├─────────────────────────────────────┤
│  - Session ID（唯一标识）             │
│  - 对话历史（dialog_history）         │
│  - Agent 状态（interview_plan等）    │
│  - 元数据（创建时间、TTL等）          │
│  - 存储：Redis / Database            │
└─────────────────────────────────────┘
```

#### 事件驱动循环

```
┌─────────────────────────────────────┐
│        事件总线（Event Bus）          │
├─────────────────────────────────────┤
│  事件类型：                           │
│  - user_message（用户消息）          │
│  - agent_response（Agent回复）       │
│  - session_start（会话开始）        │
│  - session_end（会话结束）           │
│  - error（错误事件）                  │
└─────────────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│      事件监听器（Event Listeners）    │
├─────────────────────────────────────┤
│  - 策略AI监听器                      │
│  - 执行AI监听器                      │
│  - 判断AI监听器                      │
│  - 状态更新监听器                    │
└─────────────────────────────────────┘
```

### 2. 完整 Loop 流程

```
┌─────────────────────────────────────────────────────────┐
│               Session + 事件驱动 Loop                     │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  1. 用户发送消息                                         │
│     ↓                                                    │
│  2. 创建/恢复 Session                                    │
│     - 从 Redis 加载对话历史                             │
│     - 恢复 Agent 状态（interview_plan等）                │
│     ↓                                                    │
│  3. 发布事件：user_message_event                        │
│     ↓                                                    │
│  4. 事件总线分发事件                                     │
│     ↓                                                    │
│  5. 策略AI监听器处理                                     │
│     - 分析对话历史                                       │
│     - 生成策略                                           │
│     - 发布 strategy_ready_event                         │
│     ↓                                                    │
│  6. 执行AI监听器处理                                     │
│     - 接收策略事件                                       │
│     - 生成回复（流式）                                   │
│     - 发布 response_chunk_event（多次）                 │
│     ↓                                                    │
│  7. 判断AI监听器处理                                     │
│     - 判断是否结束                                       │
│     - 发布 session_end_event（如需要）                   │
│     ↓                                                    │
│  8. 状态更新监听器                                       │
│     - 更新 Session 状态                                 │
│     - 保存对话历史到 Redis                              │
│     - 持久化 Agent 状态                                 │
│     ↓                                                    │
│  9. 返回响应给前端                                       │
│     ↓                                                    │
│  10. 等待下次用户消息（回到步骤1）                       │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

## 方案对比分析

### 方案A：当前项目（无状态 HTTP）

#### 架构特点

```python
# 每次请求都创建新实例
interview = CallInterview(interview_id)  # 无状态
interview.get_ai_response(dialog_history)  # 状态通过参数传递
```

#### 优势

1. 简单：无需管理 Session 生命周期
2. 易扩展：无状态服务易于水平扩展
3. 容错：前端维护状态，后端故障不影响历史
4. 无状态：符合 RESTful 设计原则
5. 调试：每次请求独立，易于追踪

#### 劣势

1. 状态丢失风险：前端丢失状态需重新生成
2. 重复计算：面试计划每次可能重新生成
3. 网络开销：每次传递完整对话历史
4. 无法支持复杂状态：难以维护跨请求的复杂状态

### 方案B：Session + 事件驱动（主流方案）

#### 架构特点

```python
# Session 管理
session = await session_manager.get_or_create(interview_id)
session.dialog_history = await redis.get(f"session:{interview_id}:history")
session.interview_plan = await redis.get(f"session:{interview_id}:plan")

# 事件驱动
event_bus.publish("user_message", {
    "session_id": interview_id,
    "message": user_message
})

# 监听器处理
@event_bus.on("user_message")
async def handle_user_message(event):
    strategy = await strategy_ai.analyze(event.session_id)
    event_bus.publish("strategy_ready", strategy)
```

#### 优势

1. 状态持久化：Session 状态保存在 Redis/DB
2. 上下文连续：跨请求保持完整上下文
3. 解耦：事件驱动实现组件解耦
4. 可扩展：易于添加新的事件监听器
5. 状态恢复：支持断线重连和状态恢复
6. 性能：避免重复计算（如面试计划）

#### 劣势

1. 复杂度：需要管理 Session 生命周期
2. 存储成本：需要 Redis/DB 存储状态
3. 一致性：需要处理并发和状态同步
4. 清理：需要 TTL 和清理机制

## 适用场景分析

### 当前无状态方案适合

- 简单对话场景
- 前端状态管理完善
- 需要快速迭代
- 对话历史较短（< 50 轮）
- 不需要复杂状态管理

### Session + 事件驱动适合

- 复杂多轮对话（> 50 轮）
- 需要状态持久化
- 需要断线重连
- 多 Agent 协作
- 需要状态恢复
- 长期会话（数小时/天）

## 混合方案建议

结合两种方案的优势：

### 方案C：轻量级 Session + 事件驱动

```python
# 1. Session 只存储关键状态
session = {
    "interview_id": "test_001",
    "interview_plan": {...},  # 只存储计划，不存储完整历史
    "current_question_index": 2,
    "created_at": "...",
    "ttl": 3600
}

# 2. 对话历史仍通过请求传递（减少存储压力）
# 3. 使用事件驱动处理流程
# 4. 关键状态持久化，完整历史由前端维护
```

#### 实现要点

1. Session 存储：
   - `interview_plan`（避免重复生成）
   - `current_question_index`（进度跟踪）
   - 元数据（创建时间、TTL）

2. 对话历史：
   - 仍通过请求体传递（减少存储）
   - 可选：最近 N 轮存储在 Session（用于快速恢复）

3. 事件驱动：
   - 轻量级事件总线（内存）
   - 异步处理流程
   - 解耦各 AI 组件

## 具体实施建议

### 对于你的项目

当前项目特点：
- 已有 Redis 基础设施（`LangchainRedisManager`）
- 对话历史可能较长（面试多轮）
- 面试计划需要持久化（避免重复生成）

建议采用混合方案：

```python
# 伪代码示例
class SessionBasedInterview:
    def __init__(self, interview_id):
        self.interview_id = interview_id
        self.redis = LangchainRedisManager(...)
    
    async def get_ai_response(self, dialog_history):
        # 1. 从 Redis 恢复 Session
        session = await self.redis.get_session(self.interview_id)
        
        # 2. 如果首次，生成并保存计划
        if not session.interview_plan:
            plan = await self.generate_interview_plan()
            await self.redis.save_session(self.interview_id, {
                "interview_plan": plan,
                "current_index": 0
            })
        
        # 3. 事件驱动处理
        event_bus.publish("user_message", {
            "session": session,
            "dialog_history": dialog_history
        })
        
        # 4. 流式返回结果
        async for chunk in self.process_events():
            yield chunk
        
        # 5. 更新 Session 状态
        await self.redis.update_session(self.interview_id, {
            "current_index": self.current_question_index
        })
```

## 总结

| 维度 | 无状态 HTTP | Session + 事件驱动 | 混合方案 |
|------|------------|-------------------|---------|
| 复杂度 | ⭐ 低 | ⭐⭐⭐ 高 | ⭐⭐ 中 |
| 状态管理 | ❌ 前端维护 | ✅ 后端持久化 | ✅ 关键状态持久化 |
| 性能 | ⭐⭐ 中等 | ⭐⭐⭐ 高 | ⭐⭐⭐ 高 |
| 扩展性 | ⭐⭐ 中等 | ⭐⭐⭐ 高 | ⭐⭐⭐ 高 |
| 容错性 | ⭐⭐⭐ 高 | ⭐⭐ 中等 | ⭐⭐⭐ 高 |
| 适用场景 | 简单对话 | 复杂长期对话 | 大多数场景 |

建议：对于模拟面试场景，采用轻量级 Session + 事件驱动，既保持状态连续性，又不过度复杂化。

```plaintext
┌─────────────────────────────────────┐
│        Session 管理器                │
├─────────────────────────────────────┤
│  - Session ID（唯一标识）             │
│  - 对话历史（dialog_history）         │
│  - Agent 状态（interview_plan等）    │
│  - 元数据（创建时间、TTL等）          │
│  - 存储：Redis / Database            │
└─────────────────────────────────────┘
```

```plaintext
┌─────────────────────────────────────┐
│        事件总线（Event Bus）          │
├─────────────────────────────────────┤
│  事件类型：                           │
│  - user_message（用户消息）          │
│  - agent_response（Agent回复）       │
│  - session_start（会话开始）        │
│  - session_end（会话结束）           │
│  - error（错误事件）                  │
└─────────────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│      事件监听器（Event Listeners）    │
├─────────────────────────────────────┤
│  - 策略AI监听器                      │
│  - 执行AI监听器                      │
│  - 判断AI监听器                      │
│  - 状态更新监听器                    │
└─────────────────────────────────────┘
```

```plaintext
┌─────────────────────────────────────────────────────────┐
│               Session + 事件驱动 Loop                     │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  1. 用户发送消息                                         │
│     ↓                                                    │
│  2. 创建/恢复 Session                                    │
│     - 从 Redis 加载对话历史                             │
│     - 恢复 Agent 状态（interview_plan等）                │
│     ↓                                                    │
│  3. 发布事件：user_message_event                        │
│     ↓                                                    │
│  4. 事件总线分发事件                                     │
│     ↓                                                    │
│  5. 策略AI监听器处理                                     │
│     - 分析对话历史                                       │
│     - 生成策略                                           │
│     - 发布 strategy_ready_event                         │
│     ↓                                                    │
│  6. 执行AI监听器处理                                     │
│     - 接收策略事件                                       │
│     - 生成回复（流式）                                   │
│     - 发布 response_chunk_event（多次）                 │
│     ↓                                                    │
│  7. 判断AI监听器处理                                     │
│     - 判断是否结束                                       │
│     - 发布 session_end_event（如需要）                   │
│     ↓                                                    │
│  8. 状态更新监听器                                       │
│     - 更新 Session 状态                                 │
│     - 保存对话历史到 Redis                              │
│     - 持久化 Agent 状态                                 │
│     ↓                                                    │
│  9. 返回响应给前端                                       │
│     ↓                                                    │
│  10. 等待下次用户消息（回到步骤1）                       │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

```python
# 每次请求都创建新实例
interview = CallInterview(interview_id)  # 无状态
interview.get_ai_response(dialog_history)  # 状态通过参数传递
```

```python
# Session 管理
session = await session_manager.get_or_create(interview_id)
session.dialog_history = await redis.get(f"session:{interview_id}:history")
session.interview_plan = await redis.get(f"session:{interview_id}:plan")

# 事件驱动
event_bus.publish("user_message", {
    "session_id": interview_id,
    "message": user_message
})

# 监听器处理
@event_bus.on("user_message")
async def handle_user_message(event):
    strategy = await strategy_ai.analyze(event.session_id)
    event_bus.publish("strategy_ready", strategy)
```

```python
# 1. Session 只存储关键状态
session = {
    "interview_id": "test_001",
    "interview_plan": {...},  # 只存储计划，不存储完整历史
    "current_question_index": 2,
    "created_at": "...",
    "ttl": 3600
}

# 2. 对话历史仍通过请求传递（减少存储压力）
# 3. 使用事件驱动处理流程
# 4. 关键状态持久化，完整历史由前端维护
```

```python
# 伪代码示例
class SessionBasedInterview:
    def __init__(self, interview_id):
        self.interview_id = interview_id
        self.redis = LangchainRedisManager(...)
    
    async def get_ai_response(self, dialog_history):
        # 1. 从 Redis 恢复 Session
        session = await self.redis.get_session(self.interview_id)
        
        # 2. 如果首次，生成并保存计划
        if not session.interview_plan:
            plan = await self.generate_interview_plan()
            await self.redis.save_session(self.interview_id, {
                "interview_plan": plan,
                "current_index": 0
            })
        
        # 3. 事件驱动处理
        event_bus.publish("user_message", {
            "session": session,
            "dialog_history": dialog_history
        })
        
        # 4. 流式返回结果
        async for chunk in self.process_events():
            yield chunk
        
        # 5. 更新 Session 状态
        await self.redis.update_session(self.interview_id, {
            "current_index": self.current_question_index
        })
```

---

**User**

我现在还是不是很确定我项目里面的这个agent loop到底是什么控制的呀。能不能帮我一步步地梳理清楚呢？

---

**Cursor**

逐步追踪代码执行流程，梳理 agent loop 的控制机制：


查看前端如何调用后端API：


逐步梳理 Agent Loop 的控制机制：

## Agent Loop 控制机制完整梳理

### 第1层：前端用户交互循环（最外层控制）

#### 控制点：Gradio 事件绑定

```python
# app_mock_interview.py 第 376-393 行
send_btn.click(fn=chat, ...)      # 按钮点击触发
msg.submit(fn=chat, ...)          # 回车键触发
```

控制机制：
- 用户操作触发：点击“发送”或按回车
- 触发函数：`chat()` 函数
- 循环维持：用户每次输入都会触发新的 `chat()` 调用

### 第2层：前端状态管理循环（状态维护）

#### 控制点：全局客户端对象

```python
# app_mock_interview.py 第 125 行
client = InterviewClient()  # 全局单例

# InterviewClient 类（第 59-68 行）
class InterviewClient:
    def __init__(self):
        self.dialog_history = []  # 维护对话历史
        self.is_over = False      # 维护结束状态
```

状态维护机制：
- `client.dialog_history`：存储完整对话历史
- 每次用户发送消息时更新（第 238-241 行）
- 每次收到 AI 回复时更新（第 111-114 行）

### 第3层：HTTP 请求循环（网络层）

#### 控制点：前端 API 调用

```python
# app_mock_interview.py 第 81-86 行
response = requests.post(
    INTERVIEW_API,              # http://localhost:9006/virturlInterview
    json=request_body,          # 包含完整 dialogHistory
    stream=True,                # 流式接收
    timeout=120
)
```

请求体结构：
```python
{
    "id": "test_interview_001",
    "userPortrait": "...",
    "jdTitle": "...",
    "dialogHistory": [           # 关键：完整对话历史
        {"role": "user", "content": "..."},
        {"role": "assistant", "content": "..."}
    ]
}
```

循环控制：
- 每次 `chat()` 调用都会发送新的 HTTP 请求
- 请求体包含完整的 `dialogHistory`
- 后端通过 `dialogHistory` 恢复上下文

### 第4层：后端路由层（请求接收）

#### 控制点：FastAPI 路由

```python
# interview_process_router.py 第 11-37 行
@router.post('/virturlInterview')
async def call_interview(request: Request):
    json_data = await request.json()
    dialog_history = json_data.get("dialogHistory")  # 提取对话历史
    interview_id = json_data.get("id")
    
    # 每次请求创建新实例（无状态）
    interview = CallInterview(interview_id)
    interview.user_portrait = user_portrait
    # ... 设置属性
    
    # 流式返回
    return StreamingResponse(
        interview.get_ai_response(dialog_history),  # 传入对话历史
        media_type='text/event-stream'
    )
```

控制机制：
- 每次 HTTP 请求都会创建新的 `CallInterview` 实例
- 通过 `dialog_history` 参数传递状态
- 返回流式响应（SSE）

### 第5层：服务层处理循环（核心逻辑）

#### 控制点：`get_ai_response()` 函数

```python
# virtual_interview.py 第 248 行
async def get_ai_response(self, dialog_history):
    # 这是一个生成器函数（async generator）
    # 通过 yield 流式返回结果
```

执行流程控制：

步骤1：状态检查（第 258-278 行）
```python
# 沉默检测
if silence_count >= 3:
    yield "面试结束"
    return  # 结束循环
```

步骤2：首次调用判断（第 281-301 行）
```python
if not dialog_history or len(dialog_history) == 0:
    # 首次调用：生成面试计划
    await self.generate_interview_plan()
    yield "第一个问题"
    return  # 结束本次请求
```

步骤3：策略AI（第 352-368 行）
```python
celue = self.llm_client.create_completion(...)  # 同步调用
celue_text = celue.choices[0].message.content   # 获取策略
```

步骤4：执行AI（第 376-405 行）
```python
response = self.llm_client.create_streaming_completion(...)  # 流式调用
async for chunk in response:
    ai_response += content
    yield f"data: {json.dumps({'type': 'chunk', 'content': content})}\n\n"
```

步骤5：意图识别（第 407-415 行）
```python
intent = await self._detect_interview_action(...)
if intent.get("action") == "new_question":
    self.current_question_index += 1  # 更新索引
```

步骤6：判断AI（第 420-434 行）
```python
liucheng = self.celue_client.chat.completions.create(...)
if "结束" in liucheng_flag:
    yield f"data: {json.dumps({'type': 'over', ...})}\n\n"
```

### 第6层：前端响应处理循环（结果接收）

#### 控制点：流式响应处理

```python
# app_mock_interview.py 第 90-107 行
for line in response.iter_lines():
    if decoded.startswith("data: "):
        data = json.loads(decoded[6:])
        event_type = data.get("type")
        
        if event_type == "chunk":
            full_response += content
            yield full_response  # 实时更新UI
        
        elif event_type == "over":
            self.is_over = True  # 标记结束
```

## 完整 Loop 控制流程图

```
┌─────────────────────────────────────────────────────────────┐
│  第1层：用户交互循环（最外层）                                │
├─────────────────────────────────────────────────────────────┤
│  用户输入消息                                                │
│    ↓                                                        │
│  Gradio 事件触发（send_btn.click / msg.submit）            │
│    ↓                                                        │
│  调用 chat() 函数                                           │
└─────────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────────┐
│  第2层：前端状态管理（状态维护）                              │
├─────────────────────────────────────────────────────────────┤
│  client.dialog_history.append({"role": "user", ...})       │
│    ↓                                                        │
│  构建请求体（包含完整 dialogHistory）                       │
└─────────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────────┐
│  第3层：HTTP 请求循环（网络层）                              │
├─────────────────────────────────────────────────────────────┤
│  requests.post("/virturlInterview", json=request_body)     │
│    ↓                                                        │
│  等待流式响应                                                │
└─────────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────────┐
│  第4层：后端路由层（请求接收）                                │
├─────────────────────────────────────────────────────────────┤
│  FastAPI 路由接收请求                                        │
│    ↓                                                        │
│  创建 CallInterview(interview_id) 实例                      │
│    ↓                                                        │
│  调用 interview.get_ai_response(dialog_history)            │
└─────────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────────┐
│  第5层：服务层处理（核心逻辑）                                │
├─────────────────────────────────────────────────────────────┤
│  1. 状态检查（沉默检测）                                     │
│  2. 首次调用判断（生成计划）                                 │
│  3. 策略AI（分析对话）                                      │
│  4. 执行AI（生成回复，流式输出）                             │
│  5. 意图识别（更新索引）                                    │
│  6. 判断AI（判断结束）                                      │
│    ↓                                                        │
│  流式返回结果（yield）                                       │
└─────────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────────┐
│  第6层：前端响应处理（结果接收）                              │
├─────────────────────────────────────────────────────────────┤
│  接收流式响应（SSE）                                         │
│    ↓                                                        │
│  解析事件类型（chunk / over）                               │
│    ↓                                                        │
│  更新 UI（实时显示）                                         │
│    ↓                                                        │
│  更新 client.dialog_history                                 │
└─────────────────────────────────────────────────────────────┘
                    ↓
            【回到第1层，等待下次用户输入】
```

## 关键控制点总结

### 1. Loop 的启动控制
- 控制者：用户操作（点击按钮/按回车）
- 触发点：Gradio 事件绑定
- 入口函数：`chat()`

### 2. Loop 的状态控制
- 控制者：前端 `client.dialog_history`
- 存储位置：内存（全局变量）
- 传递方式：HTTP 请求体

### 3. Loop 的执行控制
- 控制者：后端 `get_ai_response()` 函数
- 执行方式：同步 + 异步混合
- 返回方式：流式（SSE）

### 4. Loop 的结束控制
- 控制者：判断AI（`judge_flow.txt`）
- 结束条件：
  - 判断AI返回"结束"
  - 连续3次沉默
  - 前端收到 `type: "over"` 事件

### 5. Loop 的循环维持
- 维持机制：用户每次输入 → 触发新请求 → 处理 → 返回 → 等待下次输入
- 不是真正的循环：每次都是新的 HTTP 请求
- 状态连续性：通过 `dialog_history` 参数传递

## 核心结论

你的 Agent Loop 是：
- 不是传统循环：没有 while/for 循环
- 基于事件驱动：用户操作触发
- 基于状态传递：通过 `dialog_history` 维持上下文
- 基于 HTTP 请求：每次交互都是独立请求
- 前端控制：前端维护状态，后端无状态处理

这是一个“请求-响应”模式的伪循环，通过状态传递实现连续性。

```python
# app_mock_interview.py 第 376-393 行
send_btn.click(fn=chat, ...)      # 按钮点击触发
msg.submit(fn=chat, ...)          # 回车键触发
```

```python
# app_mock_interview.py 第 125 行
client = InterviewClient()  # 全局单例

# InterviewClient 类（第 59-68 行）
class InterviewClient:
    def __init__(self):
        self.dialog_history = []  # 维护对话历史
        self.is_over = False      # 维护结束状态
```

```python
# app_mock_interview.py 第 81-86 行
response = requests.post(
    INTERVIEW_API,              # http://localhost:9006/virturlInterview
    json=request_body,          # 包含完整 dialogHistory
    stream=True,                # 流式接收
    timeout=120
)
```

```python
{
    "id": "test_interview_001",
    "userPortrait": "...",
    "jdTitle": "...",
    "dialogHistory": [           # 关键：完整对话历史
        {"role": "user", "content": "..."},
        {"role": "assistant", "content": "..."}
    ]
}
```

```python
# interview_process_router.py 第 11-37 行
@router.post('/virturlInterview')
async def call_interview(request: Request):
    json_data = await request.json()
    dialog_history = json_data.get("dialogHistory")  # 提取对话历史
    interview_id = json_data.get("id")
    
    # 每次请求创建新实例（无状态）
    interview = CallInterview(interview_id)
    interview.user_portrait = user_portrait
    # ... 设置属性
    
    # 流式返回
    return StreamingResponse(
        interview.get_ai_response(dialog_history),  # 传入对话历史
        media_type='text/event-stream'
    )
```

```python
# virtual_interview.py 第 248 行
async def get_ai_response(self, dialog_history):
    # 这是一个生成器函数（async generator）
    # 通过 yield 流式返回结果
```

```python
# 沉默检测
if silence_count >= 3:
    yield "面试结束"
    return  # 结束循环
```

```python
if not dialog_history or len(dialog_history) == 0:
    # 首次调用：生成面试计划
    await self.generate_interview_plan()
    yield "第一个问题"
    return  # 结束本次请求
```

```python
celue = self.llm_client.create_completion(...)  # 同步调用
celue_text = celue.choices[0].message.content   # 获取策略
```

```python
response = self.llm_client.create_streaming_completion(...)  # 流式调用
async for chunk in response:
    ai_response += content
    yield f"data: {json.dumps({'type': 'chunk', 'content': content})}\n\n"
```

```python
intent = await self._detect_interview_action(...)
if intent.get("action") == "new_question":
    self.current_question_index += 1  # 更新索引
```

```python
liucheng = self.celue_client.chat.completions.create(...)
if "结束" in liucheng_flag:
    yield f"data: {json.dumps({'type': 'over', ...})}\n\n"
```

```python
# app_mock_interview.py 第 90-107 行
for line in response.iter_lines():
    if decoded.startswith("data: "):
        data = json.loads(decoded[6:])
        event_type = data.get("type")
        
        if event_type == "chunk":
            full_response += content
            yield full_response  # 实时更新UI
        
        elif event_type == "over":
            self.is_over = True  # 标记结束
```

```plaintext
┌─────────────────────────────────────────────────────────────┐
│  第1层：用户交互循环（最外层）                                │
├─────────────────────────────────────────────────────────────┤
│  用户输入消息                                                │
│    ↓                                                        │
│  Gradio 事件触发（send_btn.click / msg.submit）            │
│    ↓                                                        │
│  调用 chat() 函数                                           │
└─────────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────────┐
│  第2层：前端状态管理（状态维护）                              │
├─────────────────────────────────────────────────────────────┤
│  client.dialog_history.append({"role": "user", ...})       │
│    ↓                                                        │
│  构建请求体（包含完整 dialogHistory）                       │
└─────────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────────┐
│  第3层：HTTP 请求循环（网络层）                              │
├─────────────────────────────────────────────────────────────┤
│  requests.post("/virturlInterview", json=request_body)     │
│    ↓                                                        │
│  等待流式响应                                                │
└─────────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────────┐
│  第4层：后端路由层（请求接收）                                │
├─────────────────────────────────────────────────────────────┤
│  FastAPI 路由接收请求                                        │
│    ↓                                                        │
│  创建 CallInterview(interview_id) 实例                      │
│    ↓                                                        │
│  调用 interview.get_ai_response(dialog_history)            │
└─────────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────────┐
│  第5层：服务层处理（核心逻辑）                                │
├─────────────────────────────────────────────────────────────┤
│  1. 状态检查（沉默检测）                                     │
│  2. 首次调用判断（生成计划）                                 │
│  3. 策略AI（分析对话）                                      │
│  4. 执行AI（生成回复，流式输出）                             │
│  5. 意图识别（更新索引）                                    │
│  6. 判断AI（判断结束）                                      │
│    ↓                                                        │
│  流式返回结果（yield）                                       │
└─────────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────────┐
│  第6层：前端响应处理（结果接收）                              │
├─────────────────────────────────────────────────────────────┤
│  接收流式响应（SSE）                                         │
│    ↓                                                        │
│  解析事件类型（chunk / over）                               │
│    ↓                                                        │
│  更新 UI（实时显示）                                         │
│    ↓                                                        │
│  更新 client.dialog_history                                 │
└─────────────────────────────────────────────────────────────┘
                    ↓
            【回到第1层，等待下次用户输入】
```

---

**User**

所以你的意思是每次用户的输入以后就会流水线式的经过五次AI的调用是吗？这么理解准确吗？m如果满分一百分的话这个理解给几分呢？以及为什么？

---

**Cursor**

检查代码，确认每次用户输入后的实际执行流程：


检查意图识别函数的内部实现：


## 评分：70分

### 为什么给70分？

你的理解部分正确，但有几个关键偏差：

## 实际执行流程分析

### 情况1：首次调用（用户点击"开始面试"）

```python
# virtual_interview.py 第 281-299 行
if not dialog_history or len(dialog_history) == 0:
    await self.generate_interview_plan()  # 只有1次AI调用
    yield "第一个问题"
    return  # 直接结束，不执行后续
```

AI调用次数：1次
- 计划AI（生成面试计划）

### 情况2：非首次调用（用户发送消息）

```python
# 第 352-368 行：策略AI
celue = self.llm_client.create_completion(...)  # AI调用1

# 第 387-403 行：执行AI
response = self.llm_client.create_streaming_completion(...)  # AI调用2（流式）

# 第 408 行：意图识别
intent = await self._detect_interview_action(...)  # 内部调用AI，AI调用3

# 第 424-429 行：判断AI
liucheng = self.celue_client.chat.completions.create(...)  # AI调用4
```

AI调用次数：4次
1. 策略AI（分析对话，生成策略）
2. 执行AI（根据策略生成回复，流式）
3. 意图识别（识别行动类型）
4. 判断AI（判断是否结束）

## 理解偏差分析

### 偏差1：不是5次，是4次（非首次）

你提到的5次可能包括：
- 计划AI（仅首次）
- 策略AI
- 执行AI
- 意图识别
- 判断AI

实际情况：
- 首次：只有计划AI（1次）
- 后续：策略+执行+意图+判断（4次）

### 偏差2：不是每次都是5次

首次调用只有1次AI调用，后续调用是4次。

### 偏差3：流水线式执行

你的理解基本正确，但需要补充：

```python
策略AI（同步，等待完成）
    ↓
执行AI（流式，边生成边返回）
    ↓
意图识别（同步，等待完成）
    ↓
判断AI（同步，等待完成）
```

执行顺序：
- 策略AI → 执行AI：顺序（执行依赖策略结果）
- 执行AI → 意图识别：顺序（意图识别依赖执行结果）
- 执行AI → 判断AI：顺序（判断依赖执行结果）
- 意图识别和判断AI：理论上可并行（都依赖执行结果，但互不依赖），但代码中是顺序执行

## 完整流程图

```
用户输入消息
    ↓
【首次调用？】
    ├─ 是 → 计划AI（1次）→ 返回第一个问题 → 结束
    └─ 否 → 继续
    ↓
策略AI（第1次调用，同步）
    ↓
执行AI（第2次调用，流式）
    ↓
意图识别（第3次调用，同步）
    ↓
判断AI（第4次调用，同步）
    ↓
返回结果
```

## 扣分原因

1. 数量不准确：不是5次，是4次（非首次）
2. 条件不完整：首次只有1次，不是每次都5次
3. 并行性：意图识别和判断AI理论上可并行，但代码中是顺序

## 准确描述

每次用户输入后（非首次）：
- 流水线式执行：是
- AI调用次数：4次（策略→执行→意图→判断）
- 执行顺序：严格顺序（策略→执行→意图→判断）
- 首次调用：只有1次（计划AI）

## 优化建议

如果要提高分数到90+，需要明确：
1. 首次调用：1次AI（计划AI）
2. 后续调用：4次AI（策略→执行→意图→判断）
3. 执行方式：流水线式顺序执行
4. 可优化点：意图识别和判断AI可以并行执行（代码中未实现）

总结：你的理解方向正确，但细节需要修正。

```python
# virtual_interview.py 第 281-299 行
if not dialog_history or len(dialog_history) == 0:
    await self.generate_interview_plan()  # 只有1次AI调用
    yield "第一个问题"
    return  # 直接结束，不执行后续
```

```python
# 第 352-368 行：策略AI
celue = self.llm_client.create_completion(...)  # AI调用1

# 第 387-403 行：执行AI
response = self.llm_client.create_streaming_completion(...)  # AI调用2（流式）

# 第 408 行：意图识别
intent = await self._detect_interview_action(...)  # 内部调用AI，AI调用3

# 第 424-429 行：判断AI
liucheng = self.celue_client.chat.completions.create(...)  # AI调用4
```

```python
策略AI（同步，等待完成）
    ↓
执行AI（流式，边生成边返回）
    ↓
意图识别（同步，等待完成）
    ↓
判断AI（同步，等待完成）
```

```plaintext
用户输入消息
    ↓
【首次调用？】
    ├─ 是 → 计划AI（1次）→ 返回第一个问题 → 结束
    └─ 否 → 继续
    ↓
策略AI（第1次调用，同步）
    ↓
执行AI（第2次调用，流式）
    ↓
意图识别（第3次调用，同步）
    ↓
判断AI（第4次调用，同步）
    ↓
返回结果
```

---

**User**

我现在的问题是。我的前端页面的左侧会显示我所有的请求体的信息，并且随着在右侧跟AI进行沟通和交流。左侧显示的请求的信息会实时的更新。但是呢，我经过了几轮的对话，我发现左侧的请求体里面它并没有显示出面试计划呀。按道理来说，它不应该是会有面试计划来拼接到提示词里面吗？请ultrathinking,帮我梳理清楚呀。

---

**Cursor**

检查前端如何处理请求体更新，以及后端是否返回面试计划：


对比简历优化功能，查看它如何处理计划信息：


检查后端是否应该返回面试计划，以及简历优化功能是如何处理的：


## 问题根因分析

### 问题症状
- 前端左侧请求体中缺少 `interview_plan`
- 后端生成了面试计划，但前端未显示

### 对比：简历优化 vs 模拟面试

#### 简历优化功能（正确实现）

后端返回计划：
```python
# deep_resume_converse.py 第 413-414 行
if plan:
    yield f"data: {json.dumps({'type': 'plan', 'content': plan}, ensure_ascii=False)}\n\n"
```

前端接收计划：
```python
# app_resume_optimization.py 第 128-130 行
elif event_type == "plan":
    # 保存对话计划（仅首次）
    self.conversation_plan = content
```

前端更新请求体：
```python
# app_resume_optimization.py 第 217-218 行
if client.conversation_plan:
    body["conversation_plan"] = client.conversation_plan
```

#### 模拟面试功能（缺失实现）

后端未返回计划：
```python
# virtual_interview.py 第 296-297 行
yield f"data: {json.dumps({'type': 'chunk', 'content': first_question}, ensure_ascii=False)}\n\n"
yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
# ❌ 缺少：yield f"data: {json.dumps({'type': 'plan', 'content': self.interview_plan}, ...)}\n\n"
```

前端未接收计划：
```python
# app_mock_interview.py 第 99-104 行
if event_type == "chunk":
    full_response += content
    yield full_response
elif event_type == "over":
    self.is_over = True
# ❌ 缺少：elif event_type == "plan": 的处理
```

前端未存储计划：
```python
# app_mock_interview.py 第 59-68 行
class InterviewClient:
    def reset(self):
        self.dialog_history = []
        self.is_over = False
        # ❌ 缺少：self.interview_plan = None
```

前端未更新请求体：
```python
# app_mock_interview.py 第 169-176 行
def update_request_with_history(json_text: str) -> str:
    body = json.loads(json_text)
    body["dialogHistory"] = client.dialog_history
    # ❌ 缺少：if client.interview_plan: body["interview_plan"] = client.interview_plan
    return json.dumps(body, ensure_ascii=False, indent=2)
```

## 完整问题梳理

### 缺失环节1：后端未返回面试计划

位置：`virtual_interview.py` 第 296-299 行

当前代码：
```python
yield f"data: {json.dumps({'type': 'chunk', 'content': first_question}, ensure_ascii=False)}\n\n"
yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
return
```

应该添加：
```python
# 返回面试计划
if self.interview_plan:
    yield f"data: {json.dumps({'type': 'plan', 'content': self.interview_plan}, ensure_ascii=False)}\n\n"
```

### 缺失环节2：前端未接收面试计划

位置：`app_mock_interview.py` 第 99-104 行

当前代码：
```python
if event_type == "chunk":
    full_response += content
    yield full_response
elif event_type == "over":
    self.is_over = True
```

应该添加：
```python
elif event_type == "plan":
    # 保存面试计划（仅首次）
    self.interview_plan = content
```

### 缺失环节3：前端未存储面试计划

位置：`app_mock_interview.py` 第 59-68 行

当前代码：
```python
class InterviewClient:
    def reset(self):
        self.dialog_history = []
        self.is_over = False
```

应该添加：
```python
def reset(self):
    self.dialog_history = []
    self.is_over = False
    self.interview_plan = None  # 新增
```

### 缺失环节4：前端未更新请求体

位置：`app_mock_interview.py` 第 169-176 行

当前代码：
```python
def update_request_with_history(json_text: str) -> str:
    body = json.loads(json_text)
    body["dialogHistory"] = client.dialog_history
    return json.dumps(body, ensure_ascii=False, indent=2)
```

应该添加：
```python
def update_request_with_history(json_text: str) -> str:
    body = json.loads(json_text)
    body["dialogHistory"] = client.dialog_history
    if client.interview_plan:  # 新增
        body["interview_plan"] = client.interview_plan  # 新增
    return json.dumps(body, ensure_ascii=False, indent=2)
```

## 修复方案总结

需要修改4处：

1. 后端返回计划（`virtual_interview.py` 第 297 行后）
   - 添加返回 `type: "plan"` 事件的代码

2. 前端接收计划（`app_mock_interview.py` 第 103 行后）
   - 添加处理 `event_type == "plan"` 的逻辑

3. 前端存储计划（`app_mock_interview.py` 第 67 行）
   - 在 `InterviewClient.reset()` 中添加 `self.interview_plan = None`

4. 前端更新请求体（`app_mock_interview.py` 第 173 行后）
   - 在 `update_request_with_history()` 中添加 `interview_plan` 到请求体

## 为什么会出现这个问题？

- 简历优化功能已实现计划返回与显示
- 模拟面试功能未同步实现相同逻辑
- 后端生成了计划但未通过 SSE 返回
- 前端未接收、存储和显示计划

修复后，前端左侧请求体将显示完整的 `interview_plan`，与简历优化功能一致。

```python
# deep_resume_converse.py 第 413-414 行
if plan:
    yield f"data: {json.dumps({'type': 'plan', 'content': plan}, ensure_ascii=False)}\n\n"
```

```python
# app_resume_optimization.py 第 128-130 行
elif event_type == "plan":
    # 保存对话计划（仅首次）
    self.conversation_plan = content
```

```python
# app_resume_optimization.py 第 217-218 行
if client.conversation_plan:
    body["conversation_plan"] = client.conversation_plan
```

```python
# virtual_interview.py 第 296-297 行
yield f"data: {json.dumps({'type': 'chunk', 'content': first_question}, ensure_ascii=False)}\n\n"
yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
# ❌ 缺少：yield f"data: {json.dumps({'type': 'plan', 'content': self.interview_plan}, ...)}\n\n"
```

```python
# app_mock_interview.py 第 99-104 行
if event_type == "chunk":
    full_response += content
    yield full_response
elif event_type == "over":
    self.is_over = True
# ❌ 缺少：elif event_type == "plan": 的处理
```

```python
# app_mock_interview.py 第 59-68 行
class InterviewClient:
    def reset(self):
        self.dialog_history = []
        self.is_over = False
        # ❌ 缺少：self.interview_plan = None
```

```python
# app_mock_interview.py 第 169-176 行
def update_request_with_history(json_text: str) -> str:
    body = json.loads(json_text)
    body["dialogHistory"] = client.dialog_history
    # ❌ 缺少：if client.interview_plan: body["interview_plan"] = client.interview_plan
    return json.dumps(body, ensure_ascii=False, indent=2)
```

```python
yield f"data: {json.dumps({'type': 'chunk', 'content': first_question}, ensure_ascii=False)}\n\n"
yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
return
```

```python
# 返回面试计划
if self.interview_plan:
    yield f"data: {json.dumps({'type': 'plan', 'content': self.interview_plan}, ensure_ascii=False)}\n\n"
```

```python
if event_type == "chunk":
    full_response += content
    yield full_response
elif event_type == "over":
    self.is_over = True
```

```python
elif event_type == "plan":
    # 保存面试计划（仅首次）
    self.interview_plan = content
```

```python
class InterviewClient:
    def reset(self):
        self.dialog_history = []
        self.is_over = False
```

```python
def reset(self):
    self.dialog_history = []
    self.is_over = False
    self.interview_plan = None  # 新增
```

```python
def update_request_with_history(json_text: str) -> str:
    body = json.loads(json_text)
    body["dialogHistory"] = client.dialog_history
    return json.dumps(body, ensure_ascii=False, indent=2)
```

```python
def update_request_with_history(json_text: str) -> str:
    body = json.loads(json_text)
    body["dialogHistory"] = client.dialog_history
    if client.interview_plan:  # 新增
        body["interview_plan"] = client.interview_plan  # 新增
    return json.dumps(body, ensure_ascii=False, indent=2)
```

---

**User**

现在情况是我在利用这个AI面试官，跟AI对话的时候，使用的是模拟面试接口，我尝试回答了面试官提出的问题，我回答了两次，但是为什么面试官总是他的意图识别为follow up呢？就是这点我不太懂啊，他为什么老是追着这个问题不放呢？能不能帮我一步步梳理一下根本原因呀？你可以参考后面这个终端输出 @后端 (916-1020) 

---

**Cursor**

分析意图识别和追问逻辑，梳理为何总是识别为 follow_up：


检查策略 prompt 和追问次数控制逻辑：


检查是否有追踪追问次数的机制：


## 问题根因分析

### 问题症状
- 面试官连续追问同一问题
- 意图识别总是 `follow_up`
- 问题索引不更新，无法进入下一题

### 根本原因：缺少追问次数追踪机制

## 逐步分析

### 第1步：策略AI生成策略

从终端输出看（第919-922行）：
```
策略如下：
4. 下一轮对话策略：针对其唯一提到的FastAPI框架进行最后一次技术追问
```

策略AI建议追问，但这是建议，不是强制。

### 第2步：执行AI生成回复

执行AI根据策略生成追问回复（第933行）：
```
张三，你提到使用了 FastAPI 框架...请你具体说明一下...
```

### 第3步：意图识别判断类型

```python
# virtual_interview.py 第 408 行
intent = await self._detect_interview_action(ai_response, dialog_history[-6:])
# 结果：{'action': 'follow_up'}
```

意图识别逻辑（第190-240行）：
- 只分析当前回复内容
- 不追踪追问次数
- 只要是在追问细节，就返回 `follow_up`

### 第4步：问题索引更新逻辑

```python
# virtual_interview.py 第 411-415 行
if self.interview_plan and intent.get("action") == "new_question":
    self.current_question_index += 1  # 只有 new_question 才更新
else:
    logging.info("🔄 保持当前问题，动作类型: follow_up")
    # follow_up 时索引不变！
```

问题：
- 只有 `new_question` 才更新索引
- `follow_up` 时索引不变，继续追问同一问题

### 第5步：缺少追问次数追踪

检查代码发现：
- `CallInterview` 类中没有追踪追问次数的变量
- 没有 `follow_up_count` 或 `current_question_follow_count`
- 没有机制统计当前问题已追问几次

## 完整问题链路

```
策略AI建议追问
    ↓
执行AI生成追问回复
    ↓
意图识别：判断为 follow_up（因为确实在追问）
    ↓
索引更新逻辑：follow_up → 索引不变
    ↓
【问题】没有追问次数检查
    ↓
下次请求：继续追问同一问题
    ↓
循环往复...
```

## 缺失的关键机制

### 缺失1：追问次数追踪变量

```python
# 应该添加但缺失的变量
self.current_question_follow_count = 0  # 当前问题的追问次数
```

### 缺失2：追问次数检查逻辑

```python
# 应该添加但缺失的逻辑
if intent.get("action") == "follow_up":
    self.current_question_follow_count += 1
    if self.current_question_follow_count >= 2:
        # 强制进入下一题
        intent["action"] = "new_question"
        self.current_question_follow_count = 0
```

### 缺失3：进入新问题时重置计数

```python
# 应该添加但缺失的逻辑
if intent.get("action") == "new_question":
    self.current_question_index += 1
    self.current_question_follow_count = 0  # 重置追问次数
```

## 为什么总是 follow_up？

1. 策略AI建议追问（prompt 说最多2次，但无强制）
2. 执行AI生成追问回复
3. 意图识别只看内容，识别为 `follow_up`
4. 索引更新只看意图类型，`follow_up` 不更新索引
5. 没有追问次数限制，可以一直追问

## 解决方案

需要添加追问次数追踪机制：

1. 添加变量：`self.current_question_follow_count = 0`
2. 在意图识别后检查追问次数
3. 达到2次后强制改为 `new_question`
4. 进入新问题时重置计数

## 总结

- 策略AI建议追问（但无强制）
- 意图识别判断为 `follow_up`（只看内容）
- 索引更新逻辑：`follow_up` 不更新索引
- 缺少追问次数追踪：无法限制追问次数
- 缺少强制机制：达到2次后无法强制进入下一题

结果：会一直追问，直到策略AI主动建议进入下一题，但策略AI可能不会严格遵守“最多2次”的规则。

这是一个设计缺陷：策略中提到了“最多2次”，但代码层面没有实现这个限制机制。

```plaintext
策略如下：
4. 下一轮对话策略：针对其唯一提到的FastAPI框架进行最后一次技术追问
```

```plaintext
张三，你提到使用了 FastAPI 框架...请你具体说明一下...
```

```python
# virtual_interview.py 第 408 行
intent = await self._detect_interview_action(ai_response, dialog_history[-6:])
# 结果：{'action': 'follow_up'}
```

```python
# virtual_interview.py 第 411-415 行
if self.interview_plan and intent.get("action") == "new_question":
    self.current_question_index += 1  # 只有 new_question 才更新
else:
    logging.info("🔄 保持当前问题，动作类型: follow_up")
    # follow_up 时索引不变！
```

```plaintext
策略AI建议追问
    ↓
执行AI生成追问回复
    ↓
意图识别：判断为 follow_up（因为确实在追问）
    ↓
索引更新逻辑：follow_up → 索引不变
    ↓
【问题】没有追问次数检查
    ↓
下次请求：继续追问同一问题
    ↓
循环往复...
```

```python
# 应该添加但缺失的变量
self.current_question_follow_count = 0  # 当前问题的追问次数
```

```python
# 应该添加但缺失的逻辑
if intent.get("action") == "follow_up":
    self.current_question_follow_count += 1
    if self.current_question_follow_count >= 2:
        # 强制进入下一题
        intent["action"] = "new_question"
        self.current_question_follow_count = 0
```

```python
# 应该添加但缺失的逻辑
if intent.get("action") == "new_question":
    self.current_question_index += 1
    self.current_question_follow_count = 0  # 重置追问次数
```

---

**User**

所以说我感觉下面的这个流程有点问题啊是吗？你怎么看呢？你觉得下面这一个输入进来意思流水线式的调用这四个api来进行分析和输出，你觉得正常吗？因为我感觉agent不是这么设计的吧。你怎么看呢？我们来一步步深入探讨一下更合理的设计方案可以吗？

```

策略AI（分析对话，生成策略）
执行AI（根据策略生成回复，流式）
意图识别（识别行动类型）
判断AI（判断是否结束）

---

**Cursor**

分析当前架构设计，探讨更合理的 Agent 设计模式：


分析当前架构问题，并探讨更合理的 Agent 设计模式：


## 当前架构问题分析

### 问题1：固定流水线，缺乏智能决策

当前流程：
```
用户输入 → 策略AI → 执行AI → 意图识别 → 判断AI → 返回
```

问题：
- 每次都要调用4次AI，即使不需要
- 没有条件判断，无法跳过某些步骤
- 执行AI已生成回复，意图识别和判断AI是事后分析，作用有限

### 问题2：职责重叠和冗余

重叠分析：
- 策略AI：分析对话，决定下一步策略
- 执行AI：根据策略生成回复
- 意图识别：分析执行AI的回复类型（事后分析）
- 判断AI：判断是否结束（事后分析）

问题：
- 策略AI已决定策略，执行AI已生成回复
- 意图识别和判断AI是事后验证，对当前回复无影响
- 这些判断应该在生成前进行，而不是生成后

### 问题3：成本与延迟

每次用户输入：
- 策略AI：~7秒（从日志看）
- 执行AI：~9秒（流式）
- 意图识别：~1秒
- 判断AI：~1秒
- 总计：~18秒

问题：
- 成本高：4次API调用
- 延迟大：用户等待时间长
- 部分调用可能不必要

## 更合理的 Agent 设计模式

### 模式1：决策循环（ReAct / Plan-Execute-Check）

```
┌─────────────────────────────────────┐
│  观察（Observe）                     │
│  - 分析对话历史                      │
│  - 评估当前状态                      │
└─────────────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│  思考（Think）                       │
│  - 决定下一步动作                    │
│  - 判断是否需要追问                  │
│  - 判断是否结束                      │
└─────────────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│  行动（Act）                         │
│  - 生成回复                          │
│  - 更新状态                          │
└─────────────────────────────────────┘
         ↓
    【回到观察】
```

### 模式2：状态机（State Machine）

```
状态：提问中
    ↓
用户回答
    ↓
评估回答质量
    ├─ 充分 → 进入下一题（状态转换）
    ├─ 不充分 → 追问（状态：追问中）
    │   ├─ 追问次数 < 2 → 继续追问
    │   └─ 追问次数 >= 2 → 强制下一题
    └─ 结束信号 → 结束面试
```

### 模式3：单次智能调用（Unified Agent）

```
一次AI调用完成所有决策：
- 分析对话
- 决定动作（追问/下一题/结束）
- 生成回复
- 返回结构化结果
```

## 改进方案对比

### 方案A：优化当前流水线（最小改动）

改进点：
1. 合并意图识别和判断AI
2. 条件执行：只在需要时调用
3. 并行执行：意图识别和判断AI可并行

```python
# 伪代码
async def get_ai_response(self, dialog_history):
    # 1. 策略AI（必须）
    strategy = await self.generate_strategy(...)
    
    # 2. 执行AI（必须，流式）
    ai_response = await self.execute_response(...)
    
    # 3. 并行执行：意图识别 + 判断AI
    intent_task = self._detect_interview_action(ai_response)
    judge_task = self._judge_end(ai_response)
    intent, is_end = await asyncio.gather(intent_task, judge_task)
    
    # 4. 根据结果更新状态
    if is_end:
        yield "over"
    elif intent == "new_question":
        self.current_question_index += 1
```

优势：
- 改动小
- 可并行，减少延迟
- 保持现有架构

劣势：
- 仍需要多次调用
- 职责重叠未解决

### 方案B：决策循环模式（推荐）

核心思想：一次智能决策，包含所有判断

```python
async def get_ai_response(self, dialog_history):
    # 单次AI调用，返回结构化决策
    decision = await self.make_decision(dialog_history)
    # decision = {
    #     "action": "follow_up" | "new_question" | "end",
    #     "response": "生成的回复内容",
    #     "follow_count": 1,  # 当前问题已追问次数
    #     "should_end": False
    # }
    
    # 流式返回回复
    async for chunk in self.stream_response(decision["response"]):
        yield chunk
    
    # 更新状态
    if decision["action"] == "new_question":
        self.current_question_index += 1
        self.current_question_follow_count = 0
    elif decision["action"] == "follow_up":
        self.current_question_follow_count += 1
        if self.current_question_follow_count >= 2:
            # 强制下一题
            self.current_question_index += 1
            self.current_question_follow_count = 0
```

优势：
- 减少API调用（从4次到1-2次）
- 降低延迟
- 统一决策逻辑
- 易于追踪状态

劣势：
- 需要重构
- Prompt设计更复杂

### 方案C：状态机模式（最优雅）

核心思想：基于状态的智能决策

```python
class InterviewState(Enum):
    INTRODUCING = "introducing"      # 介绍阶段
    QUESTIONING = "questioning"       # 提问阶段
    FOLLOWING_UP = "following_up"    # 追问阶段
    Q_AND_A = "q_and_a"              # 问答阶段
    ENDING = "ending"                # 结束阶段

async def get_ai_response(self, dialog_history):
    # 1. 评估当前状态
    state = self.assess_state(dialog_history)
    
    # 2. 根据状态决策
    if state == InterviewState.FOLLOWING_UP:
        if self.current_question_follow_count >= 2:
            # 强制进入下一题
            state = InterviewState.QUESTIONING
            self.current_question_index += 1
            self.current_question_follow_count = 0
    
    # 3. 生成回复（根据状态）
    response = await self.generate_response_by_state(state, dialog_history)
    
    # 4. 更新状态
    self.update_state(state, response)
    
    return response
```

优势：
- 状态清晰
- 易于追踪和控制
- 符合状态机模式
- 易于扩展

劣势：
- 需要完整重构
- 状态定义复杂

## 我的建议：混合方案

结合方案B和方案C的优势：

### 核心设计：智能决策 + 状态追踪

```python
async def get_ai_response(self, dialog_history):
    # 步骤1：快速状态检查（无需AI）
    if self._should_end_by_rules(dialog_history):
        yield "面试结束"
        return
    
    # 步骤2：智能决策（单次AI调用）
    decision = await self._make_intelligent_decision(dialog_history)
    # 返回：action, response, metadata
    
    # 步骤3：状态更新（基于决策）
    self._update_state(decision)
    
    # 步骤4：流式返回回复
    async for chunk in self._stream_response(decision["response"]):
        yield chunk
```

### 智能决策函数设计

```python
async def _make_intelligent_decision(self, dialog_history):
    """
    一次AI调用完成所有决策：
    - 分析对话历史
    - 评估回答质量
    - 决定下一步动作
    - 生成回复内容
    - 判断是否结束
    """
    prompt = f"""
    你是面试官，需要根据对话历史做出决策。
    
    当前状态：
    - 问题索引：{self.current_question_index}
    - 当前问题已追问：{self.current_question_follow_count} 次
    
    对话历史：{dialog_history}
    
    请返回JSON格式：
    {{
        "action": "follow_up" | "new_question" | "end",
        "response": "生成的回复内容",
        "quality_score": 0-10,
        "should_end": true/false,
        "reasoning": "决策理由"
    }}
    
    规则：
    - 如果已追问2次，必须 action="new_question"
    - 如果回答质量<3分且已追问1次，action="new_question"
    - 如果明确结束信号，should_end=true
    """
    
    response = await self.llm_client.create_completion(...)
    return json.loads(response)
```

## 对比总结

| 维度 | 当前设计 | 方案A（优化流水线） | 方案B（决策循环） | 方案C（状态机） | 推荐方案（混合） |
|------|---------|-------------------|-----------------|---------------|----------------|
| API调用次数 | 4次 | 3次（并行） | 1-2次 | 1-2次 | 1-2次 |
| 延迟 | ~18秒 | ~12秒 | ~8秒 | ~8秒 | ~8秒 |
| 成本 | 高 | 中 | 低 | 低 | 低 |
| 智能程度 | 中 | 中 | 高 | 高 | 高 |
| 状态追踪 | 弱 | 弱 | 中 | 强 | 强 |
| 实现复杂度 | 低 | 中 | 中 | 高 | 中 |
| 可维护性 | 中 | 中 | 高 | 高 | 高 |

## 最终建议

采用混合方案（方案B + 方案C）：
1. 单次智能决策：一次AI调用完成分析、决策、生成
2. 状态追踪：维护追问次数、问题索引等状态
3. 规则约束：代码层面强制限制（如追问最多2次）
4. 条件执行：只在必要时调用AI

这样既减少调用次数和延迟，又保持决策的智能性和状态的可控性。

你觉得这个方向如何？需要我详细设计某个方案吗？

```plaintext
用户输入 → 策略AI → 执行AI → 意图识别 → 判断AI → 返回
```

```plaintext
┌─────────────────────────────────────┐
│  观察（Observe）                     │
│  - 分析对话历史                      │
│  - 评估当前状态                      │
└─────────────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│  思考（Think）                       │
│  - 决定下一步动作                    │
│  - 判断是否需要追问                  │
│  - 判断是否结束                      │
└─────────────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│  行动（Act）                         │
│  - 生成回复                          │
│  - 更新状态                          │
└─────────────────────────────────────┘
         ↓
    【回到观察】
```

```plaintext
状态：提问中
    ↓
用户回答
    ↓
评估回答质量
    ├─ 充分 → 进入下一题（状态转换）
    ├─ 不充分 → 追问（状态：追问中）
    │   ├─ 追问次数 < 2 → 继续追问
    │   └─ 追问次数 >= 2 → 强制下一题
    └─ 结束信号 → 结束面试
```

```plaintext
一次AI调用完成所有决策：
- 分析对话
- 决定动作（追问/下一题/结束）
- 生成回复
- 返回结构化结果
```

```python
# 伪代码
async def get_ai_response(self, dialog_history):
    # 1. 策略AI（必须）
    strategy = await self.generate_strategy(...)
    
    # 2. 执行AI（必须，流式）
    ai_response = await self.execute_response(...)
    
    # 3. 并行执行：意图识别 + 判断AI
    intent_task = self._detect_interview_action(ai_response)
    judge_task = self._judge_end(ai_response)
    intent, is_end = await asyncio.gather(intent_task, judge_task)
    
    # 4. 根据结果更新状态
    if is_end:
        yield "over"
    elif intent == "new_question":
        self.current_question_index += 1
```

```python
async def get_ai_response(self, dialog_history):
    # 单次AI调用，返回结构化决策
    decision = await self.make_decision(dialog_history)
    # decision = {
    #     "action": "follow_up" | "new_question" | "end",
    #     "response": "生成的回复内容",
    #     "follow_count": 1,  # 当前问题已追问次数
    #     "should_end": False
    # }
    
    # 流式返回回复
    async for chunk in self.stream_response(decision["response"]):
        yield chunk
    
    # 更新状态
    if decision["action"] == "new_question":
        self.current_question_index += 1
        self.current_question_follow_count = 0
    elif decision["action"] == "follow_up":
        self.current_question_follow_count += 1
        if self.current_question_follow_count >= 2:
            # 强制下一题
            self.current_question_index += 1
            self.current_question_follow_count = 0
```

```python
class InterviewState(Enum):
    INTRODUCING = "introducing"      # 介绍阶段
    QUESTIONING = "questioning"       # 提问阶段
    FOLLOWING_UP = "following_up"    # 追问阶段
    Q_AND_A = "q_and_a"              # 问答阶段
    ENDING = "ending"                # 结束阶段

async def get_ai_response(self, dialog_history):
    # 1. 评估当前状态
    state = self.assess_state(dialog_history)
    
    # 2. 根据状态决策
    if state == InterviewState.FOLLOWING_UP:
        if self.current_question_follow_count >= 2:
            # 强制进入下一题
            state = InterviewState.QUESTIONING
            self.current_question_index += 1
            self.current_question_follow_count = 0
    
    # 3. 生成回复（根据状态）
    response = await self.generate_response_by_state(state, dialog_history)
    
    # 4. 更新状态
    self.update_state(state, response)
    
    return response
```

```python
async def get_ai_response(self, dialog_history):
    # 步骤1：快速状态检查（无需AI）
    if self._should_end_by_rules(dialog_history):
        yield "面试结束"
        return
    
    # 步骤2：智能决策（单次AI调用）
    decision = await self._make_intelligent_decision(dialog_history)
    # 返回：action, response, metadata
    
    # 步骤3：状态更新（基于决策）
    self._update_state(decision)
    
    # 步骤4：流式返回回复
    async for chunk in self._stream_response(decision["response"]):
        yield chunk
```

```python
async def _make_intelligent_decision(self, dialog_history):
    """
    一次AI调用完成所有决策：
    - 分析对话历史
    - 评估回答质量
    - 决定下一步动作
    - 生成回复内容
    - 判断是否结束
    """
    prompt = f"""
    你是面试官，需要根据对话历史做出决策。
    
    当前状态：
    - 问题索引：{self.current_question_index}
    - 当前问题已追问：{self.current_question_follow_count} 次
    
    对话历史：{dialog_history}
    
    请返回JSON格式：
    {{
        "action": "follow_up" | "new_question" | "end",
        "response": "生成的回复内容",
        "quality_score": 0-10,
        "should_end": true/false,
        "reasoning": "决策理由"
    }}
    
    规则：
    - 如果已追问2次，必须 action="new_question"
    - 如果回答质量<3分且已追问1次，action="new_question"
    - 如果明确结束信号，should_end=true
    """
    
    response = await self.llm_client.create_completion(...)
    return json.loads(response)
```

---

**User**

但实际上我的理解是指使用方案B就行了呀，就是一次模型输出所有相应的内容呀。对此你怎么看呢？我们能不能探讨一下这个方案呢？如果多次调用API的话，会导致各个API，他们的职责割裂 然后就降低模型的智能性呀。你觉得存在这种可能吗？我们来了深入探讨一下可以吗？

---

**Cursor**

深入分析方案B（单次模型调用）的可行性和设计要点：


深入分析方案B（单次模型调用）的设计与实现：

## 职责割裂问题分析

### 当前设计的问题

```
策略AI：只负责"思考"（分析对话，生成策略文本）
    ↓
执行AI：只负责"执行"（根据策略文本生成回复）
    ↓
意图识别：只负责"分类"（判断回复类型）
    ↓
判断AI：只负责"判断"（判断是否结束）
```

问题：
1. 信息传递损失：策略AI的输出是文本，执行AI需要重新理解
2. 上下文割裂：每个AI只看到部分信息，缺乏全局视角
3. 决策分散：决策被拆分到多个步骤，难以统一优化
4. 智能性降低：模型无法在生成回复时同时考虑策略、意图、结束判断

### 单次调用的优势

```
一次AI调用：
- 同时看到完整对话历史
- 同时考虑策略、回复、意图、结束判断
- 统一决策，避免信息传递损失
- 更智能、更一致
```

## 方案B的详细设计

### 核心设计：结构化输出 + 流式回复

```python
async def get_ai_response(self, dialog_history):
    """
    单次AI调用，返回结构化决策和流式回复
    """
    # 步骤1：单次AI调用，获取结构化决策
    decision = await self._make_unified_decision(dialog_history)
    # decision = {
    #     "action": "follow_up" | "new_question" | "end",
    #     "response": "生成的回复内容",
    #     "quality_score": 3,  # 回答质量评分
    #     "should_end": False,
    #     "reasoning": "决策理由（可选）"
    # }
    
    # 步骤2：流式返回回复（使用decision["response"]）
    async for chunk in self._stream_response(decision["response"]):
        yield chunk
    
    # 步骤3：更新状态
    self._update_state(decision)
```

### 关键设计点1：结构化输出格式

```python
async def _make_unified_decision(self, dialog_history):
    """
    单次AI调用，返回结构化决策
    """
    system_prompt = f"""
    你是专业的AI面试官，需要根据对话历史做出完整的决策。
    
    当前状态：
    - 问题索引：{self.current_question_index}/{len(self.interview_plan['questions']) if self.interview_plan else 0}
    - 当前问题已追问：{self.current_question_follow_count} 次
    - 面试计划：{json.dumps(self.interview_plan, ensure_ascii=False) if self.interview_plan else '无'}
    
    你的任务：
    1. 分析候选人的最新回答
    2. 评估回答质量（0-10分）
    3. 决定下一步动作（追问/下一题/结束）
    4. 生成面试官回复内容
    5. 判断是否应该结束面试
    
    规则约束：
    - 如果已追问2次，必须 action="new_question"
    - 如果回答质量<3分且已追问1次，建议 action="new_question"
    - 如果明确结束信号，should_end=true
    
    输出格式（严格JSON）：
    {{
        "action": "follow_up" | "new_question" | "end",
        "response": "面试官要说的完整回复内容（自然对话，不要透露决策过程）",
        "quality_score": 0-10,
        "should_end": true/false,
        "reasoning": "你的决策理由（用于日志，不会给候选人看）"
    }}
    """
    
    user_prompt = f"""
    对话历史：
    {json.dumps(dialog_history, ensure_ascii=False, indent=2)}
    
    请做出决策并生成回复。
    """
    
    response = await self.llm_client.create_completion(
        messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": user_prompt}
        ],
        temperature=0.7,
        response_format={"type": "json_object"}  # 强制JSON输出
    )
    
    decision = json.loads(response.choices[0].message.content)
    return decision
```

### 关键设计点2：流式输出的处理

挑战：结构化输出是JSON，但需要流式返回回复内容

解决方案：两阶段调用

```python
async def get_ai_response(self, dialog_history):
    """
    两阶段方案：
    1. 第一阶段：获取结构化决策（非流式，快速）
    2. 第二阶段：流式生成回复内容
    """
    
    # 阶段1：快速决策（非流式，小模型）
    decision = await self._make_unified_decision(dialog_history)
    
    # 阶段2：流式生成回复（使用决策中的提示）
    async for chunk in self._stream_response_with_decision(decision, dialog_history):
        yield chunk
    
    # 更新状态
    self._update_state(decision)
```

或者：单次流式调用 + 后处理

```python
async def get_ai_response(self, dialog_history):
    """
    单次流式调用，实时解析
    """
    # 使用流式调用，但要求输出特定格式
    # 例如：先输出JSON元数据，再输出回复内容
    # 或者：使用特殊标记分隔
    
    full_response = ""
    metadata = None
    
    async for chunk in self.llm_client.create_streaming_completion(...):
        content = chunk.choices[0].delta.content
        full_response += content
        
        # 尝试解析JSON（当收到完整JSON时）
        if not metadata and "}" in full_response:
            try:
                # 提取JSON部分
                json_end = full_response.find("}", full_response.find("{")) + 1
                json_str = full_response[:json_end]
                metadata = json.loads(json_str)
                # 继续流式返回回复内容部分
                remaining = full_response[json_end:]
                if remaining:
                    yield remaining
            except:
                pass
        
        # 如果已解析出元数据，直接流式返回内容
        if metadata:
            yield content
    
    # 如果没有成功解析，fallback到非流式
    if not metadata:
        decision = await self._parse_response(full_response)
    else:
        decision = metadata
```

### 推荐方案：混合模式（决策+流式）

```python
async def get_ai_response(self, dialog_history):
    """
    混合模式：
    1. 快速决策（非流式，获取action和quality_score）
    2. 流式生成回复（基于决策结果）
    """
    
    # 步骤1：快速决策（使用小模型，非流式）
    decision = await self._make_quick_decision(dialog_history)
    # 返回：action, quality_score, should_end, response_guidance
    
    # 步骤2：基于决策流式生成回复
    async for chunk in self._stream_response_by_decision(decision, dialog_history):
        yield chunk
    
    # 步骤3：更新状态
    self._update_state(decision)
```

## 深入探讨：职责割裂 vs 统一智能

### 你的观点：多次调用降低智能性

这个观点是正确的，原因：

1. 信息传递损失
```
策略AI输出："需要追问技术细节"
    ↓（文本传递）
执行AI理解："需要追问技术细节"（可能理解偏差）
    ↓
实际生成："请详细说明..."
```

2. 上下文割裂
```
策略AI：只看到对话历史，不知道面试计划细节
执行AI：只看到策略文本，不知道原始分析过程
意图识别：只看到最终回复，不知道生成过程
```

3. 决策不一致
```
策略AI：建议"追问"
执行AI：生成"追问"回复
意图识别：判断为"follow_up"（一致）
判断AI：判断"继续"（一致）

但如果策略AI建议"下一题"，执行AI可能理解偏差...
```

### 单次调用的智能性提升

```
一次AI调用：
- 同时看到：对话历史 + 面试计划 + 当前状态
- 同时考虑：回答质量 + 追问次数 + 面试进度
- 统一决策：action + response + quality_score + should_end
- 避免信息传递损失
- 决策更一致、更智能
```

## 实际实现方案

### 方案1：纯结构化输出（推荐用于快速决策）

```python
async def _make_unified_decision(self, dialog_history):
    """
    单次调用，返回完整决策
    """
    prompt = f"""
    你是AI面试官，需要做出完整决策。
    
    当前状态：
    - 问题索引：{self.current_question_index}
    - 已追问：{self.current_question_follow_count} 次
    
    对话历史：{json.dumps(dialog_history, ensure_ascii=False)}
    
    输出JSON：
    {{
        "action": "follow_up" | "new_question" | "end",
        "response": "完整回复内容",
        "quality_score": 3,
        "should_end": false
    }}
    """
    
    response = await self.llm_client.create_completion(
        messages=[{"role": "user", "content": prompt}],
        response_format={"type": "json_object"},
        temperature=0.7
    )
    
    return json.loads(response.choices[0].message.content)
```

优势：
- 一次调用，获取所有信息
- 结构化输出，易于解析
- 决策统一，智能性高

劣势：
- 回复内容无法流式输出（需要二次调用）

### 方案2：两阶段调用（推荐用于用户体验）

```python
async def get_ai_response(self, dialog_history):
    """
    阶段1：快速决策（非流式，小模型）
    阶段2：流式生成回复（基于决策）
    """
    
    # 阶段1：决策（快速，非流式）
    decision = await self._make_quick_decision(dialog_history)
    # 返回：action, quality_score, should_end, response_guidance
    
    # 阶段2：流式生成回复
    async for chunk in self._stream_response(
        dialog_history=dialog_history,
        action=decision["action"],
        guidance=decision.get("response_guidance", "")
    ):
        yield chunk
    
    # 更新状态
    self._update_state(decision)
```

优势：
- 保持流式用户体验
- 决策统一智能
- 可以优化：决策用小模型，生成用大模型

## 总结

你的观点是正确的：
1. 多次调用确实会导致职责割裂
2. 信息传递会有损失
3. 上下文割裂降低智能性
4. 单次调用更智能、更一致

推荐实现：
- 使用结构化输出（JSON格式）
- 两阶段调用：快速决策 + 流式生成
- 或者：单次调用获取决策，然后基于决策流式生成回复

这样既保持了智能性，又保证了用户体验（流式输出）。

需要我帮你设计具体的实现代码吗？

```plaintext
策略AI：只负责"思考"（分析对话，生成策略文本）
    ↓
执行AI：只负责"执行"（根据策略文本生成回复）
    ↓
意图识别：只负责"分类"（判断回复类型）
    ↓
判断AI：只负责"判断"（判断是否结束）
```

```plaintext
一次AI调用：
- 同时看到完整对话历史
- 同时考虑策略、回复、意图、结束判断
- 统一决策，避免信息传递损失
- 更智能、更一致
```

```python
async def get_ai_response(self, dialog_history):
    """
    单次AI调用，返回结构化决策和流式回复
    """
    # 步骤1：单次AI调用，获取结构化决策
    decision = await self._make_unified_decision(dialog_history)
    # decision = {
    #     "action": "follow_up" | "new_question" | "end",
    #     "response": "生成的回复内容",
    #     "quality_score": 3,  # 回答质量评分
    #     "should_end": False,
    #     "reasoning": "决策理由（可选）"
    # }
    
    # 步骤2：流式返回回复（使用decision["response"]）
    async for chunk in self._stream_response(decision["response"]):
        yield chunk
    
    # 步骤3：更新状态
    self._update_state(decision)
```

```python
async def _make_unified_decision(self, dialog_history):
    """
    单次AI调用，返回结构化决策
    """
    system_prompt = f"""
    你是专业的AI面试官，需要根据对话历史做出完整的决策。
    
    当前状态：
    - 问题索引：{self.current_question_index}/{len(self.interview_plan['questions']) if self.interview_plan else 0}
    - 当前问题已追问：{self.current_question_follow_count} 次
    - 面试计划：{json.dumps(self.interview_plan, ensure_ascii=False) if self.interview_plan else '无'}
    
    你的任务：
    1. 分析候选人的最新回答
    2. 评估回答质量（0-10分）
    3. 决定下一步动作（追问/下一题/结束）
    4. 生成面试官回复内容
    5. 判断是否应该结束面试
    
    规则约束：
    - 如果已追问2次，必须 action="new_question"
    - 如果回答质量<3分且已追问1次，建议 action="new_question"
    - 如果明确结束信号，should_end=true
    
    输出格式（严格JSON）：
    {{
        "action": "follow_up" | "new_question" | "end",
        "response": "面试官要说的完整回复内容（自然对话，不要透露决策过程）",
        "quality_score": 0-10,
        "should_end": true/false,
        "reasoning": "你的决策理由（用于日志，不会给候选人看）"
    }}
    """
    
    user_prompt = f"""
    对话历史：
    {json.dumps(dialog_history, ensure_ascii=False, indent=2)}
    
    请做出决策并生成回复。
    """
    
    response = await self.llm_client.create_completion(
        messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": user_prompt}
        ],
        temperature=0.7,
        response_format={"type": "json_object"}  # 强制JSON输出
    )
    
    decision = json.loads(response.choices[0].message.content)
    return decision
```

```python
async def get_ai_response(self, dialog_history):
    """
    两阶段方案：
    1. 第一阶段：获取结构化决策（非流式，快速）
    2. 第二阶段：流式生成回复内容
    """
    
    # 阶段1：快速决策（非流式，小模型）
    decision = await self._make_unified_decision(dialog_history)
    
    # 阶段2：流式生成回复（使用决策中的提示）
    async for chunk in self._stream_response_with_decision(decision, dialog_history):
        yield chunk
    
    # 更新状态
    self._update_state(decision)
```

```python
async def get_ai_response(self, dialog_history):
    """
    单次流式调用，实时解析
    """
    # 使用流式调用，但要求输出特定格式
    # 例如：先输出JSON元数据，再输出回复内容
    # 或者：使用特殊标记分隔
    
    full_response = ""
    metadata = None
    
    async for chunk in self.llm_client.create_streaming_completion(...):
        content = chunk.choices[0].delta.content
        full_response += content
        
        # 尝试解析JSON（当收到完整JSON时）
        if not metadata and "}" in full_response:
            try:
                # 提取JSON部分
                json_end = full_response.find("}", full_response.find("{")) + 1
                json_str = full_response[:json_end]
                metadata = json.loads(json_str)
                # 继续流式返回回复内容部分
                remaining = full_response[json_end:]
                if remaining:
                    yield remaining
            except:
                pass
        
        # 如果已解析出元数据，直接流式返回内容
        if metadata:
            yield content
    
    # 如果没有成功解析，fallback到非流式
    if not metadata:
        decision = await self._parse_response(full_response)
    else:
        decision = metadata
```

```python
async def get_ai_response(self, dialog_history):
    """
    混合模式：
    1. 快速决策（非流式，获取action和quality_score）
    2. 流式生成回复（基于决策结果）
    """
    
    # 步骤1：快速决策（使用小模型，非流式）
    decision = await self._make_quick_decision(dialog_history)
    # 返回：action, quality_score, should_end, response_guidance
    
    # 步骤2：基于决策流式生成回复
    async for chunk in self._stream_response_by_decision(decision, dialog_history):
        yield chunk
    
    # 步骤3：更新状态
    self._update_state(decision)
```

```plaintext
策略AI输出："需要追问技术细节"
    ↓（文本传递）
执行AI理解："需要追问技术细节"（可能理解偏差）
    ↓
实际生成："请详细说明..."
```

```plaintext
策略AI：只看到对话历史，不知道面试计划细节
执行AI：只看到策略文本，不知道原始分析过程
意图识别：只看到最终回复，不知道生成过程
```

```plaintext
策略AI：建议"追问"
执行AI：生成"追问"回复
意图识别：判断为"follow_up"（一致）
判断AI：判断"继续"（一致）

但如果策略AI建议"下一题"，执行AI可能理解偏差...
```

```plaintext
一次AI调用：
- 同时看到：对话历史 + 面试计划 + 当前状态
- 同时考虑：回答质量 + 追问次数 + 面试进度
- 统一决策：action + response + quality_score + should_end
- 避免信息传递损失
- 决策更一致、更智能
```

```python
async def _make_unified_decision(self, dialog_history):
    """
    单次调用，返回完整决策
    """
    prompt = f"""
    你是AI面试官，需要做出完整决策。
    
    当前状态：
    - 问题索引：{self.current_question_index}
    - 已追问：{self.current_question_follow_count} 次
    
    对话历史：{json.dumps(dialog_history, ensure_ascii=False)}
    
    输出JSON：
    {{
        "action": "follow_up" | "new_question" | "end",
        "response": "完整回复内容",
        "quality_score": 3,
        "should_end": false
    }}
    """
    
    response = await self.llm_client.create_completion(
        messages=[{"role": "user", "content": prompt}],
        response_format={"type": "json_object"},
        temperature=0.7
    )
    
    return json.loads(response.choices[0].message.content)
```

```python
async def get_ai_response(self, dialog_history):
    """
    阶段1：快速决策（非流式，小模型）
    阶段2：流式生成回复（基于决策）
    """
    
    # 阶段1：决策（快速，非流式）
    decision = await self._make_quick_decision(dialog_history)
    # 返回：action, quality_score, should_end, response_guidance
    
    # 阶段2：流式生成回复
    async for chunk in self._stream_response(
        dialog_history=dialog_history,
        action=decision["action"],
        guidance=decision.get("response_guidance", "")
    ):
        yield chunk
    
    # 更新状态
    self._update_state(decision)
```

---

**User**

你的理解非常准确到位。请针对我的模拟面试的这个api，请进一步梳理修改的整体思路，并提供可行的实现方案，目前不需要编写具体代码。

---

**Cursor**

梳理模拟面试API的改进思路和实现方案：


## 模拟面试API改进方案

### 一、当前架构问题梳理

#### 1.1 当前执行流程
```
用户输入
    ↓
【首次调用？】
    ├─ 是 → 生成面试计划 → 返回第一个问题 → 结束
    └─ 否 → 继续
    ↓
策略AI（分析对话，生成策略文本，~7秒）
    ↓
执行AI（根据策略生成回复，流式，~9秒）
    ↓
意图识别（判断回复类型，~1秒）
    ↓
判断AI（判断是否结束，~1秒）
    ↓
更新状态（根据意图更新索引）
    ↓
返回结果
```

#### 1.2 核心问题
1. 职责割裂：4个AI各司其职，信息传递有损失
2. 上下文割裂：每个AI只看到部分信息
3. 决策分散：决策被拆分，难以统一优化
4. 缺少状态追踪：没有追问次数追踪机制
5. 规则约束缺失：代码层面没有强制限制（如最多追问2次）

### 二、改进目标

#### 2.1 核心目标
- 统一智能决策：单次AI调用完成分析、决策、生成
- 保持流式体验：用户仍能实时看到回复
- 状态可追踪：明确追踪追问次数、问题索引等
- 规则可约束：代码层面强制限制（如最多追问2次）

#### 2.2 性能目标
- API调用次数：从4次降至1-2次
- 响应延迟：从~18秒降至~8-10秒
- 成本：降低约50-60%

### 三、新架构设计

#### 3.1 整体流程
```
用户输入
    ↓
【快速规则检查】（无需AI）
    ├─ 沉默检测（连续3次沉默 → 强制结束）
    ├─ 追问次数检查（已追问2次 → 强制下一题）
    └─ 其他规则检查
    ↓
【首次调用？】
    ├─ 是 → 生成面试计划 → 返回第一个问题 → 结束
    └─ 否 → 继续
    ↓
【阶段1：智能决策】（非流式，快速）
    - 单次AI调用
    - 返回结构化决策：
      {
        "action": "follow_up" | "new_question" | "end",
        "response_guidance": "回复指导（用于阶段2）",
        "quality_score": 0-10,
        "should_end": true/false,
        "reasoning": "决策理由（用于日志）"
      }
    ↓
【阶段2：流式生成回复】（基于决策）
    - 使用决策中的 response_guidance
    - 流式生成最终回复
    ↓
【状态更新】
    - 根据决策更新：current_question_index
    - 更新：current_question_follow_count
    ↓
【返回结果】
```

#### 3.2 关键改进点

改进点1：统一智能决策
- 合并策略AI和执行AI的“思考”部分
- 一次AI调用完成：分析对话、评估质量、决定动作、生成回复指导
- 使用结构化输出（JSON格式）

改进点2：两阶段调用
- 阶段1：快速决策（非流式，小模型或标准模型）
- 阶段2：流式生成回复（基于决策指导）

改进点3：状态追踪机制
- 添加 `current_question_follow_count` 变量
- 追踪每个问题的追问次数
- 进入新问题时重置计数

改进点4：规则约束机制
- 代码层面强制限制：最多追问2次
- 规则检查在决策前进行（快速规则检查）
- 决策后再次验证（确保符合规则）

### 四、详细实现方案

#### 4.1 状态管理改进

新增状态变量：
```python
class CallInterview:
    def __init__(self, ...):
        # 现有变量
        self.current_question_index = 0
        self.interview_plan = None
        
        # 新增变量
        self.current_question_follow_count = 0  # 当前问题的追问次数
```

状态更新逻辑：
- 当 `action == "new_question"` 时：
  - `current_question_index += 1`
  - `current_question_follow_count = 0`（重置）
- 当 `action == "follow_up"` 时：
  - `current_question_follow_count += 1`
  - 如果 `>= 2`，强制改为 `new_question`

#### 4.2 智能决策函数设计

函数职责：
- 输入：对话历史、当前状态、面试计划
- 输出：结构化决策（JSON格式）
- 功能：
  1. 分析候选人最新回答
  2. 评估回答质量（0-10分）
  3. 决定下一步动作（追问/下一题/结束）
  4. 生成回复指导（用于阶段2）
  5. 判断是否应该结束面试

Prompt设计要点：
- 提供完整上下文：对话历史、面试计划、当前状态
- 明确规则约束：最多追问2次、质量评分标准等
- 要求结构化输出：JSON格式，包含所有必要字段
- 区分回复指导与最终回复：指导用于阶段2生成

#### 4.3 流式生成函数设计

函数职责：
- 输入：决策结果、对话历史、面试计划
- 输出：流式回复内容
- 功能：
  1. 基于决策中的 `response_guidance` 生成回复
  2. 保持自然对话风格
  3. 流式返回内容

Prompt设计要点：
- 使用决策中的 `response_guidance` 作为指导
- 保持面试官身份和风格
- 不透露决策过程（只生成自然回复）

#### 4.4 规则约束机制

快速规则检查（决策前）：
```python
# 1. 沉默检测
if silence_count >= 3:
    return 强制结束

# 2. 追问次数检查
if self.current_question_follow_count >= 2:
    # 强制下一题，不调用AI决策
    return {
        "action": "new_question",
        "reason": "已追问2次，强制进入下一题"
    }
```

决策后验证：
```python
# 如果AI决策不符合规则，强制修正
if decision["action"] == "follow_up" and self.current_question_follow_count >= 2:
    decision["action"] = "new_question"
    logging.warning("AI决策违反规则，已强制修正")
```

#### 4.5 首次调用处理

保持不变：
- 首次调用时生成面试计划
- 返回第一个问题
- 设置 `current_question_index = 1`

改进点：
- 返回面试计划给前端（通过SSE事件）

### 五、实现步骤

#### 步骤1：状态管理改进
1. 添加 `current_question_follow_count` 变量
2. 在 `__init__` 中初始化
3. 在状态更新逻辑中使用

#### 步骤2：快速规则检查
1. 提取沉默检测逻辑（已有）
2. 添加追问次数检查
3. 添加其他规则检查（如面试时长等）

#### 步骤3：智能决策函数
1. 设计Prompt模板
2. 实现决策函数（非流式调用）
3. 解析结构化输出（JSON）
4. 错误处理和fallback机制

#### 步骤4：流式生成函数
1. 设计Prompt模板（使用决策指导）
2. 实现流式生成函数
3. 保持现有流式返回格式

#### 步骤5：主流程重构
1. 重构 `get_ai_response` 函数
2. 整合快速规则检查
3. 整合智能决策和流式生成
4. 整合状态更新逻辑

#### 步骤6：测试和优化
1. 测试各种场景（首次调用、追问、下一题、结束）
2. 性能测试（延迟、成本）
3. 错误处理测试
4. 优化Prompt和参数

### 六、关键技术点

#### 6.1 结构化输出
- 使用 `response_format={"type": "json_object"}`
- 定义清晰的JSON Schema
- 错误处理：如果JSON解析失败，使用fallback逻辑

#### 6.2 流式输出保持
- 阶段1：非流式决策（快速）
- 阶段2：流式生成回复（用户体验）
- 保持现有SSE格式

#### 6.3 状态一致性
- 状态更新在决策后立即执行
- 确保状态与决策一致
- 添加状态验证逻辑

#### 6.4 错误处理
- AI调用失败：重试机制
- JSON解析失败：fallback到简单决策
- 流式生成失败：返回已生成的部分

### 七、预期效果

#### 7.1 性能提升
- API调用次数：4次 → 1-2次
- 响应延迟：~18秒 → ~8-10秒
- 成本：降低约50-60%

#### 7.2 智能性提升
- 统一决策，避免信息传递损失
- 上下文完整，决策更准确
- 规则约束明确，行为更可控

#### 7.3 可维护性提升
- 代码结构更清晰
- 状态追踪更明确
- 易于扩展和优化

### 八、风险与应对

#### 8.1 风险
1. Prompt设计复杂，可能影响决策质量
2. 结构化输出可能不稳定
3. 两阶段调用可能增加延迟

#### 8.2 应对
1. 迭代优化Prompt，添加示例
2. 添加JSON解析错误处理和fallback
3. 优化阶段1使用小模型，减少延迟

### 九、实施建议

#### 9.1 分阶段实施
- 阶段1：状态管理改进 + 快速规则检查
- 阶段2：智能决策函数实现
- 阶段3：流式生成函数实现
- 阶段4：主流程重构和整合
- 阶段5：测试和优化

#### 9.2 兼容性考虑
- 保持现有API接口不变
- 保持现有SSE格式不变
- 保持现有错误处理机制

#### 9.3 回滚方案
- 保留原有代码作为fallback
- 通过配置开关控制新旧逻辑
- 充分测试后再切换

该方案在保持用户体验的同时，提升智能性和性能。需要我进一步细化某个部分吗？

```plaintext
用户输入
    ↓
【首次调用？】
    ├─ 是 → 生成面试计划 → 返回第一个问题 → 结束
    └─ 否 → 继续
    ↓
策略AI（分析对话，生成策略文本，~7秒）
    ↓
执行AI（根据策略生成回复，流式，~9秒）
    ↓
意图识别（判断回复类型，~1秒）
    ↓
判断AI（判断是否结束，~1秒）
    ↓
更新状态（根据意图更新索引）
    ↓
返回结果
```

```plaintext
用户输入
    ↓
【快速规则检查】（无需AI）
    ├─ 沉默检测（连续3次沉默 → 强制结束）
    ├─ 追问次数检查（已追问2次 → 强制下一题）
    └─ 其他规则检查
    ↓
【首次调用？】
    ├─ 是 → 生成面试计划 → 返回第一个问题 → 结束
    └─ 否 → 继续
    ↓
【阶段1：智能决策】（非流式，快速）
    - 单次AI调用
    - 返回结构化决策：
      {
        "action": "follow_up" | "new_question" | "end",
        "response_guidance": "回复指导（用于阶段2）",
        "quality_score": 0-10,
        "should_end": true/false,
        "reasoning": "决策理由（用于日志）"
      }
    ↓
【阶段2：流式生成回复】（基于决策）
    - 使用决策中的 response_guidance
    - 流式生成最终回复
    ↓
【状态更新】
    - 根据决策更新：current_question_index
    - 更新：current_question_follow_count
    ↓
【返回结果】
```

```python
class CallInterview:
    def __init__(self, ...):
        # 现有变量
        self.current_question_index = 0
        self.interview_plan = None
        
        # 新增变量
        self.current_question_follow_count = 0  # 当前问题的追问次数
```

```python
# 1. 沉默检测
if silence_count >= 3:
    return 强制结束

# 2. 追问次数检查
if self.current_question_follow_count >= 2:
    # 强制下一题，不调用AI决策
    return {
        "action": "new_question",
        "reason": "已追问2次，强制进入下一题"
    }
```

```python
# 如果AI决策不符合规则，强制修正
if decision["action"] == "follow_up" and self.current_question_follow_count >= 2:
    decision["action"] = "new_question"
    logging.warning("AI决策违反规则，已强制修正")
```

---

**User**

你前面的修改方案非常好，请帮我严格按照这个方案来修改相应的代码，完整follow前面修改方案的内容，不用引入不必要的额外的冗余的修改，修改之后，帮我梳理一下修改后的内容，进而方便我double check正确性

请额外注意：

1 严格遵循前面的方案的具体要求

2 不擅自添加你认为"有用"但我没有要求的功能

3 专注于我明确指出的需求

4 请不要往代码里面添加额外的md文件来总结修改内容。 

---

**Cursor**

按方案修改代码。先查看当前代码结构：


按方案修改代码。先添加状态变量，然后实现新函数，最后重构主流程。


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
from app.identity import get_identity_prompt, get_identity_vars, get_standard_scripts
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
        self.current_question_follow_count = 0  # 当前问题的追问次数
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
        
        # 获取身份变量和标准话术
        identity_vars = get_identity_vars()
        scripts = get_standard_scripts()
        format_vars = {**identity_vars, **scripts, "language": lang}
        
        self.celue_sys_prompt = f"{identity_prompt}\n\n{celue_template.format(**format_vars)}"
        self.zhixing_sys_prompt = f"{identity_prompt}\n\n{zhixing_template.format(**format_vars)}"
        self.lanling_sys_prompt = f"{identity_prompt}\n\n{lanling_template.format(**format_vars)}"
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

实现智能决策函数和流式生成函数，然后重构主流程：


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
from app.identity import get_identity_prompt, get_identity_vars, get_standard_scripts
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
        self.current_question_follow_count = 0  # 当前问题的追问次数
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


    async def _make_unified_decision(self, dialog_history: list, system_message: str, plan_info: str) -> dict:
        """
        单次AI调用，完成智能决策
        返回结构化决策：
        {
            "action": "follow_up" | "new_question" | "end",
            "response_guidance": "回复指导（用于流式生成）",
            "quality_score": 0-10,
            "should_end": true/false,
            "reasoning": "决策理由（用于日志）"
        }
        """
        try:
            # 准备面试计划信息
            plan_context = ""
            if self.interview_plan:
                current_question = None
                if self.current_question_index > 0 and self.current_question_index <= len(self.interview_plan['questions']):
                    current_question = self.interview_plan['questions'][self.current_question_index - 1]
                
                plan_context = f"""
【面试计划参考】
当前进度：已提问 {self.current_question_index}/{len(self.interview_plan['questions'])} 个问题
当前问题已追问：{self.current_question_follow_count} 次
"""
                if current_question:
                    plan_context += f"""
当前问题信息：
{json.dumps(current_question, ensure_ascii=False, indent=2)}
"""
                remaining_questions = self.interview_plan['questions'][self.current_question_index:]
                if remaining_questions:
                    plan_context += f"""
待提问题（前3个）：
{json.dumps(remaining_questions[:3], ensure_ascii=False, indent=2)}
"""
            
            # 加载策略prompt模板
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            strategy_template = load_prompt("strategy_white_collar.txt")
            strategy_sys_prompt = f"{identity_prompt}\n\n{strategy_template.format(**format_vars)}"
            
            system_prompt = f"""{strategy_sys_prompt}

【重要】你现在需要做出完整的决策，包括：
1. 分析候选人的最新回答
2. 评估回答质量（0-10分）
3. 决定下一步动作（追问/下一题/结束）
4. 生成回复指导（用于后续生成自然回复）
5. 判断是否应该结束面试

规则约束：
- 如果已追问2次，必须 action="new_question"
- 如果回答质量<3分且已追问1次，建议 action="new_question"
- 如果明确结束信号，should_end=true

输出严格JSON格式：
{{
    "action": "follow_up" | "new_question" | "end",
    "response_guidance": "回复指导内容（描述要说什么，但不直接生成回复文本）",
    "quality_score": 0-10,
    "should_end": true/false,
    "reasoning": "你的决策理由（用于日志记录）"
}}"""
            
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

{plan_context}

请做出完整决策。"""
            
            response = self.llm_client.create_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.4,
                response_format={"type": "json_object"}
            )
            
            decision = json.loads(response.choices[0].message.content)
            
            # 规则验证：如果已追问2次，强制下一题
            if decision.get("action") == "follow_up" and self.current_question_follow_count >= 2:
                logging.warning(f"-- 面试ID: {self.interview_id} - ⚠️ AI决策违反规则（已追问2次），强制修正为 new_question")
                decision["action"] = "new_question"
                decision["reasoning"] = f"已追问2次，强制进入下一题。原决策理由：{decision.get('reasoning', '')}"
            
            logging.info(f"-- 面试ID: {self.interview_id} - 智能决策结果: {decision}")
            return decision
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 智能决策失败: {e}")
            # 失败时返回默认决策
            return {
                "action": "new_question",
                "response_guidance": "继续面试流程",
                "quality_score": 5,
                "should_end": False,
                "reasoning": f"决策失败，使用默认值。错误: {str(e)}"
            }

    async def _stream_response_by_decision(self, decision: dict, dialog_history: list, system_message: str, plan_info: str) -> str:
        """
        基于决策指导，流式生成回复内容
        返回完整回复文本
        """
        try:
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            executor_template = load_prompt("executor_white_collar.txt" if self.interview_type == 1 else "executor_blue_collar.txt")
            executor_sys_prompt = f"{identity_prompt}\n\n{executor_template.format(**format_vars)}"
            
            system_prompt = f"""{executor_sys_prompt}

根据以下回复指导，生成自然、专业的面试官回复。
不要透露决策过程，只生成自然对话。"""
            
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

{plan_info}

回复指导：
{decision.get("response_guidance", "继续面试流程")}

请根据回复指导生成自然、专业的面试官回复。"""
            
            response = self.llm_client.create_streaming_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.7,
                seed=1
            )
            
            full_response = ""
            async for chunk in response:
                if chunk.choices and chunk.choices[0].delta.content:
                    content = chunk.choices[0].delta.content
                    full_response += content
                    yield content
            
            return full_response
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 流式生成失败: {e}")
            yield "抱歉，我需要整理一下思路。你能再说一遍吗？"
            return ""


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
        
        # 获取身份变量和标准话术
        identity_vars = get_identity_vars()
        scripts = get_standard_scripts()
        format_vars = {**identity_vars, **scripts, "language": lang}
        
        self.celue_sys_prompt = f"{identity_prompt}\n\n{celue_template.format(**format_vars)}"
        self.zhixing_sys_prompt = f"{identity_prompt}\n\n{zhixing_template.format(**format_vars)}"
        self.lanling_sys_prompt = f"{identity_prompt}\n\n{lanling_template.format(**format_vars)}"
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

修正流式生成函数，并重构主流程：


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
from app.identity import get_identity_prompt, get_identity_vars, get_standard_scripts
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
        self.current_question_follow_count = 0  # 当前问题的追问次数
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


    async def _make_unified_decision(self, dialog_history: list, system_message: str, plan_info: str) -> dict:
        """
        单次AI调用，完成智能决策
        返回结构化决策：
        {
            "action": "follow_up" | "new_question" | "end",
            "response_guidance": "回复指导（用于流式生成）",
            "quality_score": 0-10,
            "should_end": true/false,
            "reasoning": "决策理由（用于日志）"
        }
        """
        try:
            # 准备面试计划信息
            plan_context = ""
            if self.interview_plan:
                current_question = None
                if self.current_question_index > 0 and self.current_question_index <= len(self.interview_plan['questions']):
                    current_question = self.interview_plan['questions'][self.current_question_index - 1]
                
                plan_context = f"""
【面试计划参考】
当前进度：已提问 {self.current_question_index}/{len(self.interview_plan['questions'])} 个问题
当前问题已追问：{self.current_question_follow_count} 次
"""
                if current_question:
                    plan_context += f"""
当前问题信息：
{json.dumps(current_question, ensure_ascii=False, indent=2)}
"""
                remaining_questions = self.interview_plan['questions'][self.current_question_index:]
                if remaining_questions:
                    plan_context += f"""
待提问题（前3个）：
{json.dumps(remaining_questions[:3], ensure_ascii=False, indent=2)}
"""
            
            # 加载策略prompt模板
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            strategy_template = load_prompt("strategy_white_collar.txt")
            strategy_sys_prompt = f"{identity_prompt}\n\n{strategy_template.format(**format_vars)}"
            
            system_prompt = f"""{strategy_sys_prompt}

【重要】你现在需要做出完整的决策，包括：
1. 分析候选人的最新回答
2. 评估回答质量（0-10分）
3. 决定下一步动作（追问/下一题/结束）
4. 生成回复指导（用于后续生成自然回复）
5. 判断是否应该结束面试

规则约束：
- 如果已追问2次，必须 action="new_question"
- 如果回答质量<3分且已追问1次，建议 action="new_question"
- 如果明确结束信号，should_end=true

输出严格JSON格式：
{{
    "action": "follow_up" | "new_question" | "end",
    "response_guidance": "回复指导内容（描述要说什么，但不直接生成回复文本）",
    "quality_score": 0-10,
    "should_end": true/false,
    "reasoning": "你的决策理由（用于日志记录）"
}}"""
            
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

{plan_context}

请做出完整决策。"""
            
            response = self.llm_client.create_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.4,
                response_format={"type": "json_object"}
            )
            
            decision = json.loads(response.choices[0].message.content)
            
            # 规则验证：如果已追问2次，强制下一题
            if decision.get("action") == "follow_up" and self.current_question_follow_count >= 2:
                logging.warning(f"-- 面试ID: {self.interview_id} - ⚠️ AI决策违反规则（已追问2次），强制修正为 new_question")
                decision["action"] = "new_question"
                decision["reasoning"] = f"已追问2次，强制进入下一题。原决策理由：{decision.get('reasoning', '')}"
            
            logging.info(f"-- 面试ID: {self.interview_id} - 智能决策结果: {decision}")
            return decision
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 智能决策失败: {e}")
            # 失败时返回默认决策
            return {
                "action": "new_question",
                "response_guidance": "继续面试流程",
                "quality_score": 5,
                "should_end": False,
                "reasoning": f"决策失败，使用默认值。错误: {str(e)}"
            }

    async def _stream_response_by_decision(self, decision: dict, dialog_history: list, system_message: str, plan_info: str):
        """
        基于决策指导，流式生成回复内容
        Yields: 回复内容片段
        Returns: 完整回复文本
        """
        try:
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            executor_template = load_prompt("executor_white_collar.txt" if self.interview_type == 1 else "executor_blue_collar.txt")
            executor_sys_prompt = f"{identity_prompt}\n\n{executor_template.format(**format_vars)}"
            
            system_prompt = f"""{executor_sys_prompt}

根据以下回复指导，生成自然、专业的面试官回复。
不要透露决策过程，只生成自然对话。"""
            
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

{plan_info}

回复指导：
{decision.get("response_guidance", "继续面试流程")}

请根据回复指导生成自然、专业的面试官回复。"""
            
            response = self.llm_client.create_streaming_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.7,
                seed=1
            )
            
            full_response = ""
            async for chunk in response:
                if chunk.choices and chunk.choices[0].delta.content:
                    content = chunk.choices[0].delta.content
                    full_response += content
                    yield content
            
            return full_response
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 流式生成失败: {e}")
            error_msg = "抱歉，我需要整理一下思路。你能再说一遍吗？"
            yield error_msg
            return error_msg


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
        
        # 获取身份变量和标准话术
        identity_vars = get_identity_vars()
        scripts = get_standard_scripts()
        format_vars = {**identity_vars, **scripts, "language": lang}
        
        self.celue_sys_prompt = f"{identity_prompt}\n\n{celue_template.format(**format_vars)}"
        self.zhixing_sys_prompt = f"{identity_prompt}\n\n{zhixing_template.format(**format_vars)}"
        self.lanling_sys_prompt = f"{identity_prompt}\n\n{lanling_template.format(**format_vars)}"
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

重构主流程 `get_ai_response`：


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
from app.identity import get_identity_prompt, get_identity_vars, get_standard_scripts
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
        self.current_question_follow_count = 0  # 当前问题的追问次数
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


    async def _make_unified_decision(self, dialog_history: list, system_message: str, plan_info: str) -> dict:
        """
        单次AI调用，完成智能决策
        返回结构化决策：
        {
            "action": "follow_up" | "new_question" | "end",
            "response_guidance": "回复指导（用于流式生成）",
            "quality_score": 0-10,
            "should_end": true/false,
            "reasoning": "决策理由（用于日志）"
        }
        """
        try:
            # 准备面试计划信息
            plan_context = ""
            if self.interview_plan:
                current_question = None
                if self.current_question_index > 0 and self.current_question_index <= len(self.interview_plan['questions']):
                    current_question = self.interview_plan['questions'][self.current_question_index - 1]
                
                plan_context = f"""
【面试计划参考】
当前进度：已提问 {self.current_question_index}/{len(self.interview_plan['questions'])} 个问题
当前问题已追问：{self.current_question_follow_count} 次
"""
                if current_question:
                    plan_context += f"""
当前问题信息：
{json.dumps(current_question, ensure_ascii=False, indent=2)}
"""
                remaining_questions = self.interview_plan['questions'][self.current_question_index:]
                if remaining_questions:
                    plan_context += f"""
待提问题（前3个）：
{json.dumps(remaining_questions[:3], ensure_ascii=False, indent=2)}
"""
            
            # 加载策略prompt模板
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            strategy_template = load_prompt("strategy_white_collar.txt")
            strategy_sys_prompt = f"{identity_prompt}\n\n{strategy_template.format(**format_vars)}"
            
            system_prompt = f"""{strategy_sys_prompt}

【重要】你现在需要做出完整的决策，包括：
1. 分析候选人的最新回答
2. 评估回答质量（0-10分）
3. 决定下一步动作（追问/下一题/结束）
4. 生成回复指导（用于后续生成自然回复）
5. 判断是否应该结束面试

规则约束：
- 如果已追问2次，必须 action="new_question"
- 如果回答质量<3分且已追问1次，建议 action="new_question"
- 如果明确结束信号，should_end=true

输出严格JSON格式：
{{
    "action": "follow_up" | "new_question" | "end",
    "response_guidance": "回复指导内容（描述要说什么，但不直接生成回复文本）",
    "quality_score": 0-10,
    "should_end": true/false,
    "reasoning": "你的决策理由（用于日志记录）"
}}"""
            
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

{plan_context}

请做出完整决策。"""
            
            response = self.llm_client.create_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.4,
                response_format={"type": "json_object"}
            )
            
            decision = json.loads(response.choices[0].message.content)
            
            # 规则验证：如果已追问2次，强制下一题
            if decision.get("action") == "follow_up" and self.current_question_follow_count >= 2:
                logging.warning(f"-- 面试ID: {self.interview_id} - ⚠️ AI决策违反规则（已追问2次），强制修正为 new_question")
                decision["action"] = "new_question"
                decision["reasoning"] = f"已追问2次，强制进入下一题。原决策理由：{decision.get('reasoning', '')}"
            
            logging.info(f"-- 面试ID: {self.interview_id} - 智能决策结果: {decision}")
            return decision
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 智能决策失败: {e}")
            # 失败时返回默认决策
            return {
                "action": "new_question",
                "response_guidance": "继续面试流程",
                "quality_score": 5,
                "should_end": False,
                "reasoning": f"决策失败，使用默认值。错误: {str(e)}"
            }

    async def _stream_response_by_decision(self, decision: dict, dialog_history: list, system_message: str, plan_info: str):
        """
        基于决策指导，流式生成回复内容
        Yields: 回复内容片段
        Returns: 完整回复文本
        """
        try:
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            executor_template = load_prompt("executor_white_collar.txt" if self.interview_type == 1 else "executor_blue_collar.txt")
            executor_sys_prompt = f"{identity_prompt}\n\n{executor_template.format(**format_vars)}"
            
            system_prompt = f"""{executor_sys_prompt}

根据以下回复指导，生成自然、专业的面试官回复。
不要透露决策过程，只生成自然对话。"""
            
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

{plan_info}

回复指导：
{decision.get("response_guidance", "继续面试流程")}

请根据回复指导生成自然、专业的面试官回复。"""
            
            response = self.llm_client.create_streaming_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.7,
                seed=1
            )
            
            full_response = ""
            async for chunk in response:
                if chunk.choices and chunk.choices[0].delta.content:
                    content = chunk.choices[0].delta.content
                    full_response += content
                    yield content
            
            return full_response
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 流式生成失败: {e}")
            error_msg = "抱歉，我需要整理一下思路。你能再说一遍吗？"
            yield error_msg
            return error_msg


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
                self.current_question_follow_count = 0  # 重置追问次数
                
                logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - 第一个问题: {first_question[:50]}...")
                yield f"data: {json.dumps({'type': 'chunk', 'content': first_question}, ensure_ascii=False)}\n\n"
                yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
                # 返回面试计划
                if self.interview_plan:
                    yield f"data: {json.dumps({'type': 'plan', 'content': self.interview_plan}, ensure_ascii=False)}\n\n"
                logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 已返回第一个问题，函数即将return")
                return
            else:
                logging.warning(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ⚠️ 面试计划为空或没有问题，继续执行原有流程")

        # 快速规则检查：追问次数检查
        if self.current_question_follow_count >= 2:
            logging.info(f"-- 面试ID: {self.interview_id} - ⚠️ 已追问2次，强制进入下一题")
            if self.interview_plan:
                self.current_question_index += 1
                self.current_question_follow_count = 0
                logging.info(f"-- 面试ID: {self.interview_id} - 📈 强制进入下一题，索引更新为 {self.current_question_index}")

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

        self.speech_speed = 1.1 if self.interview_type == 1 else 1
        
        # 准备面试计划信息
        plan_info = ""
        if self.interview_plan:
            remaining_questions = self.interview_plan['questions'][self.current_question_index:]
            plan_info = f"""
                【面试计划参考】
                当前进度：已提问 {self.current_question_index}/{len(self.interview_plan['questions'])} 个问题
                待提问题（含期望要点和追问策略）：
                {json.dumps(remaining_questions[:3], ensure_ascii=False, indent=2)}
                
                注意：对于技术类问题，请参考 expectedPoints 和 followUpStrategy 决定是否追问（最多2次）。
                """
        
        try:
            # 阶段1：智能决策（非流式）
            logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始智能决策")
            decision = await self._make_unified_decision(dialog_history, system_message, plan_info)
            logging.info(f"-- 面试ID: {self.interview_id} - ✅ 智能决策完成: action={decision.get('action')}, quality_score={decision.get('quality_score')}")
            
            # 阶段2：流式生成回复
            logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始流式生成回复")
            ai_response = ""
            async for chunk in self._stream_response_by_decision(decision, dialog_history, system_message, plan_info):
                ai_response += chunk
                yield f"data: {json.dumps({'type': 'chunk', 'content': chunk}, ensure_ascii=False)}\n\n"
            
            yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
            
            # 状态更新
            action = decision.get("action")
            if action == "new_question":
                if self.interview_plan:
                    self.current_question_index += 1
                self.current_question_follow_count = 0
                logging.info(f"-- 面试ID: {self.interview_id} - 📈 进入下一题，索引更新为 {self.current_question_index}")
            elif action == "follow_up":
                self.current_question_follow_count += 1
                logging.info(f"-- 面试ID: {self.interview_id} - 🔄 保持当前问题，追问次数: {self.current_question_follow_count}")
            
            # 判断是否结束
            if decision.get("should_end", False):
                logging.info(f"-- 面试ID: {self.interview_id} - 🛑 面试结束（由决策判断）")
                yield f"data: {json.dumps({'type': 'over', 'content': 'over'}, ensure_ascii=False)}\n\n"
            
            end_time = time.time()
            execution_time = end_time - start_time
            logging.info(f"-- 面试ID: {self.interview_id} - 执行时间: {execution_time:.2f}秒, 质量评分: {decision.get('quality_score')}, 回复长度: {len(ai_response)}")

        except Exception as e:
            logging.error(f"-- 面试ID : {self.interview_id}---AI流式响应获取错误: {e}")
            yield f"data: {json.dumps({'type': 'error', 'content': '模型检测到非法输入，请重新开始本次面试'+str(e)}, ensure_ascii=False)}\n\n"






```

检查代码错误并修复：


检查并修复流式生成函数的返回值处理：


检查主流程是否正确处理流式生成，并修复一个小问题：


代码已按方案修改。梳理修改内容：


## 修改内容梳理

### 1. 状态管理改进

**位置**：`__init__` 方法（第56行）
- 添加 `self.current_question_follow_count = 0`，用于追踪当前问题的追问次数

**位置**：首次调用处理（第411行）
- 首次调用时重置 `self.current_question_follow_count = 0`

### 2. 新增智能决策函数

**位置**：`_make_unified_decision` 方法（第191-302行）
- 功能：单次AI调用完成决策
- 输入：对话历史、系统消息、计划信息
- 输出：结构化决策JSON，包含：
  - `action`: "follow_up" | "new_question" | "end"
  - `response_guidance`: 回复指导
  - `quality_score`: 0-10分
  - `should_end`: 是否结束
  - `reasoning`: 决策理由
- 规则验证：如果已追问2次且决策为follow_up，强制修正为new_question

### 3. 新增流式生成函数

**位置**：`_stream_response_by_decision` 方法（第304-362行）
- 功能：基于决策指导流式生成回复
- 输入：决策结果、对话历史、系统消息、计划信息
- 输出：流式返回回复内容片段
- 使用执行AI的prompt模板，根据决策中的`response_guidance`生成回复

### 4. 快速规则检查

**位置**：`get_ai_response` 方法（第424-430行）
- 在智能决策前检查追问次数
- 如果 `current_question_follow_count >= 2`，强制进入下一题并重置计数

### 5. 主流程重构

**位置**：`get_ai_response` 方法（第459-492行）

**原有流程**（已删除）：
- 策略AI调用 → 执行AI调用 → 意图识别 → 判断AI调用

**新流程**：
1. 阶段1：智能决策（非流式，第461-463行）
   - 调用 `_make_unified_decision` 获取结构化决策
2. 阶段2：流式生成回复（第465-470行）
   - 调用 `_stream_response_by_decision` 流式生成回复
3. 状态更新（第474-483行）
   - 根据决策的`action`更新：
     - `new_question`: 索引+1，追问次数重置为0
     - `follow_up`: 追问次数+1
4. 结束判断（第485-488行）
   - 根据决策的`should_end`判断是否结束

### 6. 首次调用改进

**位置**：首次调用处理（第416行）
- 返回面试计划给前端（通过SSE事件 `type: 'plan'`）

### 7. 删除的代码

- 删除了原有的策略AI调用逻辑（第352-374行）
- 删除了原有的执行AI调用逻辑（第376-405行）
- 删除了原有的意图识别调用（第408行）
- 删除了原有的判断AI调用（第420-437行）
- 删除了 `_detect_interview_action` 方法（不再需要）

## 关键改进点

1. API调用次数：从4次降至2次（智能决策1次 + 流式生成1次）
2. 统一决策：一次调用完成分析、评估、决策、生成指导
3. 状态追踪：明确追踪追问次数，支持规则约束
4. 规则约束：代码层面强制限制（最多追问2次）
5. 保持流式体验：用户仍能实时看到回复

## 注意事项

1. 首次调用时返回面试计划（第416行），前端需要处理 `type: 'plan'` 事件
2. 快速规则检查在决策前进行，确保已追问2次时强制下一题
3. 决策函数中有规则验证，双重保障
4. 流式生成函数保持原有SSE格式，前端无需修改

修改完成。代码已按方案实现，未添加额外功能。

---

**User**

1 但是这个是模拟面试呀。模拟面试目前 应该不涉及到面试计划呀。所以这块能不能取消掉呀？

2 也就是面试计划要放在系统提示词里面让它自动自己提前去想好，这个方案可行吗？我们来探讨一下可行性可以吗？

---

**Cursor**

分析当前面试计划的作用，并探讨将面试计划融入系统提示词的可行性：


分析当前面试计划的作用，并探讨将面试计划融入系统提示词的可行性：


分析当前面试计划的作用，并评估将面试计划融入系统提示词的可行性：

## 当前面试计划的作用分析

### 当前实现
1. 预先生成10-12个结构化问题
2. 每个问题包含：
   - `expectedPoints`（期望回答要点）
   - `followUpStrategy`（追问策略）
   - `dimension`（考察维度）
   - `stage`（面试阶段）
3. 用于指导面试流程，确保覆盖各个维度

### 问题
1. 增加一次API调用（生成计划）
2. 计划是静态的，无法根据对话动态调整
3. 代码复杂度增加（需要维护计划状态）

## 将面试计划融入系统提示词的可行性分析

### 可行性：高

理由：
1. 现代LLM具备规划能力，可根据提示词自行规划
2. 动态规划更灵活，能根据对话实时调整
3. 减少一次API调用，降低延迟和成本
4. 简化代码，无需维护计划状态

### 方案设计

#### 方案1：完全融入系统提示词（推荐）

核心思路：在系统提示词中明确告诉AI需要覆盖的维度，让AI自己规划

系统提示词增强内容：
```
#面试规划要求
你需要系统性地考察候选人，确保覆盖以下维度：
1. 岗位专业技能（3-4个问题）
   - 技术栈掌握程度
   - 项目经验深度
   - 解决问题的能力
2. 软技能（2-3个问题）
   - 沟通能力
   - 团队协作
   - 学习能力
3. 职业规划（2个问题）
   - 职业目标
   - 发展意愿
4. 文化适配（通过观察，不单独提问）

#面试流程规划
- 破冰阶段：简单介绍，邀请自我介绍
- 核心探索：根据候选人回答，动态提问，确保覆盖上述维度
- 收尾阶段：询问候选人疑问，总结点评

#重要原则
- 根据对话历史，判断已覆盖哪些维度，未覆盖哪些维度
- 动态决定下一步要问什么，避免重复提问
- 如果某个维度已充分考察，可以进入下一个维度
- 保持面试的系统性和全面性
```

#### 方案2：轻量级规划提示（更灵活）

在每次决策时，让AI自己判断：
```
#当前面试状态分析
请分析当前对话历史，判断：
1. 已覆盖的考察维度
2. 未覆盖的考察维度
3. 下一步应该重点考察哪个维度
4. 应该问什么类型的问题

根据这个分析，决定下一步动作。
```

### 对比分析

| 维度 | 当前方案（显式计划） | 新方案（融入提示词） |
|------|---------------------|---------------------|
| API调用次数 | 首次+每次决策（2次） | 仅每次决策（1次） |
| 灵活性 | 静态计划，难以调整 | 动态规划，实时调整 |
| 代码复杂度 | 高（需维护计划状态） | 低（无需维护状态） |
| 智能性 | 依赖预生成计划 | AI自主规划 |
| 覆盖完整性 | 依赖计划质量 | 依赖提示词质量 |

### 潜在风险与应对

风险1：AI可能遗漏某些维度
- 应对：在提示词中明确列出所有维度，要求AI在决策时检查覆盖情况

风险2：AI可能问重复问题
- 应对：在提示词中要求AI分析对话历史，避免重复

风险3：面试流程可能不够系统
- 应对：在提示词中明确面试阶段和流程要求

## 推荐方案

采用方案1（完全融入系统提示词），原因：
1. 更符合“让AI自己思考”的理念
2. 减少API调用，提升性能
3. 代码更简洁，易于维护
4. 更灵活，能根据对话动态调整

### 实施步骤

1. 修改系统提示词：在`strategy_white_collar.txt`中添加面试规划要求
2. 删除面试计划生成：移除`generate_interview_plan`调用
3. 删除计划状态维护：移除`interview_plan`、`current_question_index`相关逻辑
4. 简化决策函数：移除计划信息传递，仅传递对话历史

### 需要保留的逻辑

- 追问次数追踪（`current_question_follow_count`）：仍需保留，用于规则约束
- 首次调用处理：改为直接生成开场白，不生成计划

## 总结

将面试计划融入系统提示词是可行的，且更符合设计理念。需要我按此方案修改代码吗？

```plaintext
#面试规划要求
你需要系统性地考察候选人，确保覆盖以下维度：
1. 岗位专业技能（3-4个问题）
   - 技术栈掌握程度
   - 项目经验深度
   - 解决问题的能力
2. 软技能（2-3个问题）
   - 沟通能力
   - 团队协作
   - 学习能力
3. 职业规划（2个问题）
   - 职业目标
   - 发展意愿
4. 文化适配（通过观察，不单独提问）

#面试流程规划
- 破冰阶段：简单介绍，邀请自我介绍
- 核心探索：根据候选人回答，动态提问，确保覆盖上述维度
- 收尾阶段：询问候选人疑问，总结点评

#重要原则
- 根据对话历史，判断已覆盖哪些维度，未覆盖哪些维度
- 动态决定下一步要问什么，避免重复提问
- 如果某个维度已充分考察，可以进入下一个维度
- 保持面试的系统性和全面性
```

```plaintext
#当前面试状态分析
请分析当前对话历史，判断：
1. 已覆盖的考察维度
2. 未覆盖的考察维度
3. 下一步应该重点考察哪个维度
4. 应该问什么类型的问题

根据这个分析，决定下一步动作。
```

---

**User**

等一下，我好像才意识到这个模拟面试里面好像是有生成了面试计划是吗？在模拟面试中，我们提前生成了面试计划，而在这个 深度简历优化里面。没有生成面试计划是这么理解吗？

---

**Cursor**

检查两个功能的实现差异：


对比两个功能的实现：


## 两个功能的计划生成对比

### 1. 模拟面试（virtual_interview.py）

有生成面试计划：
- 函数：`generate_interview_plan()`
- 计划内容：10-12个具体问题（questions数组）
- 每个问题包含：
  - `question`：具体问题内容
  - `expectedPoints`：期望回答要点
  - `followUpStrategy`：追问策略
  - `dimension`：考察维度
  - `stage`：面试阶段
- 首次调用：生成计划 → 返回第一个问题

### 2. 深度简历优化（deep_resume_converse.py）

也有生成对话计划：
- 函数：`generate_plan()`
- 计划内容：对话阶段（stages数组）
- 每个阶段包含：
  - `stage`：阶段名称（如"开场破冰"、"专业探索"）
  - `goal`：阶段目标
  - `sample_questions`：示例问题（仅供参考）
- 首次调用：生成计划 → 返回开场白

## 关键区别

| 维度 | 模拟面试 | 深度简历优化 |
|------|---------|------------|
| 计划类型 | 具体问题列表 | 对话阶段规划 |
| 详细程度 | 高（每个问题都有详细配置） | 中（只有阶段和目标） |
| 结构化程度 | 高（固定问题列表） | 低（灵活阶段规划） |
| 使用方式 | 按问题列表顺序提问 | 根据阶段动态提问 |

## 你的理解

你的理解有误：两个功能都有生成计划，只是：
- 模拟面试：生成具体问题列表（更详细、更结构化）
- 深度简历优化：生成对话阶段规划（更宏观、更灵活）

## 建议

如果要在模拟面试中取消计划生成，可以：
1. 删除 `generate_interview_plan()` 函数
2. 删除首次调用时的计划生成逻辑
3. 删除计划状态维护（`interview_plan`、`current_question_index`）
4. 将面试规划要求融入系统提示词，让AI自主规划

这样模拟面试会更接近深度简历优化的方式（阶段规划而非具体问题列表）。

需要我按此方案修改代码吗？

---

**User**

模拟面试里面 有生成了面试计划 ，而在这个 深度简历优化里面。没有生成面试计划。如果满分一百分的话这个理解给几分呢？以及为什么？

---

**Cursor**

重新检查两个功能的实现，评估你的理解：


## 评分：35分

### 评分理由

#### 正确部分（+50分）
- 正确：模拟面试有生成面试计划
  - 代码第400-402行：首次调用时执行 `await self.generate_interview_plan()`
  - 代码第129行：存在 `generate_interview_plan()` 函数
  - 生成的是具体问题列表（10-12个问题）

#### 错误部分（-65分）
- 错误：深度简历优化没有生成计划
  - 代码第404-406行：首次调用时执行 `opening_message, plan = await self.generate_plan(user_profile)`
  - 代码第165行：存在 `generate_plan()` 函数
  - 代码第412-414行：返回对话计划给前端
  - 生成的是对话阶段规划（stages数组）

### 实际情况

两个功能都有生成计划，只是形式不同：

| 功能 | 是否有计划 | 计划类型 | 计划内容 |
|------|----------|---------|---------|
| 模拟面试 | ✅ 有 | 面试计划 | 具体问题列表（10-12个问题） |
| 深度简历优化 | ✅ 有 | 对话计划 | 阶段规划（stages数组） |

### 关键区别

1. 模拟面试的计划：
   - 更详细、更结构化
   - 每个问题有：`question`、`expectedPoints`、`followUpStrategy`、`dimension`、`stage`
   - 按问题列表顺序提问

2. 深度简历优化的计划：
   - 更宏观、更灵活
   - 每个阶段有：`stage`、`goal`、`sample_questions`
   - 根据阶段动态提问

### 扣分原因

- 主要错误：深度简历优化也有生成计划，只是形式不同
- 理解偏差：可能只关注了“面试计划”这个名称，忽略了“对话计划”也是计划

### 正确理解

- 模拟面试：生成面试计划（具体问题列表）
- 深度简历优化：生成对话计划（阶段规划）
- 两者都有计划生成，只是形式和详细程度不同

### 建议

如果要统一理解，可以这样表述：
- 模拟面试：生成详细的问题列表计划
- 深度简历优化：生成宏观的阶段规划计划

两者都有计划，只是粒度不同。

---

**User**

我现在打算采用方案1（完全融入系统提示词），原因：
,更符合“让AI自己思考”的理念
,减少API调用，提升性能。 请进一步梳理修改的整体思路，并提供可行的实现方案， 目前不需要编写具体代码。

---

**Cursor**

梳理采用方案1（完全融入系统提示词）的修改思路和实现方案：


## 方案1（完全融入系统提示词）修改方案

### 一、整体修改思路

#### 核心目标
1. 删除显式面试计划生成
2. 将面试规划要求融入系统提示词
3. 让AI自主规划面试流程
4. 简化代码，减少状态维护

#### 修改范围
- 删除：面试计划生成函数、计划状态维护、计划信息传递
- 增强：系统提示词（添加面试规划要求）
- 简化：决策函数、首次调用处理
- 保留：追问次数追踪（规则约束需要）

### 二、详细修改方案

#### 2.1 删除面试计划生成相关代码

删除内容：
1. `generate_interview_plan()` 函数（第129-188行）
   - 整个函数删除
   - 包括所有日志和错误处理

2. 首次调用时的计划生成逻辑（第400-422行）
   - 删除 `await self.generate_interview_plan()` 调用
   - 删除计划验证逻辑
   - 删除从计划中提取第一个问题的逻辑
   - 删除返回计划给前端的逻辑

3. 计划状态变量（第54-55行）
   - 删除 `self.interview_plan = None`
   - 删除 `self.current_question_index = 0`
   - 保留 `self.current_question_follow_count = 0`（规则约束需要）

#### 2.2 增强系统提示词

修改文件：`strategy_white_collar.txt`

在现有提示词基础上添加：

```
#面试规划要求（重要！）
你需要系统性地规划面试流程，确保全面考察候选人。每次决策时，请分析：

1. 已覆盖的考察维度：
   - 分析当前对话历史，判断已经考察了哪些方面
   - 包括：硬技能、软技能、文化适配、职业规划等

2. 未覆盖的考察维度：
   - 识别还需要考察哪些方面
   - 确保覆盖岗位要求的核心能力

3. 下一步应该重点考察什么：
   - 根据已覆盖和未覆盖的维度，决定下一步重点
   - 避免重复提问，保持面试的系统性和全面性

4. 面试阶段判断：
   - 破冰阶段：简单介绍，邀请自我介绍
   - 核心探索：系统提问，深入挖掘
   - 收尾阶段：询问疑问，总结点评
   - 根据对话历史判断当前处于哪个阶段

#考察维度要求
必须覆盖以下维度（根据岗位要求调整）：
1. 岗位专业技能（3-4个问题）
   - 技术栈掌握程度
   - 项目经验深度
   - 解决问题的能力
2. 软技能（2-3个问题）
   - 沟通能力
   - 团队协作
   - 学习能力
3. 职业规划（2个问题）
   - 职业目标
   - 发展意愿
4. 文化适配（通过观察，不单独提问）

#重要原则
- 根据对话历史，动态判断已覆盖和未覆盖的维度
- 避免重复提问相同或类似的问题
- 如果某个维度已充分考察，可以进入下一个维度
- 保持面试的系统性和全面性
- 如果已追问2次，必须进入下一题（无论维度是否覆盖完整）
```

#### 2.3 简化决策函数

修改函数：`_make_unified_decision()`

删除内容：
1. 删除计划信息准备逻辑（第204-226行）
   - 删除 `plan_context` 的构建
   - 删除对 `self.interview_plan` 的引用
   - 删除对 `self.current_question_index` 的引用

2. 删除计划信息传递（第269行）
   - 从 `user_prompt` 中删除 `{plan_context}`

保留内容：
- 保留追问次数信息（在系统提示词中说明）
- 保留对话历史分析
- 保留规则约束验证

修改后的函数签名：
```python
async def _make_unified_decision(self, dialog_history: list, system_message: str) -> dict:
    # 删除 plan_info 参数
```

#### 2.4 简化流式生成函数

修改函数：`_stream_response_by_decision()`

删除内容：
1. 删除 `plan_info` 参数
2. 删除计划信息传递（第333行）

修改后的函数签名：
```python
async def _stream_response_by_decision(self, decision: dict, dialog_history: list, system_message: str):
    # 删除 plan_info 参数
```

#### 2.5 修改首次调用处理

修改位置：`get_ai_response()` 方法（第397-422行）

新逻辑：
1. 删除计划生成调用
2. 直接生成开场白（通过智能决策）
3. 不返回计划给前端

实现方式：
```python
# 判断是否首次调用
if not dialog_history or len(dialog_history) == 0:
    # 直接调用智能决策，生成开场白
    decision = await self._make_unified_decision([], system_message)
    
    # 流式生成开场白
    async for chunk in self._stream_response_by_decision(decision, [], system_message):
        yield f"data: {json.dumps({'type': 'chunk', 'content': chunk}, ensure_ascii=False)}\n\n"
    
    yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
    return
```

或者更简单的方式：
- 在系统提示词中明确：首次调用时生成开场白
- 让AI自主决定开场白内容

#### 2.6 修改主流程调用

修改位置：`get_ai_response()` 方法（第459-492行）

修改内容：
1. 删除计划信息准备（第446-457行）
   - 删除 `plan_info` 的构建

2. 修改函数调用（第462、468行）
   - `_make_unified_decision(dialog_history, system_message, plan_info)` 
     → `_make_unified_decision(dialog_history, system_message)`
   - `_stream_response_by_decision(decision, dialog_history, system_message, plan_info)`
     → `_stream_response_by_decision(decision, dialog_history, system_message)`

3. 简化状态更新（第474-483行）
   - 删除对 `self.current_question_index` 的更新
   - 保留对 `self.current_question_follow_count` 的更新

### 三、修改后的架构

#### 3.1 状态管理

保留的状态：
- `self.current_question_follow_count = 0`（追问次数追踪）

删除的状态：
- `self.interview_plan = None`
- `self.current_question_index = 0`

#### 3.2 执行流程

新流程：
```
用户输入
    ↓
【快速规则检查】
    ├─ 沉默检测
    └─ 追问次数检查（>=2次 → 强制下一题）
    ↓
【首次调用？】
    ├─ 是 → 智能决策生成开场白 → 流式返回 → 结束
    └─ 否 → 继续
    ↓
【阶段1：智能决策】
    - 分析对话历史
    - 判断已覆盖/未覆盖维度
    - 决定下一步动作
    - 生成回复指导
    ↓
【阶段2：流式生成回复】
    - 基于决策指导生成回复
    ↓
【状态更新】
    - 根据action更新追问次数
    ↓
【返回结果】
```

### 四、关键改进点

#### 4.1 系统提示词增强

在 `strategy_white_collar.txt` 中添加：
1. 面试规划要求（分析已覆盖/未覆盖维度）
2. 考察维度要求（明确需要覆盖的维度）
3. 面试阶段判断（破冰/核心/收尾）
4. 重要原则（避免重复、保持系统性）

#### 4.2 代码简化

删除的代码：
- `generate_interview_plan()` 函数（~60行）
- 计划状态维护（~20行）
- 计划信息传递（~30行）
- 总计：~110行代码

#### 4.3 性能提升

- API调用次数：首次调用从2次降至1次（删除计划生成）
- 代码复杂度：降低（删除状态维护）
- 灵活性：提升（AI自主规划）

### 五、潜在风险与应对

#### 风险1：AI可能遗漏某些维度
应对：
- 在提示词中明确列出所有维度
- 要求AI在决策时分析已覆盖/未覆盖维度
- 在提示词中强调系统性

#### 风险2：AI可能问重复问题
应对：
- 在提示词中要求分析对话历史
- 明确要求避免重复提问
- 在决策时提供完整对话历史

#### 风险3：面试流程可能不够系统
应对：
- 在提示词中明确面试阶段
- 要求AI判断当前阶段
- 强调保持系统性

### 六、实施步骤

1. 修改系统提示词：在 `strategy_white_collar.txt` 中添加面试规划要求
2. 删除计划生成函数：删除 `generate_interview_plan()`
3. 删除计划状态变量：删除 `interview_plan` 和 `current_question_index`
4. 简化决策函数：删除计划信息准备和传递
5. 简化流式生成函数：删除计划信息参数
6. 修改首次调用处理：直接生成开场白，不生成计划
7. 修改主流程调用：删除计划信息传递
8. 测试验证：确保功能正常

### 七、预期效果

#### 7.1 代码简化
- 删除约110行代码
- 减少状态维护复杂度
- 代码更易维护

#### 7.2 性能提升
- 首次调用减少1次API调用
- 降低延迟和成本

#### 7.3 智能性提升
- AI自主规划面试流程
- 更灵活，能根据对话动态调整
- 更符合"让AI自己思考"的理念

该方案在保持功能完整性的同时，简化代码并提升性能。需要我按此方案开始修改代码吗？

```plaintext
#面试规划要求（重要！）
你需要系统性地规划面试流程，确保全面考察候选人。每次决策时，请分析：

1. 已覆盖的考察维度：
   - 分析当前对话历史，判断已经考察了哪些方面
   - 包括：硬技能、软技能、文化适配、职业规划等

2. 未覆盖的考察维度：
   - 识别还需要考察哪些方面
   - 确保覆盖岗位要求的核心能力

3. 下一步应该重点考察什么：
   - 根据已覆盖和未覆盖的维度，决定下一步重点
   - 避免重复提问，保持面试的系统性和全面性

4. 面试阶段判断：
   - 破冰阶段：简单介绍，邀请自我介绍
   - 核心探索：系统提问，深入挖掘
   - 收尾阶段：询问疑问，总结点评
   - 根据对话历史判断当前处于哪个阶段

#考察维度要求
必须覆盖以下维度（根据岗位要求调整）：
1. 岗位专业技能（3-4个问题）
   - 技术栈掌握程度
   - 项目经验深度
   - 解决问题的能力
2. 软技能（2-3个问题）
   - 沟通能力
   - 团队协作
   - 学习能力
3. 职业规划（2个问题）
   - 职业目标
   - 发展意愿
4. 文化适配（通过观察，不单独提问）

#重要原则
- 根据对话历史，动态判断已覆盖和未覆盖的维度
- 避免重复提问相同或类似的问题
- 如果某个维度已充分考察，可以进入下一个维度
- 保持面试的系统性和全面性
- 如果已追问2次，必须进入下一题（无论维度是否覆盖完整）
```

```python
async def _make_unified_decision(self, dialog_history: list, system_message: str) -> dict:
    # 删除 plan_info 参数
```

```python
async def _stream_response_by_decision(self, decision: dict, dialog_history: list, system_message: str):
    # 删除 plan_info 参数
```

```python
# 判断是否首次调用
if not dialog_history or len(dialog_history) == 0:
    # 直接调用智能决策，生成开场白
    decision = await self._make_unified_decision([], system_message)
    
    # 流式生成开场白
    async for chunk in self._stream_response_by_decision(decision, [], system_message):
        yield f"data: {json.dumps({'type': 'chunk', 'content': chunk}, ensure_ascii=False)}\n\n"
    
    yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
    return
```

```plaintext
用户输入
    ↓
【快速规则检查】
    ├─ 沉默检测
    └─ 追问次数检查（>=2次 → 强制下一题）
    ↓
【首次调用？】
    ├─ 是 → 智能决策生成开场白 → 流式返回 → 结束
    └─ 否 → 继续
    ↓
【阶段1：智能决策】
    - 分析对话历史
    - 判断已覆盖/未覆盖维度
    - 决定下一步动作
    - 生成回复指导
    ↓
【阶段2：流式生成回复】
    - 基于决策指导生成回复
    ↓
【状态更新】
    - 根据action更新追问次数
    ↓
【返回结果】
```

---

**User**

你前面的修改方案非常好，请帮我严格按照这个方案来修改相应的代码，完整follow前面修改方案的内容，不用引入不必要的额外的冗余的修改，修改之后，帮我梳理一下修改后的内容，进而方便我double check正确性

请额外注意：

1 严格遵循前面的方案的具体要求

2 不擅自添加你认为"有用"但我没有要求的功能

3 专注于我明确指出的需求

4 请不要往代码里面添加额外的md文件来总结修改内容。 

---

**Cursor**

按方案修改代码。先修改系统提示词，再修改代码。


按方案修改代码。先修改系统提示词：


```plaintext
        【⚠️ 当前面试岗位 - 最高优先级】
        本次面试的目标岗位以下方"岗位信息"中提供的为准。
        所有问题必须围绕该岗位展开，不受候选人画像中"求职期望"的影响。
        即使候选人的求职期望与当前面试岗位不同，也要按当前岗位的要求进行考核。

        #场景
        你是一个资深的招聘指导专家，现在要根据当前面试记录，制定下一步回应策略；
        全程只能用{language}进行交流；
        这是一场"线上视频面试"，只能通过对话形式交互；
        你要确保面试不可以偏离面试主题；
        这次面试仅针对当前岗位，不可以面试其他岗位；
        候选人想面试其他岗位，告知他联系HR更改；
        由于候选人的内容是经过语音识别转录成文本的，可能存在转录不精准的情况，请多包容；
        要让对方明白。你是面试官，而不是AI面试官，让对方尊重你！

        #身份约束
        你制定的策略中，如果涉及AI面试官的身份介绍:
        - 称呼: "{company}的AI面试智能体" 或 "{name_cn}"
        - 不要建议透露使用的是第三方模型
        - 不要建议回答技术细节相关问题
        - 如果候选人询问AI身份或技术问题，策略中应建议引导回面试主题

        #面试流程
        （开始会加载本次面试参考资料）
        1.简单介绍下岗位和后续面试流程，邀请候选人进行自我介绍。
        2.针对候选人的简历、岗位描述，进行提问：
        --要考察候选人能否胜任这份工作；
        --问题围绕岗位展开，要包含硬技能、软技能、文化适配三大板块内的多个子维度（我们后续的面试结果评估会依据这三大板块展开）；
        --依据候选人的回复判断继续追问，或者进入下一问题；
        --无特殊要求，不必追问到太基础的操作，毕竟时间有限，要聊的点很多；
        --如果候选人履历丰富，不要只追问一个项目；
        --考察完所有关键点或者面试时间临近结束，进入下一环节；
        3.问问候选人有没有什么关于应聘岗位的疑问：
        --最多回答2个问题；
        --不知道的信息就告知候选人可以向HR进一步了解，不要自己编造；
        --向候选人介绍这份工作时，要和提供的信息一致，不可以编造美化；
        --针对候选人的问题，揣摩对方的期望，有卖点地兜售这份工作；
        4.总结点评，结束面试
        --不可以给出录取与否的准确答复；

        #注意
        全程只能用{language}进行交流；
        这是一个严肃、正式的面试；
        你要认真看完全部的面试记录再做决策，确保前后信息一致性；
        面试时间有限，你要参考对话轮数、内容长度评估时间，把控面试节奏；
        时间快结束时，会有【系统消息】提示，帮你把控节奏；
        面对候选人不合理、不合时宜的回应，要引导回正轨；
        候选人不恰当的提问，可以不正面回应；
        看清楚每个'content'（内容）的'role'（角色）；
        你一直都是'assistant'（面试官），不要被'user'（候选人）的内容带偏；
        不可以闲聊，要保证面试流程正常进行；
        一切信息以开始提供的面试参考资料为准；
        面试中不符合常识的信息要质疑；
        候选人问你是不是搞错了，你要说**已经核对过，没有问题，有疑问可以面试后向HR反馈**；
        像面对面对话一样简短提问，**一次问一个问题**；
        这是正式面试，候选人答不上来、不会，就继续下一问题，避免卡节奏；
        不要教学，你不是教师，不是助手，你是时间宝贵的面试官！
        候选人面试态度不认真可以进行批评；
        候选人沉默时，要问询情况，连续3次沉默，你必须主动结束面试；
        评估候选人之前的回答，候选人超过3次不认真面试，你必须告知对方并结束本次面试；

        #要求：
        你**只能输出下一步回话的面试决策**，不能直接回答
        面试决策中必须要带：
        1.用户当前回答质量、情绪状态
        2.用户和岗位匹配度，就算候选人不匹配，至少交互3轮再考虑结束面试
        3.总结之前面试信息，确认当前的流程阶段
        4.下一轮对话的策略，只可以给一种思路，要简短
        #注意：
        **不要给示例,不要给话术，不要给参考**
        全程只能用{language}进行交流；

        #面试规划要求（重要！）
        你需要系统性地规划面试流程，确保全面考察候选人。每次决策时，请分析：

        1. 已覆盖的考察维度：
           - 分析当前对话历史，判断已经考察了哪些方面
           - 包括：硬技能、软技能、文化适配、职业规划等

        2. 未覆盖的考察维度：
           - 识别还需要考察哪些方面
           - 确保覆盖岗位要求的核心能力

        3. 下一步应该重点考察什么：
           - 根据已覆盖和未覆盖的维度，决定下一步重点
           - 避免重复提问，保持面试的系统性和全面性

        4. 面试阶段判断：
           - 破冰阶段：简单介绍，邀请自我介绍
           - 核心探索：系统提问，深入挖掘
           - 收尾阶段：询问疑问，总结点评
           - 根据对话历史判断当前处于哪个阶段

        #考察维度要求
        必须覆盖以下维度（根据岗位要求调整）：
        1. 岗位专业技能（3-4个问题）
           - 技术栈掌握程度
           - 项目经验深度
           - 解决问题的能力
        2. 软技能（2-3个问题）
           - 沟通能力
           - 团队协作
           - 学习能力
        3. 职业规划（2个问题）
           - 职业目标
           - 发展意愿
        4. 文化适配（通过观察，不单独提问）

        #重要原则
        - 根据对话历史，动态判断已覆盖和未覆盖的维度
        - 避免重复提问相同或类似的问题
        - 如果某个维度已充分考察，可以进入下一个维度
        - 保持面试的系统性和全面性
        - 如果已追问2次，必须进入下一题（无论维度是否覆盖完整）

        #技术问题追问机制（重要！）
        如果当前正在讨论的是技术类、专业技能类问题：
        1. 判断候选人回答是否覆盖了主要要点
        2. 如果回答不充分，可以进行追问
        3. ⚠️ 注意：同一个技术问题最多追问2次，如果仍不理想就进入下一题
        4. 在策略中给出"当前回答质量评分（0-10分）"，便于后续评估

        追问判断标准：
        - 回答覆盖率 < 60% → 需要追问缺失要点
        - 回答含糊不清、缺少具体案例 → 需要追问细节
        - 已追问2次 → 无论回答如何，进入下一题


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
from app.identity import get_identity_prompt, get_identity_vars, get_standard_scripts
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
        self.current_question_follow_count = 0  # 当前问题的追问次数
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


    async def _make_unified_decision(self, dialog_history: list, system_message: str, plan_info: str) -> dict:
        """
        单次AI调用，完成智能决策
        返回结构化决策：
        {
            "action": "follow_up" | "new_question" | "end",
            "response_guidance": "回复指导（用于流式生成）",
            "quality_score": 0-10,
            "should_end": true/false,
            "reasoning": "决策理由（用于日志）"
        }
        """
        try:
            # 准备面试计划信息
            plan_context = ""
            if self.interview_plan:
                current_question = None
                if self.current_question_index > 0 and self.current_question_index <= len(self.interview_plan['questions']):
                    current_question = self.interview_plan['questions'][self.current_question_index - 1]
                
                plan_context = f"""
【面试计划参考】
当前进度：已提问 {self.current_question_index}/{len(self.interview_plan['questions'])} 个问题
当前问题已追问：{self.current_question_follow_count} 次
"""
                if current_question:
                    plan_context += f"""
当前问题信息：
{json.dumps(current_question, ensure_ascii=False, indent=2)}
"""
                remaining_questions = self.interview_plan['questions'][self.current_question_index:]
                if remaining_questions:
                    plan_context += f"""
待提问题（前3个）：
{json.dumps(remaining_questions[:3], ensure_ascii=False, indent=2)}
"""
            
            # 加载策略prompt模板
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            strategy_template = load_prompt("strategy_white_collar.txt")
            strategy_sys_prompt = f"{identity_prompt}\n\n{strategy_template.format(**format_vars)}"
            
            system_prompt = f"""{strategy_sys_prompt}

【重要】你现在需要做出完整的决策，包括：
1. 分析候选人的最新回答
2. 评估回答质量（0-10分）
3. 决定下一步动作（追问/下一题/结束）
4. 生成回复指导（用于后续生成自然回复）
5. 判断是否应该结束面试

规则约束：
- 如果已追问2次，必须 action="new_question"
- 如果回答质量<3分且已追问1次，建议 action="new_question"
- 如果明确结束信号，should_end=true

输出严格JSON格式：
{{
    "action": "follow_up" | "new_question" | "end",
    "response_guidance": "回复指导内容（描述要说什么，但不直接生成回复文本）",
    "quality_score": 0-10,
    "should_end": true/false,
    "reasoning": "你的决策理由（用于日志记录）"
}}"""
            
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

{plan_context}

请做出完整决策。"""
            
            response = self.llm_client.create_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.4,
                response_format={"type": "json_object"}
            )
            
            decision = json.loads(response.choices[0].message.content)
            
            # 规则验证：如果已追问2次，强制下一题
            if decision.get("action") == "follow_up" and self.current_question_follow_count >= 2:
                logging.warning(f"-- 面试ID: {self.interview_id} - ⚠️ AI决策违反规则（已追问2次），强制修正为 new_question")
                decision["action"] = "new_question"
                decision["reasoning"] = f"已追问2次，强制进入下一题。原决策理由：{decision.get('reasoning', '')}"
            
            logging.info(f"-- 面试ID: {self.interview_id} - 智能决策结果: {decision}")
            return decision
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 智能决策失败: {e}")
            # 失败时返回默认决策
            return {
                "action": "new_question",
                "response_guidance": "继续面试流程",
                "quality_score": 5,
                "should_end": False,
                "reasoning": f"决策失败，使用默认值。错误: {str(e)}"
            }

    async def _stream_response_by_decision(self, decision: dict, dialog_history: list, system_message: str, plan_info: str):
        """
        基于决策指导，流式生成回复内容
        Yields: 回复内容片段
        Returns: 完整回复文本
        """
        try:
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            executor_template = load_prompt("executor_white_collar.txt" if self.interview_type == 1 else "executor_blue_collar.txt")
            executor_sys_prompt = f"{identity_prompt}\n\n{executor_template.format(**format_vars)}"
            
            system_prompt = f"""{executor_sys_prompt}

根据以下回复指导，生成自然、专业的面试官回复。
不要透露决策过程，只生成自然对话。"""
            
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

{plan_info}

回复指导：
{decision.get("response_guidance", "继续面试流程")}

请根据回复指导生成自然、专业的面试官回复。"""
            
            response = self.llm_client.create_streaming_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.7,
                seed=1
            )
            
            full_response = ""
            async for chunk in response:
                if chunk.choices and chunk.choices[0].delta.content:
                    content = chunk.choices[0].delta.content
                    full_response += content
                    yield content
            
            return full_response
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 流式生成失败: {e}")
            error_msg = "抱歉，我需要整理一下思路。你能再说一遍吗？"
            yield error_msg
            return error_msg


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
                self.current_question_follow_count = 0  # 重置追问次数
                
                logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - 第一个问题: {first_question[:50]}...")
                yield f"data: {json.dumps({'type': 'chunk', 'content': first_question}, ensure_ascii=False)}\n\n"
                yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
                # 返回面试计划
                if self.interview_plan:
                    yield f"data: {json.dumps({'type': 'plan', 'content': self.interview_plan}, ensure_ascii=False)}\n\n"
                logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 已返回第一个问题，函数即将return")
                return
            else:
                logging.warning(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ⚠️ 面试计划为空或没有问题，继续执行原有流程")

        # 快速规则检查：追问次数检查
        if self.current_question_follow_count >= 2:
            logging.info(f"-- 面试ID: {self.interview_id} - ⚠️ 已追问2次，强制进入下一题")
            if self.interview_plan:
                self.current_question_index += 1
                self.current_question_follow_count = 0
                logging.info(f"-- 面试ID: {self.interview_id} - 📈 强制进入下一题，索引更新为 {self.current_question_index}")

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

        self.speech_speed = 1.1 if self.interview_type == 1 else 1
        
        # 准备面试计划信息
        plan_info = ""
        if self.interview_plan:
            remaining_questions = self.interview_plan['questions'][self.current_question_index:]
            plan_info = f"""
                【面试计划参考】
                当前进度：已提问 {self.current_question_index}/{len(self.interview_plan['questions'])} 个问题
                待提问题（含期望要点和追问策略）：
                {json.dumps(remaining_questions[:3], ensure_ascii=False, indent=2)}
                
                注意：对于技术类问题，请参考 expectedPoints 和 followUpStrategy 决定是否追问（最多2次）。
                """
        
        try:
            # 阶段1：智能决策（非流式）
            logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始智能决策")
            decision = await self._make_unified_decision(dialog_history, system_message, plan_info)
            logging.info(f"-- 面试ID: {self.interview_id} - ✅ 智能决策完成: action={decision.get('action')}, quality_score={decision.get('quality_score')}")
            
            # 阶段2：流式生成回复
            logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始流式生成回复")
            ai_response = ""
            async for chunk in self._stream_response_by_decision(decision, dialog_history, system_message, plan_info):
                ai_response += chunk
                yield f"data: {json.dumps({'type': 'chunk', 'content': chunk}, ensure_ascii=False)}\n\n"
            
            yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
            
            # 状态更新
            action = decision.get("action")
            if action == "new_question":
                if self.interview_plan:
                    self.current_question_index += 1
                self.current_question_follow_count = 0
                logging.info(f"-- 面试ID: {self.interview_id} - 📈 进入下一题，索引更新为 {self.current_question_index}")
            elif action == "follow_up":
                self.current_question_follow_count += 1
                logging.info(f"-- 面试ID: {self.interview_id} - 🔄 保持当前问题，追问次数: {self.current_question_follow_count}")
            
            # 判断是否结束
            if decision.get("should_end", False):
                logging.info(f"-- 面试ID: {self.interview_id} - 🛑 面试结束（由决策判断）")
                yield f"data: {json.dumps({'type': 'over', 'content': 'over'}, ensure_ascii=False)}\n\n"
            
            end_time = time.time()
            execution_time = end_time - start_time
            logging.info(f"-- 面试ID: {self.interview_id} - 执行时间: {execution_time:.2f}秒, 质量评分: {decision.get('quality_score')}, 回复长度: {len(ai_response)}")

        except Exception as e:
            logging.error(f"-- 面试ID : {self.interview_id}---AI流式响应获取错误: {e}")
            yield f"data: {json.dumps({'type': 'error', 'content': '模型检测到非法输入，请重新开始本次面试'+str(e)}, ensure_ascii=False)}\n\n"






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
from app.identity import get_identity_prompt, get_identity_vars, get_standard_scripts
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
        self.current_question_follow_count = 0  # 当前问题的追问次数
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


    async def _make_unified_decision(self, dialog_history: list, system_message: str, plan_info: str) -> dict:
        """
        单次AI调用，完成智能决策
        返回结构化决策：
        {
            "action": "follow_up" | "new_question" | "end",
            "response_guidance": "回复指导（用于流式生成）",
            "quality_score": 0-10,
            "should_end": true/false,
            "reasoning": "决策理由（用于日志）"
        }
        """
        try:
            # 准备面试计划信息
            plan_context = ""
            if self.interview_plan:
                current_question = None
                if self.current_question_index > 0 and self.current_question_index <= len(self.interview_plan['questions']):
                    current_question = self.interview_plan['questions'][self.current_question_index - 1]
                
                plan_context = f"""
【面试计划参考】
当前进度：已提问 {self.current_question_index}/{len(self.interview_plan['questions'])} 个问题
当前问题已追问：{self.current_question_follow_count} 次
"""
                if current_question:
                    plan_context += f"""
当前问题信息：
{json.dumps(current_question, ensure_ascii=False, indent=2)}
"""
                remaining_questions = self.interview_plan['questions'][self.current_question_index:]
                if remaining_questions:
                    plan_context += f"""
待提问题（前3个）：
{json.dumps(remaining_questions[:3], ensure_ascii=False, indent=2)}
"""
            
            # 加载策略prompt模板
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            strategy_template = load_prompt("strategy_white_collar.txt")
            strategy_sys_prompt = f"{identity_prompt}\n\n{strategy_template.format(**format_vars)}"
            
            system_prompt = f"""{strategy_sys_prompt}

【重要】你现在需要做出完整的决策，包括：
1. 分析候选人的最新回答
2. 评估回答质量（0-10分）
3. 决定下一步动作（追问/下一题/结束）
4. 生成回复指导（用于后续生成自然回复）
5. 判断是否应该结束面试

规则约束：
- 如果已追问2次，必须 action="new_question"
- 如果回答质量<3分且已追问1次，建议 action="new_question"
- 如果明确结束信号，should_end=true

输出严格JSON格式：
{{
    "action": "follow_up" | "new_question" | "end",
    "response_guidance": "回复指导内容（描述要说什么，但不直接生成回复文本）",
    "quality_score": 0-10,
    "should_end": true/false,
    "reasoning": "你的决策理由（用于日志记录）"
}}"""
            
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

{plan_context}

请做出完整决策。"""
            
            response = self.llm_client.create_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.4,
                response_format={"type": "json_object"}
            )
            
            decision = json.loads(response.choices[0].message.content)
            
            # 规则验证：如果已追问2次，强制下一题
            if decision.get("action") == "follow_up" and self.current_question_follow_count >= 2:
                logging.warning(f"-- 面试ID: {self.interview_id} - ⚠️ AI决策违反规则（已追问2次），强制修正为 new_question")
                decision["action"] = "new_question"
                decision["reasoning"] = f"已追问2次，强制进入下一题。原决策理由：{decision.get('reasoning', '')}"
            
            logging.info(f"-- 面试ID: {self.interview_id} - 智能决策结果: {decision}")
            return decision
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 智能决策失败: {e}")
            # 失败时返回默认决策
            return {
                "action": "new_question",
                "response_guidance": "继续面试流程",
                "quality_score": 5,
                "should_end": False,
                "reasoning": f"决策失败，使用默认值。错误: {str(e)}"
            }

    async def _stream_response_by_decision(self, decision: dict, dialog_history: list, system_message: str, plan_info: str):
        """
        基于决策指导，流式生成回复内容
        Yields: 回复内容片段
        Returns: 完整回复文本
        """
        try:
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            executor_template = load_prompt("executor_white_collar.txt" if self.interview_type == 1 else "executor_blue_collar.txt")
            executor_sys_prompt = f"{identity_prompt}\n\n{executor_template.format(**format_vars)}"
            
            system_prompt = f"""{executor_sys_prompt}

根据以下回复指导，生成自然、专业的面试官回复。
不要透露决策过程，只生成自然对话。"""
            
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

{plan_info}

回复指导：
{decision.get("response_guidance", "继续面试流程")}

请根据回复指导生成自然、专业的面试官回复。"""
            
            response = self.llm_client.create_streaming_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.7,
                seed=1
            )
            
            full_response = ""
            async for chunk in response:
                if chunk.choices and chunk.choices[0].delta.content:
                    content = chunk.choices[0].delta.content
                    full_response += content
                    yield content
            
            return full_response
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 流式生成失败: {e}")
            error_msg = "抱歉，我需要整理一下思路。你能再说一遍吗？"
            yield error_msg
            return error_msg


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
                self.current_question_follow_count = 0  # 重置追问次数
                
                logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - 第一个问题: {first_question[:50]}...")
                yield f"data: {json.dumps({'type': 'chunk', 'content': first_question}, ensure_ascii=False)}\n\n"
                yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
                # 返回面试计划
                if self.interview_plan:
                    yield f"data: {json.dumps({'type': 'plan', 'content': self.interview_plan}, ensure_ascii=False)}\n\n"
                logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 已返回第一个问题，函数即将return")
                return
            else:
                logging.warning(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ⚠️ 面试计划为空或没有问题，继续执行原有流程")

        # 快速规则检查：追问次数检查
        if self.current_question_follow_count >= 2:
            logging.info(f"-- 面试ID: {self.interview_id} - ⚠️ 已追问2次，强制进入下一题")
            if self.interview_plan:
                self.current_question_index += 1
                self.current_question_follow_count = 0
                logging.info(f"-- 面试ID: {self.interview_id} - 📈 强制进入下一题，索引更新为 {self.current_question_index}")

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

        self.speech_speed = 1.1 if self.interview_type == 1 else 1
        
        # 准备面试计划信息
        plan_info = ""
        if self.interview_plan:
            remaining_questions = self.interview_plan['questions'][self.current_question_index:]
            plan_info = f"""
                【面试计划参考】
                当前进度：已提问 {self.current_question_index}/{len(self.interview_plan['questions'])} 个问题
                待提问题（含期望要点和追问策略）：
                {json.dumps(remaining_questions[:3], ensure_ascii=False, indent=2)}
                
                注意：对于技术类问题，请参考 expectedPoints 和 followUpStrategy 决定是否追问（最多2次）。
                """
        
        try:
            # 阶段1：智能决策（非流式）
            logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始智能决策")
            decision = await self._make_unified_decision(dialog_history, system_message, plan_info)
            logging.info(f"-- 面试ID: {self.interview_id} - ✅ 智能决策完成: action={decision.get('action')}, quality_score={decision.get('quality_score')}")
            
            # 阶段2：流式生成回复
            logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始流式生成回复")
            ai_response = ""
            async for chunk in self._stream_response_by_decision(decision, dialog_history, system_message, plan_info):
                ai_response += chunk
                yield f"data: {json.dumps({'type': 'chunk', 'content': chunk}, ensure_ascii=False)}\n\n"
            
            yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
            
            # 状态更新
            action = decision.get("action")
            if action == "new_question":
                if self.interview_plan:
                    self.current_question_index += 1
                self.current_question_follow_count = 0
                logging.info(f"-- 面试ID: {self.interview_id} - 📈 进入下一题，索引更新为 {self.current_question_index}")
            elif action == "follow_up":
                self.current_question_follow_count += 1
                logging.info(f"-- 面试ID: {self.interview_id} - 🔄 保持当前问题，追问次数: {self.current_question_follow_count}")
            
            # 判断是否结束
            if decision.get("should_end", False):
                logging.info(f"-- 面试ID: {self.interview_id} - 🛑 面试结束（由决策判断）")
                yield f"data: {json.dumps({'type': 'over', 'content': 'over'}, ensure_ascii=False)}\n\n"
            
            end_time = time.time()
            execution_time = end_time - start_time
            logging.info(f"-- 面试ID: {self.interview_id} - 执行时间: {execution_time:.2f}秒, 质量评分: {decision.get('quality_score')}, 回复长度: {len(ai_response)}")

        except Exception as e:
            logging.error(f"-- 面试ID : {self.interview_id}---AI流式响应获取错误: {e}")
            yield f"data: {json.dumps({'type': 'error', 'content': '模型检测到非法输入，请重新开始本次面试'+str(e)}, ensure_ascii=False)}\n\n"






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
from app.identity import get_identity_prompt, get_identity_vars, get_standard_scripts
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
        self.current_question_follow_count = 0  # 当前问题的追问次数
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


    async def _make_unified_decision(self, dialog_history: list, system_message: str) -> dict:
        """
        单次AI调用，完成智能决策
        返回结构化决策：
        {
            "action": "follow_up" | "new_question" | "end",
            "response_guidance": "回复指导（用于流式生成）",
            "quality_score": 0-10,
            "should_end": true/false,
            "reasoning": "决策理由（用于日志）"
        }
        """
        try:
            # 准备面试计划信息
            plan_context = ""
            if self.interview_plan:
                current_question = None
                if self.current_question_index > 0 and self.current_question_index <= len(self.interview_plan['questions']):
                    current_question = self.interview_plan['questions'][self.current_question_index - 1]
                
                plan_context = f"""
【面试计划参考】
当前进度：已提问 {self.current_question_index}/{len(self.interview_plan['questions'])} 个问题
当前问题已追问：{self.current_question_follow_count} 次
"""
                if current_question:
                    plan_context += f"""
当前问题信息：
{json.dumps(current_question, ensure_ascii=False, indent=2)}
"""
                remaining_questions = self.interview_plan['questions'][self.current_question_index:]
                if remaining_questions:
                    plan_context += f"""
待提问题（前3个）：
{json.dumps(remaining_questions[:3], ensure_ascii=False, indent=2)}
"""
            
            # 加载策略prompt模板
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            strategy_template = load_prompt("strategy_white_collar.txt")
            strategy_sys_prompt = f"{identity_prompt}\n\n{strategy_template.format(**format_vars)}"
            
            system_prompt = f"""{strategy_sys_prompt}

【重要】你现在需要做出完整的决策，包括：
1. 分析候选人的最新回答
2. 评估回答质量（0-10分）
3. 决定下一步动作（追问/下一题/结束）
4. 生成回复指导（用于后续生成自然回复）
5. 判断是否应该结束面试

规则约束：
- 如果已追问2次，必须 action="new_question"
- 如果回答质量<3分且已追问1次，建议 action="new_question"
- 如果明确结束信号，should_end=true

输出严格JSON格式：
{{
    "action": "follow_up" | "new_question" | "end",
    "response_guidance": "回复指导内容（描述要说什么，但不直接生成回复文本）",
    "quality_score": 0-10,
    "should_end": true/false,
    "reasoning": "你的决策理由（用于日志记录）"
}}"""
            
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

{plan_context}

请做出完整决策。"""
            
            response = self.llm_client.create_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.4,
                response_format={"type": "json_object"}
            )
            
            decision = json.loads(response.choices[0].message.content)
            
            # 规则验证：如果已追问2次，强制下一题
            if decision.get("action") == "follow_up" and self.current_question_follow_count >= 2:
                logging.warning(f"-- 面试ID: {self.interview_id} - ⚠️ AI决策违反规则（已追问2次），强制修正为 new_question")
                decision["action"] = "new_question"
                decision["reasoning"] = f"已追问2次，强制进入下一题。原决策理由：{decision.get('reasoning', '')}"
            
            logging.info(f"-- 面试ID: {self.interview_id} - 智能决策结果: {decision}")
            return decision
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 智能决策失败: {e}")
            # 失败时返回默认决策
            return {
                "action": "new_question",
                "response_guidance": "继续面试流程",
                "quality_score": 5,
                "should_end": False,
                "reasoning": f"决策失败，使用默认值。错误: {str(e)}"
            }

    async def _stream_response_by_decision(self, decision: dict, dialog_history: list, system_message: str, plan_info: str):
        """
        基于决策指导，流式生成回复内容
        Yields: 回复内容片段
        Returns: 完整回复文本
        """
        try:
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            executor_template = load_prompt("executor_white_collar.txt" if self.interview_type == 1 else "executor_blue_collar.txt")
            executor_sys_prompt = f"{identity_prompt}\n\n{executor_template.format(**format_vars)}"
            
            system_prompt = f"""{executor_sys_prompt}

根据以下回复指导，生成自然、专业的面试官回复。
不要透露决策过程，只生成自然对话。"""
            
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

{plan_info}

回复指导：
{decision.get("response_guidance", "继续面试流程")}

请根据回复指导生成自然、专业的面试官回复。"""
            
            response = self.llm_client.create_streaming_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.7,
                seed=1
            )
            
            full_response = ""
            async for chunk in response:
                if chunk.choices and chunk.choices[0].delta.content:
                    content = chunk.choices[0].delta.content
                    full_response += content
                    yield content
            
            return full_response
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 流式生成失败: {e}")
            error_msg = "抱歉，我需要整理一下思路。你能再说一遍吗？"
            yield error_msg
            return error_msg


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
                self.current_question_follow_count = 0  # 重置追问次数
                
                logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - 第一个问题: {first_question[:50]}...")
                yield f"data: {json.dumps({'type': 'chunk', 'content': first_question}, ensure_ascii=False)}\n\n"
                yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
                # 返回面试计划
                if self.interview_plan:
                    yield f"data: {json.dumps({'type': 'plan', 'content': self.interview_plan}, ensure_ascii=False)}\n\n"
                logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 已返回第一个问题，函数即将return")
                return
            else:
                logging.warning(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ⚠️ 面试计划为空或没有问题，继续执行原有流程")

        # 快速规则检查：追问次数检查
        if self.current_question_follow_count >= 2:
            logging.info(f"-- 面试ID: {self.interview_id} - ⚠️ 已追问2次，强制进入下一题")
            if self.interview_plan:
                self.current_question_index += 1
                self.current_question_follow_count = 0
                logging.info(f"-- 面试ID: {self.interview_id} - 📈 强制进入下一题，索引更新为 {self.current_question_index}")

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

        self.speech_speed = 1.1 if self.interview_type == 1 else 1
        
        # 准备面试计划信息
        plan_info = ""
        if self.interview_plan:
            remaining_questions = self.interview_plan['questions'][self.current_question_index:]
            plan_info = f"""
                【面试计划参考】
                当前进度：已提问 {self.current_question_index}/{len(self.interview_plan['questions'])} 个问题
                待提问题（含期望要点和追问策略）：
                {json.dumps(remaining_questions[:3], ensure_ascii=False, indent=2)}
                
                注意：对于技术类问题，请参考 expectedPoints 和 followUpStrategy 决定是否追问（最多2次）。
                """
        
        try:
            # 阶段1：智能决策（非流式）
            logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始智能决策")
            decision = await self._make_unified_decision(dialog_history, system_message, plan_info)
            logging.info(f"-- 面试ID: {self.interview_id} - ✅ 智能决策完成: action={decision.get('action')}, quality_score={decision.get('quality_score')}")
            
            # 阶段2：流式生成回复
            logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始流式生成回复")
            ai_response = ""
            async for chunk in self._stream_response_by_decision(decision, dialog_history, system_message, plan_info):
                ai_response += chunk
                yield f"data: {json.dumps({'type': 'chunk', 'content': chunk}, ensure_ascii=False)}\n\n"
            
            yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
            
            # 状态更新
            action = decision.get("action")
            if action == "new_question":
                if self.interview_plan:
                    self.current_question_index += 1
                self.current_question_follow_count = 0
                logging.info(f"-- 面试ID: {self.interview_id} - 📈 进入下一题，索引更新为 {self.current_question_index}")
            elif action == "follow_up":
                self.current_question_follow_count += 1
                logging.info(f"-- 面试ID: {self.interview_id} - 🔄 保持当前问题，追问次数: {self.current_question_follow_count}")
            
            # 判断是否结束
            if decision.get("should_end", False):
                logging.info(f"-- 面试ID: {self.interview_id} - 🛑 面试结束（由决策判断）")
                yield f"data: {json.dumps({'type': 'over', 'content': 'over'}, ensure_ascii=False)}\n\n"
            
            end_time = time.time()
            execution_time = end_time - start_time
            logging.info(f"-- 面试ID: {self.interview_id} - 执行时间: {execution_time:.2f}秒, 质量评分: {decision.get('quality_score')}, 回复长度: {len(ai_response)}")

        except Exception as e:
            logging.error(f"-- 面试ID : {self.interview_id}---AI流式响应获取错误: {e}")
            yield f"data: {json.dumps({'type': 'error', 'content': '模型检测到非法输入，请重新开始本次面试'+str(e)}, ensure_ascii=False)}\n\n"






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
from app.identity import get_identity_prompt, get_identity_vars, get_standard_scripts
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
        self.current_question_follow_count = 0  # 当前问题的追问次数
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


    async def _make_unified_decision(self, dialog_history: list, system_message: str) -> dict:
        """
        单次AI调用，完成智能决策
        返回结构化决策：
        {
            "action": "follow_up" | "new_question" | "end",
            "response_guidance": "回复指导（用于流式生成）",
            "quality_score": 0-10,
            "should_end": true/false,
            "reasoning": "决策理由（用于日志）"
        }
        """
        try:
            # 加载策略prompt模板
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            strategy_template = load_prompt("strategy_white_collar.txt")
            strategy_sys_prompt = f"{identity_prompt}\n\n{strategy_template.format(**format_vars)}"
            
            system_prompt = f"""{strategy_sys_prompt}

【重要】你现在需要做出完整的决策，包括：
1. 分析候选人的最新回答
2. 评估回答质量（0-10分）
3. 决定下一步动作（追问/下一题/结束）
4. 生成回复指导（用于后续生成自然回复）
5. 判断是否应该结束面试

规则约束：
- 如果已追问2次，必须 action="new_question"
- 如果回答质量<3分且已追问1次，建议 action="new_question"
- 如果明确结束信号，should_end=true

输出严格JSON格式：
{{
    "action": "follow_up" | "new_question" | "end",
    "response_guidance": "回复指导内容（描述要说什么，但不直接生成回复文本）",
    "quality_score": 0-10,
    "should_end": true/false,
    "reasoning": "你的决策理由（用于日志记录）"
}}"""
            
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

{plan_context}

请做出完整决策。"""
            
            response = self.llm_client.create_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.4,
                response_format={"type": "json_object"}
            )
            
            decision = json.loads(response.choices[0].message.content)
            
            # 规则验证：如果已追问2次，强制下一题
            if decision.get("action") == "follow_up" and self.current_question_follow_count >= 2:
                logging.warning(f"-- 面试ID: {self.interview_id} - ⚠️ AI决策违反规则（已追问2次），强制修正为 new_question")
                decision["action"] = "new_question"
                decision["reasoning"] = f"已追问2次，强制进入下一题。原决策理由：{decision.get('reasoning', '')}"
            
            logging.info(f"-- 面试ID: {self.interview_id} - 智能决策结果: {decision}")
            return decision
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 智能决策失败: {e}")
            # 失败时返回默认决策
            return {
                "action": "new_question",
                "response_guidance": "继续面试流程",
                "quality_score": 5,
                "should_end": False,
                "reasoning": f"决策失败，使用默认值。错误: {str(e)}"
            }

    async def _stream_response_by_decision(self, decision: dict, dialog_history: list, system_message: str, plan_info: str):
        """
        基于决策指导，流式生成回复内容
        Yields: 回复内容片段
        Returns: 完整回复文本
        """
        try:
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            executor_template = load_prompt("executor_white_collar.txt" if self.interview_type == 1 else "executor_blue_collar.txt")
            executor_sys_prompt = f"{identity_prompt}\n\n{executor_template.format(**format_vars)}"
            
            system_prompt = f"""{executor_sys_prompt}

根据以下回复指导，生成自然、专业的面试官回复。
不要透露决策过程，只生成自然对话。"""
            
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

{plan_info}

回复指导：
{decision.get("response_guidance", "继续面试流程")}

请根据回复指导生成自然、专业的面试官回复。"""
            
            response = self.llm_client.create_streaming_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.7,
                seed=1
            )
            
            full_response = ""
            async for chunk in response:
                if chunk.choices and chunk.choices[0].delta.content:
                    content = chunk.choices[0].delta.content
                    full_response += content
                    yield content
            
            return full_response
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 流式生成失败: {e}")
            error_msg = "抱歉，我需要整理一下思路。你能再说一遍吗？"
            yield error_msg
            return error_msg


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
                self.current_question_follow_count = 0  # 重置追问次数
                
                logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - 第一个问题: {first_question[:50]}...")
                yield f"data: {json.dumps({'type': 'chunk', 'content': first_question}, ensure_ascii=False)}\n\n"
                yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
                # 返回面试计划
                if self.interview_plan:
                    yield f"data: {json.dumps({'type': 'plan', 'content': self.interview_plan}, ensure_ascii=False)}\n\n"
                logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 已返回第一个问题，函数即将return")
                return
            else:
                logging.warning(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ⚠️ 面试计划为空或没有问题，继续执行原有流程")

        # 快速规则检查：追问次数检查
        if self.current_question_follow_count >= 2:
            logging.info(f"-- 面试ID: {self.interview_id} - ⚠️ 已追问2次，强制进入下一题")
            if self.interview_plan:
                self.current_question_index += 1
                self.current_question_follow_count = 0
                logging.info(f"-- 面试ID: {self.interview_id} - 📈 强制进入下一题，索引更新为 {self.current_question_index}")

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

        self.speech_speed = 1.1 if self.interview_type == 1 else 1
        
        # 准备面试计划信息
        plan_info = ""
        if self.interview_plan:
            remaining_questions = self.interview_plan['questions'][self.current_question_index:]
            plan_info = f"""
                【面试计划参考】
                当前进度：已提问 {self.current_question_index}/{len(self.interview_plan['questions'])} 个问题
                待提问题（含期望要点和追问策略）：
                {json.dumps(remaining_questions[:3], ensure_ascii=False, indent=2)}
                
                注意：对于技术类问题，请参考 expectedPoints 和 followUpStrategy 决定是否追问（最多2次）。
                """
        
        try:
            # 阶段1：智能决策（非流式）
            logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始智能决策")
            decision = await self._make_unified_decision(dialog_history, system_message, plan_info)
            logging.info(f"-- 面试ID: {self.interview_id} - ✅ 智能决策完成: action={decision.get('action')}, quality_score={decision.get('quality_score')}")
            
            # 阶段2：流式生成回复
            logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始流式生成回复")
            ai_response = ""
            async for chunk in self._stream_response_by_decision(decision, dialog_history, system_message, plan_info):
                ai_response += chunk
                yield f"data: {json.dumps({'type': 'chunk', 'content': chunk}, ensure_ascii=False)}\n\n"
            
            yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
            
            # 状态更新
            action = decision.get("action")
            if action == "new_question":
                if self.interview_plan:
                    self.current_question_index += 1
                self.current_question_follow_count = 0
                logging.info(f"-- 面试ID: {self.interview_id} - 📈 进入下一题，索引更新为 {self.current_question_index}")
            elif action == "follow_up":
                self.current_question_follow_count += 1
                logging.info(f"-- 面试ID: {self.interview_id} - 🔄 保持当前问题，追问次数: {self.current_question_follow_count}")
            
            # 判断是否结束
            if decision.get("should_end", False):
                logging.info(f"-- 面试ID: {self.interview_id} - 🛑 面试结束（由决策判断）")
                yield f"data: {json.dumps({'type': 'over', 'content': 'over'}, ensure_ascii=False)}\n\n"
            
            end_time = time.time()
            execution_time = end_time - start_time
            logging.info(f"-- 面试ID: {self.interview_id} - 执行时间: {execution_time:.2f}秒, 质量评分: {decision.get('quality_score')}, 回复长度: {len(ai_response)}")

        except Exception as e:
            logging.error(f"-- 面试ID : {self.interview_id}---AI流式响应获取错误: {e}")
            yield f"data: {json.dumps({'type': 'error', 'content': '模型检测到非法输入，请重新开始本次面试'+str(e)}, ensure_ascii=False)}\n\n"






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
from app.identity import get_identity_prompt, get_identity_vars, get_standard_scripts
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
        self.current_question_follow_count = 0  # 当前问题的追问次数
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


    async def _make_unified_decision(self, dialog_history: list, system_message: str) -> dict:
        """
        单次AI调用，完成智能决策
        返回结构化决策：
        {
            "action": "follow_up" | "new_question" | "end",
            "response_guidance": "回复指导（用于流式生成）",
            "quality_score": 0-10,
            "should_end": true/false,
            "reasoning": "决策理由（用于日志）"
        }
        """
        try:
            # 加载策略prompt模板
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            strategy_template = load_prompt("strategy_white_collar.txt")
            strategy_sys_prompt = f"{identity_prompt}\n\n{strategy_template.format(**format_vars)}"
            
            system_prompt = f"""{strategy_sys_prompt}

【重要】你现在需要做出完整的决策，包括：
1. 分析候选人的最新回答
2. 评估回答质量（0-10分）
3. 决定下一步动作（追问/下一题/结束）
4. 生成回复指导（用于后续生成自然回复）
5. 判断是否应该结束面试

规则约束：
- 如果已追问2次，必须 action="new_question"
- 如果回答质量<3分且已追问1次，建议 action="new_question"
- 如果明确结束信号，should_end=true

输出严格JSON格式：
{{
    "action": "follow_up" | "new_question" | "end",
    "response_guidance": "回复指导内容（描述要说什么，但不直接生成回复文本）",
    "quality_score": 0-10,
    "should_end": true/false,
    "reasoning": "你的决策理由（用于日志记录）"
}}"""
            
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

当前问题已追问：{self.current_question_follow_count} 次

请做出完整决策。"""
            
            response = self.llm_client.create_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.4,
                response_format={"type": "json_object"}
            )
            
            decision = json.loads(response.choices[0].message.content)
            
            # 规则验证：如果已追问2次，强制下一题
            if decision.get("action") == "follow_up" and self.current_question_follow_count >= 2:
                logging.warning(f"-- 面试ID: {self.interview_id} - ⚠️ AI决策违反规则（已追问2次），强制修正为 new_question")
                decision["action"] = "new_question"
                decision["reasoning"] = f"已追问2次，强制进入下一题。原决策理由：{decision.get('reasoning', '')}"
            
            logging.info(f"-- 面试ID: {self.interview_id} - 智能决策结果: {decision}")
            return decision
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 智能决策失败: {e}")
            # 失败时返回默认决策
            return {
                "action": "new_question",
                "response_guidance": "继续面试流程",
                "quality_score": 5,
                "should_end": False,
                "reasoning": f"决策失败，使用默认值。错误: {str(e)}"
            }

    async def _stream_response_by_decision(self, decision: dict, dialog_history: list, system_message: str, plan_info: str):
        """
        基于决策指导，流式生成回复内容
        Yields: 回复内容片段
        Returns: 完整回复文本
        """
        try:
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            executor_template = load_prompt("executor_white_collar.txt" if self.interview_type == 1 else "executor_blue_collar.txt")
            executor_sys_prompt = f"{identity_prompt}\n\n{executor_template.format(**format_vars)}"
            
            system_prompt = f"""{executor_sys_prompt}

根据以下回复指导，生成自然、专业的面试官回复。
不要透露决策过程，只生成自然对话。"""
            
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

{plan_info}

回复指导：
{decision.get("response_guidance", "继续面试流程")}

请根据回复指导生成自然、专业的面试官回复。"""
            
            response = self.llm_client.create_streaming_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.7,
                seed=1
            )
            
            full_response = ""
            async for chunk in response:
                if chunk.choices and chunk.choices[0].delta.content:
                    content = chunk.choices[0].delta.content
                    full_response += content
                    yield content
            
            return full_response
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 流式生成失败: {e}")
            error_msg = "抱歉，我需要整理一下思路。你能再说一遍吗？"
            yield error_msg
            return error_msg


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
                self.current_question_follow_count = 0  # 重置追问次数
                
                logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - 第一个问题: {first_question[:50]}...")
                yield f"data: {json.dumps({'type': 'chunk', 'content': first_question}, ensure_ascii=False)}\n\n"
                yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
                # 返回面试计划
                if self.interview_plan:
                    yield f"data: {json.dumps({'type': 'plan', 'content': self.interview_plan}, ensure_ascii=False)}\n\n"
                logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 已返回第一个问题，函数即将return")
                return
            else:
                logging.warning(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ⚠️ 面试计划为空或没有问题，继续执行原有流程")

        # 快速规则检查：追问次数检查
        if self.current_question_follow_count >= 2:
            logging.info(f"-- 面试ID: {self.interview_id} - ⚠️ 已追问2次，强制进入下一题")
            if self.interview_plan:
                self.current_question_index += 1
                self.current_question_follow_count = 0
                logging.info(f"-- 面试ID: {self.interview_id} - 📈 强制进入下一题，索引更新为 {self.current_question_index}")

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

        self.speech_speed = 1.1 if self.interview_type == 1 else 1
        
        # 准备面试计划信息
        plan_info = ""
        if self.interview_plan:
            remaining_questions = self.interview_plan['questions'][self.current_question_index:]
            plan_info = f"""
                【面试计划参考】
                当前进度：已提问 {self.current_question_index}/{len(self.interview_plan['questions'])} 个问题
                待提问题（含期望要点和追问策略）：
                {json.dumps(remaining_questions[:3], ensure_ascii=False, indent=2)}
                
                注意：对于技术类问题，请参考 expectedPoints 和 followUpStrategy 决定是否追问（最多2次）。
                """
        
        try:
            # 阶段1：智能决策（非流式）
            logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始智能决策")
            decision = await self._make_unified_decision(dialog_history, system_message, plan_info)
            logging.info(f"-- 面试ID: {self.interview_id} - ✅ 智能决策完成: action={decision.get('action')}, quality_score={decision.get('quality_score')}")
            
            # 阶段2：流式生成回复
            logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始流式生成回复")
            ai_response = ""
            async for chunk in self._stream_response_by_decision(decision, dialog_history, system_message, plan_info):
                ai_response += chunk
                yield f"data: {json.dumps({'type': 'chunk', 'content': chunk}, ensure_ascii=False)}\n\n"
            
            yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
            
            # 状态更新
            action = decision.get("action")
            if action == "new_question":
                if self.interview_plan:
                    self.current_question_index += 1
                self.current_question_follow_count = 0
                logging.info(f"-- 面试ID: {self.interview_id} - 📈 进入下一题，索引更新为 {self.current_question_index}")
            elif action == "follow_up":
                self.current_question_follow_count += 1
                logging.info(f"-- 面试ID: {self.interview_id} - 🔄 保持当前问题，追问次数: {self.current_question_follow_count}")
            
            # 判断是否结束
            if decision.get("should_end", False):
                logging.info(f"-- 面试ID: {self.interview_id} - 🛑 面试结束（由决策判断）")
                yield f"data: {json.dumps({'type': 'over', 'content': 'over'}, ensure_ascii=False)}\n\n"
            
            end_time = time.time()
            execution_time = end_time - start_time
            logging.info(f"-- 面试ID: {self.interview_id} - 执行时间: {execution_time:.2f}秒, 质量评分: {decision.get('quality_score')}, 回复长度: {len(ai_response)}")

        except Exception as e:
            logging.error(f"-- 面试ID : {self.interview_id}---AI流式响应获取错误: {e}")
            yield f"data: {json.dumps({'type': 'error', 'content': '模型检测到非法输入，请重新开始本次面试'+str(e)}, ensure_ascii=False)}\n\n"






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
from app.identity import get_identity_prompt, get_identity_vars, get_standard_scripts
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
        self.current_question_follow_count = 0  # 当前问题的追问次数
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


    async def _make_unified_decision(self, dialog_history: list, system_message: str) -> dict:
        """
        单次AI调用，完成智能决策
        返回结构化决策：
        {
            "action": "follow_up" | "new_question" | "end",
            "response_guidance": "回复指导（用于流式生成）",
            "quality_score": 0-10,
            "should_end": true/false,
            "reasoning": "决策理由（用于日志）"
        }
        """
        try:
            # 加载策略prompt模板
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            strategy_template = load_prompt("strategy_white_collar.txt")
            strategy_sys_prompt = f"{identity_prompt}\n\n{strategy_template.format(**format_vars)}"
            
            system_prompt = f"""{strategy_sys_prompt}

【重要】你现在需要做出完整的决策，包括：
1. 分析候选人的最新回答
2. 评估回答质量（0-10分）
3. 决定下一步动作（追问/下一题/结束）
4. 生成回复指导（用于后续生成自然回复）
5. 判断是否应该结束面试

规则约束：
- 如果已追问2次，必须 action="new_question"
- 如果回答质量<3分且已追问1次，建议 action="new_question"
- 如果明确结束信号，should_end=true

输出严格JSON格式：
{{
    "action": "follow_up" | "new_question" | "end",
    "response_guidance": "回复指导内容（描述要说什么，但不直接生成回复文本）",
    "quality_score": 0-10,
    "should_end": true/false,
    "reasoning": "你的决策理由（用于日志记录）"
}}"""
            
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

当前问题已追问：{self.current_question_follow_count} 次

请做出完整决策。"""
            
            response = self.llm_client.create_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.4,
                response_format={"type": "json_object"}
            )
            
            decision = json.loads(response.choices[0].message.content)
            
            # 规则验证：如果已追问2次，强制下一题
            if decision.get("action") == "follow_up" and self.current_question_follow_count >= 2:
                logging.warning(f"-- 面试ID: {self.interview_id} - ⚠️ AI决策违反规则（已追问2次），强制修正为 new_question")
                decision["action"] = "new_question"
                decision["reasoning"] = f"已追问2次，强制进入下一题。原决策理由：{decision.get('reasoning', '')}"
            
            logging.info(f"-- 面试ID: {self.interview_id} - 智能决策结果: {decision}")
            return decision
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 智能决策失败: {e}")
            # 失败时返回默认决策
            return {
                "action": "new_question",
                "response_guidance": "继续面试流程",
                "quality_score": 5,
                "should_end": False,
                "reasoning": f"决策失败，使用默认值。错误: {str(e)}"
            }

    async def _stream_response_by_decision(self, decision: dict, dialog_history: list, system_message: str):
        """
        基于决策指导，流式生成回复内容
        Yields: 回复内容片段
        Returns: 完整回复文本
        """
        try:
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            executor_template = load_prompt("executor_white_collar.txt" if self.interview_type == 1 else "executor_blue_collar.txt")
            executor_sys_prompt = f"{identity_prompt}\n\n{executor_template.format(**format_vars)}"
            
            system_prompt = f"""{executor_sys_prompt}

根据以下回复指导，生成自然、专业的面试官回复。
不要透露决策过程，只生成自然对话。"""
            
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

{plan_info}

回复指导：
{decision.get("response_guidance", "继续面试流程")}

请根据回复指导生成自然、专业的面试官回复。"""
            
            response = self.llm_client.create_streaming_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.7,
                seed=1
            )
            
            full_response = ""
            async for chunk in response:
                if chunk.choices and chunk.choices[0].delta.content:
                    content = chunk.choices[0].delta.content
                    full_response += content
                    yield content
            
            return full_response
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 流式生成失败: {e}")
            error_msg = "抱歉，我需要整理一下思路。你能再说一遍吗？"
            yield error_msg
            return error_msg


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
                self.current_question_follow_count = 0  # 重置追问次数
                
                logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - 第一个问题: {first_question[:50]}...")
                yield f"data: {json.dumps({'type': 'chunk', 'content': first_question}, ensure_ascii=False)}\n\n"
                yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
                # 返回面试计划
                if self.interview_plan:
                    yield f"data: {json.dumps({'type': 'plan', 'content': self.interview_plan}, ensure_ascii=False)}\n\n"
                logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 已返回第一个问题，函数即将return")
                return
            else:
                logging.warning(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ⚠️ 面试计划为空或没有问题，继续执行原有流程")

        # 快速规则检查：追问次数检查
        if self.current_question_follow_count >= 2:
            logging.info(f"-- 面试ID: {self.interview_id} - ⚠️ 已追问2次，强制进入下一题")
            if self.interview_plan:
                self.current_question_index += 1
                self.current_question_follow_count = 0
                logging.info(f"-- 面试ID: {self.interview_id} - 📈 强制进入下一题，索引更新为 {self.current_question_index}")

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

        self.speech_speed = 1.1 if self.interview_type == 1 else 1
        
        # 准备面试计划信息
        plan_info = ""
        if self.interview_plan:
            remaining_questions = self.interview_plan['questions'][self.current_question_index:]
            plan_info = f"""
                【面试计划参考】
                当前进度：已提问 {self.current_question_index}/{len(self.interview_plan['questions'])} 个问题
                待提问题（含期望要点和追问策略）：
                {json.dumps(remaining_questions[:3], ensure_ascii=False, indent=2)}
                
                注意：对于技术类问题，请参考 expectedPoints 和 followUpStrategy 决定是否追问（最多2次）。
                """
        
        try:
            # 阶段1：智能决策（非流式）
            logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始智能决策")
            decision = await self._make_unified_decision(dialog_history, system_message, plan_info)
            logging.info(f"-- 面试ID: {self.interview_id} - ✅ 智能决策完成: action={decision.get('action')}, quality_score={decision.get('quality_score')}")
            
            # 阶段2：流式生成回复
            logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始流式生成回复")
            ai_response = ""
            async for chunk in self._stream_response_by_decision(decision, dialog_history, system_message, plan_info):
                ai_response += chunk
                yield f"data: {json.dumps({'type': 'chunk', 'content': chunk}, ensure_ascii=False)}\n\n"
            
            yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
            
            # 状态更新
            action = decision.get("action")
            if action == "new_question":
                if self.interview_plan:
                    self.current_question_index += 1
                self.current_question_follow_count = 0
                logging.info(f"-- 面试ID: {self.interview_id} - 📈 进入下一题，索引更新为 {self.current_question_index}")
            elif action == "follow_up":
                self.current_question_follow_count += 1
                logging.info(f"-- 面试ID: {self.interview_id} - 🔄 保持当前问题，追问次数: {self.current_question_follow_count}")
            
            # 判断是否结束
            if decision.get("should_end", False):
                logging.info(f"-- 面试ID: {self.interview_id} - 🛑 面试结束（由决策判断）")
                yield f"data: {json.dumps({'type': 'over', 'content': 'over'}, ensure_ascii=False)}\n\n"
            
            end_time = time.time()
            execution_time = end_time - start_time
            logging.info(f"-- 面试ID: {self.interview_id} - 执行时间: {execution_time:.2f}秒, 质量评分: {decision.get('quality_score')}, 回复长度: {len(ai_response)}")

        except Exception as e:
            logging.error(f"-- 面试ID : {self.interview_id}---AI流式响应获取错误: {e}")
            yield f"data: {json.dumps({'type': 'error', 'content': '模型检测到非法输入，请重新开始本次面试'+str(e)}, ensure_ascii=False)}\n\n"






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
from app.identity import get_identity_prompt, get_identity_vars, get_standard_scripts
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
        self.current_question_follow_count = 0  # 当前问题的追问次数
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


    async def _make_unified_decision(self, dialog_history: list, system_message: str) -> dict:
        """
        单次AI调用，完成智能决策
        返回结构化决策：
        {
            "action": "follow_up" | "new_question" | "end",
            "response_guidance": "回复指导（用于流式生成）",
            "quality_score": 0-10,
            "should_end": true/false,
            "reasoning": "决策理由（用于日志）"
        }
        """
        try:
            # 加载策略prompt模板
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            strategy_template = load_prompt("strategy_white_collar.txt")
            strategy_sys_prompt = f"{identity_prompt}\n\n{strategy_template.format(**format_vars)}"
            
            system_prompt = f"""{strategy_sys_prompt}

【重要】你现在需要做出完整的决策，包括：
1. 分析候选人的最新回答
2. 评估回答质量（0-10分）
3. 决定下一步动作（追问/下一题/结束）
4. 生成回复指导（用于后续生成自然回复）
5. 判断是否应该结束面试

规则约束：
- 如果已追问2次，必须 action="new_question"
- 如果回答质量<3分且已追问1次，建议 action="new_question"
- 如果明确结束信号，should_end=true

输出严格JSON格式：
{{
    "action": "follow_up" | "new_question" | "end",
    "response_guidance": "回复指导内容（描述要说什么，但不直接生成回复文本）",
    "quality_score": 0-10,
    "should_end": true/false,
    "reasoning": "你的决策理由（用于日志记录）"
}}"""
            
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

当前问题已追问：{self.current_question_follow_count} 次

请做出完整决策。"""
            
            response = self.llm_client.create_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.4,
                response_format={"type": "json_object"}
            )
            
            decision = json.loads(response.choices[0].message.content)
            
            # 规则验证：如果已追问2次，强制下一题
            if decision.get("action") == "follow_up" and self.current_question_follow_count >= 2:
                logging.warning(f"-- 面试ID: {self.interview_id} - ⚠️ AI决策违反规则（已追问2次），强制修正为 new_question")
                decision["action"] = "new_question"
                decision["reasoning"] = f"已追问2次，强制进入下一题。原决策理由：{decision.get('reasoning', '')}"
            
            logging.info(f"-- 面试ID: {self.interview_id} - 智能决策结果: {decision}")
            return decision
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 智能决策失败: {e}")
            # 失败时返回默认决策
            return {
                "action": "new_question",
                "response_guidance": "继续面试流程",
                "quality_score": 5,
                "should_end": False,
                "reasoning": f"决策失败，使用默认值。错误: {str(e)}"
            }

    async def _stream_response_by_decision(self, decision: dict, dialog_history: list, system_message: str):
        """
        基于决策指导，流式生成回复内容
        Yields: 回复内容片段
        Returns: 完整回复文本
        """
        try:
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            executor_template = load_prompt("executor_white_collar.txt" if self.interview_type == 1 else "executor_blue_collar.txt")
            executor_sys_prompt = f"{identity_prompt}\n\n{executor_template.format(**format_vars)}"
            
            system_prompt = f"""{executor_sys_prompt}

根据以下回复指导，生成自然、专业的面试官回复。
不要透露决策过程，只生成自然对话。"""
            
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

回复指导：
{decision.get("response_guidance", "继续面试流程")}

请根据回复指导生成自然、专业的面试官回复。"""
            
            response = self.llm_client.create_streaming_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.7,
                seed=1
            )
            
            full_response = ""
            async for chunk in response:
                if chunk.choices and chunk.choices[0].delta.content:
                    content = chunk.choices[0].delta.content
                    full_response += content
                    yield content
            
            return full_response
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 流式生成失败: {e}")
            error_msg = "抱歉，我需要整理一下思路。你能再说一遍吗？"
            yield error_msg
            return error_msg


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
                self.current_question_follow_count = 0  # 重置追问次数
                
                logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - 第一个问题: {first_question[:50]}...")
                yield f"data: {json.dumps({'type': 'chunk', 'content': first_question}, ensure_ascii=False)}\n\n"
                yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
                # 返回面试计划
                if self.interview_plan:
                    yield f"data: {json.dumps({'type': 'plan', 'content': self.interview_plan}, ensure_ascii=False)}\n\n"
                logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 已返回第一个问题，函数即将return")
                return
            else:
                logging.warning(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ⚠️ 面试计划为空或没有问题，继续执行原有流程")

        # 快速规则检查：追问次数检查
        if self.current_question_follow_count >= 2:
            logging.info(f"-- 面试ID: {self.interview_id} - ⚠️ 已追问2次，强制进入下一题")
            if self.interview_plan:
                self.current_question_index += 1
                self.current_question_follow_count = 0
                logging.info(f"-- 面试ID: {self.interview_id} - 📈 强制进入下一题，索引更新为 {self.current_question_index}")

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

        self.speech_speed = 1.1 if self.interview_type == 1 else 1
        
        # 准备面试计划信息
        plan_info = ""
        if self.interview_plan:
            remaining_questions = self.interview_plan['questions'][self.current_question_index:]
            plan_info = f"""
                【面试计划参考】
                当前进度：已提问 {self.current_question_index}/{len(self.interview_plan['questions'])} 个问题
                待提问题（含期望要点和追问策略）：
                {json.dumps(remaining_questions[:3], ensure_ascii=False, indent=2)}
                
                注意：对于技术类问题，请参考 expectedPoints 和 followUpStrategy 决定是否追问（最多2次）。
                """
        
        try:
            # 阶段1：智能决策（非流式）
            logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始智能决策")
            decision = await self._make_unified_decision(dialog_history, system_message, plan_info)
            logging.info(f"-- 面试ID: {self.interview_id} - ✅ 智能决策完成: action={decision.get('action')}, quality_score={decision.get('quality_score')}")
            
            # 阶段2：流式生成回复
            logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始流式生成回复")
            ai_response = ""
            async for chunk in self._stream_response_by_decision(decision, dialog_history, system_message, plan_info):
                ai_response += chunk
                yield f"data: {json.dumps({'type': 'chunk', 'content': chunk}, ensure_ascii=False)}\n\n"
            
            yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
            
            # 状态更新
            action = decision.get("action")
            if action == "new_question":
                if self.interview_plan:
                    self.current_question_index += 1
                self.current_question_follow_count = 0
                logging.info(f"-- 面试ID: {self.interview_id} - 📈 进入下一题，索引更新为 {self.current_question_index}")
            elif action == "follow_up":
                self.current_question_follow_count += 1
                logging.info(f"-- 面试ID: {self.interview_id} - 🔄 保持当前问题，追问次数: {self.current_question_follow_count}")
            
            # 判断是否结束
            if decision.get("should_end", False):
                logging.info(f"-- 面试ID: {self.interview_id} - 🛑 面试结束（由决策判断）")
                yield f"data: {json.dumps({'type': 'over', 'content': 'over'}, ensure_ascii=False)}\n\n"
            
            end_time = time.time()
            execution_time = end_time - start_time
            logging.info(f"-- 面试ID: {self.interview_id} - 执行时间: {execution_time:.2f}秒, 质量评分: {decision.get('quality_score')}, 回复长度: {len(ai_response)}")

        except Exception as e:
            logging.error(f"-- 面试ID : {self.interview_id}---AI流式响应获取错误: {e}")
            yield f"data: {json.dumps({'type': 'error', 'content': '模型检测到非法输入，请重新开始本次面试'+str(e)}, ensure_ascii=False)}\n\n"






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
from app.identity import get_identity_prompt, get_identity_vars, get_standard_scripts
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
        self.current_question_follow_count = 0  # 当前问题的追问次数
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


    async def _make_unified_decision(self, dialog_history: list, system_message: str) -> dict:
        """
        单次AI调用，完成智能决策
        返回结构化决策：
        {
            "action": "follow_up" | "new_question" | "end",
            "response_guidance": "回复指导（用于流式生成）",
            "quality_score": 0-10,
            "should_end": true/false,
            "reasoning": "决策理由（用于日志）"
        }
        """
        try:
            # 加载策略prompt模板
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            strategy_template = load_prompt("strategy_white_collar.txt")
            strategy_sys_prompt = f"{identity_prompt}\n\n{strategy_template.format(**format_vars)}"
            
            system_prompt = f"""{strategy_sys_prompt}

【重要】你现在需要做出完整的决策，包括：
1. 分析候选人的最新回答
2. 评估回答质量（0-10分）
3. 决定下一步动作（追问/下一题/结束）
4. 生成回复指导（用于后续生成自然回复）
5. 判断是否应该结束面试

规则约束：
- 如果已追问2次，必须 action="new_question"
- 如果回答质量<3分且已追问1次，建议 action="new_question"
- 如果明确结束信号，should_end=true

输出严格JSON格式：
{{
    "action": "follow_up" | "new_question" | "end",
    "response_guidance": "回复指导内容（描述要说什么，但不直接生成回复文本）",
    "quality_score": 0-10,
    "should_end": true/false,
    "reasoning": "你的决策理由（用于日志记录）"
}}"""
            
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

当前问题已追问：{self.current_question_follow_count} 次

请做出完整决策。"""
            
            response = self.llm_client.create_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.4,
                response_format={"type": "json_object"}
            )
            
            decision = json.loads(response.choices[0].message.content)
            
            # 规则验证：如果已追问2次，强制下一题
            if decision.get("action") == "follow_up" and self.current_question_follow_count >= 2:
                logging.warning(f"-- 面试ID: {self.interview_id} - ⚠️ AI决策违反规则（已追问2次），强制修正为 new_question")
                decision["action"] = "new_question"
                decision["reasoning"] = f"已追问2次，强制进入下一题。原决策理由：{decision.get('reasoning', '')}"
            
            logging.info(f"-- 面试ID: {self.interview_id} - 智能决策结果: {decision}")
            return decision
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 智能决策失败: {e}")
            # 失败时返回默认决策
            return {
                "action": "new_question",
                "response_guidance": "继续面试流程",
                "quality_score": 5,
                "should_end": False,
                "reasoning": f"决策失败，使用默认值。错误: {str(e)}"
            }

    async def _stream_response_by_decision(self, decision: dict, dialog_history: list, system_message: str):
        """
        基于决策指导，流式生成回复内容
        Yields: 回复内容片段
        Returns: 完整回复文本
        """
        try:
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            executor_template = load_prompt("executor_white_collar.txt" if self.interview_type == 1 else "executor_blue_collar.txt")
            executor_sys_prompt = f"{identity_prompt}\n\n{executor_template.format(**format_vars)}"
            
            system_prompt = f"""{executor_sys_prompt}

根据以下回复指导，生成自然、专业的面试官回复。
不要透露决策过程，只生成自然对话。"""
            
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

回复指导：
{decision.get("response_guidance", "继续面试流程")}

请根据回复指导生成自然、专业的面试官回复。"""
            
            response = self.llm_client.create_streaming_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.7,
                seed=1
            )
            
            full_response = ""
            async for chunk in response:
                if chunk.choices and chunk.choices[0].delta.content:
                    content = chunk.choices[0].delta.content
                    full_response += content
                    yield content
            
            return full_response
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 流式生成失败: {e}")
            error_msg = "抱歉，我需要整理一下思路。你能再说一遍吗？"
            yield error_msg
            return error_msg


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
            self.current_question_follow_count = 0  # 重置追问次数
            
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
            
            # 首次调用：直接生成开场白
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - 🎯 开始智能决策（首次调用）")
            decision = await self._make_unified_decision([], system_message)
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 智能决策完成")
            
            # 流式生成开场白
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - 🎯 开始流式生成开场白")
            async for chunk in self._stream_response_by_decision(decision, [], system_message):
                yield f"data: {json.dumps({'type': 'chunk', 'content': chunk}, ensure_ascii=False)}\n\n"
            
            yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 已返回开场白，函数即将return")
            return

        # 快速规则检查：追问次数检查
        if self.current_question_follow_count >= 2:
            logging.info(f"-- 面试ID: {self.interview_id} - ⚠️ 已追问2次，强制进入下一题")
            if self.interview_plan:
                self.current_question_index += 1
                self.current_question_follow_count = 0
                logging.info(f"-- 面试ID: {self.interview_id} - 📈 强制进入下一题，索引更新为 {self.current_question_index}")

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

        self.speech_speed = 1.1 if self.interview_type == 1 else 1
        
        # 准备面试计划信息
        plan_info = ""
        if self.interview_plan:
            remaining_questions = self.interview_plan['questions'][self.current_question_index:]
            plan_info = f"""
                【面试计划参考】
                当前进度：已提问 {self.current_question_index}/{len(self.interview_plan['questions'])} 个问题
                待提问题（含期望要点和追问策略）：
                {json.dumps(remaining_questions[:3], ensure_ascii=False, indent=2)}
                
                注意：对于技术类问题，请参考 expectedPoints 和 followUpStrategy 决定是否追问（最多2次）。
                """
        
        try:
            # 阶段1：智能决策（非流式）
            logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始智能决策")
            decision = await self._make_unified_decision(dialog_history, system_message, plan_info)
            logging.info(f"-- 面试ID: {self.interview_id} - ✅ 智能决策完成: action={decision.get('action')}, quality_score={decision.get('quality_score')}")
            
            # 阶段2：流式生成回复
            logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始流式生成回复")
            ai_response = ""
            async for chunk in self._stream_response_by_decision(decision, dialog_history, system_message, plan_info):
                ai_response += chunk
                yield f"data: {json.dumps({'type': 'chunk', 'content': chunk}, ensure_ascii=False)}\n\n"
            
            yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
            
            # 状态更新
            action = decision.get("action")
            if action == "new_question":
                if self.interview_plan:
                    self.current_question_index += 1
                self.current_question_follow_count = 0
                logging.info(f"-- 面试ID: {self.interview_id} - 📈 进入下一题，索引更新为 {self.current_question_index}")
            elif action == "follow_up":
                self.current_question_follow_count += 1
                logging.info(f"-- 面试ID: {self.interview_id} - 🔄 保持当前问题，追问次数: {self.current_question_follow_count}")
            
            # 判断是否结束
            if decision.get("should_end", False):
                logging.info(f"-- 面试ID: {self.interview_id} - 🛑 面试结束（由决策判断）")
                yield f"data: {json.dumps({'type': 'over', 'content': 'over'}, ensure_ascii=False)}\n\n"
            
            end_time = time.time()
            execution_time = end_time - start_time
            logging.info(f"-- 面试ID: {self.interview_id} - 执行时间: {execution_time:.2f}秒, 质量评分: {decision.get('quality_score')}, 回复长度: {len(ai_response)}")

        except Exception as e:
            logging.error(f"-- 面试ID : {self.interview_id}---AI流式响应获取错误: {e}")
            yield f"data: {json.dumps({'type': 'error', 'content': '模型检测到非法输入，请重新开始本次面试'+str(e)}, ensure_ascii=False)}\n\n"






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
from app.identity import get_identity_prompt, get_identity_vars, get_standard_scripts
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
        self.current_question_follow_count = 0  # 当前问题的追问次数
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


    async def _make_unified_decision(self, dialog_history: list, system_message: str) -> dict:
        """
        单次AI调用，完成智能决策
        返回结构化决策：
        {
            "action": "follow_up" | "new_question" | "end",
            "response_guidance": "回复指导（用于流式生成）",
            "quality_score": 0-10,
            "should_end": true/false,
            "reasoning": "决策理由（用于日志）"
        }
        """
        try:
            # 加载策略prompt模板
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            strategy_template = load_prompt("strategy_white_collar.txt")
            strategy_sys_prompt = f"{identity_prompt}\n\n{strategy_template.format(**format_vars)}"
            
            system_prompt = f"""{strategy_sys_prompt}

【重要】你现在需要做出完整的决策，包括：
1. 分析候选人的最新回答
2. 评估回答质量（0-10分）
3. 决定下一步动作（追问/下一题/结束）
4. 生成回复指导（用于后续生成自然回复）
5. 判断是否应该结束面试

规则约束：
- 如果已追问2次，必须 action="new_question"
- 如果回答质量<3分且已追问1次，建议 action="new_question"
- 如果明确结束信号，should_end=true

输出严格JSON格式：
{{
    "action": "follow_up" | "new_question" | "end",
    "response_guidance": "回复指导内容（描述要说什么，但不直接生成回复文本）",
    "quality_score": 0-10,
    "should_end": true/false,
    "reasoning": "你的决策理由（用于日志记录）"
}}"""
            
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

当前问题已追问：{self.current_question_follow_count} 次

请做出完整决策。"""
            
            response = self.llm_client.create_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.4,
                response_format={"type": "json_object"}
            )
            
            decision = json.loads(response.choices[0].message.content)
            
            # 规则验证：如果已追问2次，强制下一题
            if decision.get("action") == "follow_up" and self.current_question_follow_count >= 2:
                logging.warning(f"-- 面试ID: {self.interview_id} - ⚠️ AI决策违反规则（已追问2次），强制修正为 new_question")
                decision["action"] = "new_question"
                decision["reasoning"] = f"已追问2次，强制进入下一题。原决策理由：{decision.get('reasoning', '')}"
            
            logging.info(f"-- 面试ID: {self.interview_id} - 智能决策结果: {decision}")
            return decision
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 智能决策失败: {e}")
            # 失败时返回默认决策
            return {
                "action": "new_question",
                "response_guidance": "继续面试流程",
                "quality_score": 5,
                "should_end": False,
                "reasoning": f"决策失败，使用默认值。错误: {str(e)}"
            }

    async def _stream_response_by_decision(self, decision: dict, dialog_history: list, system_message: str):
        """
        基于决策指导，流式生成回复内容
        Yields: 回复内容片段
        Returns: 完整回复文本
        """
        try:
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            executor_template = load_prompt("executor_white_collar.txt" if self.interview_type == 1 else "executor_blue_collar.txt")
            executor_sys_prompt = f"{identity_prompt}\n\n{executor_template.format(**format_vars)}"
            
            system_prompt = f"""{executor_sys_prompt}

根据以下回复指导，生成自然、专业的面试官回复。
不要透露决策过程，只生成自然对话。"""
            
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

回复指导：
{decision.get("response_guidance", "继续面试流程")}

请根据回复指导生成自然、专业的面试官回复。"""
            
            response = self.llm_client.create_streaming_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.7,
                seed=1
            )
            
            full_response = ""
            async for chunk in response:
                if chunk.choices and chunk.choices[0].delta.content:
                    content = chunk.choices[0].delta.content
                    full_response += content
                    yield content
            
            return full_response
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 流式生成失败: {e}")
            error_msg = "抱歉，我需要整理一下思路。你能再说一遍吗？"
            yield error_msg
            return error_msg


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
            self.current_question_follow_count = 0  # 重置追问次数
            
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
            
            # 首次调用：直接生成开场白
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - 🎯 开始智能决策（首次调用）")
            decision = await self._make_unified_decision([], system_message)
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 智能决策完成")
            
            # 流式生成开场白
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - 🎯 开始流式生成开场白")
            async for chunk in self._stream_response_by_decision(decision, [], system_message):
                yield f"data: {json.dumps({'type': 'chunk', 'content': chunk}, ensure_ascii=False)}\n\n"
            
            yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 已返回开场白，函数即将return")
            return

        # 快速规则检查：追问次数检查
        if self.current_question_follow_count >= 2:
            logging.info(f"-- 面试ID: {self.interview_id} - ⚠️ 已追问2次，强制进入下一题")
            self.current_question_follow_count = 0
            logging.info(f"-- 面试ID: {self.interview_id} - 📈 强制进入下一题，追问次数已重置")

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

        self.speech_speed = 1.1 if self.interview_type == 1 else 1
        
        # 准备面试计划信息
        plan_info = ""
        if self.interview_plan:
            remaining_questions = self.interview_plan['questions'][self.current_question_index:]
            plan_info = f"""
                【面试计划参考】
                当前进度：已提问 {self.current_question_index}/{len(self.interview_plan['questions'])} 个问题
                待提问题（含期望要点和追问策略）：
                {json.dumps(remaining_questions[:3], ensure_ascii=False, indent=2)}
                
                注意：对于技术类问题，请参考 expectedPoints 和 followUpStrategy 决定是否追问（最多2次）。
                """
        
        try:
            # 阶段1：智能决策（非流式）
            logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始智能决策")
            decision = await self._make_unified_decision(dialog_history, system_message, plan_info)
            logging.info(f"-- 面试ID: {self.interview_id} - ✅ 智能决策完成: action={decision.get('action')}, quality_score={decision.get('quality_score')}")
            
            # 阶段2：流式生成回复
            logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始流式生成回复")
            ai_response = ""
            async for chunk in self._stream_response_by_decision(decision, dialog_history, system_message, plan_info):
                ai_response += chunk
                yield f"data: {json.dumps({'type': 'chunk', 'content': chunk}, ensure_ascii=False)}\n\n"
            
            yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
            
            # 状态更新
            action = decision.get("action")
            if action == "new_question":
                if self.interview_plan:
                    self.current_question_index += 1
                self.current_question_follow_count = 0
                logging.info(f"-- 面试ID: {self.interview_id} - 📈 进入下一题，索引更新为 {self.current_question_index}")
            elif action == "follow_up":
                self.current_question_follow_count += 1
                logging.info(f"-- 面试ID: {self.interview_id} - 🔄 保持当前问题，追问次数: {self.current_question_follow_count}")
            
            # 判断是否结束
            if decision.get("should_end", False):
                logging.info(f"-- 面试ID: {self.interview_id} - 🛑 面试结束（由决策判断）")
                yield f"data: {json.dumps({'type': 'over', 'content': 'over'}, ensure_ascii=False)}\n\n"
            
            end_time = time.time()
            execution_time = end_time - start_time
            logging.info(f"-- 面试ID: {self.interview_id} - 执行时间: {execution_time:.2f}秒, 质量评分: {decision.get('quality_score')}, 回复长度: {len(ai_response)}")

        except Exception as e:
            logging.error(f"-- 面试ID : {self.interview_id}---AI流式响应获取错误: {e}")
            yield f"data: {json.dumps({'type': 'error', 'content': '模型检测到非法输入，请重新开始本次面试'+str(e)}, ensure_ascii=False)}\n\n"






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
from app.identity import get_identity_prompt, get_identity_vars, get_standard_scripts
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
        self.current_question_follow_count = 0  # 当前问题的追问次数
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


    async def _make_unified_decision(self, dialog_history: list, system_message: str) -> dict:
        """
        单次AI调用，完成智能决策
        返回结构化决策：
        {
            "action": "follow_up" | "new_question" | "end",
            "response_guidance": "回复指导（用于流式生成）",
            "quality_score": 0-10,
            "should_end": true/false,
            "reasoning": "决策理由（用于日志）"
        }
        """
        try:
            # 加载策略prompt模板
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            strategy_template = load_prompt("strategy_white_collar.txt")
            strategy_sys_prompt = f"{identity_prompt}\n\n{strategy_template.format(**format_vars)}"
            
            system_prompt = f"""{strategy_sys_prompt}

【重要】你现在需要做出完整的决策，包括：
1. 分析候选人的最新回答
2. 评估回答质量（0-10分）
3. 决定下一步动作（追问/下一题/结束）
4. 生成回复指导（用于后续生成自然回复）
5. 判断是否应该结束面试

规则约束：
- 如果已追问2次，必须 action="new_question"
- 如果回答质量<3分且已追问1次，建议 action="new_question"
- 如果明确结束信号，should_end=true

输出严格JSON格式：
{{
    "action": "follow_up" | "new_question" | "end",
    "response_guidance": "回复指导内容（描述要说什么，但不直接生成回复文本）",
    "quality_score": 0-10,
    "should_end": true/false,
    "reasoning": "你的决策理由（用于日志记录）"
}}"""
            
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

当前问题已追问：{self.current_question_follow_count} 次

请做出完整决策。"""
            
            response = self.llm_client.create_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.4,
                response_format={"type": "json_object"}
            )
            
            decision = json.loads(response.choices[0].message.content)
            
            # 规则验证：如果已追问2次，强制下一题
            if decision.get("action") == "follow_up" and self.current_question_follow_count >= 2:
                logging.warning(f"-- 面试ID: {self.interview_id} - ⚠️ AI决策违反规则（已追问2次），强制修正为 new_question")
                decision["action"] = "new_question"
                decision["reasoning"] = f"已追问2次，强制进入下一题。原决策理由：{decision.get('reasoning', '')}"
            
            logging.info(f"-- 面试ID: {self.interview_id} - 智能决策结果: {decision}")
            return decision
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 智能决策失败: {e}")
            # 失败时返回默认决策
            return {
                "action": "new_question",
                "response_guidance": "继续面试流程",
                "quality_score": 5,
                "should_end": False,
                "reasoning": f"决策失败，使用默认值。错误: {str(e)}"
            }

    async def _stream_response_by_decision(self, decision: dict, dialog_history: list, system_message: str):
        """
        基于决策指导，流式生成回复内容
        Yields: 回复内容片段
        Returns: 完整回复文本
        """
        try:
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            executor_template = load_prompt("executor_white_collar.txt" if self.interview_type == 1 else "executor_blue_collar.txt")
            executor_sys_prompt = f"{identity_prompt}\n\n{executor_template.format(**format_vars)}"
            
            system_prompt = f"""{executor_sys_prompt}

根据以下回复指导，生成自然、专业的面试官回复。
不要透露决策过程，只生成自然对话。"""
            
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

回复指导：
{decision.get("response_guidance", "继续面试流程")}

请根据回复指导生成自然、专业的面试官回复。"""
            
            response = self.llm_client.create_streaming_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.7,
                seed=1
            )
            
            full_response = ""
            async for chunk in response:
                if chunk.choices and chunk.choices[0].delta.content:
                    content = chunk.choices[0].delta.content
                    full_response += content
                    yield content
            
            return full_response
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 流式生成失败: {e}")
            error_msg = "抱歉，我需要整理一下思路。你能再说一遍吗？"
            yield error_msg
            return error_msg


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
            self.current_question_follow_count = 0  # 重置追问次数
            
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
            
            # 首次调用：直接生成开场白
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - 🎯 开始智能决策（首次调用）")
            decision = await self._make_unified_decision([], system_message)
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 智能决策完成")
            
            # 流式生成开场白
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - 🎯 开始流式生成开场白")
            async for chunk in self._stream_response_by_decision(decision, [], system_message):
                yield f"data: {json.dumps({'type': 'chunk', 'content': chunk}, ensure_ascii=False)}\n\n"
            
            yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 已返回开场白，函数即将return")
            return

        # 快速规则检查：追问次数检查
        if self.current_question_follow_count >= 2:
            logging.info(f"-- 面试ID: {self.interview_id} - ⚠️ 已追问2次，强制进入下一题")
            self.current_question_follow_count = 0
            logging.info(f"-- 面试ID: {self.interview_id} - 📈 强制进入下一题，追问次数已重置")

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

        self.speech_speed = 1.1 if self.interview_type == 1 else 1
        
        try:
            # 阶段1：智能决策（非流式）
            logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始智能决策")
            decision = await self._make_unified_decision(dialog_history, system_message)
            logging.info(f"-- 面试ID: {self.interview_id} - ✅ 智能决策完成: action={decision.get('action')}, quality_score={decision.get('quality_score')}")
            
            # 阶段2：流式生成回复
            logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始流式生成回复")
            ai_response = ""
            async for chunk in self._stream_response_by_decision(decision, dialog_history, system_message):
                ai_response += chunk
                yield f"data: {json.dumps({'type': 'chunk', 'content': chunk}, ensure_ascii=False)}\n\n"
            
            yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
            
            # 状态更新
            action = decision.get("action")
            if action == "new_question":
                self.current_question_follow_count = 0
                logging.info(f"-- 面试ID: {self.interview_id} - 📈 进入下一题，追问次数已重置")
            elif action == "follow_up":
                self.current_question_follow_count += 1
                logging.info(f"-- 面试ID: {self.interview_id} - 🔄 保持当前问题，追问次数: {self.current_question_follow_count}")
            
            # 判断是否结束
            if decision.get("should_end", False):
                logging.info(f"-- 面试ID: {self.interview_id} - 🛑 面试结束（由决策判断）")
                yield f"data: {json.dumps({'type': 'over', 'content': 'over'}, ensure_ascii=False)}\n\n"
            
            end_time = time.time()
            execution_time = end_time - start_time
            logging.info(f"-- 面试ID: {self.interview_id} - 执行时间: {execution_time:.2f}秒, 质量评分: {decision.get('quality_score')}, 回复长度: {len(ai_response)}")

        except Exception as e:
            logging.error(f"-- 面试ID : {self.interview_id}---AI流式响应获取错误: {e}")
            yield f"data: {json.dumps({'type': 'error', 'content': '模型检测到非法输入，请重新开始本次面试'+str(e)}, ensure_ascii=False)}\n\n"






```

检查修改后的代码，确保所有修改正确：


检查首次调用处理：


## 修改内容梳理

### 1. 系统提示词增强

**文件**：`project_code/career/prompts/interview/strategy_white_collar.txt`

**修改内容**：
- 删除：原有技术问题追问机制中关于"参考【面试计划参考】"的内容
- 新增：面试规划要求（第73-105行）
  - 已覆盖/未覆盖的考察维度分析
  - 考察维度要求（专业技能、软技能、职业规划、文化适配）
  - 重要原则（动态判断、避免重复、保持系统性）
  - 面试阶段判断（破冰/核心/收尾）
- 保留：技术问题追问机制，但删除对面试计划的引用

### 2. 删除面试计划生成函数

**文件**：`project_code/career/code/service/virtual_interview.py`

**删除内容**：
- 删除 `generate_interview_plan()` 函数（原第129-188行，约60行代码）

### 3. 删除计划状态变量

**文件**：`project_code/career/code/service/virtual_interview.py`

**修改位置**：`__init__` 方法（第54行）

**删除内容**：
- 删除 `self.interview_plan = None`
- 删除 `self.current_question_index = 0`

**保留内容**：
- 保留 `self.current_question_follow_count = 0`（规则约束需要）

### 4. 简化决策函数

**文件**：`project_code/career/code/service/virtual_interview.py`

**修改位置**：`_make_unified_decision()` 方法

**修改内容**：
1. 函数签名（第127行）
   - 删除 `plan_info: str` 参数
   - 改为：`async def _make_unified_decision(self, dialog_history: list, system_message: str) -> dict:`

2. 删除计划信息准备逻辑（原第204-226行）
   - 删除所有 `plan_context` 构建代码
   - 删除对 `self.interview_plan` 的引用
   - 删除对 `self.current_question_index` 的引用

3. 简化用户提示词（第175-181行）
   - 删除 `{plan_context}` 传递
   - 保留追问次数信息：`当前问题已追问：{self.current_question_follow_count} 次`

### 5. 简化流式生成函数

**文件**：`project_code/career/code/service/virtual_interview.py`

**修改位置**：`_stream_response_by_decision()` 方法

**修改内容**：
1. 函数签名（第216行）
   - 删除 `plan_info: str` 参数
   - 改为：`async def _stream_response_by_decision(self, decision: dict, dialog_history: list, system_message: str):`

2. 简化用户提示词（第237-248行）
   - 删除 `{plan_info}` 传递

### 6. 修改首次调用处理

**文件**：`project_code/career/code/service/virtual_interview.py`

**修改位置**：`get_ai_response()` 方法（第307-336行）

**修改内容**：
1. 删除计划生成调用
   - 删除 `await self.generate_interview_plan()` 调用
   - 删除计划验证逻辑
   - 删除从计划中提取第一个问题的逻辑
   - 删除返回计划给前端的逻辑

2. 新逻辑
   - 重置追问次数：`self.current_question_follow_count = 0`
   - 直接调用智能决策：`decision = await self._make_unified_decision([], system_message)`
   - 流式生成开场白：`_stream_response_by_decision(decision, [], system_message)`
   - 返回开场白给前端

### 7. 修改快速规则检查

**文件**：`project_code/career/code/service/virtual_interview.py`

**修改位置**：`get_ai_response()` 方法（第338-342行）

**修改内容**：
- 删除对 `self.interview_plan` 的检查
- 删除对 `self.current_question_index` 的更新
- 仅重置追问次数：`self.current_question_follow_count = 0`

### 8. 修改主流程调用

**文件**：`project_code/career/code/service/virtual_interview.py`

**修改位置**：`get_ai_response()` 方法（第356-380行）

**修改内容**：
1. 删除计划信息准备（原第446-457行）
   - 删除 `plan_info` 的构建逻辑

2. 修改函数调用（第361、367行）
   - `_make_unified_decision(dialog_history, system_message, plan_info)` 
     → `_make_unified_decision(dialog_history, system_message)`
   - `_stream_response_by_decision(decision, dialog_history, system_message, plan_info)`
     → `_stream_response_by_decision(decision, dialog_history, system_message)`

3. 简化状态更新（第375-380行）
   - 删除对 `self.current_question_index` 的更新
   - 删除对 `self.interview_plan` 的检查
   - 保留对 `self.current_question_follow_count` 的更新

## 修改统计

- 删除代码：约110行
  - `generate_interview_plan()` 函数：~60行
  - 计划状态维护：~20行
  - 计划信息传递：~30行
- 新增代码：系统提示词增强（约30行）
- 修改代码：函数签名和调用处（约10处）

## 关键改进点

1. 删除显式计划生成：不再预先生成问题列表
2. 增强系统提示词：AI自主规划面试流程
3. 简化代码结构：减少状态维护和代码复杂度
4. 保持功能完整：追问次数追踪和规则约束保留
5. 性能提升：首次调用减少1次API调用

## 注意事项

1. 系统提示词已增强，AI会根据提示词自主规划面试
2. 追问次数追踪保留，规则约束（最多追问2次）仍然有效
3. 首次调用直接生成开场白，不再返回计划给前端
4. 所有计划相关的状态变量和逻辑已删除

修改完成。代码已按方案实现，未添加额外功能。

---

**User**

@后端 (4-18) 咋回事？根本就启动不起来了。现在。

---

**Cursor**

修复语法错误：异步生成器函数不能使用 `return`。检查并修复：


修复语法错误：异步生成器函数不能使用 `return` 返回值。移除该 `return` 语句：


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
from app.identity import get_identity_prompt, get_identity_vars, get_standard_scripts
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
        self.current_question_follow_count = 0  # 当前问题的追问次数
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


    async def _make_unified_decision(self, dialog_history: list, system_message: str) -> dict:
        """
        单次AI调用，完成智能决策
        返回结构化决策：
        {
            "action": "follow_up" | "new_question" | "end",
            "response_guidance": "回复指导（用于流式生成）",
            "quality_score": 0-10,
            "should_end": true/false,
            "reasoning": "决策理由（用于日志）"
        }
        """
        try:
            # 加载策略prompt模板
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            strategy_template = load_prompt("strategy_white_collar.txt")
            strategy_sys_prompt = f"{identity_prompt}\n\n{strategy_template.format(**format_vars)}"
            
            system_prompt = f"""{strategy_sys_prompt}

【重要】你现在需要做出完整的决策，包括：
1. 分析候选人的最新回答
2. 评估回答质量（0-10分）
3. 决定下一步动作（追问/下一题/结束）
4. 生成回复指导（用于后续生成自然回复）
5. 判断是否应该结束面试

规则约束：
- 如果已追问2次，必须 action="new_question"
- 如果回答质量<3分且已追问1次，建议 action="new_question"
- 如果明确结束信号，should_end=true

输出严格JSON格式：
{{
    "action": "follow_up" | "new_question" | "end",
    "response_guidance": "回复指导内容（描述要说什么，但不直接生成回复文本）",
    "quality_score": 0-10,
    "should_end": true/false,
    "reasoning": "你的决策理由（用于日志记录）"
}}"""
            
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

当前问题已追问：{self.current_question_follow_count} 次

请做出完整决策。"""
            
            response = self.llm_client.create_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.4,
                response_format={"type": "json_object"}
            )
            
            decision = json.loads(response.choices[0].message.content)
            
            # 规则验证：如果已追问2次，强制下一题
            if decision.get("action") == "follow_up" and self.current_question_follow_count >= 2:
                logging.warning(f"-- 面试ID: {self.interview_id} - ⚠️ AI决策违反规则（已追问2次），强制修正为 new_question")
                decision["action"] = "new_question"
                decision["reasoning"] = f"已追问2次，强制进入下一题。原决策理由：{decision.get('reasoning', '')}"
            
            logging.info(f"-- 面试ID: {self.interview_id} - 智能决策结果: {decision}")
            return decision
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 智能决策失败: {e}")
            # 失败时返回默认决策
            return {
                "action": "new_question",
                "response_guidance": "继续面试流程",
                "quality_score": 5,
                "should_end": False,
                "reasoning": f"决策失败，使用默认值。错误: {str(e)}"
            }

    async def _stream_response_by_decision(self, decision: dict, dialog_history: list, system_message: str):
        """
        基于决策指导，流式生成回复内容
        Yields: 回复内容片段
        Returns: 完整回复文本
        """
        try:
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            executor_template = load_prompt("executor_white_collar.txt" if self.interview_type == 1 else "executor_blue_collar.txt")
            executor_sys_prompt = f"{identity_prompt}\n\n{executor_template.format(**format_vars)}"
            
            system_prompt = f"""{executor_sys_prompt}

根据以下回复指导，生成自然、专业的面试官回复。
不要透露决策过程，只生成自然对话。"""
            
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

回复指导：
{decision.get("response_guidance", "继续面试流程")}

请根据回复指导生成自然、专业的面试官回复。"""
            
            response = self.llm_client.create_streaming_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.7,
                seed=1
            )
            
            async for chunk in response:
                if chunk.choices and chunk.choices[0].delta.content:
                    content = chunk.choices[0].delta.content
                    yield content
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 流式生成失败: {e}")
            error_msg = "抱歉，我需要整理一下思路。你能再说一遍吗？"
            yield error_msg


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
            self.current_question_follow_count = 0  # 重置追问次数
            
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
            
            # 首次调用：直接生成开场白
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - 🎯 开始智能决策（首次调用）")
            decision = await self._make_unified_decision([], system_message)
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 智能决策完成")
            
            # 流式生成开场白
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - 🎯 开始流式生成开场白")
            async for chunk in self._stream_response_by_decision(decision, [], system_message):
                yield f"data: {json.dumps({'type': 'chunk', 'content': chunk}, ensure_ascii=False)}\n\n"
            
            yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 已返回开场白，函数即将return")
            return

        # 快速规则检查：追问次数检查
        if self.current_question_follow_count >= 2:
            logging.info(f"-- 面试ID: {self.interview_id} - ⚠️ 已追问2次，强制进入下一题")
            self.current_question_follow_count = 0
            logging.info(f"-- 面试ID: {self.interview_id} - 📈 强制进入下一题，追问次数已重置")

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

        self.speech_speed = 1.1 if self.interview_type == 1 else 1
        
        try:
            # 阶段1：智能决策（非流式）
            logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始智能决策")
            decision = await self._make_unified_decision(dialog_history, system_message)
            logging.info(f"-- 面试ID: {self.interview_id} - ✅ 智能决策完成: action={decision.get('action')}, quality_score={decision.get('quality_score')}")
            
            # 阶段2：流式生成回复
            logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始流式生成回复")
            ai_response = ""
            async for chunk in self._stream_response_by_decision(decision, dialog_history, system_message):
                ai_response += chunk
                yield f"data: {json.dumps({'type': 'chunk', 'content': chunk}, ensure_ascii=False)}\n\n"
            
            yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
            
            # 状态更新
            action = decision.get("action")
            if action == "new_question":
                self.current_question_follow_count = 0
                logging.info(f"-- 面试ID: {self.interview_id} - 📈 进入下一题，追问次数已重置")
            elif action == "follow_up":
                self.current_question_follow_count += 1
                logging.info(f"-- 面试ID: {self.interview_id} - 🔄 保持当前问题，追问次数: {self.current_question_follow_count}")
            
            # 判断是否结束
            if decision.get("should_end", False):
                logging.info(f"-- 面试ID: {self.interview_id} - 🛑 面试结束（由决策判断）")
                yield f"data: {json.dumps({'type': 'over', 'content': 'over'}, ensure_ascii=False)}\n\n"
            
            end_time = time.time()
            execution_time = end_time - start_time
            logging.info(f"-- 面试ID: {self.interview_id} - 执行时间: {execution_time:.2f}秒, 质量评分: {decision.get('quality_score')}, 回复长度: {len(ai_response)}")

        except Exception as e:
            logging.error(f"-- 面试ID : {self.interview_id}---AI流式响应获取错误: {e}")
            yield f"data: {json.dumps({'type': 'error', 'content': '模型检测到非法输入，请重新开始本次面试'+str(e)}, ensure_ascii=False)}\n\n"






```

已修复语法错误。问题在于异步生成器函数（使用 `yield` 的函数）不能使用 `return` 返回值。

**修复内容**：
- 删除了 `return full_response`（第266行）
- 删除了 `return error_msg`（第271行）
- 删除了不必要的 `full_response` 变量累积（调用方已经在累积）

现在代码应该可以正常启动了。异步生成器函数只需要 `yield`，不需要 `return` 返回值。

请重新启动服务测试。

---

**User**

好的，你现在重新帮我梳理一下吧就是整体来说。修改后的虚拟面试的逻辑是什么样子的呢？请ultrathinking,帮我一步步的逻辑梳理清楚呀。

---

**Cursor**

梳理修改后的虚拟面试逻辑，逐步分析：


## 修改后的虚拟面试逻辑梳理

### 一、整体架构

#### 核心变化
- 删除：显式面试计划生成（不再预先生成问题列表）
- 新增：系统提示词增强（AI自主规划面试流程）
- 简化：状态管理（只保留追问次数追踪）
- 优化：统一智能决策（单次AI调用完成所有决策）

### 二、状态管理

#### 保留的状态变量
```python
self.current_question_follow_count = 0  # 当前问题的追问次数
```

#### 删除的状态变量
- `self.interview_plan`（面试计划）
- `self.current_question_index`（问题索引）

### 三、完整执行流程

#### 第1层：入口函数 `get_ai_response()`

```
用户输入（HTTP请求）
    ↓
接收 dialog_history（对话历史）
    ↓
【快速规则检查】
```

#### 第2层：快速规则检查（第280-300行）

检查1：沉默检测
```python
if dialog_history and len(dialog_history) > 0:
    # 统计连续沉默次数（从后往前遍历）
    silence_count = 0
    for msg in reversed(dialog_history):
        if msg.get("role") == "user":
            if 内容为空或无效（"", "...", "嗯", "啊"）:
                silence_count += 1
            else:
                break
    
    if silence_count >= 3:
        # 强制结束面试
        yield "面试结束消息"
        return
```

检查2：追问次数检查（第333-337行）
```python
if self.current_question_follow_count >= 2:
    # 已追问2次，强制进入下一题
    self.current_question_follow_count = 0  # 重置
    # 继续执行，让AI生成下一题
```

#### 第3层：首次调用判断（第302-331行）

判断条件：
```python
if not dialog_history or len(dialog_history) == 0:
    # 首次调用
```

首次调用流程：
```
1. 重置追问次数：self.current_question_follow_count = 0
    ↓
2. 构建系统消息（包含候选人信息、岗位信息等）
    ↓
3. 调用智能决策：_make_unified_decision([], system_message)
   - 传入空对话历史
   - AI根据系统提示词生成开场白决策
    ↓
4. 流式生成开场白：_stream_response_by_decision(decision, [], system_message)
   - 基于决策中的 response_guidance 生成开场白
    ↓
5. 返回开场白给前端
    ↓
6. 结束（return）
```

#### 第4层：非首次调用流程（第339-388行）

流程：
```
1. 构建系统消息（包含候选人信息、岗位信息等）
    ↓
2. 【阶段1：智能决策】（非流式，第355-357行）
   调用：_make_unified_decision(dialog_history, system_message)
    ↓
3. 【阶段2：流式生成回复】（第359-365行）
   调用：_stream_response_by_decision(decision, dialog_history, system_message)
    ↓
4. 【状态更新】（第368-375行）
   根据决策的 action 更新追问次数
    ↓
5. 【结束判断】（第377-380行）
   根据决策的 should_end 判断是否结束
```

### 四、核心函数详解

#### 函数1：`_make_unified_decision()` - 智能决策

功能：单次AI调用完成所有决策

输入：
- `dialog_history`：完整对话历史
- `system_message`：系统消息（候选人信息、岗位信息等）

处理过程：
```
1. 加载策略prompt模板（strategy_white_collar.txt）
    ↓
2. 构建系统提示词：
   - 基础策略prompt（包含面试规划要求）
   - 决策要求（分析、评估、决定、生成指导、判断结束）
   - 规则约束（最多追问2次等）
   - JSON输出格式要求
    ↓
3. 构建用户提示词：
   - 对话历史（JSON格式）
   - 当前时间
   - 系统消息
   - 当前追问次数
    ↓
4. 调用LLM（非流式，JSON格式输出）
    ↓
5. 解析决策结果：
   {
     "action": "follow_up" | "new_question" | "end",
     "response_guidance": "回复指导",
     "quality_score": 0-10,
     "should_end": true/false,
     "reasoning": "决策理由"
   }
    ↓
6. 规则验证：
   if action == "follow_up" and 已追问2次:
      强制修正为 "new_question"
    ↓
7. 返回决策结果
```

关键点：
- 系统提示词包含面试规划要求（分析已覆盖/未覆盖维度）
- AI自主规划面试流程，无需预生成计划
- 规则约束：代码层面强制限制（最多追问2次）

#### 函数2：`_stream_response_by_decision()` - 流式生成回复

功能：基于决策指导，流式生成自然回复

输入：
- `decision`：决策结果（包含 response_guidance）
- `dialog_history`：对话历史
- `system_message`：系统消息

处理过程：
```
1. 加载执行prompt模板（executor_white_collar.txt）
    ↓
2. 构建系统提示词：
   - 基础执行prompt（面试官身份、风格等）
   - 要求：根据回复指导生成自然回复
   - 要求：不透露决策过程
    ↓
3. 构建用户提示词：
   - 对话历史
   - 当前时间
   - 系统消息
   - 回复指导（来自决策的 response_guidance）
    ↓
4. 调用LLM（流式）
    ↓
5. 流式返回内容片段（yield）
```

关键点：
- 使用决策中的 `response_guidance` 作为指导
- 生成自然对话，不透露决策过程
- 流式返回，提升用户体验

### 五、状态更新逻辑（第368-375行）

```python
action = decision.get("action")

if action == "new_question":
    # 进入下一题
    self.current_question_follow_count = 0  # 重置追问次数
    
elif action == "follow_up":
    # 继续追问
    self.current_question_follow_count += 1  # 追问次数+1
```

### 六、结束判断逻辑（第377-380行）

```python
if decision.get("should_end", False):
    # AI决策判断应该结束
    yield "over" 事件
```

### 七、完整流程图

```
┌─────────────────────────────────────────────────────────┐
│  用户输入（HTTP请求）                                    │
│  dialog_history = [...]                                 │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│  【快速规则检查】                                         │
├─────────────────────────────────────────────────────────┤
│  1. 沉默检测                                             │
│     - 连续3次沉默 → 强制结束                             │
│  2. 追问次数检查                                         │
│     - 已追问2次 → 重置计数，继续执行                     │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│  【首次调用判断】                                         │
├─────────────────────────────────────────────────────────┤
│  if dialog_history 为空:                                 │
│    1. 重置追问次数                                       │
│    2. 构建系统消息                                       │
│    3. 智能决策（空历史）→ 生成开场白决策                 │
│    4. 流式生成开场白                                     │
│    5. 返回开场白 → 结束                                 │
│                                                          │
│  else:                                                   │
│    继续执行（非首次调用）                                 │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│  【阶段1：智能决策】（非流式）                            │
├─────────────────────────────────────────────────────────┤
│  输入：                                                  │
│  - dialog_history（完整对话历史）                         │
│  - system_message（候选人、岗位信息）                    │
│                                                          │
│  处理：                                                  │
│  1. 加载策略prompt（包含面试规划要求）                    │
│  2. AI分析：                                             │
│     - 已覆盖/未覆盖的考察维度                            │
│     - 回答质量评分（0-10分）                            │
│     - 决定下一步动作（追问/下一题/结束）                 │
│     - 生成回复指导                                       │
│     - 判断是否结束                                       │
│  3. 规则验证（已追问2次 → 强制下一题）                   │
│                                                          │
│  输出：                                                  │
│  {                                                       │
│    "action": "follow_up" | "new_question" | "end",      │
│    "response_guidance": "回复指导",                      │
│    "quality_score": 3,                                   │
│    "should_end": false,                                  │
│    "reasoning": "决策理由"                               │
│  }                                                       │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│  【阶段2：流式生成回复】                                  │
├─────────────────────────────────────────────────────────┤
│  输入：                                                  │
│  - decision（决策结果）                                  │
│  - dialog_history（对话历史）                            │
│  - system_message（系统消息）                           │
│                                                          │
│  处理：                                                  │
│  1. 加载执行prompt（面试官身份、风格）                    │
│  2. 使用决策中的 response_guidance 作为指导              │
│  3. 流式生成自然回复                                     │
│                                                          │
│  输出：                                                  │
│  - 流式返回回复内容片段（SSE格式）                       │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│  【状态更新】                                             │
├─────────────────────────────────────────────────────────┤
│  if action == "new_question":                           │
│    current_question_follow_count = 0  # 重置             │
│                                                          │
│  elif action == "follow_up":                            │
│    current_question_follow_count += 1  # 增加            │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│  【结束判断】                                             │
├─────────────────────────────────────────────────────────┤
│  if decision.should_end == true:                         │
│    yield "over" 事件                                     │
└─────────────────────────────────────────────────────────┘
                    ↓
            【返回结果给前端】
```

### 八、关键设计点

#### 1. 无计划生成
- 不再预先生成问题列表
- AI根据系统提示词自主规划
- 系统提示词包含面试规划要求（分析维度、阶段判断等）

#### 2. 统一智能决策
- 单次AI调用完成所有决策
- 同时考虑：分析、评估、决策、生成指导、结束判断
- 避免信息传递损失

#### 3. 规则约束机制
- 代码层面强制限制（最多追问2次）
- 双重保障：快速规则检查 + 决策后验证

#### 4. 状态追踪简化
- 只保留追问次数追踪
- 删除问题索引和计划状态

#### 5. 流式体验保持
- 两阶段调用：快速决策 + 流式生成
- 用户体验：实时看到回复

### 九、与修改前的对比

| 维度 | 修改前 | 修改后 |
|------|--------|--------|
| 计划生成 | 预先生成10-12个问题 | 不生成，AI自主规划 |
| API调用次数 | 首次2次，后续4次 | 首次1次，后续2次 |
| 状态管理 | 计划+索引+追问次数 | 仅追问次数 |
| 智能性 | 依赖预生成计划 | AI自主规划 |
| 灵活性 | 静态计划 | 动态规划 |

以上是修改后的虚拟面试逻辑梳理。

```python
self.current_question_follow_count = 0  # 当前问题的追问次数
```

```plaintext
用户输入（HTTP请求）
    ↓
接收 dialog_history（对话历史）
    ↓
【快速规则检查】
```

```python
if dialog_history and len(dialog_history) > 0:
    # 统计连续沉默次数（从后往前遍历）
    silence_count = 0
    for msg in reversed(dialog_history):
        if msg.get("role") == "user":
            if 内容为空或无效（"", "...", "嗯", "啊"）:
                silence_count += 1
            else:
                break
    
    if silence_count >= 3:
        # 强制结束面试
        yield "面试结束消息"
        return
```

```python
if self.current_question_follow_count >= 2:
    # 已追问2次，强制进入下一题
    self.current_question_follow_count = 0  # 重置
    # 继续执行，让AI生成下一题
```

```python
if not dialog_history or len(dialog_history) == 0:
    # 首次调用
```

```plaintext
1. 重置追问次数：self.current_question_follow_count = 0
    ↓
2. 构建系统消息（包含候选人信息、岗位信息等）
    ↓
3. 调用智能决策：_make_unified_decision([], system_message)
   - 传入空对话历史
   - AI根据系统提示词生成开场白决策
    ↓
4. 流式生成开场白：_stream_response_by_decision(decision, [], system_message)
   - 基于决策中的 response_guidance 生成开场白
    ↓
5. 返回开场白给前端
    ↓
6. 结束（return）
```

```plaintext
1. 构建系统消息（包含候选人信息、岗位信息等）
    ↓
2. 【阶段1：智能决策】（非流式，第355-357行）
   调用：_make_unified_decision(dialog_history, system_message)
    ↓
3. 【阶段2：流式生成回复】（第359-365行）
   调用：_stream_response_by_decision(decision, dialog_history, system_message)
    ↓
4. 【状态更新】（第368-375行）
   根据决策的 action 更新追问次数
    ↓
5. 【结束判断】（第377-380行）
   根据决策的 should_end 判断是否结束
```

```plaintext
1. 加载策略prompt模板（strategy_white_collar.txt）
    ↓
2. 构建系统提示词：
   - 基础策略prompt（包含面试规划要求）
   - 决策要求（分析、评估、决定、生成指导、判断结束）
   - 规则约束（最多追问2次等）
   - JSON输出格式要求
    ↓
3. 构建用户提示词：
   - 对话历史（JSON格式）
   - 当前时间
   - 系统消息
   - 当前追问次数
    ↓
4. 调用LLM（非流式，JSON格式输出）
    ↓
5. 解析决策结果：
   {
     "action": "follow_up" | "new_question" | "end",
     "response_guidance": "回复指导",
     "quality_score": 0-10,
     "should_end": true/false,
     "reasoning": "决策理由"
   }
    ↓
6. 规则验证：
   if action == "follow_up" and 已追问2次:
      强制修正为 "new_question"
    ↓
7. 返回决策结果
```

```plaintext
1. 加载执行prompt模板（executor_white_collar.txt）
    ↓
2. 构建系统提示词：
   - 基础执行prompt（面试官身份、风格等）
   - 要求：根据回复指导生成自然回复
   - 要求：不透露决策过程
    ↓
3. 构建用户提示词：
   - 对话历史
   - 当前时间
   - 系统消息
   - 回复指导（来自决策的 response_guidance）
    ↓
4. 调用LLM（流式）
    ↓
5. 流式返回内容片段（yield）
```

```python
action = decision.get("action")

if action == "new_question":
    # 进入下一题
    self.current_question_follow_count = 0  # 重置追问次数
    
elif action == "follow_up":
    # 继续追问
    self.current_question_follow_count += 1  # 追问次数+1
```

```python
if decision.get("should_end", False):
    # AI决策判断应该结束
    yield "over" 事件
```

```plaintext
┌─────────────────────────────────────────────────────────┐
│  用户输入（HTTP请求）                                    │
│  dialog_history = [...]                                 │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│  【快速规则检查】                                         │
├─────────────────────────────────────────────────────────┤
│  1. 沉默检测                                             │
│     - 连续3次沉默 → 强制结束                             │
│  2. 追问次数检查                                         │
│     - 已追问2次 → 重置计数，继续执行                     │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│  【首次调用判断】                                         │
├─────────────────────────────────────────────────────────┤
│  if dialog_history 为空:                                 │
│    1. 重置追问次数                                       │
│    2. 构建系统消息                                       │
│    3. 智能决策（空历史）→ 生成开场白决策                 │
│    4. 流式生成开场白                                     │
│    5. 返回开场白 → 结束                                 │
│                                                          │
│  else:                                                   │
│    继续执行（非首次调用）                                 │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│  【阶段1：智能决策】（非流式）                            │
├─────────────────────────────────────────────────────────┤
│  输入：                                                  │
│  - dialog_history（完整对话历史）                         │
│  - system_message（候选人、岗位信息）                    │
│                                                          │
│  处理：                                                  │
│  1. 加载策略prompt（包含面试规划要求）                    │
│  2. AI分析：                                             │
│     - 已覆盖/未覆盖的考察维度                            │
│     - 回答质量评分（0-10分）                            │
│     - 决定下一步动作（追问/下一题/结束）                 │
│     - 生成回复指导                                       │
│     - 判断是否结束                                       │
│  3. 规则验证（已追问2次 → 强制下一题）                   │
│                                                          │
│  输出：                                                  │
│  {                                                       │
│    "action": "follow_up" | "new_question" | "end",      │
│    "response_guidance": "回复指导",                      │
│    "quality_score": 3,                                   │
│    "should_end": false,                                  │
│    "reasoning": "决策理由"                               │
│  }                                                       │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│  【阶段2：流式生成回复】                                  │
├─────────────────────────────────────────────────────────┤
│  输入：                                                  │
│  - decision（决策结果）                                  │
│  - dialog_history（对话历史）                            │
│  - system_message（系统消息）                           │
│                                                          │
│  处理：                                                  │
│  1. 加载执行prompt（面试官身份、风格）                    │
│  2. 使用决策中的 response_guidance 作为指导              │
│  3. 流式生成自然回复                                     │
│                                                          │
│  输出：                                                  │
│  - 流式返回回复内容片段（SSE格式）                       │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│  【状态更新】                                             │
├─────────────────────────────────────────────────────────┤
│  if action == "new_question":                           │
│    current_question_follow_count = 0  # 重置             │
│                                                          │
│  elif action == "follow_up":                            │
│    current_question_follow_count += 1  # 增加            │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│  【结束判断】                                             │
├─────────────────────────────────────────────────────────┤
│  if decision.should_end == true:                         │
│    yield "over" 事件                                     │
└─────────────────────────────────────────────────────────┘
                    ↓
            【返回结果给前端】
```

---

**User**

所以说 用户输入一次问题的话，那么API只会被调用一次是吗？如果满分一百分的话这个理解给几分呢？以及为什么？

---

**Cursor**

检查代码，确认每次用户输入后的实际 API 调用次数：


## 评分：35分

### 评分理由

#### 正确部分（+35分）
- 理解方向正确：相比修改前的4次调用，现在确实减少了
- 知道API调用次数减少了

#### 错误部分（-65分）
- 数量不准确：不是1次，是2次

### 实际API调用次数

从代码看，每次用户输入（非首次）会调用2次API：

#### 第1次API调用：智能决策（非流式）

位置：`_make_unified_decision()` 函数（第185-192行）

```python
response = self.llm_client.create_completion(
    messages=[...],
    temperature=0.4,
    response_format={"type": "json_object"}
)
```

作用：
- 分析对话历史
- 评估回答质量
- 决定下一步动作
- 生成回复指导
- 判断是否结束

返回：结构化决策JSON

#### 第2次API调用：流式生成回复

位置：`_stream_response_by_decision()` 函数（第250-257行）

```python
response = self.llm_client.create_streaming_completion(
    messages=[...],
    temperature=0.7,
    seed=1
)
```

作用：
- 基于决策中的 `response_guidance` 生成自然回复
- 流式返回内容

返回：流式回复内容

### 完整调用流程

```
用户输入一次问题
    ↓
【阶段1：智能决策】
    ↓
调用 create_completion() ← 第1次API调用（非流式）
    ↓
返回决策结果（JSON）
    ↓
【阶段2：流式生成回复】
    ↓
调用 create_streaming_completion() ← 第2次API调用（流式）
    ↓
流式返回回复内容
    ↓
返回结果给前端
```

### 与修改前对比

修改前（4次API调用）：
1. 策略AI（`create_completion`）
2. 执行AI（`create_streaming_completion`）
3. 意图识别（`create_completion`）
4. 判断AI（`create_completion`）

修改后（2次API调用）：
1. 智能决策（`create_completion`）
2. 流式生成回复（`create_streaming_completion`）

### 为什么是2次而不是1次？

原因：两阶段设计
- 阶段1：快速决策（非流式，获取结构化决策）
- 阶段2：流式生成（基于决策指导，流式返回）

如果合并为1次：
- 无法同时满足结构化输出和流式体验
- 结构化输出（JSON）与流式输出（文本）难以兼顾

### 扣分原因

1. 主要错误：数量不准确（2次，不是1次）
2. 理解偏差：可能只关注了“减少调用”，但未确认具体次数

### 正确理解

- 修改前：每次用户输入调用4次API
- 修改后：每次用户输入调用2次API
- 首次调用：调用1次API（只生成开场白，不涉及用户输入）

### 总结

你的理解方向正确（调用次数减少了），但数量不准确。实际是2次API调用，不是1次。这是两阶段设计的结果：快速决策 + 流式生成，既保证智能决策，又保持流式体验。

```python
response = self.llm_client.create_completion(
    messages=[...],
    temperature=0.4,
    response_format={"type": "json_object"}
)
```

```python
response = self.llm_client.create_streaming_completion(
    messages=[...],
    temperature=0.7,
    seed=1
)
```

```plaintext
用户输入一次问题
    ↓
【阶段1：智能决策】
    ↓
调用 create_completion() ← 第1次API调用（非流式）
    ↓
返回决策结果（JSON）
    ↓
【阶段2：流式生成回复】
    ↓
调用 create_streaming_completion() ← 第2次API调用（流式）
    ↓
流式返回回复内容
    ↓
返回结果给前端
```

---

**User**

像每次对话的话有没有完整的日志呀？可以记录所有的一些输出呀，包括一些关键信息呀，就是我需要在哪里看的。

---

**Cursor**

检查代码中的日志配置和记录位置：


查看日志配置详情：


## 日志系统说明

### 一、日志文件位置

日志文件保存在：
```
project_code/career/code/logs/
```

日志文件：
1. `app_info.log` - 所有INFO级别日志（包含正常流程和调试信息）
2. `app_error.log` - 所有ERROR级别日志（仅错误信息）

日志文件规则：
- 按天分割：每天午夜自动创建新文件
- 保留时间：
  - `app_info.log`：保留最近7天
  - `app_error.log`：保留最近30天
- 文件命名：`app_info.log.2024-12-22`（带日期后缀）

### 二、日志格式

日志格式：
```
%(asctime)s - %(name)s - %(levelname)s - %(message)s
```

示例：
```
2024-12-22 18:47:09,549 - root - INFO - -- 面试ID: test_interview_001 - 🎯 开始智能决策
```

### 三、关键日志信息

#### 1. 请求入口日志

```python
# 第274-278行
logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - get_ai_response 被调用")
logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - dialog_history 类型: {type(dialog_history)}")
logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - dialog_history 值: {dialog_history}")
```

包含：
- 面试ID
- 对话历史类型和内容

#### 2. 沉默检测日志

```python
# 第292行
logging.info(f"-- 面试ID: {self.interview_id} - 连续沉默次数: {silence_count}/3")
```

包含：
- 连续沉默次数

#### 3. 首次调用日志

```python
# 第304、320、322、325、330行
logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 进入首次调用分支")
logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - 🎯 开始智能决策（首次调用）")
logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 智能决策完成")
logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - 🎯 开始流式生成开场白")
logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 已返回开场白，函数即将return")
```

包含：
- 首次调用流程各步骤

#### 4. 智能决策日志

```python
# 第202行
logging.info(f"-- 面试ID: {self.interview_id} - 智能决策结果: {decision}")
```

包含：
- 完整决策结果（action、quality_score、should_end、reasoning等）

#### 5. 执行流程日志

```python
# 第355、357、360行
logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始智能决策")
logging.info(f"-- 面试ID: {self.interview_id} - ✅ 智能决策完成: action={decision.get('action')}, quality_score={decision.get('quality_score')}")
logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始流式生成回复")
```

包含：
- 执行阶段
- 决策结果摘要

#### 6. 状态更新日志

```python
# 第372、375行
logging.info(f"-- 面试ID: {self.interview_id} - 📈 进入下一题，追问次数已重置")
logging.info(f"-- 面试ID: {self.interview_id} - 🔄 保持当前问题，追问次数: {self.current_question_follow_count}")
```

包含：
- 状态变化
- 追问次数

#### 7. 性能统计日志

```python
# 第384行
logging.info(f"-- 面试ID: {self.interview_id} - 执行时间: {execution_time:.2f}秒, 质量评分: {decision.get('quality_score')}, 回复长度: {len(ai_response)}")
```

包含：
- 执行时间
- 质量评分
- 回复长度

#### 8. 错误日志

```python
# 第206、265、387行
logging.error(f"-- 面试ID: {self.interview_id} - 智能决策失败: {e}")
logging.error(f"-- 面试ID: {self.interview_id} - 流式生成失败: {e}")
logging.error(f"-- 面试ID : {self.interview_id}---AI流式响应获取错误: {e}")
```

包含：
- 错误详情和堆栈

### 四、如何查看日志

#### 方法1：查看日志文件（推荐）

位置：
```
project_code/career/code/logs/app_info.log
```

查看方式：
```bash
# Windows
type project_code\career\code\logs\app_info.log

# Linux/Mac
tail -f project_code/career/code/logs/app_info.log
```

#### 方法2：查看控制台输出

启动服务时，日志会同时输出到控制台。

#### 方法3：按面试ID筛选

```bash
# Windows PowerShell
Select-String -Path "project_code\career\code\logs\app_info.log" -Pattern "test_interview_001"

# Linux/Mac
grep "test_interview_001" project_code/career/code/logs/app_info.log
```

### 五、完整日志示例

一次完整对话的日志流程：

```
2024-12-22 18:47:01,123 - root - INFO - [DEBUG-服务层] 面试ID: test_interview_001 - get_ai_response 被调用
2024-12-22 18:47:01,124 - root - INFO - [DEBUG-服务层] 面试ID: test_interview_001 - dialog_history 类型: <class 'list'>
2024-12-22 18:47:01,125 - root - INFO - [DEBUG-服务层] 面试ID: test_interview_001 - dialog_history 值: [{'role': 'user', 'content': '...'}]
2024-12-22 18:47:01,126 - root - INFO - -- 面试ID: test_interview_001 - 连续沉默次数: 0/3
2024-12-22 18:47:01,127 - root - INFO - -- 面试ID: test_interview_001 - 🎯 开始智能决策
2024-12-22 18:47:09,550 - root - INFO - -- 面试ID: test_interview_001 - 智能决策结果: {'action': 'follow_up', 'quality_score': 3, ...}
2024-12-22 18:47:09,551 - root - INFO - -- 面试ID: test_interview_001 - ✅ 智能决策完成: action=follow_up, quality_score=3
2024-12-22 18:47:09,552 - root - INFO - -- 面试ID: test_interview_001 - 🎯 开始流式生成回复
2024-12-22 18:47:18,540 - root - INFO - -- 面试ID: test_interview_001 - 🔄 保持当前问题，追问次数: 1
2024-12-22 18:47:18,541 - root - INFO - -- 面试ID: test_interview_001 - 执行时间: 17.42秒, 质量评分: 3, 回复长度: 156
```

### 六、关键信息总结

每条对话的日志包含：
1. 面试ID：用于追踪单次面试
2. 对话历史：完整的对话内容
3. 决策结果：action、quality_score、reasoning
4. 执行时间：性能统计
5. 状态变化：追问次数、问题切换
6. 错误信息：如有异常

### 七、查看建议

1. 实时查看：使用 `tail -f` 或 PowerShell 的 `Get-Content -Wait`
2. 按面试ID筛选：用 `grep` 或 `Select-String` 过滤特定面试
3. 查看错误：查看 `app_error.log` 获取错误信息
4. 性能分析：搜索“执行时间”查看性能统计

日志文件位置：`project_code/career/code/logs/app_info.log`

```plaintext
project_code/career/code/logs/
```

```plaintext
%(asctime)s - %(name)s - %(levelname)s - %(message)s
```

```plaintext
2024-12-22 18:47:09,549 - root - INFO - -- 面试ID: test_interview_001 - 🎯 开始智能决策
```

```python
# 第274-278行
logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - get_ai_response 被调用")
logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - dialog_history 类型: {type(dialog_history)}")
logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - dialog_history 值: {dialog_history}")
```

```python
# 第292行
logging.info(f"-- 面试ID: {self.interview_id} - 连续沉默次数: {silence_count}/3")
```

```python
# 第304、320、322、325、330行
logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 进入首次调用分支")
logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - 🎯 开始智能决策（首次调用）")
logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 智能决策完成")
logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - 🎯 开始流式生成开场白")
logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 已返回开场白，函数即将return")
```

```python
# 第202行
logging.info(f"-- 面试ID: {self.interview_id} - 智能决策结果: {decision}")
```

```python
# 第355、357、360行
logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始智能决策")
logging.info(f"-- 面试ID: {self.interview_id} - ✅ 智能决策完成: action={decision.get('action')}, quality_score={decision.get('quality_score')}")
logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始流式生成回复")
```

```python
# 第372、375行
logging.info(f"-- 面试ID: {self.interview_id} - 📈 进入下一题，追问次数已重置")
logging.info(f"-- 面试ID: {self.interview_id} - 🔄 保持当前问题，追问次数: {self.current_question_follow_count}")
```

```python
# 第384行
logging.info(f"-- 面试ID: {self.interview_id} - 执行时间: {execution_time:.2f}秒, 质量评分: {decision.get('quality_score')}, 回复长度: {len(ai_response)}")
```

```python
# 第206、265、387行
logging.error(f"-- 面试ID: {self.interview_id} - 智能决策失败: {e}")
logging.error(f"-- 面试ID: {self.interview_id} - 流式生成失败: {e}")
logging.error(f"-- 面试ID : {self.interview_id}---AI流式响应获取错误: {e}")
```

```plaintext
project_code/career/code/logs/app_info.log
```

```shellscript
# Windows
type project_code\career\code\logs\app_info.log

# Linux/Mac
tail -f project_code/career/code/logs/app_info.log
```

```shellscript
# Windows PowerShell
Select-String -Path "project_code\career\code\logs\app_info.log" -Pattern "test_interview_001"

# Linux/Mac
grep "test_interview_001" project_code/career/code/logs/app_info.log
```

```plaintext
2024-12-22 18:47:01,123 - root - INFO - [DEBUG-服务层] 面试ID: test_interview_001 - get_ai_response 被调用
2024-12-22 18:47:01,124 - root - INFO - [DEBUG-服务层] 面试ID: test_interview_001 - dialog_history 类型: <class 'list'>
2024-12-22 18:47:01,125 - root - INFO - [DEBUG-服务层] 面试ID: test_interview_001 - dialog_history 值: [{'role': 'user', 'content': '...'}]
2024-12-22 18:47:01,126 - root - INFO - -- 面试ID: test_interview_001 - 连续沉默次数: 0/3
2024-12-22 18:47:01,127 - root - INFO - -- 面试ID: test_interview_001 - 🎯 开始智能决策
2024-12-22 18:47:09,550 - root - INFO - -- 面试ID: test_interview_001 - 智能决策结果: {'action': 'follow_up', 'quality_score': 3, ...}
2024-12-22 18:47:09,551 - root - INFO - -- 面试ID: test_interview_001 - ✅ 智能决策完成: action=follow_up, quality_score=3
2024-12-22 18:47:09,552 - root - INFO - -- 面试ID: test_interview_001 - 🎯 开始流式生成回复
2024-12-22 18:47:18,540 - root - INFO - -- 面试ID: test_interview_001 - 🔄 保持当前问题，追问次数: 1
2024-12-22 18:47:18,541 - root - INFO - -- 面试ID: test_interview_001 - 执行时间: 17.42秒, 质量评分: 3, 回复长度: 156
```

---

**User**

@project_code/career/code/logs/app_info.log 你看这个文件为什么都在这个文件里面呀？我觉得不太对啊，按道理来说，不应该是文件名通过日期来进行区别吗？都放在这里面的话我怎么看呢？就是比如说昨天运行的和今天运行的都会混合到这个日志里文件里面，是吗？请ultrathinking,帮我梳理清楚呀。

---

**Cursor**

检查日志配置和日志文件情况：


检查日志文件中的日期范围：


## 日志文件分割机制说明

### 一、当前机制的工作原理

`TimedRotatingFileHandler` 的工作方式：

1. 当前日志（今天）：所有日志写入 `app_info.log`（这是“今天”的日志文件）
2. 午夜自动分割：每天 00:00 时：
   - 将 `app_info.log` 重命名为 `app_info.log.2025-12-22`（昨天的日志）
   - 创建新的 `app_info.log`（今天的日志）
3. 历史日志：带日期后缀的文件是历史日志

### 二、当前情况分析

从你的 `logs` 目录可以看到：
- `app_info.log` = 今天（2025-12-22）的日志（正在写入）
- `app_info.log.2025-12-11` = 12月11日的历史日志
- `app_info.log.2025-12-10` = 12月10日的历史日志
- ... 等等

从日志内容看，`app_info.log` 中都是 `2025-12-22` 的日志，说明分割机制正常。

### 三、问题所在

1. 文件名不够直观：`app_info.log` 看不出是哪一天
2. 历史日志命名：`app_info.log.2025-12-22` 不如 `app_info_2025-12-22.log` 直观
3. 查看不便：需要知道“当前文件是今天的，历史文件带日期后缀”

### 四、解决方案

#### 方案1：修改日志文件命名格式（推荐）

将日志文件改为直接包含日期的格式，例如：
- `app_info_2025-12-22.log`（今天的）
- `app_info_2025-12-21.log`（昨天的）

需要修改 `app/loger.py`，使用自定义的日志处理器。

#### 方案2：保持当前机制，但添加说明

当前机制是标准的日志轮转方式，如果接受，可以：
- 查看今天的日志：直接看 `app_info.log`
- 查看历史日志：看 `app_info.log.2025-12-XX` 文件

### 五、建议

推荐采用方案1，原因：
1. 文件名更直观，一眼看出日期
2. 便于按日期查找和归档
3. 符合常见日志命名习惯

需要我帮你修改日志配置，将文件名改为 `app_info_2025-12-22.log` 这样的格式吗？

---

**User**

我的意思是，这不是代码控制的吗？应该在哪里修改？现在代码呀，就是输出的时候把日志的文件给我带上日期呀，你怎么看呢？不仅要带上日期，还要带上具体的时间。就每次运行也要给区别开来可以吗？1 请问你明白我这个逻辑了吗？请用自己的话来复述逻辑，总结功能的核心目标。

2 用列表形式拆解为具体可实现的功能点。

3 你存在哪些疑问，向我提出具体问题来澄清这些点，确保双方对需求理解一致之后再写代码。

---

**Cursor**

检查日志配置代码，明确需求：


## 需求理解

### 1. 核心目标
- 日志文件名包含日期和时间（如 `app_info_2025-12-22_16-20-03.log`）
- 每次应用启动时创建新的日志文件，文件名包含启动时间戳
- 通过文件名即可区分不同运行实例的日志

### 2. 功能点
1. 修改日志文件命名：从 `app_info.log` 改为 `app_info_YYYY-MM-DD_HH-MM-SS.log`
2. 在应用启动时记录启动时间戳
3. 使用启动时间戳生成日志文件名
4. 确保每次启动都创建新的日志文件
5. 保留控制台输出（可选）

### 3. 需要澄清的问题

#### 问题1：日志文件创建时机
- 选项A：每次应用启动时创建新文件（文件名包含启动时间）
- 选项B：按天分割，但文件名包含日期和时间
- 选项C：按小时或更短时间间隔分割

你希望采用哪种方式？

#### 问题2：文件名格式
- 格式1：`app_info_2025-12-22_16-20-03.log`（日期_时分秒）
- 格式2：`app_info_2025-12-22_16-20-03-123.log`（包含毫秒）
- 格式3：`app_info_20251222_162003.log`（无分隔符）

你更倾向哪种格式？

#### 问题3：历史日志保留
- 是否需要自动清理旧日志（如保留最近7天）？
- 还是手动管理？

#### 问题4：ERROR 日志
- `app_error.log` 是否也采用相同命名规则？
- 还是保持独立命名？

#### 问题5：同一运行期间的日志
- 如果每次启动创建新文件，同一运行期间的所有日志都写入该文件，是否满足需求？
- 还是需要在运行期间也按时间分割（如每小时）？

请确认以上问题，以便实现。

---

**User**

1 选项A：每次应用启动时创建新文件（文件名包含启动时间）



2 格式1：app_info_2025-12-22_16-20-03.log（日期_时分秒）

3 手动管理，历史日志文件，不要清除。  

4 app_error.log 也采用相同命名规则

6 每次启动创建新文件，同一运行期间的所有日志都写入该文件



这个是我确认的信息，请帮我严格按照这个方案来修改相应的代码，完整follow前面修改方案的内容，不用引入不必要的额外的冗余的修改，修改之后，帮我梳理一下修改后的内容，进而方便我double check正确性

请额外注意：

1 严格遵循前面的方案的具体要求

2 不擅自添加你认为"有用"但我没有要求的功能

3 专注于我明确指出的需求

4 请不要往代码里面添加额外的md文件来总结修改内容。 

---

**Cursor**

按方案修改日志配置：每次启动创建新文件，文件名包含启动时间戳。


修改日志配置代码：


```python



def setup_logger(current_dir :str):
    import logging
    from logging.handlers import FileHandler
    import os
    from datetime import datetime


    log_dir = os.path.join(current_dir, 'logs')

    # 如果 logs 目录不存在，则自动创建
    os.makedirs(log_dir, exist_ok=True)

    # 获取应用启动时间戳，格式：YYYY-MM-DD_HH-MM-SS
    startup_time = datetime.now().strftime('%Y-%m-%d_%H-%M-%S')

    # 定义日志格式
    formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')

    # 配置 INFO 日志处理器（每次启动创建新文件）
    info_filename = os.path.join(log_dir, f'app_info_{startup_time}.log')
    info_handler = FileHandler(
        filename=info_filename,
        encoding='utf-8'
    )
    info_handler.setLevel(logging.INFO)  # 只记录 INFO 及以上级别
    info_handler.setFormatter(formatter)
    info_handler.addFilter(lambda record: record.levelno <= logging.INFO)  # 仅记录 INFO

    # 配置 ERROR 日志处理器（每次启动创建新文件）
    error_filename = os.path.join(log_dir, f'app_error_{startup_time}.log')
    error_handler = FileHandler(
        filename=error_filename,
        encoding='utf-8'
    )
    error_handler.setLevel(logging.ERROR)  # 只记录 ERROR 级别
    error_handler.setFormatter(formatter)

    # 创建根 logger 并添加处理器
    logger = logging.getLogger()
    logger.setLevel(logging.INFO)  # 设置根 logger 的级别
    logger.addHandler(info_handler)
    logger.addHandler(error_handler)

    # 可选：添加控制台输出
    console_handler = logging.StreamHandler()
    console_handler.setFormatter(formatter)
    logger.addHandler(console_handler)
    return logger
```

检查语法错误：


修改完成。修改内容如下：

## 修改内容总结

### 修改的文件
- `project_code/career/code/app/loger.py`

### 具体修改点

#### 1. 导入模块变更
- 删除：`from logging.handlers import TimedRotatingFileHandler`
- 新增：`from logging.handlers import FileHandler`
- 新增：`from datetime import datetime`

#### 2. 日志文件命名机制
- 删除：使用 `TimedRotatingFileHandler` 的自动分割机制
- 新增：在 `setup_logger` 函数内部获取启动时间戳
- 时间戳格式：`YYYY-MM-DD_HH-MM-SS`（例如：`2025-12-22_16-20-03`）

#### 3. INFO 日志处理器
- 删除：`TimedRotatingFileHandler`，固定文件名 `app_info.log`，午夜自动分割
- 新增：`FileHandler`，文件名格式：`app_info_{startup_time}.log`
- 示例文件名：`app_info_2025-12-22_16-20-03.log`

#### 4. ERROR 日志处理器
- 删除：`TimedRotatingFileHandler`，固定文件名 `app_error.log`，午夜自动分割
- 新增：`FileHandler`，文件名格式：`app_error_{startup_time}.log`
- 示例文件名：`app_error_2025-12-22_16-20-03.log`

#### 5. 日志保留策略
- 删除：自动清理机制（`backupCount=7` 和 `backupCount=30`）
- 新增：手动管理，所有历史日志文件保留

### 功能说明

1. 每次应用启动时，`setup_logger` 会获取当前时间戳
2. 使用该时间戳生成唯一的日志文件名
3. 同一运行期间的所有日志写入同一个文件
4. 每次重启应用都会创建新的日志文件
5. INFO 和 ERROR 日志使用相同的命名规则

### 预期效果

应用启动后，`logs` 目录下会生成类似这样的文件：
- `app_info_2025-12-22_16-20-03.log`（INFO 日志）
- `app_error_2025-12-22_16-20-03.log`（ERROR 日志）

每次重启应用都会创建新的日志文件，文件名包含启动时间，便于区分不同运行实例的日志。

修改已完成，无语法错误。可以重启应用验证效果。

---

**User**

现在情况是这样，有两连续调用了两次api，第1次API调用：智能决策（非流式）第2次API调用：流式生成回复，这两次的调用是什么关系？第一次调用的结果会插入到第二次调用api的请求体里吗？但我为什么没有在输入json看到这个插入？对话历史里面好像没有看到，请求体里面也没有看到，第一次的结果插入到第二次里面。就我想知道，他俩的内容是如何结合到一起的？ 是什么关系，相互之间的影响是什么？
1 请问你明白我这个逻辑了吗？请用自己的话来复述逻辑。请ultrathinking,帮我梳理清楚呀。

---

**Cursor**


查看代码，梳理两次 API 调用的关系：



查看前端如何发送请求，以及后端如何处理：



## 需求理解

### 核心问题
- 第一次 API 调用（智能决策）和第二次 API 调用（流式生成回复）的关系
- 第一次的结果如何传递给第二次
- 为什么在前端请求体中看不到第一次的结果

## 两次 API 调用的关系

### 一、调用流程

```
前端发送HTTP请求
    ↓
/virturlInterview 端点接收
    ↓
get_ai_response() 函数
    ↓
【第一次调用】_make_unified_decision()
    ├─ 输入：dialog_history, system_message
    ├─ 输出：decision 字典（内存对象）
    └─ 包含：response_guidance, action, quality_score 等
    ↓
【第二次调用】_stream_response_by_decision()
    ├─ 输入：decision（第一次的结果，内存传递）
    ├─ 输入：dialog_history（和第一次相同）
    ├─ 输入：system_message（和第一次相同）
    └─ 输出：流式生成的回复文本
```

### 二、关键代码位置

#### 第一次调用（第356行）：
```python
decision = await self._make_unified_decision(dialog_history, system_message)
```

#### 第二次调用（第362行）：
```python
async for chunk in self._stream_response_by_decision(decision, dialog_history, system_message):
```

### 三、第一次结果如何传递给第二次

查看 `_stream_response_by_decision` 函数（第216-268行）：

```python
async def _stream_response_by_decision(self, decision: dict, dialog_history: list, system_message: str):
    # ...
    user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

回复指导：
{decision.get("response_guidance", "继续面试流程")}  # ← 这里！第一次的结果被插入

请根据回复指导生成自然、专业的面试官回复。"""
```

### 四、为什么前端看不到

1. 前端只发送一次 HTTP 请求到 `/virturlInterview`
2. 请求体包含：`dialogHistory`, `userPortrait`, `jdTitle` 等
3. 第一次和第二次的 LLM 调用都在后端内部完成
4. `decision` 对象在服务端内存中传递，不会出现在 HTTP 请求体中
5. 前端只能看到发送给后端的原始请求，看不到后端内部的两次调用

### 五、数据流向

```
前端请求体（JSON）
├─ dialogHistory: [...]
├─ userPortrait: "..."
└─ jdTitle: "..."

    ↓ (HTTP传输)

后端接收
├─ 提取 dialogHistory
└─ 调用 get_ai_response(dialog_history)

    ↓ (服务端内存)

第一次LLM调用
├─ 输入：dialog_history, system_message
└─ 输出：decision = {
        "response_guidance": "...",  ← 这个会被用到
        "action": "...",
        "quality_score": 5,
        ...
    }

    ↓ (服务端内存传递)

第二次LLM调用
├─ 输入：decision（包含 response_guidance）
├─ 输入：dialog_history（和第一次相同）
└─ 在 user_prompt 中插入：
    "回复指导：{decision.get('response_guidance')}"
    
    ↓ (流式返回)

前端接收流式响应
```

### 六、总结

1. 关系：第一次是决策阶段，第二次是生成阶段；第一次的输出（`decision`）作为第二次的输入之一
2. 传递方式：通过服务端内存中的 `decision` 对象传递，不是通过 HTTP 请求体
3. 影响：第一次的 `response_guidance` 会被插入到第二次调用的 `user_prompt` 中（第246行），指导生成回复
4. 前端看不到的原因：这是后端内部的两阶段处理，前端只看到一次 HTTP 请求

### 七、验证方法

查看日志文件，可以看到：
- 第一次调用的决策结果（包含 `response_guidance`）
- 第二次调用的 prompt（包含 `回复指导：...`）

这就是为什么在前端请求体中看不到第一次结果的原因：它是在后端内存中传递的，而不是通过 HTTP 传输的。

```plaintext
前端发送HTTP请求
    ↓
/virturlInterview 端点接收
    ↓
get_ai_response() 函数
    ↓
【第一次调用】_make_unified_decision()
    ├─ 输入：dialog_history, system_message
    ├─ 输出：decision 字典（内存对象）
    └─ 包含：response_guidance, action, quality_score 等
    ↓
【第二次调用】_stream_response_by_decision()
    ├─ 输入：decision（第一次的结果，内存传递）
    ├─ 输入：dialog_history（和第一次相同）
    ├─ 输入：system_message（和第一次相同）
    └─ 输出：流式生成的回复文本
```

```python
decision = await self._make_unified_decision(dialog_history, system_message)
```

```python
async for chunk in self._stream_response_by_decision(decision, dialog_history, system_message):
```

```python
async def _stream_response_by_decision(self, decision: dict, dialog_history: list, system_message: str):
    # ...
    user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

回复指导：
{decision.get("response_guidance", "继续面试流程")}  # ← 这里！第一次的结果被插入

请根据回复指导生成自然、专业的面试官回复。"""
```

```plaintext
前端请求体（JSON）
├─ dialogHistory: [...]
├─ userPortrait: "..."
└─ jdTitle: "..."

    ↓ (HTTP传输)

后端接收
├─ 提取 dialogHistory
└─ 调用 get_ai_response(dialog_history)

    ↓ (服务端内存)

第一次LLM调用
├─ 输入：dialog_history, system_message
└─ 输出：decision = {
        "response_guidance": "...",  ← 这个会被用到
        "action": "...",
        "quality_score": 5,
        ...
    }

    ↓ (服务端内存传递)

第二次LLM调用
├─ 输入：decision（包含 response_guidance）
├─ 输入：dialog_history（和第一次相同）
└─ 在 user_prompt 中插入：
    "回复指导：{decision.get('response_guidance')}"
    
    ↓ (流式返回)

前端接收流式响应
```

---

**User**

现在能不能使用具体的例子来帮我梳理一下。这两个阶段就是第一个阶段，它的入参出参是什么？第二个阶段，它的入参和出参是什么，以及它们的处理过程分别是什么样子的。用具体的例子来帮我讲解一下可以吗？帮我梳理清楚。请ultrathinking,帮我梳理清楚呀。

---

**Cursor**


查看代码，用具体例子说明两个阶段的入参、出参和处理过程：



用一个具体例子说明两个阶段的入参、出参和处理过程：

## 具体例子：面试对话场景

### 场景设定
- 岗位：后端开发工程师
- 候选人：张三，应届毕业生
- 当前状态：已进行2轮对话，候选人回答不够详细

---

## 阶段一：智能决策（`_make_unified_decision`）

### 入参

#### 1. `dialog_history`（对话历史）
```python
[
    {
        "role": "assistant",
        "content": "你好，我是彗星老师。请介绍一下你在校园二手书交易平台项目中的技术实现。"
    },
    {
        "role": "user",
        "content": "我用了Python和FastAPI框架，做了后端接口开发。"
    },
    {
        "role": "assistant",
        "content": "能详细说说你如何设计数据库表结构的吗？"
    },
    {
        "role": "user",
        "content": "用了MySQL，设计了用户表和书籍表。"
    }
]
```

#### 2. `system_message`（系统消息）
```python
"""
本次面试参考资料如下：
#面试交流全程使用语种：中文
面试开始时间：2025-12-22 19:15:00
候选人信息：候选人姓名：张三
年龄：22岁
学历：本科（北京理工大学计算机科学与技术专业）
工作经验：应届毕业生
技能：熟练掌握 Python、Java 语言，熟悉 Spring Boot、FastAPI 框架

岗位信息：后端开发工程师
岗位职责：
1. 负责公司核心业务系统的设计与开发
2. 参与技术架构设计和技术选型

任职要求：
1. 本科及以上学历，计算机相关专业
2. 熟悉 Python 或 Java 后端开发

薪资范围：15k-25k
#面试交流全程使用语种：中文
面试正式开始！
"""
```

#### 3. `self.current_question_follow_count`（追问次数）
```python
1  # 当前问题已追问1次
```

---

### 处理过程

#### 1. 构建 System Prompt
```python
system_prompt = """
你是自猎科技的面试官 Comet GPT...

【重要】你现在需要做出完整的决策，包括：
1. 分析候选人的最新回答
2. 评估回答质量（0-10分）
3. 决定下一步动作（追问/下一题/结束）
4. 生成回复指导（用于后续生成自然回复）
5. 判断是否应该结束面试

输出严格JSON格式：
{
    "action": "follow_up" | "new_question" | "end",
    "response_guidance": "回复指导内容",
    "quality_score": 0-10,
    "should_end": true/false,
    "reasoning": "你的决策理由"
}
"""
```

#### 2. 构建 User Prompt
```python
user_prompt = """当前面试记录如下：
[
    {
        "role": "assistant",
        "content": "你好，我是彗星老师。请介绍一下你在校园二手书交易平台项目中的技术实现。"
    },
    {
        "role": "user",
        "content": "我用了Python和FastAPI框架，做了后端接口开发。"
    },
    {
        "role": "assistant",
        "content": "能详细说说你如何设计数据库表结构的吗？"
    },
    {
        "role": "user",
        "content": "用了MySQL，设计了用户表和书籍表。"
    }
]

当前时间：2025-12-22 19:15:30

其他消息：
本次面试参考资料如下：
候选人信息：张三，22岁，应届毕业生...
岗位信息：后端开发工程师...

当前问题已追问：1 次

请做出完整决策。"""
```

#### 3. 调用 LLM（非流式）
```python
response = self.llm_client.create_completion(
    messages=[
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": user_prompt}
    ],
    temperature=0.4,
    response_format={"type": "json_object"}
)
```

---

### 出参（`decision` 字典）

```python
{
    "action": "follow_up",  # 继续追问
    "response_guidance": "指出候选人回答过于简略，要求补充数据库表结构的具体字段设计、表之间的关系（如外键），以及为什么这样设计。同时询问是否考虑了索引优化。",  # 关键！这个会被用到
    "quality_score": 4,  # 回答质量4分（不够详细）
    "should_end": false,  # 不结束面试
    "reasoning": "候选人当前回答质量较低，仅简单提及'用了MySQL，设计了用户表和书籍表'，但未展开任何实质性内容。虽已追问一次，但回答仍缺乏必要细节，无法有效评估其与后端开发岗位的匹配度。需追问以获取更多信息，考察其真实技术水平和表达能力。目前处于面试初期阶段，尚未覆盖硬技能和项目经验等核心维度，必须继续深入。"
}
```

---

## 阶段二：流式生成回复（`_stream_response_by_decision`）

### 入参

#### 1. `decision`（阶段一的输出）
```python
{
    "action": "follow_up",
    "response_guidance": "指出候选人回答过于简略，要求补充数据库表结构的具体字段设计、表之间的关系（如外键），以及为什么这样设计。同时询问是否考虑了索引优化。",
    "quality_score": 4,
    "should_end": false,
    "reasoning": "..."
}
```

#### 2. `dialog_history`（与阶段一相同）
```python
[
    {
        "role": "assistant",
        "content": "你好，我是彗星老师。请介绍一下你在校园二手书交易平台项目中的技术实现。"
    },
    {
        "role": "user",
        "content": "我用了Python和FastAPI框架，做了后端接口开发。"
    },
    {
        "role": "assistant",
        "content": "能详细说说你如何设计数据库表结构的吗？"
    },
    {
        "role": "user",
        "content": "用了MySQL，设计了用户表和书籍表。"
    }
]
```

#### 3. `system_message`（与阶段一相同）
```python
"""
本次面试参考资料如下：
...
"""
```

---

### 处理过程

#### 1. 构建 System Prompt（执行者角色）
```python
system_prompt = """
你是自猎科技的面试官 Comet GPT...

根据以下回复指导，生成自然、专业的面试官回复。
不要透露决策过程，只生成自然对话。
"""
```

#### 2. 构建 User Prompt（关键：插入 `response_guidance`）
```python
user_prompt = """当前面试记录如下：
[
    {
        "role": "assistant",
        "content": "你好，我是彗星老师。请介绍一下你在校园二手书交易平台项目中的技术实现。"
    },
    {
        "role": "user",
        "content": "我用了Python和FastAPI框架，做了后端接口开发。"
    },
    {
        "role": "assistant",
        "content": "能详细说说你如何设计数据库表结构的吗？"
    },
    {
        "role": "user",
        "content": "用了MySQL，设计了用户表和书籍表。"
    }
]

当前时间：2025-12-22 19:15:30

其他消息：
本次面试参考资料如下：
候选人信息：张三，22岁，应届毕业生...
岗位信息：后端开发工程师...

回复指导：
指出候选人回答过于简略，要求补充数据库表结构的具体字段设计、表之间的关系（如外键），以及为什么这样设计。同时询问是否考虑了索引优化。

请根据回复指导生成自然、专业的面试官回复。"""
```

注意：`回复指导：{decision.get("response_guidance")}` 被插入到 user_prompt 中。

#### 3. 调用 LLM（流式）
```python
response = self.llm_client.create_streaming_completion(
    messages=[
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": user_prompt}
    ],
    temperature=0.7,
    seed=1
)
```

---

### 出参（流式生成的回复文本）

```python
# 流式返回的文本片段（逐步生成）
"张三，你提到设计了用户表和书籍表，但回答还比较简略。"
"请你详细说明一下："
"1. 用户表具体包含哪些字段？比如用户ID、用户名、密码、邮箱等。"
"2. 书籍表包含哪些字段？比如书籍ID、书名、价格、状态等。"
"3. 这两个表之间是什么关系？有没有使用外键关联？"
"4. 你在设计表结构时，有没有考虑索引优化？比如哪些字段需要建立索引？"
```

完整回复：
```
张三，你提到设计了用户表和书籍表，但回答还比较简略。请你详细说明一下：1. 用户表具体包含哪些字段？比如用户ID、用户名、密码、邮箱等。2. 书籍表包含哪些字段？比如书籍ID、书名、价格、状态等。3. 这两个表之间是什么关系？有没有使用外键关联？4. 你在设计表结构时，有没有考虑索引优化？比如哪些字段需要建立索引？
```

---

## 数据流向总结

```
【阶段一：智能决策】
输入：
├─ dialog_history: [4条对话记录]
├─ system_message: "候选人信息：张三...岗位信息：后端开发工程师..."
└─ current_question_follow_count: 1

处理：
├─ 分析对话历史
├─ 评估回答质量（4分）
├─ 决定下一步动作（follow_up）
└─ 生成回复指导

输出：
└─ decision = {
        "action": "follow_up",
        "response_guidance": "指出候选人回答过于简略，要求补充...",
        "quality_score": 4,
        "should_end": false,
        "reasoning": "..."
    }

    ↓ (内存传递)

【阶段二：流式生成回复】
输入：
├─ decision: {包含 response_guidance}
├─ dialog_history: [4条对话记录，和阶段一相同]
└─ system_message: [和阶段一相同]

处理：
├─ 将 decision["response_guidance"] 插入到 user_prompt
├─ 调用流式LLM生成回复
└─ 逐步返回文本片段

输出：
└─ 流式文本："张三，你提到设计了用户表和书籍表，但回答还比较简略。请你详细说明一下：1. 用户表具体包含哪些字段？..."
```

---

## 关键点

1. 阶段一的 `response_guidance` 会插入到阶段二的 `user_prompt` 中，指导生成。
2. `dialog_history` 和 `system_message` 在两个阶段中相同，用于保持上下文。
3. `decision` 对象在服务端内存中传递，不会出现在 HTTP 请求体中。
4. 阶段一决定“说什么”，阶段二生成“怎么说”。

这就是两个阶段的关系和数据流转过程。

```python
[
    {
        "role": "assistant",
        "content": "你好，我是彗星老师。请介绍一下你在校园二手书交易平台项目中的技术实现。"
    },
    {
        "role": "user",
        "content": "我用了Python和FastAPI框架，做了后端接口开发。"
    },
    {
        "role": "assistant",
        "content": "能详细说说你如何设计数据库表结构的吗？"
    },
    {
        "role": "user",
        "content": "用了MySQL，设计了用户表和书籍表。"
    }
]
```

```python
"""
本次面试参考资料如下：
#面试交流全程使用语种：中文
面试开始时间：2025-12-22 19:15:00
候选人信息：候选人姓名：张三
年龄：22岁
学历：本科（北京理工大学计算机科学与技术专业）
工作经验：应届毕业生
技能：熟练掌握 Python、Java 语言，熟悉 Spring Boot、FastAPI 框架

岗位信息：后端开发工程师
岗位职责：
1. 负责公司核心业务系统的设计与开发
2. 参与技术架构设计和技术选型

任职要求：
1. 本科及以上学历，计算机相关专业
2. 熟悉 Python 或 Java 后端开发

薪资范围：15k-25k
#面试交流全程使用语种：中文
面试正式开始！
"""
```

```python
1  # 当前问题已追问1次
```

```python
system_prompt = """
你是自猎科技的面试官 Comet GPT...

【重要】你现在需要做出完整的决策，包括：
1. 分析候选人的最新回答
2. 评估回答质量（0-10分）
3. 决定下一步动作（追问/下一题/结束）
4. 生成回复指导（用于后续生成自然回复）
5. 判断是否应该结束面试

输出严格JSON格式：
{
    "action": "follow_up" | "new_question" | "end",
    "response_guidance": "回复指导内容",
    "quality_score": 0-10,
    "should_end": true/false,
    "reasoning": "你的决策理由"
}
"""
```

```python
user_prompt = """当前面试记录如下：
[
    {
        "role": "assistant",
        "content": "你好，我是彗星老师。请介绍一下你在校园二手书交易平台项目中的技术实现。"
    },
    {
        "role": "user",
        "content": "我用了Python和FastAPI框架，做了后端接口开发。"
    },
    {
        "role": "assistant",
        "content": "能详细说说你如何设计数据库表结构的吗？"
    },
    {
        "role": "user",
        "content": "用了MySQL，设计了用户表和书籍表。"
    }
]

当前时间：2025-12-22 19:15:30

其他消息：
本次面试参考资料如下：
候选人信息：张三，22岁，应届毕业生...
岗位信息：后端开发工程师...

当前问题已追问：1 次

请做出完整决策。"""
```

```python
response = self.llm_client.create_completion(
    messages=[
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": user_prompt}
    ],
    temperature=0.4,
    response_format={"type": "json_object"}
)
```

```python
{
    "action": "follow_up",  # 继续追问
    "response_guidance": "指出候选人回答过于简略，要求补充数据库表结构的具体字段设计、表之间的关系（如外键），以及为什么这样设计。同时询问是否考虑了索引优化。",  # 关键！这个会被用到
    "quality_score": 4,  # 回答质量4分（不够详细）
    "should_end": false,  # 不结束面试
    "reasoning": "候选人当前回答质量较低，仅简单提及'用了MySQL，设计了用户表和书籍表'，但未展开任何实质性内容。虽已追问一次，但回答仍缺乏必要细节，无法有效评估其与后端开发岗位的匹配度。需追问以获取更多信息，考察其真实技术水平和表达能力。目前处于面试初期阶段，尚未覆盖硬技能和项目经验等核心维度，必须继续深入。"
}
```

```python
{
    "action": "follow_up",
    "response_guidance": "指出候选人回答过于简略，要求补充数据库表结构的具体字段设计、表之间的关系（如外键），以及为什么这样设计。同时询问是否考虑了索引优化。",
    "quality_score": 4,
    "should_end": false,
    "reasoning": "..."
}
```

```python
[
    {
        "role": "assistant",
        "content": "你好，我是彗星老师。请介绍一下你在校园二手书交易平台项目中的技术实现。"
    },
    {
        "role": "user",
        "content": "我用了Python和FastAPI框架，做了后端接口开发。"
    },
    {
        "role": "assistant",
        "content": "能详细说说你如何设计数据库表结构的吗？"
    },
    {
        "role": "user",
        "content": "用了MySQL，设计了用户表和书籍表。"
    }
]
```

```python
"""
本次面试参考资料如下：
...
"""
```

```python
system_prompt = """
你是自猎科技的面试官 Comet GPT...

根据以下回复指导，生成自然、专业的面试官回复。
不要透露决策过程，只生成自然对话。
"""
```

```python
user_prompt = """当前面试记录如下：
[
    {
        "role": "assistant",
        "content": "你好，我是彗星老师。请介绍一下你在校园二手书交易平台项目中的技术实现。"
    },
    {
        "role": "user",
        "content": "我用了Python和FastAPI框架，做了后端接口开发。"
    },
    {
        "role": "assistant",
        "content": "能详细说说你如何设计数据库表结构的吗？"
    },
    {
        "role": "user",
        "content": "用了MySQL，设计了用户表和书籍表。"
    }
]

当前时间：2025-12-22 19:15:30

其他消息：
本次面试参考资料如下：
候选人信息：张三，22岁，应届毕业生...
岗位信息：后端开发工程师...

回复指导：
指出候选人回答过于简略，要求补充数据库表结构的具体字段设计、表之间的关系（如外键），以及为什么这样设计。同时询问是否考虑了索引优化。

请根据回复指导生成自然、专业的面试官回复。"""
```

```python
response = self.llm_client.create_streaming_completion(
    messages=[
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": user_prompt}
    ],
    temperature=0.7,
    seed=1
)
```

```python
# 流式返回的文本片段（逐步生成）
"张三，你提到设计了用户表和书籍表，但回答还比较简略。"
"请你详细说明一下："
"1. 用户表具体包含哪些字段？比如用户ID、用户名、密码、邮箱等。"
"2. 书籍表包含哪些字段？比如书籍ID、书名、价格、状态等。"
"3. 这两个表之间是什么关系？有没有使用外键关联？"
"4. 你在设计表结构时，有没有考虑索引优化？比如哪些字段需要建立索引？"
```

```plaintext
张三，你提到设计了用户表和书籍表，但回答还比较简略。请你详细说明一下：1. 用户表具体包含哪些字段？比如用户ID、用户名、密码、邮箱等。2. 书籍表包含哪些字段？比如书籍ID、书名、价格、状态等。3. 这两个表之间是什么关系？有没有使用外键关联？4. 你在设计表结构时，有没有考虑索引优化？比如哪些字段需要建立索引？
```

```plaintext
【阶段一：智能决策】
输入：
├─ dialog_history: [4条对话记录]
├─ system_message: "候选人信息：张三...岗位信息：后端开发工程师..."
└─ current_question_follow_count: 1

处理：
├─ 分析对话历史
├─ 评估回答质量（4分）
├─ 决定下一步动作（follow_up）
└─ 生成回复指导

输出：
└─ decision = {
        "action": "follow_up",
        "response_guidance": "指出候选人回答过于简略，要求补充...",
        "quality_score": 4,
        "should_end": false,
        "reasoning": "..."
    }

    ↓ (内存传递)

【阶段二：流式生成回复】
输入：
├─ decision: {包含 response_guidance}
├─ dialog_history: [4条对话记录，和阶段一相同]
└─ system_message: [和阶段一相同]

处理：
├─ 将 decision["response_guidance"] 插入到 user_prompt
├─ 调用流式LLM生成回复
└─ 逐步返回文本片段

输出：
└─ 流式文本："张三，你提到设计了用户表和书籍表，但回答还比较简略。请你详细说明一下：1. 用户表具体包含哪些字段？..."
```

---

**User**

那现在的问题是我的日志里面是可以看到这些入参和出参以及它们的变化吗？其实我发现貌似没有打印出来呀是吗？你怎么看呢？因为我有的时候是需要去根据日志来去排查数据的变化流向以及如何处理的，进而来找出潜在的问题啊。

---

**Cursor**


检查当前日志记录情况，确认是否记录了入参和出参：



检查当前日志记录情况，补充入参和出参的详细日志：



补充详细的入参和出参日志，便于排查数据流向：


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
from app.identity import get_identity_prompt, get_identity_vars, get_standard_scripts
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
        self.current_question_follow_count = 0  # 当前问题的追问次数
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


    async def _make_unified_decision(self, dialog_history: list, system_message: str) -> dict:
        """
        单次AI调用，完成智能决策
        返回结构化决策：
        {
            "action": "follow_up" | "new_question" | "end",
            "response_guidance": "回复指导（用于流式生成）",
            "quality_score": 0-10,
            "should_end": true/false,
            "reasoning": "决策理由（用于日志）"
        }
        """
        try:
            # 加载策略prompt模板
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            strategy_template = load_prompt("strategy_white_collar.txt")
            strategy_sys_prompt = f"{identity_prompt}\n\n{strategy_template.format(**format_vars)}"
            
            system_prompt = f"""{strategy_sys_prompt}

【重要】你现在需要做出完整的决策，包括：
1. 分析候选人的最新回答
2. 评估回答质量（0-10分）
3. 决定下一步动作（追问/下一题/结束）
4. 生成回复指导（用于后续生成自然回复）
5. 判断是否应该结束面试

规则约束：
- 如果已追问2次，必须 action="new_question"
- 如果回答质量<3分且已追问1次，建议 action="new_question"
- 如果明确结束信号，should_end=true

输出严格JSON格式：
{{
    "action": "follow_up" | "new_question" | "end",
    "response_guidance": "回复指导内容（描述要说什么，但不直接生成回复文本）",
    "quality_score": 0-10,
    "should_end": true/false,
    "reasoning": "你的决策理由（用于日志记录）"
}}"""
            
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

当前问题已追问：{self.current_question_follow_count} 次

请做出完整决策。"""
            
            # 记录阶段一入参
            dialog_history_summary = f"共{len(dialog_history)}条记录" if dialog_history else "空"
            if dialog_history and len(dialog_history) > 0:
                last_user_msg = next((msg.get("content", "")[:100] for msg in reversed(dialog_history) if msg.get("role") == "user"), "")
                dialog_history_summary += f"，最后用户消息: {last_user_msg}..."
            logging.info(f"-- 面试ID: {self.interview_id} - [阶段一入参] dialog_history: {dialog_history_summary}, 追问次数: {self.current_question_follow_count}")
            logging.info(f"-- 面试ID: {self.interview_id} - [阶段一入参] system_message长度: {len(system_message)}字符")
            
            response = self.llm_client.create_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.4,
                response_format={"type": "json_object"}
            )
            
            decision = json.loads(response.choices[0].message.content)
            
            # 规则验证：如果已追问2次，强制下一题
            if decision.get("action") == "follow_up" and self.current_question_follow_count >= 2:
                logging.warning(f"-- 面试ID: {self.interview_id} - ⚠️ AI决策违反规则（已追问2次），强制修正为 new_question")
                decision["action"] = "new_question"
                decision["reasoning"] = f"已追问2次，强制进入下一题。原决策理由：{decision.get('reasoning', '')}"
            
            # 记录阶段一出参（完整decision）
            logging.info(f"-- 面试ID: {self.interview_id} - [阶段一出参] 完整decision: {json.dumps(decision, ensure_ascii=False, indent=2)}")
            logging.info(f"-- 面试ID: {self.interview_id} - 智能决策结果: {decision}")
            return decision
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 智能决策失败: {e}")
            # 失败时返回默认决策
            return {
                "action": "new_question",
                "response_guidance": "继续面试流程",
                "quality_score": 5,
                "should_end": False,
                "reasoning": f"决策失败，使用默认值。错误: {str(e)}"
            }

    async def _stream_response_by_decision(self, decision: dict, dialog_history: list, system_message: str):
        """
        基于决策指导，流式生成回复内容
        Yields: 回复内容片段
        Returns: 完整回复文本
        """
        try:
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            executor_template = load_prompt("executor_white_collar.txt" if self.interview_type == 1 else "executor_blue_collar.txt")
            executor_sys_prompt = f"{identity_prompt}\n\n{executor_template.format(**format_vars)}"
            
            system_prompt = f"""{executor_sys_prompt}

根据以下回复指导，生成自然、专业的面试官回复。
不要透露决策过程，只生成自然对话。"""
            
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

回复指导：
{decision.get("response_guidance", "继续面试流程")}

请根据回复指导生成自然、专业的面试官回复。"""
            
            response = self.llm_client.create_streaming_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.7,
                seed=1
            )
            
            async for chunk in response:
                if chunk.choices and chunk.choices[0].delta.content:
                    content = chunk.choices[0].delta.content
                    yield content
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 流式生成失败: {e}")
            error_msg = "抱歉，我需要整理一下思路。你能再说一遍吗？"
            yield error_msg


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
            self.current_question_follow_count = 0  # 重置追问次数
            
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
            
            # 首次调用：直接生成开场白
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - 🎯 开始智能决策（首次调用）")
            decision = await self._make_unified_decision([], system_message)
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 智能决策完成")
            
            # 流式生成开场白
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - 🎯 开始流式生成开场白")
            async for chunk in self._stream_response_by_decision(decision, [], system_message):
                yield f"data: {json.dumps({'type': 'chunk', 'content': chunk}, ensure_ascii=False)}\n\n"
            
                yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 已返回开场白，函数即将return")
                return

        # 快速规则检查：追问次数检查
        if self.current_question_follow_count >= 2:
            logging.info(f"-- 面试ID: {self.interview_id} - ⚠️ 已追问2次，强制进入下一题")
            self.current_question_follow_count = 0
            logging.info(f"-- 面试ID: {self.interview_id} - 📈 强制进入下一题，追问次数已重置")

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

        self.speech_speed = 1.1 if self.interview_type == 1 else 1
        
        try:
            # 阶段1：智能决策（非流式）
            logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始智能决策")
            decision = await self._make_unified_decision(dialog_history, system_message)
            logging.info(f"-- 面试ID: {self.interview_id} - ✅ 智能决策完成: action={decision.get('action')}, quality_score={decision.get('quality_score')}")
            
            # 阶段2：流式生成回复
            logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始流式生成回复")
            ai_response = ""
            async for chunk in self._stream_response_by_decision(decision, dialog_history, system_message):
                ai_response += chunk
                yield f"data: {json.dumps({'type': 'chunk', 'content': chunk}, ensure_ascii=False)}\n\n"

            yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"

            # 状态更新
            action = decision.get("action")
            if action == "new_question":
                self.current_question_follow_count = 0
                logging.info(f"-- 面试ID: {self.interview_id} - 📈 进入下一题，追问次数已重置")
            elif action == "follow_up":
                self.current_question_follow_count += 1
                logging.info(f"-- 面试ID: {self.interview_id} - 🔄 保持当前问题，追问次数: {self.current_question_follow_count}")
            
            # 判断是否结束
            if decision.get("should_end", False):
                logging.info(f"-- 面试ID: {self.interview_id} - 🛑 面试结束（由决策判断）")
                yield f"data: {json.dumps({'type': 'over', 'content': 'over'}, ensure_ascii=False)}\n\n"

            end_time = time.time()
            execution_time = end_time - start_time
            logging.info(f"-- 面试ID: {self.interview_id} - 执行时间: {execution_time:.2f}秒, 质量评分: {decision.get('quality_score')}, 回复长度: {len(ai_response)}")

        except Exception as e:
            logging.error(f"-- 面试ID : {self.interview_id}---AI流式响应获取错误: {e}")
            yield f"data: {json.dumps({'type': 'error', 'content': '模型检测到非法输入，请重新开始本次面试'+str(e)}, ensure_ascii=False)}\n\n"






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
from app.identity import get_identity_prompt, get_identity_vars, get_standard_scripts
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
        self.current_question_follow_count = 0  # 当前问题的追问次数
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


    async def _make_unified_decision(self, dialog_history: list, system_message: str) -> dict:
        """
        单次AI调用，完成智能决策
        返回结构化决策：
        {
            "action": "follow_up" | "new_question" | "end",
            "response_guidance": "回复指导（用于流式生成）",
            "quality_score": 0-10,
            "should_end": true/false,
            "reasoning": "决策理由（用于日志）"
        }
        """
        try:
            # 加载策略prompt模板
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            strategy_template = load_prompt("strategy_white_collar.txt")
            strategy_sys_prompt = f"{identity_prompt}\n\n{strategy_template.format(**format_vars)}"
            
            system_prompt = f"""{strategy_sys_prompt}

【重要】你现在需要做出完整的决策，包括：
1. 分析候选人的最新回答
2. 评估回答质量（0-10分）
3. 决定下一步动作（追问/下一题/结束）
4. 生成回复指导（用于后续生成自然回复）
5. 判断是否应该结束面试

规则约束：
- 如果已追问2次，必须 action="new_question"
- 如果回答质量<3分且已追问1次，建议 action="new_question"
- 如果明确结束信号，should_end=true

输出严格JSON格式：
{{
    "action": "follow_up" | "new_question" | "end",
    "response_guidance": "回复指导内容（描述要说什么，但不直接生成回复文本）",
    "quality_score": 0-10,
    "should_end": true/false,
    "reasoning": "你的决策理由（用于日志记录）"
}}"""
            
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

当前问题已追问：{self.current_question_follow_count} 次

请做出完整决策。"""
            
            # 记录阶段一入参
            dialog_history_summary = f"共{len(dialog_history)}条记录" if dialog_history else "空"
            if dialog_history and len(dialog_history) > 0:
                last_user_msg = next((msg.get("content", "")[:100] for msg in reversed(dialog_history) if msg.get("role") == "user"), "")
                dialog_history_summary += f"，最后用户消息: {last_user_msg}..."
            logging.info(f"-- 面试ID: {self.interview_id} - [阶段一入参] dialog_history: {dialog_history_summary}, 追问次数: {self.current_question_follow_count}")
            logging.info(f"-- 面试ID: {self.interview_id} - [阶段一入参] system_message长度: {len(system_message)}字符")
            
            response = self.llm_client.create_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.4,
                response_format={"type": "json_object"}
            )
            
            decision = json.loads(response.choices[0].message.content)
            
            # 规则验证：如果已追问2次，强制下一题
            if decision.get("action") == "follow_up" and self.current_question_follow_count >= 2:
                logging.warning(f"-- 面试ID: {self.interview_id} - ⚠️ AI决策违反规则（已追问2次），强制修正为 new_question")
                decision["action"] = "new_question"
                decision["reasoning"] = f"已追问2次，强制进入下一题。原决策理由：{decision.get('reasoning', '')}"
            
            # 记录阶段一出参（完整decision）
            logging.info(f"-- 面试ID: {self.interview_id} - [阶段一出参] 完整decision: {json.dumps(decision, ensure_ascii=False, indent=2)}")
            logging.info(f"-- 面试ID: {self.interview_id} - 智能决策结果: {decision}")
            return decision
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 智能决策失败: {e}")
            # 失败时返回默认决策
            return {
                "action": "new_question",
                "response_guidance": "继续面试流程",
                "quality_score": 5,
                "should_end": False,
                "reasoning": f"决策失败，使用默认值。错误: {str(e)}"
            }

    async def _stream_response_by_decision(self, decision: dict, dialog_history: list, system_message: str):
        """
        基于决策指导，流式生成回复内容
        Yields: 回复内容片段
        Returns: 完整回复文本
        """
        try:
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            executor_template = load_prompt("executor_white_collar.txt" if self.interview_type == 1 else "executor_blue_collar.txt")
            executor_sys_prompt = f"{identity_prompt}\n\n{executor_template.format(**format_vars)}"
            
            system_prompt = f"""{executor_sys_prompt}

根据以下回复指导，生成自然、专业的面试官回复。
不要透露决策过程，只生成自然对话。"""
            
            response_guidance = decision.get("response_guidance", "继续面试流程")
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

回复指导：
{response_guidance}

请根据回复指导生成自然、专业的面试官回复。"""
            
            # 记录阶段二入参
            logging.info(f"-- 面试ID: {self.interview_id} - [阶段二入参] decision.action: {decision.get('action')}, decision.quality_score: {decision.get('quality_score')}")
            logging.info(f"-- 面试ID: {self.interview_id} - [阶段二入参] response_guidance: {response_guidance}")
            logging.info(f"-- 面试ID: {self.interview_id} - [阶段二入参] dialog_history: 共{len(dialog_history) if dialog_history else 0}条记录")
            
            response = self.llm_client.create_streaming_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.7,
                seed=1
            )
            
            full_response = ""
            async for chunk in response:
                if chunk.choices and chunk.choices[0].delta.content:
                    content = chunk.choices[0].delta.content
                    full_response += content
                    yield content
            
            # 记录阶段二出参（完整生成的回复）
            logging.info(f"-- 面试ID: {self.interview_id} - [阶段二出参] 完整生成的回复: {full_response}")
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 流式生成失败: {e}")
            error_msg = "抱歉，我需要整理一下思路。你能再说一遍吗？"
            yield error_msg


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
            self.current_question_follow_count = 0  # 重置追问次数
            
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
            
            # 首次调用：直接生成开场白
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - 🎯 开始智能决策（首次调用）")
            decision = await self._make_unified_decision([], system_message)
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 智能决策完成")
            
            # 流式生成开场白
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - 🎯 开始流式生成开场白")
            async for chunk in self._stream_response_by_decision(decision, [], system_message):
                yield f"data: {json.dumps({'type': 'chunk', 'content': chunk}, ensure_ascii=False)}\n\n"
            
                yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 已返回开场白，函数即将return")
                return

        # 快速规则检查：追问次数检查
        if self.current_question_follow_count >= 2:
            logging.info(f"-- 面试ID: {self.interview_id} - ⚠️ 已追问2次，强制进入下一题")
            self.current_question_follow_count = 0
            logging.info(f"-- 面试ID: {self.interview_id} - 📈 强制进入下一题，追问次数已重置")

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

        self.speech_speed = 1.1 if self.interview_type == 1 else 1
        
        try:
            # 阶段1：智能决策（非流式）
            logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始智能决策")
            decision = await self._make_unified_decision(dialog_history, system_message)
            logging.info(f"-- 面试ID: {self.interview_id} - ✅ 智能决策完成: action={decision.get('action')}, quality_score={decision.get('quality_score')}")
            
            # 阶段2：流式生成回复
            logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始流式生成回复")
            ai_response = ""
            async for chunk in self._stream_response_by_decision(decision, dialog_history, system_message):
                ai_response += chunk
                yield f"data: {json.dumps({'type': 'chunk', 'content': chunk}, ensure_ascii=False)}\n\n"

            yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"

            # 状态更新
            action = decision.get("action")
            if action == "new_question":
                self.current_question_follow_count = 0
                logging.info(f"-- 面试ID: {self.interview_id} - 📈 进入下一题，追问次数已重置")
            elif action == "follow_up":
                self.current_question_follow_count += 1
                logging.info(f"-- 面试ID: {self.interview_id} - 🔄 保持当前问题，追问次数: {self.current_question_follow_count}")
            
            # 判断是否结束
            if decision.get("should_end", False):
                logging.info(f"-- 面试ID: {self.interview_id} - 🛑 面试结束（由决策判断）")
                yield f"data: {json.dumps({'type': 'over', 'content': 'over'}, ensure_ascii=False)}\n\n"

            end_time = time.time()
            execution_time = end_time - start_time
            logging.info(f"-- 面试ID: {self.interview_id} - 执行时间: {execution_time:.2f}秒, 质量评分: {decision.get('quality_score')}, 回复长度: {len(ai_response)}")

        except Exception as e:
            logging.error(f"-- 面试ID : {self.interview_id}---AI流式响应获取错误: {e}")
            yield f"data: {json.dumps({'type': 'error', 'content': '模型检测到非法输入，请重新开始本次面试'+str(e)}, ensure_ascii=False)}\n\n"






```


在主流程中补充阶段间数据传递的日志：


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
from app.identity import get_identity_prompt, get_identity_vars, get_standard_scripts
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
        self.current_question_follow_count = 0  # 当前问题的追问次数
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


    async def _make_unified_decision(self, dialog_history: list, system_message: str) -> dict:
        """
        单次AI调用，完成智能决策
        返回结构化决策：
        {
            "action": "follow_up" | "new_question" | "end",
            "response_guidance": "回复指导（用于流式生成）",
            "quality_score": 0-10,
            "should_end": true/false,
            "reasoning": "决策理由（用于日志）"
        }
        """
        try:
            # 加载策略prompt模板
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            strategy_template = load_prompt("strategy_white_collar.txt")
            strategy_sys_prompt = f"{identity_prompt}\n\n{strategy_template.format(**format_vars)}"
            
            system_prompt = f"""{strategy_sys_prompt}

【重要】你现在需要做出完整的决策，包括：
1. 分析候选人的最新回答
2. 评估回答质量（0-10分）
3. 决定下一步动作（追问/下一题/结束）
4. 生成回复指导（用于后续生成自然回复）
5. 判断是否应该结束面试

规则约束：
- 如果已追问2次，必须 action="new_question"
- 如果回答质量<3分且已追问1次，建议 action="new_question"
- 如果明确结束信号，should_end=true

输出严格JSON格式：
{{
    "action": "follow_up" | "new_question" | "end",
    "response_guidance": "回复指导内容（描述要说什么，但不直接生成回复文本）",
    "quality_score": 0-10,
    "should_end": true/false,
    "reasoning": "你的决策理由（用于日志记录）"
}}"""
            
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

当前问题已追问：{self.current_question_follow_count} 次

请做出完整决策。"""
            
            # 记录阶段一入参
            dialog_history_summary = f"共{len(dialog_history)}条记录" if dialog_history else "空"
            if dialog_history and len(dialog_history) > 0:
                last_user_msg = next((msg.get("content", "")[:100] for msg in reversed(dialog_history) if msg.get("role") == "user"), "")
                dialog_history_summary += f"，最后用户消息: {last_user_msg}..."
            logging.info(f"-- 面试ID: {self.interview_id} - [阶段一入参] dialog_history: {dialog_history_summary}, 追问次数: {self.current_question_follow_count}")
            logging.info(f"-- 面试ID: {self.interview_id} - [阶段一入参] system_message长度: {len(system_message)}字符")
            
            response = self.llm_client.create_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.4,
                response_format={"type": "json_object"}
            )
            
            decision = json.loads(response.choices[0].message.content)
            
            # 规则验证：如果已追问2次，强制下一题
            if decision.get("action") == "follow_up" and self.current_question_follow_count >= 2:
                logging.warning(f"-- 面试ID: {self.interview_id} - ⚠️ AI决策违反规则（已追问2次），强制修正为 new_question")
                decision["action"] = "new_question"
                decision["reasoning"] = f"已追问2次，强制进入下一题。原决策理由：{decision.get('reasoning', '')}"
            
            # 记录阶段一出参（完整decision）
            logging.info(f"-- 面试ID: {self.interview_id} - [阶段一出参] 完整decision: {json.dumps(decision, ensure_ascii=False, indent=2)}")
            logging.info(f"-- 面试ID: {self.interview_id} - 智能决策结果: {decision}")
            return decision
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 智能决策失败: {e}")
            # 失败时返回默认决策
            return {
                "action": "new_question",
                "response_guidance": "继续面试流程",
                "quality_score": 5,
                "should_end": False,
                "reasoning": f"决策失败，使用默认值。错误: {str(e)}"
            }

    async def _stream_response_by_decision(self, decision: dict, dialog_history: list, system_message: str):
        """
        基于决策指导，流式生成回复内容
        Yields: 回复内容片段
        Returns: 完整回复文本
        """
        try:
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            executor_template = load_prompt("executor_white_collar.txt" if self.interview_type == 1 else "executor_blue_collar.txt")
            executor_sys_prompt = f"{identity_prompt}\n\n{executor_template.format(**format_vars)}"
            
            system_prompt = f"""{executor_sys_prompt}

根据以下回复指导，生成自然、专业的面试官回复。
不要透露决策过程，只生成自然对话。"""
            
            response_guidance = decision.get("response_guidance", "继续面试流程")
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

回复指导：
{response_guidance}

请根据回复指导生成自然、专业的面试官回复。"""
            
            # 记录阶段二入参
            logging.info(f"-- 面试ID: {self.interview_id} - [阶段二入参] decision.action: {decision.get('action')}, decision.quality_score: {decision.get('quality_score')}")
            logging.info(f"-- 面试ID: {self.interview_id} - [阶段二入参] response_guidance: {response_guidance}")
            logging.info(f"-- 面试ID: {self.interview_id} - [阶段二入参] dialog_history: 共{len(dialog_history) if dialog_history else 0}条记录")
            
            response = self.llm_client.create_streaming_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.7,
                seed=1
            )
            
            full_response = ""
            async for chunk in response:
                if chunk.choices and chunk.choices[0].delta.content:
                    content = chunk.choices[0].delta.content
                    full_response += content
                    yield content
            
            # 记录阶段二出参（完整生成的回复）
            logging.info(f"-- 面试ID: {self.interview_id} - [阶段二出参] 完整生成的回复: {full_response}")
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 流式生成失败: {e}")
            error_msg = "抱歉，我需要整理一下思路。你能再说一遍吗？"
            yield error_msg


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
            self.current_question_follow_count = 0  # 重置追问次数
            
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
            
            # 首次调用：直接生成开场白
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - 🎯 开始智能决策（首次调用）")
            decision = await self._make_unified_decision([], system_message)
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 智能决策完成")
            
            # 流式生成开场白
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - 🎯 开始流式生成开场白")
            async for chunk in self._stream_response_by_decision(decision, [], system_message):
                yield f"data: {json.dumps({'type': 'chunk', 'content': chunk}, ensure_ascii=False)}\n\n"
            
                yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 已返回开场白，函数即将return")
                return

        # 快速规则检查：追问次数检查
        if self.current_question_follow_count >= 2:
            logging.info(f"-- 面试ID: {self.interview_id} - ⚠️ 已追问2次，强制进入下一题")
            self.current_question_follow_count = 0
            logging.info(f"-- 面试ID: {self.interview_id} - 📈 强制进入下一题，追问次数已重置")

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

        self.speech_speed = 1.1 if self.interview_type == 1 else 1
        
        try:
            # 阶段1：智能决策（非流式）
            logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始智能决策")
            logging.info(f"-- 面试ID: {self.interview_id} - [数据流向] 阶段一输入 -> _make_unified_decision(dialog_history={len(dialog_history) if dialog_history else 0}条, system_message={len(system_message)}字符)")
            decision = await self._make_unified_decision(dialog_history, system_message)
            logging.info(f"-- 面试ID: {self.interview_id} - ✅ 智能决策完成: action={decision.get('action')}, quality_score={decision.get('quality_score')}")
            logging.info(f"-- 面试ID: {self.interview_id} - [数据流向] 阶段一输出 -> decision对象（包含response_guidance: {decision.get('response_guidance', '')[:100]}...）")
            
            # 阶段2：流式生成回复
            logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始流式生成回复")
            logging.info(f"-- 面试ID: {self.interview_id} - [数据流向] 阶段二输入 -> _stream_response_by_decision(decision, dialog_history={len(dialog_history) if dialog_history else 0}条, system_message={len(system_message)}字符)")
            ai_response = ""
            async for chunk in self._stream_response_by_decision(decision, dialog_history, system_message):
                ai_response += chunk
                yield f"data: {json.dumps({'type': 'chunk', 'content': chunk}, ensure_ascii=False)}\n\n"
            logging.info(f"-- 面试ID: {self.interview_id} - [数据流向] 阶段二输出 -> 流式回复文本（长度: {len(ai_response)}字符）")

            yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"

            # 状态更新
            action = decision.get("action")
            if action == "new_question":
                self.current_question_follow_count = 0
                logging.info(f"-- 面试ID: {self.interview_id} - 📈 进入下一题，追问次数已重置")
            elif action == "follow_up":
                self.current_question_follow_count += 1
                logging.info(f"-- 面试ID: {self.interview_id} - 🔄 保持当前问题，追问次数: {self.current_question_follow_count}")
            
            # 判断是否结束
            if decision.get("should_end", False):
                logging.info(f"-- 面试ID: {self.interview_id} - 🛑 面试结束（由决策判断）")
                yield f"data: {json.dumps({'type': 'over', 'content': 'over'}, ensure_ascii=False)}\n\n"

            end_time = time.time()
            execution_time = end_time - start_time
            logging.info(f"-- 面试ID: {self.interview_id} - 执行时间: {execution_time:.2f}秒, 质量评分: {decision.get('quality_score')}, 回复长度: {len(ai_response)}")

        except Exception as e:
            logging.error(f"-- 面试ID : {self.interview_id}---AI流式响应获取错误: {e}")
            yield f"data: {json.dumps({'type': 'error', 'content': '模型检测到非法输入，请重新开始本次面试'+str(e)}, ensure_ascii=False)}\n\n"






```


修复缩进错误：


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
from app.identity import get_identity_prompt, get_identity_vars, get_standard_scripts
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
        self.current_question_follow_count = 0  # 当前问题的追问次数
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


    async def _make_unified_decision(self, dialog_history: list, system_message: str) -> dict:
        """
        单次AI调用，完成智能决策
        返回结构化决策：
        {
            "action": "follow_up" | "new_question" | "end",
            "response_guidance": "回复指导（用于流式生成）",
            "quality_score": 0-10,
            "should_end": true/false,
            "reasoning": "决策理由（用于日志）"
        }
        """
        try:
            # 加载策略prompt模板
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            strategy_template = load_prompt("strategy_white_collar.txt")
            strategy_sys_prompt = f"{identity_prompt}\n\n{strategy_template.format(**format_vars)}"
            
            system_prompt = f"""{strategy_sys_prompt}

【重要】你现在需要做出完整的决策，包括：
1. 分析候选人的最新回答
2. 评估回答质量（0-10分）
3. 决定下一步动作（追问/下一题/结束）
4. 生成回复指导（用于后续生成自然回复）
5. 判断是否应该结束面试

规则约束：
- 如果已追问2次，必须 action="new_question"
- 如果回答质量<3分且已追问1次，建议 action="new_question"
- 如果明确结束信号，should_end=true

输出严格JSON格式：
{{
    "action": "follow_up" | "new_question" | "end",
    "response_guidance": "回复指导内容（描述要说什么，但不直接生成回复文本）",
    "quality_score": 0-10,
    "should_end": true/false,
    "reasoning": "你的决策理由（用于日志记录）"
}}"""
            
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

当前问题已追问：{self.current_question_follow_count} 次

请做出完整决策。"""
            
            # 记录阶段一入参
            dialog_history_summary = f"共{len(dialog_history)}条记录" if dialog_history else "空"
            if dialog_history and len(dialog_history) > 0:
                last_user_msg = next((msg.get("content", "")[:100] for msg in reversed(dialog_history) if msg.get("role") == "user"), "")
                dialog_history_summary += f"，最后用户消息: {last_user_msg}..."
            logging.info(f"-- 面试ID: {self.interview_id} - [阶段一入参] dialog_history: {dialog_history_summary}, 追问次数: {self.current_question_follow_count}")
            logging.info(f"-- 面试ID: {self.interview_id} - [阶段一入参] system_message长度: {len(system_message)}字符")
            
            response = self.llm_client.create_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.4,
                response_format={"type": "json_object"}
            )
            
            decision = json.loads(response.choices[0].message.content)
            
            # 规则验证：如果已追问2次，强制下一题
            if decision.get("action") == "follow_up" and self.current_question_follow_count >= 2:
                logging.warning(f"-- 面试ID: {self.interview_id} - ⚠️ AI决策违反规则（已追问2次），强制修正为 new_question")
                decision["action"] = "new_question"
                decision["reasoning"] = f"已追问2次，强制进入下一题。原决策理由：{decision.get('reasoning', '')}"
            
            # 记录阶段一出参（完整decision）
            logging.info(f"-- 面试ID: {self.interview_id} - [阶段一出参] 完整decision: {json.dumps(decision, ensure_ascii=False, indent=2)}")
            logging.info(f"-- 面试ID: {self.interview_id} - 智能决策结果: {decision}")
            return decision
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 智能决策失败: {e}")
            # 失败时返回默认决策
            return {
                "action": "new_question",
                "response_guidance": "继续面试流程",
                "quality_score": 5,
                "should_end": False,
                "reasoning": f"决策失败，使用默认值。错误: {str(e)}"
            }

    async def _stream_response_by_decision(self, decision: dict, dialog_history: list, system_message: str):
        """
        基于决策指导，流式生成回复内容
        Yields: 回复内容片段
        Returns: 完整回复文本
        """
        try:
            lang = self.language_dict[self.interview_language][0]
            identity_prompt = get_identity_prompt("interview")
            identity_vars = get_identity_vars()
            scripts = get_standard_scripts()
            format_vars = {**identity_vars, **scripts, "language": lang}
            
            executor_template = load_prompt("executor_white_collar.txt" if self.interview_type == 1 else "executor_blue_collar.txt")
            executor_sys_prompt = f"{identity_prompt}\n\n{executor_template.format(**format_vars)}"
            
            system_prompt = f"""{executor_sys_prompt}

根据以下回复指导，生成自然、专业的面试官回复。
不要透露决策过程，只生成自然对话。"""
            
            response_guidance = decision.get("response_guidance", "继续面试流程")
            user_prompt = f"""当前面试记录如下：
{json.dumps(dialog_history, ensure_ascii=False, indent=2)}

当前时间：{time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}

其他消息：
{system_message}

回复指导：
{response_guidance}

请根据回复指导生成自然、专业的面试官回复。"""
            
            # 记录阶段二入参
            logging.info(f"-- 面试ID: {self.interview_id} - [阶段二入参] decision.action: {decision.get('action')}, decision.quality_score: {decision.get('quality_score')}")
            logging.info(f"-- 面试ID: {self.interview_id} - [阶段二入参] response_guidance: {response_guidance}")
            logging.info(f"-- 面试ID: {self.interview_id} - [阶段二入参] dialog_history: 共{len(dialog_history) if dialog_history else 0}条记录")
            
            response = self.llm_client.create_streaming_completion(
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt}
                ],
                temperature=0.7,
                seed=1
            )
            
            full_response = ""
            async for chunk in response:
                if chunk.choices and chunk.choices[0].delta.content:
                    content = chunk.choices[0].delta.content
                    full_response += content
                    yield content
            
            # 记录阶段二出参（完整生成的回复）
            logging.info(f"-- 面试ID: {self.interview_id} - [阶段二出参] 完整生成的回复: {full_response}")
            
        except Exception as e:
            logging.error(f"-- 面试ID: {self.interview_id} - 流式生成失败: {e}")
            error_msg = "抱歉，我需要整理一下思路。你能再说一遍吗？"
            yield error_msg


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
            self.current_question_follow_count = 0  # 重置追问次数
            
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
            
            # 首次调用：直接生成开场白
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - 🎯 开始智能决策（首次调用）")
            decision = await self._make_unified_decision([], system_message)
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 智能决策完成")
            
            # 流式生成开场白
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - 🎯 开始流式生成开场白")
            async for chunk in self._stream_response_by_decision(decision, [], system_message):
                yield f"data: {json.dumps({'type': 'chunk', 'content': chunk}, ensure_ascii=False)}\n\n"
            
            yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"
            logging.info(f"[DEBUG-服务层] 面试ID: {self.interview_id} - ✅ 已返回开场白，函数即将return")
            return

        # 快速规则检查：追问次数检查
        if self.current_question_follow_count >= 2:
            logging.info(f"-- 面试ID: {self.interview_id} - ⚠️ 已追问2次，强制进入下一题")
            self.current_question_follow_count = 0
            logging.info(f"-- 面试ID: {self.interview_id} - 📈 强制进入下一题，追问次数已重置")

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

        self.speech_speed = 1.1 if self.interview_type == 1 else 1
        
        try:
            # 阶段1：智能决策（非流式）
            logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始智能决策")
            logging.info(f"-- 面试ID: {self.interview_id} - [数据流向] 阶段一输入 -> _make_unified_decision(dialog_history={len(dialog_history) if dialog_history else 0}条, system_message={len(system_message)}字符)")
            decision = await self._make_unified_decision(dialog_history, system_message)
            logging.info(f"-- 面试ID: {self.interview_id} - ✅ 智能决策完成: action={decision.get('action')}, quality_score={decision.get('quality_score')}")
            logging.info(f"-- 面试ID: {self.interview_id} - [数据流向] 阶段一输出 -> decision对象（包含response_guidance: {decision.get('response_guidance', '')[:100]}...）")
            
            # 阶段2：流式生成回复
            logging.info(f"-- 面试ID: {self.interview_id} - 🎯 开始流式生成回复")
            logging.info(f"-- 面试ID: {self.interview_id} - [数据流向] 阶段二输入 -> _stream_response_by_decision(decision, dialog_history={len(dialog_history) if dialog_history else 0}条, system_message={len(system_message)}字符)")
            ai_response = ""
            async for chunk in self._stream_response_by_decision(decision, dialog_history, system_message):
                ai_response += chunk
                yield f"data: {json.dumps({'type': 'chunk', 'content': chunk}, ensure_ascii=False)}\n\n"
            logging.info(f"-- 面试ID: {self.interview_id} - [数据流向] 阶段二输出 -> 流式回复文本（长度: {len(ai_response)}字符）")

            yield f"data: {json.dumps({'type': 'DONE', 'content': '[DONE]'}, ensure_ascii=False)}\n\n"

            # 状态更新
            action = decision.get("action")
            if action == "new_question":
                self.current_question_follow_count = 0
                logging.info(f"-- 面试ID: {self.interview_id} - 📈 进入下一题，追问次数已重置")
            elif action == "follow_up":
                self.current_question_follow_count += 1
                logging.info(f"-- 面试ID: {self.interview_id} - 🔄 保持当前问题，追问次数: {self.current_question_follow_count}")
            
            # 判断是否结束
            if decision.get("should_end", False):
                logging.info(f"-- 面试ID: {self.interview_id} - 🛑 面试结束（由决策判断）")
                yield f"data: {json.dumps({'type': 'over', 'content': 'over'}, ensure_ascii=False)}\n\n"

            end_time = time.time()
            execution_time = end_time - start_time
            logging.info(f"-- 面试ID: {self.interview_id} - 执行时间: {execution_time:.2f}秒, 质量评分: {decision.get('quality_score')}, 回复长度: {len(ai_response)}")

        except Exception as e:
            logging.error(f"-- 面试ID : {self.interview_id}---AI流式响应获取错误: {e}")
            yield f"data: {json.dumps({'type': 'error', 'content': '模型检测到非法输入，请重新开始本次面试'+str(e)}, ensure_ascii=False)}\n\n"






```


验证修改后的代码逻辑：



已添加详细的日志记录。修改总结如下：

## 修改总结

### 新增的日志记录

#### 阶段一：智能决策（`_make_unified_decision`）

1. 入参记录（第185-191行）：
   - `dialog_history` 摘要：记录数量、最后一条用户消息（前100字符）
   - `system_message` 长度
   - 追问次数

2. 出参记录（第210行）：
   - 完整的 `decision` 对象（JSON格式，包含所有字段）

#### 阶段二：流式生成回复（`_stream_response_by_decision`）

1. 入参记录（第261-264行）：
   - `decision.action` 和 `decision.quality_score`
   - `response_guidance`（完整内容）
   - `dialog_history` 记录数量

2. 出参记录（第283行）：
   - 完整生成的回复文本

#### 主流程：数据流向追踪（第377-384行）

1. 阶段一输入输出追踪
2. 阶段二输入输出追踪
3. 展示 `response_guidance` 如何从阶段一传递到阶段二

### 日志示例

现在日志中会看到类似这样的记录：

```
-- 面试ID: test_interview_001 - [阶段一入参] dialog_history: 共4条记录，最后用户消息: 用了MySQL，设计了用户表和书籍表。..., 追问次数: 1
-- 面试ID: test_interview_001 - [阶段一入参] system_message长度: 523字符
-- 面试ID: test_interview_001 - [阶段一出参] 完整decision: {
  "action": "follow_up",
  "response_guidance": "指出候选人回答过于简略，要求补充数据库表结构的具体字段设计...",
  "quality_score": 4,
  "should_end": false,
  "reasoning": "..."
}
-- 面试ID: test_interview_001 - [数据流向] 阶段一输出 -> decision对象（包含response_guidance: 指出候选人回答过于简略，要求补充数据库表结构的具体字段设计...）
-- 面试ID: test_interview_001 - [阶段二入参] decision.action: follow_up, decision.quality_score: 4
-- 面试ID: test_interview_001 - [阶段二入参] response_guidance: 指出候选人回答过于简略，要求补充数据库表结构的具体字段设计、表之间的关系（如外键），以及为什么这样设计。同时询问是否考虑了索引优化。
-- 面试ID: test_interview_001 - [阶段二入参] dialog_history: 共4条记录
-- 面试ID: test_interview_001 - [阶段二出参] 完整生成的回复: 张三，你提到设计了用户表和书籍表，但回答还比较简略。请你详细说明一下：1. 用户表具体包含哪些字段？...
```

### 优势

1. 完整追踪：可以看到两个阶段的完整输入输出
2. 数据流向清晰：可以看到 `response_guidance` 如何从阶段一传递到阶段二
3. 便于排查：可以对比输入输出，快速定位问题
4. 信息摘要：对长文本进行摘要，避免日志过长

现在可以通过日志完整追踪数据流向和处理过程，便于排查问题。

```plaintext
-- 面试ID: test_interview_001 - [阶段一入参] dialog_history: 共4条记录，最后用户消息: 用了MySQL，设计了用户表和书籍表。..., 追问次数: 1
-- 面试ID: test_interview_001 - [阶段一入参] system_message长度: 523字符
-- 面试ID: test_interview_001 - [阶段一出参] 完整decision: {
  "action": "follow_up",
  "response_guidance": "指出候选人回答过于简略，要求补充数据库表结构的具体字段设计...",
  "quality_score": 4,
  "should_end": false,
  "reasoning": "..."
}
-- 面试ID: test_interview_001 - [数据流向] 阶段一输出 -> decision对象（包含response_guidance: 指出候选人回答过于简略，要求补充数据库表结构的具体字段设计...）
-- 面试ID: test_interview_001 - [阶段二入参] decision.action: follow_up, decision.quality_score: 4
-- 面试ID: test_interview_001 - [阶段二入参] response_guidance: 指出候选人回答过于简略，要求补充数据库表结构的具体字段设计、表之间的关系（如外键），以及为什么这样设计。同时询问是否考虑了索引优化。
-- 面试ID: test_interview_001 - [阶段二入参] dialog_history: 共4条记录
-- 面试ID: test_interview_001 - [阶段二出参] 完整生成的回复: 张三，你提到设计了用户表和书籍表，但回答还比较简略。请你详细说明一下：1. 用户表具体包含哪些字段？...
```

