

**User**

1 因为我想我是在做一个开发人员。我是不可能像专业的测试人员一样，进行详细的就从底往上的详细的测试，就比如说从单元测试出发，所有单元测试做完以后再做 集成测试，集成测试确定完全没问题了，再做契约测试或者API测试，这些都没问题，这之后再做端到端测试，请问专业的测试人员，他们是这样的流程吗？

2 我作为一个专业的开发人员，我肯定无法做这么详尽的测试。因为没有这么多的时间和精力。我想的是，做一个端到端测试，测试他是没问题即可。你怎么看呢？ 

---

**Cursor**

你的想法非常实际！让我从专业角度给你分析一下。

## 1️⃣ 专业测试人员的流程是怎样的？

### 理想的测试金字塔（理论）

```
         /\
        /  \  E2E (少量，10%)
       /----\
      /      \  Integration/API (中量，20%)
     /--------\
    /          \  Unit Tests (大量，70%)
   /------------\
```

**是的，专业测试团队通常会按照这个顺序：**

1. **开发阶段** → 开发人员写单元测试
2. **集成阶段** → 测试人员写集成测试
3. **系统测试** → 测试契约/API测试
4. **发布前** → 端到端测试（最少，最慢，最贵）

**但这是有前提的：**
- ✅ 有专门的 QA 团队
- ✅ 有充足的时间和预算
- ✅ 项目规模大、复杂度高
- ✅ 对质量要求极高（金融、医疗等）

## 2️⃣ 作为开发人员，你的想法完全合理！

### 现实中的开发实践

你的想法**非常务实**，很多成功的项目都是这样做的：

```
实际开发中的测试策略（推荐）
┌────────────────────────────────────┐
│ 优先级 1: 端到端测试 (E2E)        │  ← 先保证核心功能能跑通
│ - 测试最关键的用户流程              │
│ - 覆盖主要业务场景                 │
│ - 能发现最明显的问题                │
└────────────────────────────────────┘
          ↓ 有时间再补充
┌────────────────────────────────────┐
│ 优先级 2: 集成测试                 │  ← 测试模块间交互
│ - 测试 API 接口                    │
│ - 测试数据库交互                   │
└────────────────────────────────────┘
          ↓ 最后考虑（可选）
┌────────────────────────────────────┐
│ 优先级 3: 单元测试                 │  ← 只给核心逻辑写
│ - 复杂算法                         │
│ - 关键业务逻辑                     │
└────────────────────────────────────┘
```

### 为什么这样是合理的？

#### ✅ 投入产出比最高

```
E2E 测试:
- 时间投入: 1-2 小时写几个核心场景
- 覆盖范围: 80% 的关键功能
- ROI: ⭐⭐⭐⭐⭐

单元测试:
- 时间投入: 可能需要几天写几百个测试
- 覆盖范围: 代码行覆盖率高，但业务覆盖率不一定
- ROI: ⭐⭐⭐ (对某些项目)
```

#### ✅ 符合"80/20 法则"

20% 的测试工作，可以发现 80% 的问题：

```python
# 一个好的 E2E 测试可以发现：
- ✓ API 是否正常
- ✓ 数据库连接是否正常
- ✓ 业务逻辑是否正确
- ✓ 前后端集成是否有问题
- ✓ 配置是否正确
```

## 3️⃣ 推荐的务实测试策略

### 方案 A: 最小可行测试（MVP）

**适合：初创项目、快速迭代、小团队**

```python
tests/
├── e2e/
│   └── test_critical_flows.py     # 只测最核心的 3-5 个流程
│       ├── test_user_can_submit_interview()
│       ├── test_generate_report()
│       └── test_view_report()
```

**优点：**
- ⚡ 快速验证系统能用
- 🎯 覆盖最重要的功能
- 💰 成本最低

### 方案 B: 实用主义测试（推荐给你）

**适合：有一定规模的项目、需要持续维护**

```python
tests/
├── e2e/                            # 🔥 核心流程（必须有）
│   ├── test_main_user_flows.py    # 用户主要使用场景
│   └── test_critical_paths.py     # 关键业务路径
│
├── integration/                    # ⚡ 重点模块（选择性写）
│   └── test_ai_api_integration.py # 外部 API 集成
│
└── unit/                           # 💎 核心逻辑（只写关键的）
    └── test_complex_algorithms.py  # 复杂算法或计算逻辑
```

**写测试的判断标准：**

| 测试类型 | 什么时候写 | 例子 |
|---------|-----------|------|
| E2E | **必须写** | 完整的面试流程 |
| Integration | **重要外部依赖** | OpenAI API 调用 |
| Unit | **复杂逻辑/算法** | 评分计算算法 |

### 方案 C: 渐进式测试

**边开发边补充：**

```
第 1 周: 写 2-3 个 E2E 测试覆盖核心功能
         ↓
第 2 周: 发现某个模块老出问题 → 给它加集成测试
         ↓
第 3 周: 某个算法太复杂 → 给它加单元测试
         ↓
持续迭代...
```

## 4️⃣ 具体建议：你的项目应该这样做

看你的项目（面试评估系统），我建议：

### 核心测试（必须有）

```python
# tests/e2e/test_evaluation_report_e2e.py

@pytest.mark.e2e
class TestEvaluationReportE2E:
    
    async def test_complete_interview_evaluation_flow(self):
        """完整流程：提交面试 → 生成报告 → 查看报告"""
        # 这一个测试就覆盖了 80% 的功能
        
    async def test_report_with_multimodal_data(self):
        """测试视频/图片处理"""
        
    async def test_different_job_levels(self):
        """测试不同级别岗位（初级/高级）"""
```

**3 个测试，覆盖主要场景 → 完成！** ✅

### 可选补充（有时间再做）

```python
# tests/integration/test_openai_integration.py
def test_openai_api_returns_valid_response():
    """测试 OpenAI API 返回格式是否正确"""
    # 因为这是外部依赖，容易出问题，值得测

# tests/unit/test_score_calculation.py
def test_match_score_calculation():
    """测试评分算法"""
    # 如果评分逻辑很复杂，值得单独测试
```

## 5️⃣ 业界实践参考

### Google/Facebook 等大厂

- 有专门的测试团队
- 有自动化测试基础设施
- 单元测试覆盖率要求 80%+

### 创业公司/小团队

- **主要依赖 E2E 测试** ✅
- 关键功能加集成测试
- 单元测试只写核心算法
- **测试覆盖率 30-50% 就很好了**

### 开源项目

- **优先保证核心功能的 E2E 测试**
- 社区贡献者会逐步补充测试
- 不强求高覆盖率

## 🎯 最终建议

**你的想法是对的！作为开发人员：**

1. ✅ **先写 3-5 个 E2E 测试**，覆盖最核心的功能
2. ✅ **运行这些测试确保没问题** → 就可以发布了
3. ✅ 后续发现问题 → 补充相应的测试
4. ✅ 不要追求完美的测试覆盖率

**记住这句话：**
> "有测试比没测试好，但不测试不代表就不能发布。最重要的是**测试关键路径**，而不是测试所有代码。"

**实用的测试策略：**
- 💚 2-3 个 E2E 测试 = 及格（60分）
- 💙 5-10 个 E2E + 关键集成测试 = 良好（80分）
- 💛 完整的测试体系 = 优秀（95分）← 这是专业QA团队的工作

**你现在只需要做到 60-80 分就够了！** 🎉

```plaintext
         /\
        /  \  E2E (少量，10%)
       /----\
      /      \  Integration/API (中量，20%)
     /--------\
    /          \  Unit Tests (大量，70%)
   /------------\
```

```plaintext
实际开发中的测试策略（推荐）
┌────────────────────────────────────┐
│ 优先级 1: 端到端测试 (E2E)        │  ← 先保证核心功能能跑通
│ - 测试最关键的用户流程              │
│ - 覆盖主要业务场景                 │
│ - 能发现最明显的问题                │
└────────────────────────────────────┘
          ↓ 有时间再补充
┌────────────────────────────────────┐
│ 优先级 2: 集成测试                 │  ← 测试模块间交互
│ - 测试 API 接口                    │
│ - 测试数据库交互                   │
└────────────────────────────────────┘
          ↓ 最后考虑（可选）
┌────────────────────────────────────┐
│ 优先级 3: 单元测试                 │  ← 只给核心逻辑写
│ - 复杂算法                         │
│ - 关键业务逻辑                     │
└────────────────────────────────────┘
```

```plaintext
E2E 测试:
- 时间投入: 1-2 小时写几个核心场景
- 覆盖范围: 80% 的关键功能
- ROI: ⭐⭐⭐⭐⭐

单元测试:
- 时间投入: 可能需要几天写几百个测试
- 覆盖范围: 代码行覆盖率高，但业务覆盖率不一定
- ROI: ⭐⭐⭐ (对某些项目)
```

```python
# 一个好的 E2E 测试可以发现：
- ✓ API 是否正常
- ✓ 数据库连接是否正常
- ✓ 业务逻辑是否正确
- ✓ 前后端集成是否有问题
- ✓ 配置是否正确
```

```python
tests/
├── e2e/
│   └── test_critical_flows.py     # 只测最核心的 3-5 个流程
│       ├── test_user_can_submit_interview()
│       ├── test_generate_report()
│       └── test_view_report()
```

```python
tests/
├── e2e/                            # 🔥 核心流程（必须有）
│   ├── test_main_user_flows.py    # 用户主要使用场景
│   └── test_critical_paths.py     # 关键业务路径
│
├── integration/                    # ⚡ 重点模块（选择性写）
│   └── test_ai_api_integration.py # 外部 API 集成
│
└── unit/                           # 💎 核心逻辑（只写关键的）
    └── test_complex_algorithms.py  # 复杂算法或计算逻辑
```

```plaintext
第 1 周: 写 2-3 个 E2E 测试覆盖核心功能
         ↓
第 2 周: 发现某个模块老出问题 → 给它加集成测试
         ↓
第 3 周: 某个算法太复杂 → 给它加单元测试
         ↓
持续迭代...
```

```python
# tests/e2e/test_evaluation_report_e2e.py

@pytest.mark.e2e
class TestEvaluationReportE2E:
    
    async def test_complete_interview_evaluation_flow(self):
        """完整流程：提交面试 → 生成报告 → 查看报告"""
        # 这一个测试就覆盖了 80% 的功能
        
    async def test_report_with_multimodal_data(self):
        """测试视频/图片处理"""
        
    async def test_different_job_levels(self):
        """测试不同级别岗位（初级/高级）"""
```

```python
# tests/integration/test_openai_integration.py
def test_openai_api_returns_valid_response():
    """测试 OpenAI API 返回格式是否正确"""
    # 因为这是外部依赖，容易出问题，值得测

# tests/unit/test_score_calculation.py
def test_match_score_calculation():
    """测试评分算法"""
    # 如果评分逻辑很复杂，值得单独测试
```

---

**User**

@bash (224-303) 这是端到端测试的输出，我感觉测的根本不到位呀。这是端到端测试吗？能不能帮我梳理一下分析一下呢？我理解的端到端不应该是输入和输出这样的格式会被检查吗？这也没有检查呀我也不知道输入和输出是什么呢？你怎么看呢？

@test_evaluation_report_e2e.py 这个文件你帮我评估一下写的怎么样呀？是否满足了端到端的基本要求？你觉得这个端到端测试的核心是什么呢？能不能教一下我呢？我都是头一次接触，不是特别了解。

---

**Cursor**

你的观察非常敏锐！让我详细分析一下这个端到端测试的问题。

看了你的测试和输出，我完全理解你的困惑！让我深入分析一下。

## 🔍 问题诊断

### 1️⃣ 你看到的测试输出（bash 224-303）

这个是 **`test_evaluation_report_simple.py`** 的输出，**不是真正的端到端测试**！

```
✅ 测试模块导入       ← 这是单元测试级别
✅ 测试评估器创建     ← 这是单元测试级别
✅ 测试数据格式       ← 这是单元测试级别
✅ 测试API配置        ← 这是单元测试级别
```

**问题：这些测试只是检查"组件能不能创建"，根本没有测试"功能能不能用"！**

### 2️⃣ 你的理解是对的！

真正的端到端测试应该是：

```
输入 → [整个系统] → 输出
  ↓                    ↓
验证输入格式        验证输出结果
```

## 📊 端到端测试的核心要素（教学）

### ✅ 真正的端到端测试应该包含：

```
1. 准备真实的输入数据
   ↓
2. 调用真实的系统（不Mock任何东西）
   ↓
3. 获得真实的输出
   ↓
4. 验证输出的格式和内容是否正确
   ↓
5. 验证输出是否符合业务需求
```

### 🎯 端到端测试的"三要素"

| 要素 | 说明 | 你的测试有吗？ |
|------|------|---------------|
| **真实输入** | 完整的、真实场景的输入数据 | ❌ 有数据但没验证输入 |
| **真实流程** | 从头到尾跑完整个系统 | ⚠️ 有，但测试不调用AI |
| **真实输出验证** | 详细验证输出的格式和内容 | ❌ 验证不够详细 |

## 📝 评估你的测试文件

### `test_evaluation_report_e2e.py` 的优缺点

#### ✅ 做得好的地方：

1. **有完整的测试数据**（第38-109行）
2. **调用了真实的 `generate_report` 方法**（第137-144行）
3. **有基本的结构验证**（第151-181行）
4. **有性能测试**（第306-351行）

#### ❌ 缺少的关键部分：

1. **没有详细验证输出内容**
2. **没有验证业务逻辑是否正确**
3. **没有验证边界情况**
4. **没有验证错误处理**

### 具体问题分析

```python
# 现在的测试（第151-163行）
assert "overall_assessment" in report, "缺少 overall_assessment 字段"
assert "match_score" in overall, "缺少 match_score"
assert 0 <= overall["match_score"] <= 100, "match_score 应该在0-100之间"

print(f"✅ 综合评分: {overall['match_score']}")
print(f"✅ 结论: {overall['conclusion']}")
```

**问题：只检查了"有没有"，没检查"对不对"！**

比如：
- ❌ 如果 AI 返回的评分是 `999`？→ 会通过（因为只检查了 0-100）
- ❌ 如果结论是乱码或空字符串？→ 会通过（因为没检查内容）
- ❌ 如果分析完全不符合面试内容？→ 会通过（因为没验证业务逻辑）

## 🎓 改进建议：如何写好端到端测试

### 方案 1：基础版（60分）- 验证输入输出

```python
async def test_complete_evaluation_flow_basic(self):
    """完整的评估流程测试 - 基础版"""
    
    # ========== 第1步：准备并验证输入 ==========
    input_data = {
        "interview_id": "e2e_test_001",
        "transcript": "面试官：请介绍你的Python经验。\n候选人：我有3年Python经验...",
        "jd": "招聘高级Python工程师，要求3年以上经验...",
        "interview_language": "zhongwen"
    }
    
    # 验证输入格式
    assert len(input_data["transcript"]) > 100, "面试转录内容应该足够长"
    assert "Python" in input_data["jd"], "JD应该包含关键技能要求"
    
    print(f"\n📥 输入数据:")
    print(f"   面试ID: {input_data['interview_id']}")
    print(f"   转录长度: {len(input_data['transcript'])} 字符")
    print(f"   JD长度: {len(input_data['jd'])} 字符")
    
    # ========== 第2步：执行真实流程 ==========
    assessor = AIInterviewAssessor()
    
    transcript_json = json.dumps({"content": input_data["transcript"]}, ensure_ascii=False)
    
    report = assessor.generate_report(
        interview_id=input_data["interview_id"],
        transcript_json_str=transcript_json,
        jd=input_data["jd"],
        interview_language=input_data["interview_language"]
    )
    
    # ========== 第3步：详细验证输出 ==========
    print(f"\n📤 输出验证:")
    
    # 3.1 验证结构完整性
    required_fields = ["overall_assessment", "detailed_analysis", "user_report"]
    for field in required_fields:
        assert field in report, f"缺少必需字段: {field}"
    print(f"   ✅ 报告结构完整")
    
    # 3.2 验证综合评估
    overall = report["overall_assessment"]
    assert "match_score" in overall
    assert "conclusion" in overall
    
    # 验证评分合理性
    score = overall["match_score"]
    assert isinstance(score, (int, float)), "评分应该是数字"
    assert 0 <= score <= 100, f"评分超出范围: {score}"
    print(f"   ✅ 综合评分: {score}")
    
    # 验证结论不是空的
    conclusion = overall["conclusion"]
    assert len(conclusion) > 10, f"结论内容太短: {conclusion}"
    assert any(word in conclusion for word in ["录用", "胜任", "不推荐", "考虑"]), \
        f"结论应该包含明确的建议: {conclusion}"
    print(f"   ✅ 结论: {conclusion[:50]}...")
    
    # 3.3 验证详细分析
    detailed = report["detailed_analysis"]
    assert isinstance(detailed, list), "详细分析应该是列表"
    assert len(detailed) >= 3, f"能力分析项太少: {len(detailed)}"
    
    # 验证每个分析项的内容
    for i, analysis in enumerate(detailed):
        assert "competency" in analysis, f"第{i}项缺少能力名称"
        assert "score" in analysis, f"第{i}项缺少评分"
        assert "rationale" in analysis, f"第{i}项缺少理由"
        
        # 验证评分合理
        assert 0 <= analysis["score"] <= 100, f"第{i}项评分超出范围"
        
        # 验证理由不是空的
        assert len(analysis["rationale"]) > 20, f"第{i}项理由太简短"
    
    print(f"   ✅ 详细分析: {len(detailed)} 个能力维度")
    
    # 3.4 验证用户报告
    user_report = report["user_report"]
    
    # 验证必需字段
    assert "highlights" in user_report, "缺少亮点"
    assert "insufficient" in user_report, "缺少不足"
    assert "short_term_actions" in user_report, "缺少短期行动建议"
    assert "career_advice" in user_report, "缺少职业建议"
    
    # 验证内容不是空的
    assert len(user_report["highlights"]) > 20, "亮点内容太少"
    assert len(user_report["insufficient"]) > 20, "不足分析太少"
    assert isinstance(user_report["short_term_actions"], list), "行动建议应该是列表"
    assert len(user_report["short_term_actions"]) >= 2, "至少应该有2条行动建议"
    
    print(f"   ✅ 用户报告完整")
    
    # ========== 第4步：验证业务逻辑 ==========
    print(f"\n🔍 业务逻辑验证:")
    
    # 验证：如果JD要求Python，分析中应该包含Python相关能力
    competencies = [item["competency"] for item in detailed]
    has_python_skill = any("Python" in comp or "开发" in comp for comp in competencies)
    assert has_python_skill, f"应该有Python相关的能力评估，但只有: {competencies}"
    print(f"   ✅ 能力评估包含关键技能")
    
    # 验证：评分和结论应该一致
    if score >= 80:
        assert any(word in conclusion for word in ["录用", "推荐", "胜任"]), \
            f"高分({score})应该推荐录用，但结论是: {conclusion}"
    elif score < 60:
        assert any(word in conclusion for word in ["不推荐", "不符合", "需要提升"]), \
            f"低分({score})不应该推荐录用，但结论是: {conclusion}"
    print(f"   ✅ 评分与结论一致")
    
    print(f"\n🎉 端到端测试通过！")
```

### 方案 2：进阶版（80分）- 测试不同场景

```python
@pytest.mark.parametrize("scenario", [
    {
        "name": "高级工程师-匹配度高",
        "transcript": "我有5年Python经验，做过大规模分布式系统...",
        "jd": "招聘高级Python工程师，要求3-5年经验...",
        "expected_score_range": (70, 100),
        "expected_keywords": ["录用", "推荐", "胜任"]
    },
    {
        "name": "初级工程师-匹配度低",
        "transcript": "我刚毕业，学过一点Python...",
        "jd": "招聘高级Python工程师，要求5年以上经验...",
        "expected_score_range": (0, 50),
        "expected_keywords": ["不推荐", "经验不足"]
    },
    {
        "name": "技能不匹配",
        "transcript": "我主要做Java开发...",
        "jd": "招聘Python工程师...",
        "expected_score_range": (0, 40),
        "expected_keywords": ["不符合", "技能"]
    }
])
async def test_different_scenarios(self, scenario):
    """测试不同场景下的评估结果"""
    
    print(f"\n🎯 测试场景: {scenario['name']}")
    
    # 执行评估
    report = ... # 调用 generate_report
    
    # 验证评分在预期范围内
    score = report["overall_assessment"]["match_score"]
    min_score, max_score = scenario["expected_score_range"]
    assert min_score <= score <= max_score, \
        f"场景'{scenario['name']}'的评分({score})不在预期范围({min_score}-{max_score})"
    
    # 验证结论包含预期关键词
    conclusion = report["overall_assessment"]["conclusion"]
    has_expected_keyword = any(kw in conclusion for kw in scenario["expected_keywords"])
    assert has_expected_keyword, \
        f"场景'{scenario['name']}'的结论应包含{scenario['expected_keywords']}之一，但实际是: {conclusion}"
    
    print(f"✅ 场景验证通过")
```

## 🎯 总结：端到端测试的核心

### 核心1：真实性
- ✅ 使用真实的输入数据
- ✅ 调用真实的 API（不 Mock）
- ✅ 获得真实的输出

### 核心2：完整性
- ✅ 从用户输入到最终输出的完整流程
- ✅ 包含所有中间步骤（API调用、数据处理等）

### 核心3：业务验证
- ✅ 不只验证"有没有"，还要验证"对不对"
- ✅ 验证业务逻辑是否正确
- ✅ 验证不同场景下的行为是否符合预期

### 核心4：可读性
- ✅ 清晰的输入输出日志
- ✅ 详细的断言错误信息
- ✅ 易于定位问题

## 💡 你的测试评分

| 维度 | 得分 | 评价 |
|------|-----|------|
| 真实性 | 7/10 | 有真实调用，但有些测试不调用AI |
| 完整性 | 6/10 | 流程完整，但验证不够 |
| 业务验证 | 3/10 | 只验证结构，不验证内容 |
| 可读性 | 8/10 | 输出清晰，容易理解 |
| **总分** | **6/10** | **及格，但有很大改进空间** |

**建议：按照上面的"基础版"改进，就能达到 8/10 分了！** 🚀

```plaintext
✅ 测试模块导入       ← 这是单元测试级别
✅ 测试评估器创建     ← 这是单元测试级别
✅ 测试数据格式       ← 这是单元测试级别
✅ 测试API配置        ← 这是单元测试级别
```

```plaintext
输入 → [整个系统] → 输出
  ↓                    ↓
验证输入格式        验证输出结果
```

```plaintext
1. 准备真实的输入数据
   ↓
2. 调用真实的系统（不Mock任何东西）
   ↓
3. 获得真实的输出
   ↓
4. 验证输出的格式和内容是否正确
   ↓
5. 验证输出是否符合业务需求
```

```python
# 现在的测试（第151-163行）
assert "overall_assessment" in report, "缺少 overall_assessment 字段"
assert "match_score" in overall, "缺少 match_score"
assert 0 <= overall["match_score"] <= 100, "match_score 应该在0-100之间"

print(f"✅ 综合评分: {overall['match_score']}")
print(f"✅ 结论: {overall['conclusion']}")
```

```python
async def test_complete_evaluation_flow_basic(self):
    """完整的评估流程测试 - 基础版"""
    
    # ========== 第1步：准备并验证输入 ==========
    input_data = {
        "interview_id": "e2e_test_001",
        "transcript": "面试官：请介绍你的Python经验。\n候选人：我有3年Python经验...",
        "jd": "招聘高级Python工程师，要求3年以上经验...",
        "interview_language": "zhongwen"
    }
    
    # 验证输入格式
    assert len(input_data["transcript"]) > 100, "面试转录内容应该足够长"
    assert "Python" in input_data["jd"], "JD应该包含关键技能要求"
    
    print(f"\n📥 输入数据:")
    print(f"   面试ID: {input_data['interview_id']}")
    print(f"   转录长度: {len(input_data['transcript'])} 字符")
    print(f"   JD长度: {len(input_data['jd'])} 字符")
    
    # ========== 第2步：执行真实流程 ==========
    assessor = AIInterviewAssessor()
    
    transcript_json = json.dumps({"content": input_data["transcript"]}, ensure_ascii=False)
    
    report = assessor.generate_report(
        interview_id=input_data["interview_id"],
        transcript_json_str=transcript_json,
        jd=input_data["jd"],
        interview_language=input_data["interview_language"]
    )
    
    # ========== 第3步：详细验证输出 ==========
    print(f"\n📤 输出验证:")
    
    # 3.1 验证结构完整性
    required_fields = ["overall_assessment", "detailed_analysis", "user_report"]
    for field in required_fields:
        assert field in report, f"缺少必需字段: {field}"
    print(f"   ✅ 报告结构完整")
    
    # 3.2 验证综合评估
    overall = report["overall_assessment"]
    assert "match_score" in overall
    assert "conclusion" in overall
    
    # 验证评分合理性
    score = overall["match_score"]
    assert isinstance(score, (int, float)), "评分应该是数字"
    assert 0 <= score <= 100, f"评分超出范围: {score}"
    print(f"   ✅ 综合评分: {score}")
    
    # 验证结论不是空的
    conclusion = overall["conclusion"]
    assert len(conclusion) > 10, f"结论内容太短: {conclusion}"
    assert any(word in conclusion for word in ["录用", "胜任", "不推荐", "考虑"]), \
        f"结论应该包含明确的建议: {conclusion}"
    print(f"   ✅ 结论: {conclusion[:50]}...")
    
    # 3.3 验证详细分析
    detailed = report["detailed_analysis"]
    assert isinstance(detailed, list), "详细分析应该是列表"
    assert len(detailed) >= 3, f"能力分析项太少: {len(detailed)}"
    
    # 验证每个分析项的内容
    for i, analysis in enumerate(detailed):
        assert "competency" in analysis, f"第{i}项缺少能力名称"
        assert "score" in analysis, f"第{i}项缺少评分"
        assert "rationale" in analysis, f"第{i}项缺少理由"
        
        # 验证评分合理
        assert 0 <= analysis["score"] <= 100, f"第{i}项评分超出范围"
        
        # 验证理由不是空的
        assert len(analysis["rationale"]) > 20, f"第{i}项理由太简短"
    
    print(f"   ✅ 详细分析: {len(detailed)} 个能力维度")
    
    # 3.4 验证用户报告
    user_report = report["user_report"]
    
    # 验证必需字段
    assert "highlights" in user_report, "缺少亮点"
    assert "insufficient" in user_report, "缺少不足"
    assert "short_term_actions" in user_report, "缺少短期行动建议"
    assert "career_advice" in user_report, "缺少职业建议"
    
    # 验证内容不是空的
    assert len(user_report["highlights"]) > 20, "亮点内容太少"
    assert len(user_report["insufficient"]) > 20, "不足分析太少"
    assert isinstance(user_report["short_term_actions"], list), "行动建议应该是列表"
    assert len(user_report["short_term_actions"]) >= 2, "至少应该有2条行动建议"
    
    print(f"   ✅ 用户报告完整")
    
    # ========== 第4步：验证业务逻辑 ==========
    print(f"\n🔍 业务逻辑验证:")
    
    # 验证：如果JD要求Python，分析中应该包含Python相关能力
    competencies = [item["competency"] for item in detailed]
    has_python_skill = any("Python" in comp or "开发" in comp for comp in competencies)
    assert has_python_skill, f"应该有Python相关的能力评估，但只有: {competencies}"
    print(f"   ✅ 能力评估包含关键技能")
    
    # 验证：评分和结论应该一致
    if score >= 80:
        assert any(word in conclusion for word in ["录用", "推荐", "胜任"]), \
            f"高分({score})应该推荐录用，但结论是: {conclusion}"
    elif score < 60:
        assert any(word in conclusion for word in ["不推荐", "不符合", "需要提升"]), \
            f"低分({score})不应该推荐录用，但结论是: {conclusion}"
    print(f"   ✅ 评分与结论一致")
    
    print(f"\n🎉 端到端测试通过！")
```

```python
@pytest.mark.parametrize("scenario", [
    {
        "name": "高级工程师-匹配度高",
        "transcript": "我有5年Python经验，做过大规模分布式系统...",
        "jd": "招聘高级Python工程师，要求3-5年经验...",
        "expected_score_range": (70, 100),
        "expected_keywords": ["录用", "推荐", "胜任"]
    },
    {
        "name": "初级工程师-匹配度低",
        "transcript": "我刚毕业，学过一点Python...",
        "jd": "招聘高级Python工程师，要求5年以上经验...",
        "expected_score_range": (0, 50),
        "expected_keywords": ["不推荐", "经验不足"]
    },
    {
        "name": "技能不匹配",
        "transcript": "我主要做Java开发...",
        "jd": "招聘Python工程师...",
        "expected_score_range": (0, 40),
        "expected_keywords": ["不符合", "技能"]
    }
])
async def test_different_scenarios(self, scenario):
    """测试不同场景下的评估结果"""
    
    print(f"\n🎯 测试场景: {scenario['name']}")
    
    # 执行评估
    report = ... # 调用 generate_report
    
    # 验证评分在预期范围内
    score = report["overall_assessment"]["match_score"]
    min_score, max_score = scenario["expected_score_range"]
    assert min_score <= score <= max_score, \
        f"场景'{scenario['name']}'的评分({score})不在预期范围({min_score}-{max_score})"
    
    # 验证结论包含预期关键词
    conclusion = report["overall_assessment"]["conclusion"]
    has_expected_keyword = any(kw in conclusion for kw in scenario["expected_keywords"])
    assert has_expected_keyword, \
        f"场景'{scenario['name']}'的结论应包含{scenario['expected_keywords']}之一，但实际是: {conclusion}"
    
    print(f"✅ 场景验证通过")
```

---

**User**

好的，那你现在能不能帮我思考一下我的提示词模板应该怎么写来引导模型帮我去生成一个相应的端到端的测试呢？就是我给llm提供的是代码或者是相应的spec文档。 其实我的理解是这个提示词里面有包含要引导模型去思考这个e2e测试里面最重要的部分是什么，然后再根据思考完以后，根据它来生成相应的测试。然后让它涵盖最核心的逻辑，并且专注于输入和输出这部分。请问是这样吗？以及你有什么其他好的看法和观点吗？

---

**Cursor**

你的思路非常正确！让我帮你设计一个高质量的提示词模板，用于引导 LLM 生成端到端测试。

## 🎯 提示词设计思路

你的理解完全对：
1. ✅ 引导模型思考 E2E 测试的核心
2. ✅ 专注于输入和输出验证
3. ✅ 覆盖最关键的业务逻辑

让我给你设计一个**分步骤、有思考过程**的提示词模板：

## 📝 提示词模板（推荐版本）

```markdown
# 任务：生成端到端（E2E）测试代码

## 背景
你是一位专业的测试工程师，需要为给定的代码/规格文档生成高质量的端到端测试。

## 输入
{INPUT_TYPE} # 可以是 "源代码" 或 "规格文档"

```
{CODE_OR_SPEC}
```

## 测试框架
- 使用 pytest
- 使用 @pytest.mark.e2e 标记
- 如果是异步函数，使用 @pytest.mark.asyncio

## 生成要求

### 第一步：分析核心功能
请先分析这段代码/文档，回答以下问题：

1. **这个功能的主要目的是什么？**（用一句话概括）
2. **关键的输入参数有哪些？**（列出参数名和类型）
3. **期望的输出是什么？**（格式、类型、必需字段）
4. **核心业务逻辑是什么？**（例如：评分、分类、转换等）
5. **有哪些重要的业务规则需要验证？**（例如：评分范围、数据完整性等）
6. **可能的异常场景有哪些？**（例如：输入为空、格式错误等）

### 第二步：确定测试场景
基于上述分析，确定需要测试的核心场景（优先级排序）：

1. **主场景（Happy Path）**：正常输入 → 正常输出
2. **边界场景**：极端但有效的输入
3. **异常场景**：无效输入的处理

### 第三步：设计测试用例
对于每个场景，明确：
- **输入数据**：具体的、真实的测试数据
- **预期输出**：详细的验证点
- **业务验证**：除了格式，还要验证逻辑是否正确

### 第四步：生成测试代码
遵循以下原则生成测试代码：

#### 必须包含的部分：
1. **详细的输入准备**
   - 使用真实、完整的数据
   - 在注释中说明为什么选择这些数据
   - 验证输入数据的格式

2. **清晰的执行过程**
   - 调用真实的函数/API（不 Mock）
   - 记录执行时间（如果需要）
   - 打印关键信息便于调试

3. **完整的输出验证**
   - 结构验证：检查必需字段是否存在
   - 类型验证：检查数据类型是否正确
   - 范围验证：检查数值是否在合理范围内
   - 内容验证：检查内容是否非空、是否合理
   - 业务逻辑验证：检查业务规则是否满足

4. **可读的断言信息**
   - 每个 assert 都要有清晰的错误提示
   - 使用 f-string 显示实际值

#### 代码结构模板：
```python
@pytest.mark.e2e
async def test_[功能名称]_[场景描述](self):
    """测试 [功能描述] - [场景说明]"""
    
    # ========== 第1步：准备输入数据 ==========
    print(f"\n{'='*60}")
    print(f"🎯 测试场景：[场景名称]")
    print(f"{'='*60}")
    
    input_data = {
        # 真实的、完整的输入数据
    }
    
    # 验证输入数据格式
    assert [输入验证], "输入数据验证失败"
    
    print(f"\n📥 输入数据:")
    print(f"   [关键输入信息]")
    
    # ========== 第2步：执行功能 ==========
    print(f"\n🚀 执行 [功能名称]...")
    
    result = await [调用真实函数]
    
    # ========== 第3步：验证输出结构 ==========
    print(f"\n📊 验证输出结构...")
    
    # 必需字段检查
    required_fields = [...]
    for field in required_fields:
        assert field in result, f"缺少必需字段: {field}"
    print(f"   ✅ 输出结构完整")
    
    # ========== 第4步：验证输出内容 ==========
    print(f"\n📤 验证输出内容...")
    
    # 类型检查
    assert isinstance(result['field'], expected_type), \
        f"字段类型错误，期望 {expected_type}，实际 {type(result['field'])}"
    
    # 范围检查
    assert min_value <= result['value'] <= max_value, \
        f"值超出范围: {result['value']} (期望 {min_value}-{max_value})"
    
    # 内容检查
    assert len(result['text']) > min_length, \
        f"内容太短: {len(result['text'])} 字符"
    
    print(f"   ✅ [字段名]: {result['field']}")
    
    # ========== 第5步：验证业务逻辑 ==========
    print(f"\n🔍 验证业务逻辑...")
    
    # 业务规则验证
    assert [业务规则], "业务逻辑验证失败: [具体说明]"
    
    print(f"   ✅ 业务逻辑正确")
    print(f"\n🎉 测试通过！")
```

## 输出格式
请按以下格式输出：

### 分析结果
```
[第一步的分析结果]
```

### 测试场景
```
[第二步确定的测试场景]
```

### 测试代码
```python
[第三步和第四步生成的完整测试代码]
```

### 使用说明
```
[如何运行这些测试，以及注意事项]
```

## 注意事项
1. 测试代码必须是可直接运行的，包含所有必要的 import
2. 使用真实数据，不要用过于简单的示例（如 "test123"）
3. 每个验证点都要有清晰的断言信息
4. 优先测试核心功能的主场景，不要被边缘情况分散注意力
5. E2E 测试应该测试"整个流程"，不要只测试某个小函数
6. 测试应该是独立的，不依赖其他测试的执行顺序
```

## 🎨 优化版（更简洁）

如果你觉得上面的太长，这是一个**精简但高效**的版本：

```markdown
# 任务：为以下代码/规格生成端到端测试

## 输入
{INPUT_TYPE}: {CODE_OR_SPEC}

## 要求

### 1. 先思考（在代码注释中体现）
- 核心功能是什么？
- 关键输入和输出是什么？
- 需要验证哪些业务规则？

### 2. 生成测试（遵循这个结构）

```python
@pytest.mark.e2e
async def test_[场景名](self):
    """端到端测试：[场景描述]"""
    
    # === 准备输入（要真实、完整） ===
    input_data = {...}
    print(f"📥 输入: {input_data}")
    
    # === 执行真实流程（不Mock） ===
    result = await real_function(input_data)
    
    # === 验证输出 ===
    # 1. 结构：必需字段是否存在
    assert "key_field" in result, "缺少关键字段"
    
    # 2. 类型：数据类型是否正确
    assert isinstance(result['field'], ExpectedType)
    
    # 3. 范围：数值是否合理
    assert min <= result['value'] <= max, f"值超出范围: {result['value']}"
    
    # 4. 内容：内容是否有效（不为空、不是占位符）
    assert len(result['text']) > 20, "内容太简短"
    
    # 5. 业务：业务逻辑是否正确
    assert [业务规则], "业务逻辑错误"
    
    print(f"📤 输出验证通过: {result}")
```

### 3. 测试场景优先级
1. **必须测**：主要业务流程（Happy Path）
2. **建议测**：不同类型的输入（高/中/低匹配度等）
3. **可选测**：异常处理（输入为空等）

### 4. 关键原则
- ✅ 使用真实数据（不要 "test123"）
- ✅ 验证业务逻辑（不只是格式）
- ✅ 清晰的错误信息（每个 assert 都有说明）
- ✅ 从用户角度测试（输入→输出的完整流程）
- ❌ 不要 Mock 核心功能
- ❌ 不要只测试数据结构
```

## 🔥 针对你的项目的具体提示词

```markdown
# 任务：为面试评估系统生成端到端测试

## 输入代码
```python
class AIInterviewAssessor:
    def generate_report(
        self,
        interview_id: int,
        transcript_json_str: str,
        jd: str,
        url_list: list = [],
        is_mp4: bool = True,
        interview_language: str = "zhongwen"
    ) -> dict:
        """生成面试评估报告"""
        # ... 实现代码
```

## 功能说明
这个函数根据面试转录和岗位描述，生成包含评分、分析和建议的评估报告。

## 生成要求

### 思考过程（必须先回答）
1. **核心功能**：这个函数做什么？
2. **输入**：需要哪些参数？什么格式？
3. **输出**：返回什么？包含哪些字段？
4. **业务规则**：
   - 评分范围是多少？
   - 高分/低分时结论应该是什么？
   - 必须包含哪些分析维度？

### 测试场景设计
1. **场景1**：高级工程师+匹配的JD → 期望高分+推荐录用
2. **场景2**：初级工程师+高级JD → 期望低分+不推荐
3. **场景3**：技能不匹配 → 期望低分+明确指出问题

### 测试代码要求
- 每个场景生成一个独立的测试函数
- 输入数据要真实（包含完整的面试对话）
- 验证要全面：
  - ✅ 报告结构完整
  - ✅ 评分在 0-100 范围内
  - ✅ 结论长度合理（不能只有几个字）
  - ✅ 详细分析至少3个维度
  - ✅ 用户报告包含亮点、不足、建议
  - ✅ **评分和结论要一致**（高分→推荐，低分→不推荐）
  - ✅ **能力分析要和JD相关**（JD要求Python，就要评估Python能力）

### 输出格式
```python
# 完整的测试类，包含：
# 1. 必要的 import
# 2. 测试数据 fixture
# 3. 3个测试函数（对应3个场景）
# 4. 详细的验证逻辑
```
```

## 💡 提示词设计的关键点

### 1. **引导思考**（最重要！）
```
❌ 错误："生成测试代码"
✅ 正确："先分析核心功能和业务规则，然后生成测试代码"
```

### 2. **提供结构模板**
给 LLM 一个清晰的结构，它会填充内容

### 3. **强调验证重点**
```
明确告诉模型要验证：
- 结构（有没有）
- 类型（对不对）
- 范围（合不合理）
- 内容（是不是空的/占位符）
- 业务（符不符合逻辑）
```

### 4. **给出具体例子**
```
❌ "验证输出"
✅ "验证评分在0-100之间，结论不少于10个字，且高分时包含'推荐'关键词"
```

### 5. **分步骤输出**
```
1. 先让模型输出分析结果
2. 再让模型输出测试场景
3. 最后生成代码
```
这样可以提高质量，避免直接生成代码导致思考不充分

## 🎯 提示词使用流程

```
步骤1：准备输入
  ├─ 收集源代码或规格文档
  └─ 确定测试框架（pytest、unittest等）

步骤2：构建提示词
  ├─ 使用上面的模板
  ├─ 填入具体代码/规格
  └─ 根据项目调整验证要点

步骤3：调用 LLM
  └─ 使用温度 0.3-0.5（需要准确性）

步骤4：审查输出
  ├─ 检查思考过程是否合理
  ├─ 检查测试场景是否覆盖核心功能
  └─ 检查验证逻辑是否全面

步骤5：迭代优化
  └─ 根据输出质量调整提示词
```

## 📊 评估生成质量的检查清单

生成的测试代码应该满足：

- [ ] 包含思考/分析过程（注释或单独说明）
- [ ] 使用真实、完整的测试数据
- [ ] 验证输入数据格式
- [ ] 调用真实函数（不 Mock）
- [ ] 验证输出结构（必需字段）
- [ ] 验证输出类型
- [ ] 验证数值范围
- [ ] 验证内容不为空
- [ ] **验证业务逻辑**（最重要！）
- [ ] 有清晰的断言错误信息
- [ ] 有进度日志输出
- [ ] 可以直接运行

你觉得这个提示词设计思路如何？需要我根据你的具体需求再调整吗？

```markdown
# 任务：生成端到端（E2E）测试代码

## 背景
你是一位专业的测试工程师，需要为给定的代码/规格文档生成高质量的端到端测试。

## 输入
{INPUT_TYPE} # 可以是 "源代码" 或 "规格文档"

```

```plaintext

## 测试框架
- 使用 pytest
- 使用 @pytest.mark.e2e 标记
- 如果是异步函数，使用 @pytest.mark.asyncio

## 生成要求

### 第一步：分析核心功能
请先分析这段代码/文档，回答以下问题：

1. **这个功能的主要目的是什么？**（用一句话概括）
2. **关键的输入参数有哪些？**（列出参数名和类型）
3. **期望的输出是什么？**（格式、类型、必需字段）
4. **核心业务逻辑是什么？**（例如：评分、分类、转换等）
5. **有哪些重要的业务规则需要验证？**（例如：评分范围、数据完整性等）
6. **可能的异常场景有哪些？**（例如：输入为空、格式错误等）

### 第二步：确定测试场景
基于上述分析，确定需要测试的核心场景（优先级排序）：

1. **主场景（Happy Path）**：正常输入 → 正常输出
2. **边界场景**：极端但有效的输入
3. **异常场景**：无效输入的处理

### 第三步：设计测试用例
对于每个场景，明确：
- **输入数据**：具体的、真实的测试数据
- **预期输出**：详细的验证点
- **业务验证**：除了格式，还要验证逻辑是否正确

### 第四步：生成测试代码
遵循以下原则生成测试代码：

#### 必须包含的部分：
1. **详细的输入准备**
   - 使用真实、完整的数据
   - 在注释中说明为什么选择这些数据
   - 验证输入数据的格式

2. **清晰的执行过程**
   - 调用真实的函数/API（不 Mock）
   - 记录执行时间（如果需要）
   - 打印关键信息便于调试

3. **完整的输出验证**
   - 结构验证：检查必需字段是否存在
   - 类型验证：检查数据类型是否正确
   - 范围验证：检查数值是否在合理范围内
   - 内容验证：检查内容是否非空、是否合理
   - 业务逻辑验证：检查业务规则是否满足

4. **可读的断言信息**
   - 每个 assert 都要有清晰的错误提示
   - 使用 f-string 显示实际值

#### 代码结构模板：
@pytest.mark.e2e
async def test_[功能名称]_[场景描述](self):
    """测试 [功能描述] - [场景说明]"""
    
    # ========== 第1步：准备输入数据 ==========
    print(f"\n{'='*60}")
    print(f"🎯 测试场景：[场景名称]")
    print(f"{'='*60}")
    
    input_data = {
        # 真实的、完整的输入数据
    }
    
    # 验证输入数据格式
    assert [输入验证], "输入数据验证失败"
    
    print(f"\n📥 输入数据:")
    print(f"   [关键输入信息]")
    
    # ========== 第2步：执行功能 ==========
    print(f"\n🚀 执行 [功能名称]...")
    
    result = await [调用真实函数]
    
    # ========== 第3步：验证输出结构 ==========
    print(f"\n📊 验证输出结构...")
    
    # 必需字段检查
    required_fields = [...]
    for field in required_fields:
        assert field in result, f"缺少必需字段: {field}"
    print(f"   ✅ 输出结构完整")
    
    # ========== 第4步：验证输出内容 ==========
    print(f"\n📤 验证输出内容...")
    
    # 类型检查
    assert isinstance(result['field'], expected_type), \
        f"字段类型错误，期望 {expected_type}，实际 {type(result['field'])}"
    
    # 范围检查
    assert min_value <= result['value'] <= max_value, \
        f"值超出范围: {result['value']} (期望 {min_value}-{max_value})"
    
    # 内容检查
    assert len(result['text']) > min_length, \
        f"内容太短: {len(result['text'])} 字符"
    
    print(f"   ✅ [字段名]: {result['field']}")
    
    # ========== 第5步：验证业务逻辑 ==========
    print(f"\n🔍 验证业务逻辑...")
    
    # 业务规则验证
    assert [业务规则], "业务逻辑验证失败: [具体说明]"
    
    print(f"   ✅ 业务逻辑正确")
    print(f"\n🎉 测试通过！")## 输出格式
请按以下格式输出：

### 分析结果
```

```plaintext

### 测试场景分析结果]
```

```plaintext

### 测试场景分析结果]
```

```plaintext

### 测试代码
[第三步和第四步生成的完整测试代码]### 使用说明
```

```plaintext

### 测试代码
[第三步和第四步生成的完整测试代码]### 使用说明
```

```python

## 注意事项
1. 测试代码必须是可直接运行的，包含所有必要的 import
2. 使用真实数据，不要用过于简单的示例（如 "test123"）
3. 每个验证点都要有清晰的断言信息
4. 优先测试核心功能的主场景，不要被边缘情况分散注意力
5. E2E 测试应该测试"整个流程"，不要只测试某个小函数
6. 测试应该是独立的，不依赖其他测试的执行顺序
```

```plaintext

## 注意事项
1. 测试代码必须是可直接运行的，包含所有必要的 import
2. 使用真实数据，不要用过于简单的示例（如 "test123"）
3. 每个验证点都要有清晰的断言信息
4. 优先测试核心功能的主场景，不要被边缘情况分散注意力
5. E2E 测试应该测试"整个流程"，不要只测试某个小函数
6. 测试应该是独立的，不依赖其他测试的执行顺序
```

```plaintext
# 任务：为以下代码/规格生成端到端测试

## 输入
{INPUT_TYPE}: {CODE_OR_SPEC}

## 要求

### 1. 先思考（在代码注释中体现）
- 核心功能是什么？
- 关键输入和输出是什么？
- 需要验证哪些业务规则？

### 2. 生成测试（遵循这个结构）

@pytest.mark.e2e
async def test_[场景名](self):
    """端到端测试：[场景描述]"""
    
    # === 准备输入（要真实、完整） ===
    input_data = {...}
    print(f"📥 输入: {input_data}")
    
    # === 执行真实流程（不Mock） ===
    result = await real_function(input_data)
    
    # === 验证输出 ===
    # 1. 结构：必需字段是否存在
    assert "key_field" in result, "缺少关键字段"
    
    # 2. 类型：数据类型是否正确
    assert isinstance(result['field'], ExpectedType)
    
    # 3. 范围：数值是否合理
    assert min <= result['value'] <= max, f"值超出范围: {result['value']}"
    
    # 4. 内容：内容是否有效（不为空、不是占位符）
    assert len(result['text']) > 20, "内容太简短"
    
    # 5. 业务：业务逻辑是否正确
    assert [业务规则], "业务逻辑错误"
    
    print(f"📤 输出验证通过: {result}")### 3. 测试场景优先级
1. **必须测**：主要业务流程（Happy Path）
2. **建议测**：不同类型的输入（高/中/低匹配度等）
3. **可选测**：异常处理（输入为空等）

### 4. 关键原则
- ✅ 使用真实数据（不要 "test123"）
- ✅ 验证业务逻辑（不只是格式）
- ✅ 清晰的错误信息（每个 assert 都有说明）
- ✅ 从用户角度测试（输入→输出的完整流程）
- ❌ 不要 Mock 核心功能
- ❌ 不要只测试数据结构
```

```markdown
# 任务：为以下代码/规格生成端到端测试

## 输入
{INPUT_TYPE}: {CODE_OR_SPEC}

## 要求

### 1. 先思考（在代码注释中体现）
- 核心功能是什么？
- 关键输入和输出是什么？
- 需要验证哪些业务规则？

### 2. 生成测试（遵循这个结构）

@pytest.mark.e2e
async def test_[场景名](self):
    """端到端测试：[场景描述]"""
    
    # === 准备输入（要真实、完整） ===
    input_data = {...}
    print(f"📥 输入: {input_data}")
    
    # === 执行真实流程（不Mock） ===
    result = await real_function(input_data)
    
    # === 验证输出 ===
    # 1. 结构：必需字段是否存在
    assert "key_field" in result, "缺少关键字段"
    
    # 2. 类型：数据类型是否正确
    assert isinstance(result['field'], ExpectedType)
    
    # 3. 范围：数值是否合理
    assert min <= result['value'] <= max, f"值超出范围: {result['value']}"
    
    # 4. 内容：内容是否有效（不为空、不是占位符）
    assert len(result['text']) > 20, "内容太简短"
    
    # 5. 业务：业务逻辑是否正确
    assert [业务规则], "业务逻辑错误"
    
    print(f"📤 输出验证通过: {result}")### 3. 测试场景优先级
1. **必须测**：主要业务流程（Happy Path）
2. **建议测**：不同类型的输入（高/中/低匹配度等）
3. **可选测**：异常处理（输入为空等）

### 4. 关键原则
- ✅ 使用真实数据（不要 "test123"）
- ✅ 验证业务逻辑（不只是格式）
- ✅ 清晰的错误信息（每个 assert 都有说明）
- ✅ 从用户角度测试（输入→输出的完整流程）
- ❌ 不要 Mock 核心功能
- ❌ 不要只测试数据结构
```

```plaintext
# 任务：为面试评估系统生成端到端测试

## 输入代码
class AIInterviewAssessor:
    def generate_report(
        self,
        interview_id: int,
        transcript_json_str: str,
        jd: str,
        url_list: list = [],
        is_mp4: bool = True,
        interview_language: str = "zhongwen"
    ) -> dict:
        """生成面试评估报告"""
        # ... 实现代码## 功能说明
这个函数根据面试转录和岗位描述，生成包含评分、分析和建议的评估报告。

## 生成要求

### 思考过程（必须先回答）
1. **核心功能**：这个函数做什么？
2. **输入**：需要哪些参数？什么格式？
3. **输出**：返回什么？包含哪些字段？
4. **业务规则**：
   - 评分范围是多少？
   - 高分/低分时结论应该是什么？
   - 必须包含哪些分析维度？

### 测试场景设计
1. **场景1**：高级工程师+匹配的JD → 期望高分+推荐录用
2. **场景2**：初级工程师+高级JD → 期望低分+不推荐
3. **场景3**：技能不匹配 → 期望低分+明确指出问题

### 测试代码要求
- 每个场景生成一个独立的测试函数
- 输入数据要真实（包含完整的面试对话）
- 验证要全面：
  - ✅ 报告结构完整
  - ✅ 评分在 0-100 范围内
  - ✅ 结论长度合理（不能只有几个字）
  - ✅ 详细分析至少3个维度
  - ✅ 用户报告包含亮点、不足、建议
  - ✅ **评分和结论要一致**（高分→推荐，低分→不推荐）
  - ✅ **能力分析要和JD相关**（JD要求Python，就要评估Python能力）

### 输出格式
# 完整的测试类，包含：
# 1. 必要的 import
# 2. 测试数据 fixture
# 3. 3个测试函数（对应3个场景）
# 4. 详细的验证逻辑
```

```markdown
# 任务：为面试评估系统生成端到端测试

## 输入代码
class AIInterviewAssessor:
    def generate_report(
        self,
        interview_id: int,
        transcript_json_str: str,
        jd: str,
        url_list: list = [],
        is_mp4: bool = True,
        interview_language: str = "zhongwen"
    ) -> dict:
        """生成面试评估报告"""
        # ... 实现代码## 功能说明
这个函数根据面试转录和岗位描述，生成包含评分、分析和建议的评估报告。

## 生成要求

### 思考过程（必须先回答）
1. **核心功能**：这个函数做什么？
2. **输入**：需要哪些参数？什么格式？
3. **输出**：返回什么？包含哪些字段？
4. **业务规则**：
   - 评分范围是多少？
   - 高分/低分时结论应该是什么？
   - 必须包含哪些分析维度？

### 测试场景设计
1. **场景1**：高级工程师+匹配的JD → 期望高分+推荐录用
2. **场景2**：初级工程师+高级JD → 期望低分+不推荐
3. **场景3**：技能不匹配 → 期望低分+明确指出问题

### 测试代码要求
- 每个场景生成一个独立的测试函数
- 输入数据要真实（包含完整的面试对话）
- 验证要全面：
  - ✅ 报告结构完整
  - ✅ 评分在 0-100 范围内
  - ✅ 结论长度合理（不能只有几个字）
  - ✅ 详细分析至少3个维度
  - ✅ 用户报告包含亮点、不足、建议
  - ✅ **评分和结论要一致**（高分→推荐，低分→不推荐）
  - ✅ **能力分析要和JD相关**（JD要求Python，就要评估Python能力）

### 输出格式
# 完整的测试类，包含：
# 1. 必要的 import
# 2. 测试数据 fixture
# 3. 3个测试函数（对应3个场景）
# 4. 详细的验证逻辑
```

```plaintext
❌ 错误："生成测试代码"
✅ 正确："先分析核心功能和业务规则，然后生成测试代码"
```

```plaintext
❌ 错误："生成测试代码"
✅ 正确："先分析核心功能和业务规则，然后生成测试代码"
```

```markdown
明确告诉模型要验证：
- 结构（有没有）
- 类型（对不对）
- 范围（合不合理）
- 内容（是不是空的/占位符）
- 业务（符不符合逻辑）
```

```plaintext
明确告诉模型要验证：
- 结构（有没有）
- 类型（对不对）
- 范围（合不合理）
- 内容（是不是空的/占位符）
- 业务（符不符合逻辑）
```

```python
❌ "验证输出"
✅ "验证评分在0-100之间，结论不少于10个字，且高分时包含'推荐'关键词"
```

```plaintext
❌ "验证输出"
✅ "验证评分在0-100之间，结论不少于10个字，且高分时包含'推荐'关键词"
```

```plaintext
1. 先让模型输出分析结果
2. 再让模型输出测试场景
3. 最后生成代码
```

```plaintext
1. 先让模型输出分析结果
2. 再让模型输出测试场景
3. 最后生成代码
```

```plaintext
步骤1：准备输入
  ├─ 收集源代码或规格文档
  └─ 确定测试框架（pytest、unittest等）

步骤2：构建提示词
  ├─ 使用上面的模板
  ├─ 填入具体代码/规格
  └─ 根据项目调整验证要点

步骤3：调用 LLM
  └─ 使用温度 0.3-0.5（需要准确性）

步骤4：审查输出
  ├─ 检查思考过程是否合理
  ├─ 检查测试场景是否覆盖核心功能
  └─ 检查验证逻辑是否全面

步骤5：迭代优化
  └─ 根据输出质量调整提示词
```

```plaintext
步骤1：准备输入
  ├─ 收集源代码或规格文档
  └─ 确定测试框架（pytest、unittest等）

步骤2：构建提示词
  ├─ 使用上面的模板
  ├─ 填入具体代码/规格
  └─ 根据项目调整验证要点

步骤3：调用 LLM
  └─ 使用温度 0.3-0.5（需要准确性）

步骤4：审查输出
  ├─ 检查思考过程是否合理
  ├─ 检查测试场景是否覆盖核心功能
  └─ 检查验证逻辑是否全面

步骤5：迭代优化
  └─ 根据输出质量调整提示词
```

```plaintext

### 4. **给出具体例子**
```

```plaintext

### 5. **分步骤输出**
```

```plaintext
这样可以提高质量，避免直接生成代码导致思考不充分

## 🎯 提示词使用流程

```

```plaintext

## 📊 评估生成质量的检查清单

生成的测试代码应该满足：

- [ ] 包含思考/分析过程（注释或单独说明）
- [ ] 使用真实、完整的测试数据
- [ ] 验证输入数据格式
- [ ] 调用真实函数（不 Mock）
- [ ] 验证输出结构（必需字段）
- [ ] 验证输出类型
- [ ] 验证数值范围
- [ ] 验证内容不为空
- [ ] **验证业务逻辑**（最重要！）
- [ ] 有清晰的断言错误信息
- [ ] 有进度日志输出
- [ ] 可以直接运行

你觉得这个提示词设计思路如何？需要我根据你的具体需求再调整吗？
```

---

**User**



下面这个内容是你前面给我的提示词，我看了一下，非常好，非常不错，非常全面。然后我相信模型会生成非常好的 端到端测试，但是我的问题就在于我是想让他把e2e测试写到特定的文件里面呀，模型这样返回来,测试代码和它的相应的长篇大论的解释会融杂在一起吧？这怎么解决呀？ 

```

# 任务：生成端到端（E2E）测试代码



## 背景

你是一位专业的测试工程师，需要为给定的代码/规格文档生成高质量的端到端测试。

## 输入

{可以是 "源代码" 或 "规格文档"}  

## 测试框架

- 使用 pytest

- 使用 @pytest.mark.e2e 标记

- 如果是异步函数，使用 @pytest.mark.asyncio

## 生成要求

### 第一步：分析核心功能

请先分析这段代码/文档，回答以下问题：

1. **这个功能的主要目的是什么？**（用一句话概括）

2. **关键的输入参数有哪些？**（列出参数名和类型）

3. **期望的输出是什么？**（格式、类型、必需字段）

4. **核心业务逻辑是什么？**（例如：评分、分类、转换等）

5. **有哪些重要的业务规则需要验证？**（例如：评分范围、数据完整性等）

6. **可能的异常场景有哪些？**（例如：输入为空、格式错误等）

### 第二步：确定测试场景

基于上述分析，确定需要测试的核心场景（优先级排序）：

1. **主场景（Happy Path）**：正常输入 → 正常输出

2. **边界场景**：极端但有效的输入

3. **异常场景**：无效输入的处理

### 第三步：设计测试用例

对于每个场景，明确：

- **输入数据**：具体的、真实的测试数据

- **预期输出**：详细的验证点

- **业务验证**：除了格式，还要验证逻辑是否正确

### 第四步：生成测试代码

遵循以下原则生成测试代码：

#### 必须包含的部分：

1. **详细的输入准备**

   - 使用真实、完整的数据

   - 在注释中说明为什么选择这些数据

   - 验证输入数据的格式

2. **清晰的执行过程**

   - 调用真实的函数/API（不 Mock）

   - 记录执行时间（如果需要）

   - 打印关键信息便于调试

3. **完整的输出验证**

   - 结构验证：检查必需字段是否存在

   - 类型验证：检查数据类型是否正确

   - 范围验证：检查数值是否在合理范围内

   - 内容验证：检查内容是否非空、是否合理

   - 业务逻辑验证：检查业务规则是否满足

4. **可读的断言信息**

   - 每个 assert 都要有清晰的错误提示

   - 使用 f-string 显示实际值

## 输出格式

请按以下格式输出：

### 分析结果

```

[第一步的分析结果]

```

### 测试场景

```

[第二步确定的测试场景]

```

### 测试代码

```python

[第三步和第四步生成的完整测试代码]

```

### 使用说明

```

[如何运行这些测试，以及注意事项]

```

## 注意事项

1. 测试代码必须是可直接运行的，包含所有必要的 import

2. 使用真实数据，不要用过于简单的示例（如 "test123"）

3. 每个验证点都要有清晰的断言信息

4. 优先测试核心功能的主场景，不要被边缘情况分散注意力

5. E2E 测试应该测试"整个流程"，不要只测试某个小函数

6. 测试应该是独立的，不依赖其他测试的执行顺序

```

---

**Cursor**

非常好的问题！你完全理解了实际使用中的痛点。让我给你几个解决方案：

## 🎯 解决方案对比

### 方案 1：两阶段 Prompt（推荐！）⭐⭐⭐⭐⭐

**思路**：先让 LLM 思考，再让它只生成代码

#### 第一次调用 LLM（分析阶段）
```markdown
# 任务：分析功能并设计测试场景

## 输入代码
```python
{YOUR_CODE}
```

## 分析要求
请分析以下内容：

1. **核心功能**：这个功能做什么？（1句话）
2. **输入输出**：
   - 输入参数及类型
   - 输出格式及必需字段
3. **业务规则**：需要验证的关键业务逻辑（3-5条）
4. **测试场景**：列出2-3个最重要的测试场景

## 输出格式（纯文本，用于下一步）
```json
{
  "功能描述": "...",
  "输入参数": [...],
  "输出字段": [...],
  "业务规则": [...],
  "测试场景": [
    {
      "场景名": "...",
      "输入数据": {...},
      "预期结果": {...}
    }
  ]
}
```
```

#### 第二次调用 LLM（生成代码）
```markdown
# 任务：生成端到端测试代码（纯代码输出）

## 功能分析结果
{第一次调用的输出}

## 要求
根据上述分析，生成完整的 pytest 测试代码。

## 输出格式
**只输出可执行的 Python 代码，不要任何解释文字。**
代码应该：
- 包含所有必要的 import
- 包含测试类和测试方法
- 包含详细的验证逻辑
- 在代码注释中体现思考过程

## 示例输出格式
```python
"""
端到端测试 - [功能名称]
基于分析结果生成的测试
"""
import pytest
# ... 其他 import

@pytest.mark.e2e
class Test[功能名]E2E:
    # 测试代码...
```

**重要：只输出 Python 代码块内的内容，不要输出任何其他文字！**
```

**优点**：
- ✅ 分析和代码分离，清晰
- ✅ 第二次输出纯代码，可直接写入文件
- ✅ 思考过程保留在第一次输出中，可供审查
- ✅ 灵活：可以手动调整分析结果后再生成代码

---

### 方案 2：结构化输出 Prompt（OpenAI 最新功能）⭐⭐⭐⭐

使用 OpenAI 的 **Structured Output** 功能，强制 LLM 按照指定格式输出：

```python
# 定义输出结构
response_format = {
    "type": "json_schema",
    "json_schema": {
        "name": "e2e_test_generation",
        "schema": {
            "type": "object",
            "properties": {
                "analysis": {
                    "type": "object",
                    "properties": {
                        "core_function": {"type": "string"},
                        "business_rules": {"type": "array", "items": {"type": "string"}}
                    }
                },
                "test_code": {
                    "type": "string",
                    "description": "完整的 Python 测试代码"
                }
            },
            "required": ["analysis", "test_code"]
        }
    }
}

# 调用 API
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": your_prompt}],
    response_format=response_format
)

# 解析输出
result = json.loads(response.choices[0].message.content)
analysis = result["analysis"]  # 分析内容
test_code = result["test_code"]  # 纯测试代码

# 直接写入文件
with open("test_file.py", "w") as f:
    f.write(test_code)
```

**提示词示例**：
```markdown
# 任务：生成端到端测试

## 输入
{YOUR_CODE}

## 输出要求
请以 JSON 格式输出：
{
  "analysis": {
    "core_function": "功能描述",
    "business_rules": ["规则1", "规则2"]
  },
  "test_code": "完整的Python测试代码（字符串）"
}

注意：test_code 字段应该包含完整可运行的 Python 代码。
```

**优点**：
- ✅ 输出格式可预测
- ✅ 分析和代码分离
- ✅ 易于程序化处理

---

### 方案 3：单次 Prompt + 标记分割⭐⭐⭐

在 Prompt 中要求 LLM 使用特殊标记分隔代码和解释：

```markdown
# 任务：生成端到端测试代码

{你原来的 Prompt 内容...}

## 输出格式要求（重要！）

请按以下格式严格输出，使用特定标记分隔各部分：

---ANALYSIS_START---
[你的分析内容]
---ANALYSIS_END---

---TEST_CODE_START---
```python
[完整的测试代码，不要有任何其他文字]
```
---TEST_CODE_END---

---USAGE_START---
[使用说明]
---USAGE_END---

**关键**：TEST_CODE 部分只能包含 Python 代码，不要有任何解释文字。
```

**处理代码**：
```python
def parse_llm_output(output: str) -> dict:
    """解析 LLM 输出"""
    
    # 提取分析部分
    analysis = extract_between(output, "---ANALYSIS_START---", "---ANALYSIS_END---")
    
    # 提取测试代码（去除 markdown 代码块标记）
    code_section = extract_between(output, "---TEST_CODE_START---", "---TEST_CODE_END---")
    test_code = code_section.replace("```python", "").replace("```", "").strip()
    
    # 提取使用说明
    usage = extract_between(output, "---USAGE_START---", "---USAGE_END---")
    
    return {
        "analysis": analysis.strip(),
        "test_code": test_code,
        "usage": usage.strip()
    }

def extract_between(text: str, start_marker: str, end_marker: str) -> str:
    """提取两个标记之间的内容"""
    start = text.find(start_marker)
    end = text.find(end_marker)
    if start == -1 or end == -1:
        return ""
    return text[start + len(start_marker):end]

# 使用
result = parse_llm_output(llm_response)
with open("test_e2e.py", "w", encoding="utf-8") as f:
    f.write(result["test_code"])
```

**优点**：
- ✅ 一次调用完成
- ✅ 可程序化提取代码
- ⚠️ 需要 LLM 严格遵循格式（可能不稳定）

---

### 方案 4：极简 Prompt（只要代码）⭐⭐⭐

如果你只想要代码，不要解释：

```markdown
# 任务：生成端到端测试代码

## 输入代码
```python
{YOUR_CODE}
```

## 要求
为上述代码生成完整的 pytest 端到端测试。

### 测试要求
- 使用 @pytest.mark.e2e 标记
- 测试核心业务流程
- 验证输入输出格式
- 验证业务逻辑正确性
- 包含详细的断言和错误信息

### 输出要求
**只输出 Python 代码，不要输出任何解释文字！**
**不要输出 "分析"、"说明" 等额外内容！**
**代码中可以包含注释来说明测试逻辑。**

直接从 import 语句开始输出。

## 输出示例
```python
"""端到端测试文件"""
import pytest
...
```
```

**优点**：
- ✅ 最简单
- ✅ 输出最干净
- ⚠️ 缺少分析过程（不知道 LLM 的思考逻辑）

---

## 🏆 我的推荐：方案 1（两阶段）

**为什么？**

1. **可控性最强**：你可以审查分析结果，确保 LLM 理解正确
2. **输出最干净**：第二次只输出代码
3. **可复现**：如果代码有问题，可以调整分析结果重新生成
4. **成本优化**：分析一次，可以多次生成代码（调整提示词）

**实际工作流程**：

```python
# 步骤 1：分析功能
analysis_prompt = """
# 任务：分析功能
{CODE}

输出 JSON 格式的分析结果，包括：功能描述、输入输出、业务规则、测试场景
"""

analysis_result = call_llm(analysis_prompt)
print("📋 分析结果：")
print(analysis_result)

# 🔍 人工审查分析结果，确认无误

# 步骤 2：生成测试代码
code_gen_prompt = f"""
# 任务：生成测试代码

## 分析结果
{analysis_result}

## 要求
根据上述分析生成完整的 pytest 测试代码。
**只输出 Python 代码，不要任何解释！**

输出格式：
```python
# 测试代码从这里开始
import pytest
...
```
"""

test_code = call_llm(code_gen_prompt)

# 提取代码（去除 markdown 标记）
clean_code = extract_code_from_markdown(test_code)

# 写入文件
with open("tests/e2e/test_generated.py", "w", encoding="utf-8") as f:
    f.write(clean_code)

print("✅ 测试代码已生成到: tests/e2e/test_generated.py")
```

**辅助函数**：
```python
def extract_code_from_markdown(text: str) -> str:
    """从 markdown 代码块中提取代码"""
    # 匹配 ```python ... ``` 
    import re
    pattern = r'```python\n(.*?)\n```'
    matches = re.findall(pattern, text, re.DOTALL)
    
    if matches:
        return matches[0].strip()
    
    # 如果没有 markdown 标记，尝试直接返回（去除前后空白）
    return text.strip()
```

---

## 💡 额外技巧

### 技巧 1：明确告诉 LLM 输出会被直接写入文件

```markdown
**重要提示**：
你的输出将被直接写入 Python 文件执行，不会经过人工编辑。
因此必须确保：
1. 输出的是纯 Python 代码
2. 不包含任何解释文字（注释除外）
3. 代码可以直接运行
```

### 技巧 2：给 LLM 看一个完整的例子

```markdown
## 输出格式示例（你的输出应该和这个完全一样的格式）

```python
"""端到端测试 - 示例功能"""
import pytest
import json

@pytest.mark.e2e
class TestExampleE2E:
    def test_main_scenario(self):
        """测试主要场景"""
        # 准备输入
        input_data = {...}
        
        # 执行
        result = function(input_data)
        
        # 验证
        assert "field" in result
        assert result["field"] == expected_value
```

**你的输出应该完全遵循这个格式！**
```

### 技巧 3：后处理清理

即使有上述要求，LLM 可能还是输出额外内容，做个后处理：

```python
def clean_llm_code_output(text: str) -> str:
    """清理 LLM 输出，只保留代码"""
    
    # 移除常见的解释性文字
    lines_to_remove = [
        "以下是生成的测试代码",
        "这是测试代码",
        "Here is the test code",
        "根据分析",
    ]
    
    lines = text.split('\n')
    cleaned_lines = []
    
    for line in lines:
        # 跳过解释性文字
        if any(phrase in line for phrase in lines_to_remove):
            continue
        cleaned_lines.append(line)
    
    result = '\n'.join(cleaned_lines)
    
    # 提取 ```python ... ``` 之间的内容
    result = extract_code_from_markdown(result)
    
    return result
```

---

## 🎯 最终推荐方案

结合你的需求，我建议使用 **方案 1（两阶段）+ 后处理清理**：

```python
def generate_e2e_test(code: str, output_file: str):
    """生成端到端测试"""
    
    # 阶段 1：分析
    analysis = analyze_code(code)
    print("📋 分析完成")
    
    # 阶段 2：生成代码
    raw_code = generate_test_code(analysis)
    
    # 清理输出
    clean_code = clean_llm_code_output(raw_code)
    
    # 写入文件
    with open(output_file, "w", encoding="utf-8") as f:
        f.write(clean_code)
    
    print(f"✅ 测试已生成: {output_file}")
    
    return {
        "analysis": analysis,
        "code": clean_code
    }
```

这样你就能得到干净的、可直接使用的测试代码了！🎉

```markdown
# 任务：分析功能并设计测试场景

## 输入代码
{YOUR_CODE}## 分析要求
请分析以下内容：

1. **核心功能**：这个功能做什么？（1句话）
2. **输入输出**：
   - 输入参数及类型
   - 输出格式及必需字段
3. **业务规则**：需要验证的关键业务逻辑（3-5条）
4. **测试场景**：列出2-3个最重要的测试场景

## 输出格式（纯文本，用于下一步）
{
  "功能描述": "...",
  "输入参数": [...],
  "输出字段": [...],
  "业务规则": [...],
  "测试场景": [
    {
      "场景名": "...",
      "输入数据": {...},
      "预期结果": {...}
    }
  ]
}
```

```json
# 任务：生成端到端测试代码（纯代码输出）

## 功能分析结果
{第一次调用的输出}

## 要求
根据上述分析，生成完整的 pytest 测试代码。

## 输出格式
**只输出可执行的 Python 代码，不要任何解释文字。**
代码应该：
- 包含所有必要的 import
- 包含测试类和测试方法
- 包含详细的验证逻辑
- 在代码注释中体现思考过程

## 示例输出格式
"""
端到端测试 - [功能名称]
基于分析结果生成的测试
"""
import pytest
# ... 其他 import

@pytest.mark.e2e
class Test[功能名]E2E:
    # 测试代码...**重要：只输出 Python 代码块内的内容，不要输出任何其他文字！**
```

```markdown
# 任务：生成端到端测试代码（纯代码输出）

## 功能分析结果
{第一次调用的输出}

## 要求
根据上述分析，生成完整的 pytest 测试代码。

## 输出格式
**只输出可执行的 Python 代码，不要任何解释文字。**
代码应该：
- 包含所有必要的 import
- 包含测试类和测试方法
- 包含详细的验证逻辑
- 在代码注释中体现思考过程

## 示例输出格式
"""
端到端测试 - [功能名称]
基于分析结果生成的测试
"""
import pytest
# ... 其他 import

@pytest.mark.e2e
class Test[功能名]E2E:
    # 测试代码...**重要：只输出 Python 代码块内的内容，不要输出任何其他文字！**
```

```plaintext
# 定义输出结构
response_format = {
    "type": "json_schema",
    "json_schema": {
        "name": "e2e_test_generation",
        "schema": {
            "type": "object",
            "properties": {
                "analysis": {
                    "type": "object",
                    "properties": {
                        "core_function": {"type": "string"},
                        "business_rules": {"type": "array", "items": {"type": "string"}}
                    }
                },
                "test_code": {
                    "type": "string",
                    "description": "完整的 Python 测试代码"
                }
            },
            "required": ["analysis", "test_code"]
        }
    }
}

# 调用 API
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": your_prompt}],
    response_format=response_format
)

# 解析输出
result = json.loads(response.choices[0].message.content)
analysis = result["analysis"]  # 分析内容
test_code = result["test_code"]  # 纯测试代码

# 直接写入文件
with open("test_file.py", "w") as f:
    f.write(test_code)
```

```python
# 定义输出结构
response_format = {
    "type": "json_schema",
    "json_schema": {
        "name": "e2e_test_generation",
        "schema": {
            "type": "object",
            "properties": {
                "analysis": {
                    "type": "object",
                    "properties": {
                        "core_function": {"type": "string"},
                        "business_rules": {"type": "array", "items": {"type": "string"}}
                    }
                },
                "test_code": {
                    "type": "string",
                    "description": "完整的 Python 测试代码"
                }
            },
            "required": ["analysis", "test_code"]
        }
    }
}

# 调用 API
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": your_prompt}],
    response_format=response_format
)

# 解析输出
result = json.loads(response.choices[0].message.content)
analysis = result["analysis"]  # 分析内容
test_code = result["test_code"]  # 纯测试代码

# 直接写入文件
with open("test_file.py", "w") as f:
    f.write(test_code)
```

```python
# 任务：生成端到端测试

## 输入
{YOUR_CODE}

## 输出要求
请以 JSON 格式输出：
{
  "analysis": {
    "core_function": "功能描述",
    "business_rules": ["规则1", "规则2"]
  },
  "test_code": "完整的Python测试代码（字符串）"
}

注意：test_code 字段应该包含完整可运行的 Python 代码。
```

```markdown
# 任务：生成端到端测试

## 输入
{YOUR_CODE}

## 输出要求
请以 JSON 格式输出：
{
  "analysis": {
    "core_function": "功能描述",
    "business_rules": ["规则1", "规则2"]
  },
  "test_code": "完整的Python测试代码（字符串）"
}

注意：test_code 字段应该包含完整可运行的 Python 代码。
```

```markdown
# 任务：生成端到端测试代码

{你原来的 Prompt 内容...}

## 输出格式要求（重要！）

请按以下格式严格输出，使用特定标记分隔各部分：

---ANALYSIS_START---
[你的分析内容]
---ANALYSIS_END---

---TEST_CODE_START---
[完整的测试代码，不要有任何其他文字]---TEST_CODE_END---

---USAGE_START---
[使用说明]
---USAGE_END---

**关键**：TEST_CODE 部分只能包含 Python 代码，不要有任何解释文字。
```

```markdown
# 任务：生成端到端测试代码

{你原来的 Prompt 内容...}

## 输出格式要求（重要！）

请按以下格式严格输出，使用特定标记分隔各部分：

---ANALYSIS_START---
[你的分析内容]
---ANALYSIS_END---

---TEST_CODE_START---
[完整的测试代码，不要有任何其他文字]---TEST_CODE_END---

---USAGE_START---
[使用说明]
---USAGE_END---

**关键**：TEST_CODE 部分只能包含 Python 代码，不要有任何解释文字。
```

```markdown
def parse_llm_output(output: str) -> dict:
    """解析 LLM 输出"""
    
    # 提取分析部分
    analysis = extract_between(output, "---ANALYSIS_START---", "---ANALYSIS_END---")
    
    # 提取测试代码（去除 markdown 代码块标记）
    code_section = extract_between(output, "---TEST_CODE_START---", "---TEST_CODE_END---")
    test_code = code_section.replace("", "").replace("```", "").strip()
    
    # 提取使用说明
    usage = extract_between(output, "---USAGE_START---", "---USAGE_END---")
    
    return {
        "analysis": analysis.strip(),
        "test_code": test_code,
        "usage": usage.strip()
    }

def extract_between(text: str, start_marker: str, end_marker: str) -> str:
    """提取两个标记之间的内容"""
    start = text.find(start_marker)
    end = text.find(end_marker)
    if start == -1 or end == -1:
        return ""
    return text[start + len(start_marker):end]

# 使用
result = parse_llm_output(llm_response)
with open("test_e2e.py", "w", encoding="utf-8") as f:
    f.write(result["test_code"])**优点**：
- ✅ 一次调用完成
- ✅ 可程序化提取代码
- ⚠️ 需要 LLM 严格遵循格式（可能不稳定）

---

### 方案 4：极简 Prompt（只要代码）⭐⭐⭐

如果你只想要代码，不要解释：

# 任务：生成端到端测试代码

## 输入代码
{YOUR_CODE}## 要求
为上述代码生成完整的 pytest 端到端测试。

### 测试要求
- 使用 @pytest.mark.e2e 标记
- 测试核心业务流程
- 验证输入输出格式
- 验证业务逻辑正确性
- 包含详细的断言和错误信息

### 输出要求
**只输出 Python 代码，不要输出任何解释文字！**
**不要输出 "分析"、"说明" 等额外内容！**
**代码中可以包含注释来说明测试逻辑。**

直接从 import 语句开始输出。

## 输出示例
"""端到端测试文件"""
import pytest
...**优点**：
- ✅ 最简单
- ✅ 输出最干净
- ⚠️ 缺少分析过程（不知道 LLM 的思考逻辑）

---

## 🏆 我的推荐：方案 1（两阶段）

**为什么？**

1. **可控性最强**：你可以审查分析结果，确保 LLM 理解正确
2. **输出最干净**：第二次只输出代码
3. **可复现**：如果代码有问题，可以调整分析结果重新生成
4. **成本优化**：分析一次，可以多次生成代码（调整提示词）

**实际工作流程**：

# 步骤 1：分析功能
analysis_prompt = """
# 任务：分析功能
{CODE}

输出 JSON 格式的分析结果，包括：功能描述、输入输出、业务规则、测试场景
"""

analysis_result = call_llm(analysis_prompt)
print("📋 分析结果：")
print(analysis_result)

# 🔍 人工审查分析结果，确认无误

# 步骤 2：生成测试代码
code_gen_prompt = f"""
# 任务：生成测试代码

## 分析结果
{analysis_result}

## 要求
根据上述分析生成完整的 pytest 测试代码。
**只输出 Python 代码，不要任何解释！**

输出格式：
# 测试代码从这里开始
import pytest
..."""

test_code = call_llm(code_gen_prompt)

# 提取代码（去除 markdown 标记）
clean_code = extract_code_from_markdown(test_code)

# 写入文件
with open("tests/e2e/test_generated.py", "w", encoding="utf-8") as f:
    f.write(clean_code)

print("✅ 测试代码已生成到: tests/e2e/test_generated.py")**辅助函数**：
def extract_code_from_markdown(text: str) -> str:
    """从 markdown 代码块中提取代码"""
    # 匹配 ...     import re
    pattern = r'\n(.*?)\n```'
    matches = re.findall(pattern, text, re.DOTALL)
    
    if matches:
        return matches[0].strip()
    
    # 如果没有 markdown 标记，尝试直接返回（去除前后空白）
    return text.strip()---

## 💡 额外技巧

### 技巧 1：明确告诉 LLM 输出会被直接写入文件

**重要提示**：
你的输出将被直接写入 Python 文件执行，不会经过人工编辑。
因此必须确保：
1. 输出的是纯 Python 代码
2. 不包含任何解释文字（注释除外）
3. 代码可以直接运行### 技巧 2：给 LLM 看一个完整的例子

## 输出格式示例（你的输出应该和这个完全一样的格式）

"""端到端测试 - 示例功能"""
import pytest
import json

@pytest.mark.e2e
class TestExampleE2E:
    def test_main_scenario(self):
        """测试主要场景"""
        # 准备输入
        input_data = {...}
        
        # 执行
        result = function(input_data)
        
        # 验证
        assert "field" in result
        assert result["field"] == expected_value**你的输出应该完全遵循这个格式！**### 技巧 3：后处理清理

即使有上述要求，LLM 可能还是输出额外内容，做个后处理：

def clean_llm_code_output(text: str) -> str:
    """清理 LLM 输出，只保留代码"""
    
    # 移除常见的解释性文字
    lines_to_remove = [
        "以下是生成的测试代码",
        "这是测试代码",
        "Here is the test code",
        "根据分析",
    ]
    
    lines = text.split('\n')
    cleaned_lines = []
    
    for line in lines:
        # 跳过解释性文字
        if any(phrase in line for phrase in lines_to_remove):
            continue
        cleaned_lines.append(line)
    
    result = '\n'.join(cleaned_lines)
    
    # 提取  ... ``` 之间的内容
    result = extract_code_from_markdown(result)
    
    return result---

## 🎯 最终推荐方案

结合你的需求，我建议使用 **方案 1（两阶段）+ 后处理清理**：

def generate_e2e_test(code: str, output_file: str):
    """生成端到端测试"""
    
    # 阶段 1：分析
    analysis = analyze_code(code)
    print("📋 分析完成")
    
    # 阶段 2：生成代码
    raw_code = generate_test_code(analysis)
    
    # 清理输出
    clean_code = clean_llm_code_output(raw_code)
    
    # 写入文件
    with open(output_file, "w", encoding="utf-8") as f:
        f.write(clean_code)
    
    print(f"✅ 测试已生成: {output_file}")
    
    return {
        "analysis": analysis,
        "code": clean_code
    }这样你就能得到干净的、可直接使用的测试代码了！🎉
```

```python
def parse_llm_output(output: str) -> dict:
    """解析 LLM 输出"""
    
    # 提取分析部分
    analysis = extract_between(output, "---ANALYSIS_START---", "---ANALYSIS_END---")
    
    # 提取测试代码（去除 markdown 代码块标记）
    code_section = extract_between(output, "---TEST_CODE_START---", "---TEST_CODE_END---")
    test_code = code_section.replace("", "").replace("```", "").strip()
    
    # 提取使用说明
    usage = extract_between(output, "---USAGE_START---", "---USAGE_END---")
    
    return {
        "analysis": analysis.strip(),
        "test_code": test_code,
        "usage": usage.strip()
    }

def extract_between(text: str, start_marker: str, end_marker: str) -> str:
    """提取两个标记之间的内容"""
    start = text.find(start_marker)
    end = text.find(end_marker)
    if start == -1 or end == -1:
        return ""
    return text[start + len(start_marker):end]

# 使用
result = parse_llm_output(llm_response)
with open("test_e2e.py", "w", encoding="utf-8") as f:
    f.write(result["test_code"])**优点**：
- ✅ 一次调用完成
- ✅ 可程序化提取代码
- ⚠️ 需要 LLM 严格遵循格式（可能不稳定）

---

### 方案 4：极简 Prompt（只要代码）⭐⭐⭐

如果你只想要代码，不要解释：

# 任务：生成端到端测试代码

## 输入代码
{YOUR_CODE}## 要求
为上述代码生成完整的 pytest 端到端测试。

### 测试要求
- 使用 @pytest.mark.e2e 标记
- 测试核心业务流程
- 验证输入输出格式
- 验证业务逻辑正确性
- 包含详细的断言和错误信息

### 输出要求
**只输出 Python 代码，不要输出任何解释文字！**
**不要输出 "分析"、"说明" 等额外内容！**
**代码中可以包含注释来说明测试逻辑。**

直接从 import 语句开始输出。

## 输出示例
"""端到端测试文件"""
import pytest
...**优点**：
- ✅ 最简单
- ✅ 输出最干净
- ⚠️ 缺少分析过程（不知道 LLM 的思考逻辑）

---

## 🏆 我的推荐：方案 1（两阶段）

**为什么？**

1. **可控性最强**：你可以审查分析结果，确保 LLM 理解正确
2. **输出最干净**：第二次只输出代码
3. **可复现**：如果代码有问题，可以调整分析结果重新生成
4. **成本优化**：分析一次，可以多次生成代码（调整提示词）

**实际工作流程**：

# 步骤 1：分析功能
analysis_prompt = """
# 任务：分析功能
{CODE}

输出 JSON 格式的分析结果，包括：功能描述、输入输出、业务规则、测试场景
"""

analysis_result = call_llm(analysis_prompt)
print("📋 分析结果：")
print(analysis_result)

# 🔍 人工审查分析结果，确认无误

# 步骤 2：生成测试代码
code_gen_prompt = f"""
# 任务：生成测试代码

## 分析结果
{analysis_result}

## 要求
根据上述分析生成完整的 pytest 测试代码。
**只输出 Python 代码，不要任何解释！**

输出格式：
# 测试代码从这里开始
import pytest
..."""

test_code = call_llm(code_gen_prompt)

# 提取代码（去除 markdown 标记）
clean_code = extract_code_from_markdown(test_code)

# 写入文件
with open("tests/e2e/test_generated.py", "w", encoding="utf-8") as f:
    f.write(clean_code)

print("✅ 测试代码已生成到: tests/e2e/test_generated.py")**辅助函数**：
def extract_code_from_markdown(text: str) -> str:
    """从 markdown 代码块中提取代码"""
    # 匹配 ...     import re
    pattern = r'\n(.*?)\n```'
    matches = re.findall(pattern, text, re.DOTALL)
    
    if matches:
        return matches[0].strip()
    
    # 如果没有 markdown 标记，尝试直接返回（去除前后空白）
    return text.strip()---

## 💡 额外技巧

### 技巧 1：明确告诉 LLM 输出会被直接写入文件

**重要提示**：
你的输出将被直接写入 Python 文件执行，不会经过人工编辑。
因此必须确保：
1. 输出的是纯 Python 代码
2. 不包含任何解释文字（注释除外）
3. 代码可以直接运行### 技巧 2：给 LLM 看一个完整的例子

## 输出格式示例（你的输出应该和这个完全一样的格式）

"""端到端测试 - 示例功能"""
import pytest
import json

@pytest.mark.e2e
class TestExampleE2E:
    def test_main_scenario(self):
        """测试主要场景"""
        # 准备输入
        input_data = {...}
        
        # 执行
        result = function(input_data)
        
        # 验证
        assert "field" in result
        assert result["field"] == expected_value**你的输出应该完全遵循这个格式！**### 技巧 3：后处理清理

即使有上述要求，LLM 可能还是输出额外内容，做个后处理：

def clean_llm_code_output(text: str) -> str:
    """清理 LLM 输出，只保留代码"""
    
    # 移除常见的解释性文字
    lines_to_remove = [
        "以下是生成的测试代码",
        "这是测试代码",
        "Here is the test code",
        "根据分析",
    ]
    
    lines = text.split('\n')
    cleaned_lines = []
    
    for line in lines:
        # 跳过解释性文字
        if any(phrase in line for phrase in lines_to_remove):
            continue
        cleaned_lines.append(line)
    
    result = '\n'.join(cleaned_lines)
    
    # 提取  ... ``` 之间的内容
    result = extract_code_from_markdown(result)
    
    return result---

## 🎯 最终推荐方案

结合你的需求，我建议使用 **方案 1（两阶段）+ 后处理清理**：

def generate_e2e_test(code: str, output_file: str):
    """生成端到端测试"""
    
    # 阶段 1：分析
    analysis = analyze_code(code)
    print("📋 分析完成")
    
    # 阶段 2：生成代码
    raw_code = generate_test_code(analysis)
    
    # 清理输出
    clean_code = clean_llm_code_output(raw_code)
    
    # 写入文件
    with open(output_file, "w", encoding="utf-8") as f:
        f.write(clean_code)
    
    print(f"✅ 测试已生成: {output_file}")
    
    return {
        "analysis": analysis,
        "code": clean_code
    }这样你就能得到干净的、可直接使用的测试代码了！🎉
```

```plaintext
---TEST_CODE_END---

---USAGE_START---
[使用说明]
---USAGE_END---

**关键**：TEST_CODE 部分只能包含 Python 代码，不要有任何解释文字。
```

```python
def parse_llm_output(output: str) -> dict:
    """解析 LLM 输出"""
    
    # 提取分析部分
    analysis = extract_between(output, "---ANALYSIS_START---", "---ANALYSIS_END---")
    
    # 提取测试代码（去除 markdown 代码块标记）
    code_section = extract_between(output, "---TEST_CODE_START---", "---TEST_CODE_END---")
    test_code = code_section.replace("", "").replace("```", "").strip()
    
    # 提取使用说明
    usage = extract_between(output, "---USAGE_START---", "---USAGE_END---")
    
    return {
        "analysis": analysis.strip(),
        "test_code": test_code,
        "usage": usage.strip()
    }

def extract_between(text: str, start_marker: str, end_marker: str) -> str:
    """提取两个标记之间的内容"""
    start = text.find(start_marker)
    end = text.find(end_marker)
    if start == -1 or end == -1:
        return ""
    return text[start + len(start_marker):end]

# 使用
result = parse_llm_output(llm_response)
with open("test_e2e.py", "w", encoding="utf-8") as f:
    f.write(result["test_code"])
```

```markdown
# 任务：生成端到端测试代码

## 输入代码
{YOUR_CODE}

```

```python
"""端到端测试文件"""
import pytest
...
```

```plaintext

**优点**：
- ✅ 最简单
- ✅ 输出最干净
- ⚠️ 缺少分析过程（不知道 LLM 的思考逻辑）

---

## 🏆 我的推荐：方案 1（两阶段）

**为什么？**

1. **可控性最强**：你可以审查分析结果，确保 LLM 理解正确
2. **输出最干净**：第二次只输出代码
3. **可复现**：如果代码有问题，可以调整分析结果重新生成
4. **成本优化**：分析一次，可以多次生成代码（调整提示词）

**实际工作流程**：

# 步骤 1：分析功能
analysis_prompt = """
# 任务：分析功能
{CODE}

输出 JSON 格式的分析结果，包括：功能描述、输入输出、业务规则、测试场景
"""

analysis_result = call_llm(analysis_prompt)
print("📋 分析结果：")
print(analysis_result)

# 🔍 人工审查分析结果，确认无误

# 步骤 2：生成测试代码
code_gen_prompt = f"""
# 任务：生成测试代码

## 分析结果
{analysis_result}

## 要求
根据上述分析生成完整的 pytest 测试代码。
**只输出 Python 代码，不要任何解释！**

输出格式：
# 测试代码从这里开始
import pytest
...
```
"""

test_code = call_llm(code_gen_prompt)

# 提取代码（去除 markdown 标记）
clean_code = extract_code_from_markdown(test_code)

# 写入文件
with open("tests/e2e/test_generated.py", "w", encoding="utf-8") as f:
    f.write(clean_code)

print("✅ 测试代码已生成到: tests/e2e/test_generated.py")
```

```python
def extract_code_from_markdown(text: str) -> str:
    """从 markdown 代码块中提取代码"""
    # 匹配 ... 
```

```plaintext

---

## 💡 额外技巧

### 技巧 1：明确告诉 LLM 输出会被直接写入文件

**重要提示**：
你的输出将被直接写入 Python 文件执行，不会经过人工编辑。
因此必须确保：
1. 输出的是纯 Python 代码
2. 不包含任何解释文字（注释除外）
3. 代码可以直接运行

```

```markdown
## 输出格式示例（你的输出应该和这个完全一样的格式）

"""端到端测试 - 示例功能"""
import pytest
import json

@pytest.mark.e2e
class TestExampleE2E:
    def test_main_scenario(self):
        """测试主要场景"""
        # 准备输入
        input_data = {...}
        
        # 执行
        result = function(input_data)
        
        # 验证
        assert "field" in result
        assert result["field"] == expected_value

```

```plaintext

### 技巧 3：后处理清理

即使有上述要求，LLM 可能还是输出额外内容，做个后处理：

def clean_llm_code_output(text: str) -> str:
    """清理 LLM 输出，只保留代码"""
    
    # 移除常见的解释性文字
    lines_to_remove = [
        "以下是生成的测试代码",
        "这是测试代码",
        "Here is the test code",
        "根据分析",
    ]
    
    lines = text.split('\n')
    cleaned_lines = []
    
    for line in lines:
        # 跳过解释性文字
        if any(phrase in line for phrase in lines_to_remove):
            continue
        cleaned_lines.append(line)
    
    result = '\n'.join(cleaned_lines)
    
    # 提取 ... ``` 之间的内容
    result = extract_code_from_markdown(result)
    
    return result
```

```python
def generate_e2e_test(code: str, output_file: str):
    """生成端到端测试"""
    
    # 阶段 1：分析
    analysis = analyze_code(code)
    print("📋 分析完成")
    
    # 阶段 2：生成代码
    raw_code = generate_test_code(analysis)
    
    # 清理输出
    clean_code = clean_llm_code_output(raw_code)
    
    # 写入文件
    with open(output_file, "w", encoding="utf-8") as f:
        f.write(clean_code)
    
    print(f"✅ 测试已生成: {output_file}")
    
    return {
        "analysis": analysis,
        "code": clean_code
    }
```

---

**User**

我理解了,你的意思是调用两次是吧？先是引导模型去分析，分析要生成哪些内容以及目的什么，测试的逻辑是什么以及确定测试场景。然后呢第二个提示词在是让模型根据前面的分析结果去特定的文件里面生成e2e测试代码，是吗？

---

**Cursor**

完全正确！你理解得非常准确！👍

让我用图示和具体例子帮你确认一下整个流程：

## 📊 完整流程图

```
┌─────────────────────────────────────────────────────────────┐
│  输入：源代码 / 规格文档                                      │
└─────────────────┬───────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────────────────────────┐
│  第一次 LLM 调用 - 分析阶段                                   │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ Prompt 1: "请分析这段代码的核心功能、业务规则、测试场景" │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────┬───────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────────────────────────┐
│  LLM 输出 1 - 分析结果（JSON 或纯文本）                       │
│  {                                                           │
│    "功能": "根据面试评估生成报告",                           │
│    "业务规则": ["评分0-100", "高分推荐录用"],                │
│    "测试场景": [                                             │
│      {"场景": "高级工程师匹配", "期望": "高分+推荐"},        │
│      {"场景": "初级工程师不匹配", "期望": "低分+不推荐"}     │
│    ]                                                         │
│  }                                                           │
└─────────────────┬───────────────────────────────────────────┘
                  ↓
        【可选：人工审查分析结果】
                  ↓
┌─────────────────────────────────────────────────────────────┐
│  第二次 LLM 调用 - 代码生成阶段                               │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ Prompt 2: "根据上述分析，生成纯测试代码，不要解释"      │  │
│  │ 包含：第一次的分析结果                                  │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────┬───────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────────────────────────┐
│  LLM 输出 2 - 纯测试代码                                     │
│  ```python                                                   │
│  import pytest                                               │
│  @pytest.mark.e2e                                           │
│  class TestEvaluationE2E:                                   │
│      def test_senior_match(self): ...                       │
│  ```                                                         │
└─────────────────┬───────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────────────────────────┐
│  后处理：清理代码（去除 markdown 标记等）                     │
└─────────────────┬───────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────────────────────────┐
│  写入文件：tests/e2e/test_generated.py                       │
└─────────────────────────────────────────────────────────────┘
```

## 💻 具体代码实现示例

```python
def generate_e2e_test_two_stage(source_code: str, target_file: str):
    """
    两阶段生成端到端测试
    
    Args:
        source_code: 要测试的源代码
        target_file: 测试文件路径，如 "tests/e2e/test_evaluation_report_e2e.py"
    """
    
    # ============ 第一阶段：分析 ============
    print("🔍 第一阶段：分析代码...")
    
    analysis_prompt = f"""
# 任务：分析代码并设计测试场景

## 输入代码
```python
{source_code}
```

## 要求
请分析并以 JSON 格式输出：

```json
{{
  "功能描述": "一句话概括这个功能",
  "输入参数": [
    {{"名称": "参数名", "类型": "类型", "说明": "用途"}}
  ],
  "输出结构": {{
    "必需字段": ["field1", "field2"],
    "字段说明": {{"field1": "说明"}}
  }},
  "业务规则": [
    "规则1：评分必须在0-100之间",
    "规则2：高分时结论应包含推荐关键词"
  ],
  "测试场景": [
    {{
      "场景名": "高级工程师-匹配度高",
      "优先级": "P0",
      "输入描述": "3年以上经验，技能匹配JD",
      "预期结果": "评分70-100，结论推荐录用"
    }}
  ]
}}
```

只输出 JSON，不要其他文字。
"""
    
    # 调用 LLM（第一次）
    analysis_result = call_llm(analysis_prompt, temperature=0.3)
    
    print("✅ 分析完成：")
    print(analysis_result[:200] + "...")  # 显示部分结果
    
    # 可选：保存分析结果供审查
    with open(target_file.replace(".py", "_analysis.json"), "w", encoding="utf-8") as f:
        f.write(analysis_result)
    
    
    # ============ 第二阶段：生成代码 ============
    print("\n⚙️ 第二阶段：生成测试代码...")
    
    code_generation_prompt = f"""
# 任务：生成端到端测试代码

## 功能分析结果
{analysis_result}

## 生成要求
根据上述分析结果，生成完整的 pytest 端到端测试代码。

### 代码要求
1. 使用 @pytest.mark.e2e 和 @pytest.mark.asyncio（如需要）
2. 为每个测试场景生成一个测试方法
3. 包含详细的输入验证、输出验证、业务逻辑验证
4. 每个 assert 都要有清晰的错误信息
5. 使用 print 输出测试进度

### 验证要点（必须包含）
- ✅ 结构验证：检查必需字段
- ✅ 类型验证：检查数据类型
- ✅ 范围验证：检查数值范围
- ✅ 内容验证：检查内容非空且有意义
- ✅ 业务验证：检查业务规则

### 输出格式
**重要：只输出 Python 代码，不要任何解释文字！**
**你的输出将直接写入文件 {target_file}**

从以下格式开始输出：

```python
\"\"\"
端到端测试 - [功能名称]
自动生成的测试代码
\"\"\"
import pytest
import sys
import os
...
```

直接输出代码，不要输出 "以下是代码" 等说明！
"""
    
    # 调用 LLM（第二次）
    raw_code = call_llm(code_generation_prompt, temperature=0.2)
    
    print("✅ 代码生成完成")
    
    
    # ============ 第三阶段：清理和写入 ============
    print("\n🧹 第三阶段：清理代码...")
    
    # 清理 markdown 标记和多余文字
    clean_code = extract_and_clean_code(raw_code)
    
    # 写入目标文件
    with open(target_file, "w", encoding="utf-8") as f:
        f.write(clean_code)
    
    print(f"✅ 测试代码已写入: {target_file}")
    print(f"📝 分析结果已保存: {target_file.replace('.py', '_analysis.json')}")
    
    return {
        "analysis": analysis_result,
        "test_code": clean_code,
        "test_file": target_file
    }


def extract_and_clean_code(llm_output: str) -> str:
    """提取并清理 LLM 输出的代码"""
    import re
    
    # 方法1：提取 ```python ... ``` 之间的内容
    pattern = r'```python\s*\n(.*?)```'
    matches = re.findall(pattern, llm_output, re.DOTALL)
    
    if matches:
        code = matches[0]
    else:
        # 方法2：如果没有 markdown 标记，尝试智能清理
        code = llm_output
        
        # 移除常见的说明文字
        lines_to_skip = [
            "以下是", "这是", "代码如下", "生成的代码",
            "Here is", "The code", "Following is"
        ]
        
        lines = code.split('\n')
        cleaned_lines = []
        
        for line in lines:
            # 跳过说明性文字
            if any(phrase in line for phrase in lines_to_skip) and not line.strip().startswith('#'):
                continue
            cleaned_lines.append(line)
        
        code = '\n'.join(cleaned_lines)
    
    # 清理前后空白
    code = code.strip()
    
    # 确保以 docstring 或 import 开始
    if not (code.startswith('"""') or code.startswith('import') or code.startswith('#')):
        # 找到第一个有效的 Python 代码行
        for i, line in enumerate(code.split('\n')):
            if line.strip() and (line.startswith('"""') or line.startswith('import') or line.startswith('#')):
                code = '\n'.join(code.split('\n')[i:])
                break
    
    return code


def call_llm(prompt: str, temperature: float = 0.3) -> str:
    """调用 LLM API（示例）"""
    from openai import OpenAI
    
    client = OpenAI(api_key="your-api-key")
    
    response = client.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": prompt}],
        temperature=temperature
    )
    
    return response.choices[0].message.content


# ============ 使用示例 ============
if __name__ == "__main__":
    
    # 读取要测试的源代码
    with open("code/service/AIInterviewAssessor.py", "r", encoding="utf-8") as f:
        source_code = f.read()
    
    # 生成测试
    result = generate_e2e_test_two_stage(
        source_code=source_code,
        target_file="tests/e2e/evaluation_report/test_evaluation_report_e2e_generated.py"
    )
    
    print("\n" + "="*60)
    print("🎉 测试生成完成！")
    print("="*60)
    print(f"📄 测试文件: {result['test_file']}")
    print(f"📊 分析结果: {result['test_file'].replace('.py', '_analysis.json')}")
    print("\n运行测试:")
    print(f"  pytest {result['test_file']} -v -s")
```

## 🎯 关键点总结

### ✅ 你的理解完全正确：

1. **第一次调用**：
   - 目的：分析和设计
   - 输入：源代码/规格
   - 输出：分析结果（JSON 或文本）
   - 作用：让 LLM 深入思考，确定测试策略

2. **第二次调用**：
   - 目的：生成代码
   - 输入：第一次的分析结果
   - 输出：纯测试代码
   - 作用：基于清晰的分析，生成结构化的测试

3. **为什么这样好**：
   - 🧠 **分离思考和执行**：分析和编码分开，质量更高
   - 🎯 **输出更纯净**：第二次只输出代码，易于直接写文件
   - 🔍 **可审查**：可以检查分析结果，确保理解正确
   - 🔄 **可迭代**：分析一次，可以调整第二个 prompt 多次生成

### 💡 额外提示

如果你想更自动化，可以加个验证步骤：

```python
# 在写入文件前，验证生成的代码
def validate_generated_code(code: str) -> bool:
    """验证生成的代码是否合法"""
    try:
        # 检查是否是合法的 Python 语法
        compile(code, '<string>', 'exec')
        
        # 检查必要的内容
        assert '@pytest.mark.e2e' in code, "缺少 @pytest.mark.e2e 标记"
        assert 'import pytest' in code, "缺少 pytest 导入"
        assert 'assert' in code, "缺少断言"
        
        return True
    except Exception as e:
        print(f"❌ 代码验证失败: {e}")
        return False

# 使用
if validate_generated_code(clean_code):
    with open(target_file, "w", encoding="utf-8") as f:
        f.write(clean_code)
else:
    print("⚠️ 生成的代码有问题，请检查")
```

你的理解完全准确！这就是最佳实践的工作流程。🎉

```plaintext
┌─────────────────────────────────────────────────────────────┐
│  输入：源代码 / 规格文档                                      │
└─────────────────┬───────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────────────────────────┐
│  第一次 LLM 调用 - 分析阶段                                   │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ Prompt 1: "请分析这段代码的核心功能、业务规则、测试场景" │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────┬───────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────────────────────────┐
│  LLM 输出 1 - 分析结果（JSON 或纯文本）                       │
│  {                                                           │
│    "功能": "根据面试评估生成报告",                           │
│    "业务规则": ["评分0-100", "高分推荐录用"],                │
│    "测试场景": [                                             │
│      {"场景": "高级工程师匹配", "期望": "高分+推荐"},        │
│      {"场景": "初级工程师不匹配", "期望": "低分+不推荐"}     │
│    ]                                                         │
│  }                                                           │
└─────────────────┬───────────────────────────────────────────┘
                  ↓
        【可选：人工审查分析结果】
                  ↓
┌─────────────────────────────────────────────────────────────┐
│  第二次 LLM 调用 - 代码生成阶段                               │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ Prompt 2: "根据上述分析，生成纯测试代码，不要解释"      │  │
│  │ 包含：第一次的分析结果                                  │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────┬───────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────────────────────────┐
│  LLM 输出 2 - 纯测试代码                                     │
│                                                   │
│  import pytest                                               │
│  @pytest.mark.e2e                                           │
│  class TestEvaluationE2E:                                   │
│      def test_senior_match(self): ...                       │
│  ```                                                         │
└─────────────────┬───────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────────────────────────┐
│  后处理：清理代码（去除 markdown 标记等）                     │
└─────────────────┬───────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────────────────────────┐
│  写入文件：tests/e2e/test_generated.py                       │
└─────────────────────────────────────────────────────────────┘## 💻 具体代码实现示例

def generate_e2e_test_two_stage(source_code: str, target_file: str):
    """
    两阶段生成端到端测试
    
    Args:
        source_code: 要测试的源代码
        target_file: 测试文件路径，如 "tests/e2e/test_evaluation_report_e2e.py"
    """
    
    # ============ 第一阶段：分析 ============
    print("🔍 第一阶段：分析代码...")
    
    analysis_prompt = f"""
# 任务：分析代码并设计测试场景

## 输入代码
{source_code}## 要求
请分析并以 JSON 格式输出：

{{
  "功能描述": "一句话概括这个功能",
  "输入参数": [
    {{"名称": "参数名", "类型": "类型", "说明": "用途"}}
  ],
  "输出结构": {{
    "必需字段": ["field1", "field2"],
    "字段说明": {{"field1": "说明"}}
  }},
  "业务规则": [
    "规则1：评分必须在0-100之间",
    "规则2：高分时结论应包含推荐关键词"
  ],
  "测试场景": [
    {{
      "场景名": "高级工程师-匹配度高",
      "优先级": "P0",
      "输入描述": "3年以上经验，技能匹配JD",
      "预期结果": "评分70-100，结论推荐录用"
    }}
  ]
}}只输出 JSON，不要其他文字。
"""
    
    # 调用 LLM（第一次）
    analysis_result = call_llm(analysis_prompt, temperature=0.3)
    
    print("✅ 分析完成：")
    print(analysis_result[:200] + "...")  # 显示部分结果
    
    # 可选：保存分析结果供审查
    with open(target_file.replace(".py", "_analysis.json"), "w", encoding="utf-8") as f:
        f.write(analysis_result)
    
    
    # ============ 第二阶段：生成代码 ============
    print("\n⚙️ 第二阶段：生成测试代码...")
    
    code_generation_prompt = f"""
# 任务：生成端到端测试代码

## 功能分析结果
{analysis_result}

## 生成要求
根据上述分析结果，生成完整的 pytest 端到端测试代码。

### 代码要求
1. 使用 @pytest.mark.e2e 和 @pytest.mark.asyncio（如需要）
2. 为每个测试场景生成一个测试方法
3. 包含详细的输入验证、输出验证、业务逻辑验证
4. 每个 assert 都要有清晰的错误信息
5. 使用 print 输出测试进度

### 验证要点（必须包含）
- ✅ 结构验证：检查必需字段
- ✅ 类型验证：检查数据类型
- ✅ 范围验证：检查数值范围
- ✅ 内容验证：检查内容非空且有意义
- ✅ 业务验证：检查业务规则

### 输出格式
**重要：只输出 Python 代码，不要任何解释文字！**
**你的输出将直接写入文件 {target_file}**

从以下格式开始输出：

\"\"\"
端到端测试 - [功能名称]
自动生成的测试代码
\"\"\"
import pytest
import sys
import os
...直接输出代码，不要输出 "以下是代码" 等说明！
"""
    
    # 调用 LLM（第二次）
    raw_code = call_llm(code_generation_prompt, temperature=0.2)
    
    print("✅ 代码生成完成")
    
    
    # ============ 第三阶段：清理和写入 ============
    print("\n🧹 第三阶段：清理代码...")
    
    # 清理 markdown 标记和多余文字
    clean_code = extract_and_clean_code(raw_code)
    
    # 写入目标文件
    with open(target_file, "w", encoding="utf-8") as f:
        f.write(clean_code)
    
    print(f"✅ 测试代码已写入: {target_file}")
    print(f"📝 分析结果已保存: {target_file.replace('.py', '_analysis.json')}")
    
    return {
        "analysis": analysis_result,
        "test_code": clean_code,
        "test_file": target_file
    }


def extract_and_clean_code(llm_output: str) -> str:
    """提取并清理 LLM 输出的代码"""
    import re
    
    # 方法1：提取 ... ``` 之间的内容
    pattern = r'\s*\n(.*?)```'
    matches = re.findall(pattern, llm_output, re.DOTALL)
    
    if matches:
        code = matches[0]
    else:
        # 方法2：如果没有 markdown 标记，尝试智能清理
        code = llm_output
        
        # 移除常见的说明文字
        lines_to_skip = [
            "以下是", "这是", "代码如下", "生成的代码",
            "Here is", "The code", "Following is"
        ]
        
        lines = code.split('\n')
        cleaned_lines = []
        
        for line in lines:
            # 跳过说明性文字
            if any(phrase in line for phrase in lines_to_skip) and not line.strip().startswith('#'):
                continue
            cleaned_lines.append(line)
        
        code = '\n'.join(cleaned_lines)
    
    # 清理前后空白
    code = code.strip()
    
    # 确保以 docstring 或 import 开始
    if not (code.startswith('"""') or code.startswith('import') or code.startswith('#')):
        # 找到第一个有效的 Python 代码行
        for i, line in enumerate(code.split('\n')):
            if line.strip() and (line.startswith('"""') or line.startswith('import') or line.startswith('#')):
                code = '\n'.join(code.split('\n')[i:])
                break
    
    return code


def call_llm(prompt: str, temperature: float = 0.3) -> str:
    """调用 LLM API（示例）"""
    from openai import OpenAI
    
    client = OpenAI(api_key="your-api-key")
    
    response = client.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": prompt}],
        temperature=temperature
    )
    
    return response.choices[0].message.content


# ============ 使用示例 ============
if __name__ == "__main__":
    
    # 读取要测试的源代码
    with open("code/service/AIInterviewAssessor.py", "r", encoding="utf-8") as f:
        source_code = f.read()
    
    # 生成测试
    result = generate_e2e_test_two_stage(
        source_code=source_code,
        target_file="tests/e2e/evaluation_report/test_evaluation_report_e2e_generated.py"
    )
    
    print("\n" + "="*60)
    print("🎉 测试生成完成！")
    print("="*60)
    print(f"📄 测试文件: {result['test_file']}")
    print(f"📊 分析结果: {result['test_file'].replace('.py', '_analysis.json')}")
    print("\n运行测试:")
    print(f"  pytest {result['test_file']} -v -s")## 🎯 关键点总结

### ✅ 你的理解完全正确：

1. **第一次调用**：
   - 目的：分析和设计
   - 输入：源代码/规格
   - 输出：分析结果（JSON 或文本）
   - 作用：让 LLM 深入思考，确定测试策略

2. **第二次调用**：
   - 目的：生成代码
   - 输入：第一次的分析结果
   - 输出：纯测试代码
   - 作用：基于清晰的分析，生成结构化的测试

3. **为什么这样好**：
   - 🧠 **分离思考和执行**：分析和编码分开，质量更高
   - 🎯 **输出更纯净**：第二次只输出代码，易于直接写文件
   - 🔍 **可审查**：可以检查分析结果，确保理解正确
   - 🔄 **可迭代**：分析一次，可以调整第二个 prompt 多次生成

### 💡 额外提示

如果你想更自动化，可以加个验证步骤：

# 在写入文件前，验证生成的代码
def validate_generated_code(code: str) -> bool:
    """验证生成的代码是否合法"""
    try:
        # 检查是否是合法的 Python 语法
        compile(code, '<string>', 'exec')
        
        # 检查必要的内容
        assert '@pytest.mark.e2e' in code, "缺少 @pytest.mark.e2e 标记"
        assert 'import pytest' in code, "缺少 pytest 导入"
        assert 'assert' in code, "缺少断言"
        
        return True
    except Exception as e:
        print(f"❌ 代码验证失败: {e}")
        return False

# 使用
if validate_generated_code(clean_code):
    with open(target_file, "w", encoding="utf-8") as f:
        f.write(clean_code)
else:
    print("⚠️ 生成的代码有问题，请检查")你的理解完全准确！这就是最佳实践的工作流程。🎉
```

```plaintext
└─────────────────┬───────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────────────────────────┐
│  后处理：清理代码（去除 markdown 标记等）                     │
└─────────────────┬───────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────────────────────────┐
│  写入文件：tests/e2e/test_generated.py                       │
└─────────────────────────────────────────────────────────────┘
```

```python
def generate_e2e_test_two_stage(source_code: str, target_file: str):
    """
    两阶段生成端到端测试
    
    Args:
        source_code: 要测试的源代码
        target_file: 测试文件路径，如 "tests/e2e/test_evaluation_report_e2e.py"
    """
    
    # ============ 第一阶段：分析 ============
    print("🔍 第一阶段：分析代码...")
    
    analysis_prompt = f"""
# 任务：分析代码并设计测试场景

## 输入代码
{source_code}

```

```json
{{
  "功能描述": "一句话概括这个功能",
  "输入参数": [
    {{"名称": "参数名", "类型": "类型", "说明": "用途"}}
  ],
  "输出结构": {{
    "必需字段": ["field1", "field2"],
    "字段说明": {{"field1": "说明"}}
  }},
  "业务规则": [
    "规则1：评分必须在0-100之间",
    "规则2：高分时结论应包含推荐关键词"
  ],
  "测试场景": [
    {{
      "场景名": "高级工程师-匹配度高",
      "优先级": "P0",
      "输入描述": "3年以上经验，技能匹配JD",
      "预期结果": "评分70-100，结论推荐录用"
    }}
  ]
}}
```

```python
\"\"\"
端到端测试 - [功能名称]
自动生成的测试代码
\"\"\"
import pytest
import sys
import os
...
```

```plaintext

## 🎯 关键点总结

### ✅ 你的理解完全正确：

1. **第一次调用**：
   - 目的：分析和设计
   - 输入：源代码/规格
   - 输出：分析结果（JSON 或文本）
   - 作用：让 LLM 深入思考，确定测试策略

2. **第二次调用**：
   - 目的：生成代码
   - 输入：第一次的分析结果
   - 输出：纯测试代码
   - 作用：基于清晰的分析，生成结构化的测试

3. **为什么这样好**：
   - 🧠 **分离思考和执行**：分析和编码分开，质量更高
   - 🎯 **输出更纯净**：第二次只输出代码，易于直接写文件
   - 🔍 **可审查**：可以检查分析结果，确保理解正确
   - 🔄 **可迭代**：分析一次，可以调整第二个 prompt 多次生成

### 💡 额外提示

如果你想更自动化，可以加个验证步骤：

# 在写入文件前，验证生成的代码
def validate_generated_code(code: str) -> bool:
    """验证生成的代码是否合法"""
    try:
        # 检查是否是合法的 Python 语法
        compile(code, '<string>', 'exec')
        
        # 检查必要的内容
        assert '@pytest.mark.e2e' in code, "缺少 @pytest.mark.e2e 标记"
        assert 'import pytest' in code, "缺少 pytest 导入"
        assert 'assert' in code, "缺少断言"
        
        return True
    except Exception as e:
        print(f"❌ 代码验证失败: {e}")
        return False

# 使用
if validate_generated_code(clean_code):
    with open(target_file, "w", encoding="utf-8") as f:
        f.write(clean_code)
else:
    print("⚠️ 生成的代码有问题，请检查")

```

