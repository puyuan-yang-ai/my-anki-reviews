好的，我来根据我们对齐的需求生成规格文档。

[1 tool called]

已经生成了规格文档。让我检查一下是否有格式问题：

[1 tool called]

规格文档已生成完毕！文件位置：`project_code/career/docs/specs/deep-resume-optimization/conversation-spec.md`

---

## 📄 文档结构总结

| 章节 | 内容 |
|------|------|
| **1. 核心思路** | 模块定位、解决的问题、与 Generate 接口的关系 |
| **2. 核心功能** | 输入参数、输出事件类型、处理逻辑流程图 |
| **3. 核心实现** | 文件结构、5 个模块的具体步骤 |
| **4. API 接口** | 请求/响应示例（JSON 格式） |
| **5. 技术细节** | 状态管理、流式响应、沉默检测、AI 角色等 |

---

## ✅ 需求对齐确认

文档中已包含我们之前讨论确认的所有内容：

- [x] 前端透传 `conversation_plan` 方案
- [x] 流式 SSE 响应
- [x] 5 大模块（计划生成、策略、执行、意图识别、结束判断）
- [x] 沉默检测（3 次无效强制结束）
- [x] 暂无轮次限制
- [x] AI 语气：温和鼓励型
- [x] 案例分析：给出场景让学生分析
- [x] 结束由 AI 判断，返回 `over` 信号

---

如果文档内容没问题，下一步我可以帮你：
1. 编写 prompt 文件（`generate_plan.txt`、`strategy.txt`、`executor.txt`、`judge_end.txt`）
2. 实现服务层代码（`deep_resume_converse.py`）
3. 更新路由层代码（`resume_router.py`）

请确认是否继续？
