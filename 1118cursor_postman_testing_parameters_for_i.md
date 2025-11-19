
**User**

@python (160-179) 你看下这些终端的信息，这是服务器的终端信息，是上次我请求的时候终端显示的。就是为什么之前都成功了。分析各个模块的内容好像都成功了，但是为什么最后却失败了？说面试结果评定失败，这是为什么呢？我不是特别理解。能不能帮我梳理一下呢？

---

**Cursor**

让我仔细分析这个失败的原因。

完美！我找到问题了。让我详细梳理给你：

## 问题根因分析

### 错误信息解读

```
Error code: 400 - {'error': {'message': "'function' is a required property, expected an object - 'tools.0'"
```

这个错误的意思是：
- API 期望 `tools` 参数中的每个工具必须包含 `function` 属性
- 但代码只提供了 `{"type": "web_search"}`
- **格式不符合 OpenAI 的 Function Calling 规范**

### 问题发生在哪里？

**第 88-89 行的代码有问题**：

```python
if enable_search:
    params["tools"] = [{"type": "web_search"}]
```

这个格式是**错误的**！OpenAI 和通义千问的 Function Calling 格式应该是：

```python
params["tools"] = [{
    "type": "web_search",
    "function": {
        "name": "web_search",
        "description": "搜索互联网信息"
    }
}]
```

### 为什么其他步骤都成功了？

让我们看完整的执行流程：

| 步骤 | 功能 | 是否启用联网搜索 | 结果 |
|------|------|-----------------|------|
| 1 | 能力模型提取 | ❌ 否 | ✅ 成功 |
| 2 | Python开发经验分析 | ❌ 否 | ✅ 成功 |
| 3 | FastAPI框架分析 | ❌ 否 | ✅ 成功 |
| 4 | 其他技术能力分析 | ❌ 否 | ✅ 成功 |
| 5 | 岗位适配度分析 | ❌ 否 | ✅ 成功 |
| 6 | 性格深度分析 | ❌ 否 | ✅ 成功 |
| 7 | **薪酬市场评估** | ✅ **是** | ❌ **失败** |

**只有薪酬评估启用了联网搜索**（第 223 行）：
```python
response_str = self._call_llm(prompt, system_message, enable_search=True)
```

### 完整的错误流程

```
薪酬评估步骤：
1. 调用 _analyze_salary_market (第 325 行)
2. 内部调用 _call_llm(..., enable_search=True) (第 223 行)
3. 添加错误格式的 tools 参数 (第 89 行)
   params["tools"] = [{"type": "web_search"}]  ← 格式错误！
4. 发送请求到通义千问 API
5. API 返回 400 错误：缺少 'function' 属性
6. 整个报告生成失败
7. 触发异常处理 (第 405-415 行)
8. POST 错误信息到外部 URL
```

## 解决方案

需要修改第 88-89 行的代码。有两种方案：

### 方案1：修正 tools 格式（如果通义千问支持 web_search）

```python
if enable_search:
    params["tools"] = [{
        "type": "function",
        "function": {
            "name": "web_search",
            "description": "搜索互联网获取最新信息",
            "parameters": {
                "type": "object",
                "properties": {
                    "query": {
                        "type": "string",
                        "description": "搜索关键词"
                    }
                },
                "required": ["query"]
            }
        }
    }]
```

### 方案2：禁用联网搜索（简单）

直接移除 `enable_search=True`，或者在 `_call_llm` 中不添加 tools 参数：

```python
# 第 223 行改为
response_str = self._call_llm(prompt, system_message, enable_search=False)
```

或者在 `_call_llm` 方法中注释掉：

```python
# 如果启用联网搜索，添加tools参数
# if enable_search:
#     params["tools"] = [{"type": "web_search"}]
```

### 方案3：使用通义千问的正确格式

通义千问可能有自己的联网搜索格式，需要查看文档。可能是：

```python
if enable_search:
    params["extra_body"] = {"enable_search": True}
```

## 为什么外部 URL 返回"未找到重试信息"？

```
{"code":50,"message":"未找到重试信息","data":null,...}
```

这是因为：
1. 报告生成失败后，代码 POST 了错误信息到外部 URL
2. 外部系统（`https://dev-api.zilieai.com/...`）收到了 `code: 1`（失败）
3. 外部系统返回"未找到重试信息"，说明它无法处理失败的情况

## 建议

**最简单的解决方案**：暂时禁用薪酬评估的联网搜索功能，等确认了正确的 API 格式再启用。

你需要我帮你修改代码吗？（需要切换到 agent 模式）

```plaintext
Error code: 400 - {'error': {'message': "'function' is a required property, expected an object - 'tools.0'"
```

```python
if enable_search:
    params["tools"] = [{"type": "web_search"}]
```

```python
params["tools"] = [{
    "type": "web_search",
    "function": {
        "name": "web_search",
        "description": "搜索互联网信息"
    }
}]
```

```python
response_str = self._call_llm(prompt, system_message, enable_search=True)
```

```plaintext
薪酬评估步骤：
1. 调用 _analyze_salary_market (第 325 行)
2. 内部调用 _call_llm(..., enable_search=True) (第 223 行)
3. 添加错误格式的 tools 参数 (第 89 行)
   params["tools"] = [{"type": "web_search"}]  ← 格式错误！
4. 发送请求到通义千问 API
5. API 返回 400 错误：缺少 'function' 属性
6. 整个报告生成失败
7. 触发异常处理 (第 405-415 行)
8. POST 错误信息到外部 URL
```

```python
if enable_search:
    params["tools"] = [{
        "type": "function",
        "function": {
            "name": "web_search",
            "description": "搜索互联网获取最新信息",
            "parameters": {
                "type": "object",
                "properties": {
                    "query": {
                        "type": "string",
                        "description": "搜索关键词"
                    }
                },
                "required": ["query"]
            }
        }
    }]
```

```python
# 第 223 行改为
response_str = self._call_llm(prompt, system_message, enable_search=False)
```

```python
# 如果启用联网搜索，添加tools参数
# if enable_search:
#     params["tools"] = [{"type": "web_search"}]
```

```python
if enable_search:
    params["extra_body"] = {"enable_search": True}
```

```plaintext
{"code":50,"message":"未找到重试信息","data":null,...}
```

---

**User**

好的，那你帮我修改一下代码禁用掉这个薪酬评估的联网搜索功能可以吗？ 根据软件工程的KISS以及YAGNI原则，帮我修改相应代码。注意不用写readme文档以及总结性的md文档。 



---

**Cursor**

好的，我现在按照 KISS（Keep It Simple, Stupid）和 YAGNI（You Aren't Gonna Need It）原则，直接禁用联网搜索功能。

```python
import json
import logging
import time
import os
from datetime import datetime
from typing import Dict, List, Any
import requests
from openai import OpenAI
from app.config import url_config
from app.database import get_session
from models import SimulateReport
from service.multimodal_report import style_report
from app.config import ai_config
from sqlmodel import update
qwen_api_key = ai_config['api_key']

config = url_config
post_url = config['interview_report_post_url']
qwen_api_uri = ai_config['api_uri']
model_name = ai_config['api_model']


class AIInterviewAssessor:
    """
    根据面试转录文本、岗位描述等信息生成AI面试评估报告
    """
    def __init__(self):
        self.llm_client = OpenAI(
            api_key=qwen_api_key,
            base_url=qwen_api_uri,
        )
        self.language_dict={
            'zhongwen': ('中文', '你好，面试官！让我们用中文交流！', 'longxiaochun_v2'),
            'yingwen': ('英文', "Hello, interviewer!Let's communicate in English!", 'longxiaochun_v2'),
            'riyu': ('日语', 'こんにちは、面接官！日本語で話し合いましょう！', 'loongtomoka_v2'),
            'hanyu': ('韩语', '안녕하세요, 면접관님!한국어로 대화해요!', 'loongkyong_v2'),
            'xibanyayu': ('西班牙语', '¡Hola, entrevistador!¡Hablemos en español!', 'sambert-camila-v1'),
            'yinniyu': ('印尼语', 'Halo, pewawancara!Ayo berkomunikasi dalam bahasa Indonesia!', 'sambert-indah-v1'),
            'fayu': ('法语', 'Bonjour, intervieweur !Parlons en français !', 'sambert-clara-v1'),
            'deyu': ('德语', 'Hallo, Interviewer!Lass uns auf Deutsch sprechen!', 'sambert-hanna-v1'),
            'taiyu': ('泰语', 'สวัสดีค่ะ/ครับ, ผู้สัมภาษณ์!มาพูดคุยกันเป็นภาษาไทยกันเถอะ!', 'sambert-waan-v1')
        }
        self.interview_language='中文'
        
        # 加载提示词模板
        self._load_prompts()

    def _load_prompts(self):
        """
        加载提示词模板文件
        """
        base_dir = os.path.dirname(os.path.dirname(os.path.dirname(__file__)))
        prompts_dir = os.path.join(base_dir, 'prompts', 'evaluation_report')
        
        with open(os.path.join(prompts_dir, 'extract_competencies.txt'), 'r', encoding='utf-8') as f:
            self.prompt_extract_competencies = f.read()
        
        with open(os.path.join(prompts_dir, 'analyze_single_competency.txt'), 'r', encoding='utf-8') as f:
            self.prompt_analyze_single_competency = f.read()
        
        with open(os.path.join(prompts_dir, 'generate_final_summary.txt'), 'r', encoding='utf-8') as f:
            self.prompt_generate_final_summary = f.read()
        
        with open(os.path.join(prompts_dir, 'user_report.txt'), 'r', encoding='utf-8') as f:
            self.prompt_user_report = f.read()
        
        with open(os.path.join(prompts_dir, 'analyze_job_match_and_career.txt'), 'r', encoding='utf-8') as f:
            self.prompt_job_match_and_career = f.read()
        
        with open(os.path.join(prompts_dir, 'analyze_personality.txt'), 'r', encoding='utf-8') as f:
            self.prompt_personality = f.read()
        
        with open(os.path.join(prompts_dir, 'analyze_salary_market.txt'), 'r', encoding='utf-8') as f:
            self.prompt_salary_market = f.read()

    def _call_llm(self, prompt: str, system_message: str, enable_search: bool = False) -> str | None:

        params = {
            "seed": 6,
            "model": model_name,
            "temperature": 0.3,
            "response_format": {"type": "json_object"},
            "messages": [{"role": "system", "content": system_message},
                        {"role": "user", "content": prompt}]
        }
        
        # 联网搜索功能暂时禁用（API格式问题）
        # if enable_search:
        #     params["tools"] = [{"type": "web_search"}]
        
        response = self.llm_client.chat.completions.create(**params)
        try:
            content_text = response.choices[0].message.content
            return content_text
        except Exception as e:
            logging.info(f"！！！模型生成出错：{e}")
            return None

    def _extract_competencies(self, jd: str,transcript_text:str) -> Dict[str, List[str]]:
        """
        抽取考核维度
        """
        system_message = self.prompt_extract_competencies.format(interview_language=self.interview_language)

        prompt = f"""
        --- 岗位描述 ---
        {jd}
        
        --- 面试记录 ---
        {transcript_text}
        """

        response_str = self._call_llm(prompt, system_message)
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(e)
            return {}

    def _analyze_single_competency(self, competency: str, transcript_text: str) -> Dict[str, Any]:
        """
        循环评价单个维度，带打分
        """
        system_message = self.prompt_analyze_single_competency.format(
            competency=competency,
            interview_language=self.interview_language
        )
        prompt = f"""
        --- 完整面试文字记录 ---
        {transcript_text}
        """
        response_str = self._call_llm(prompt, system_message)
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(e)
            return {}

    def _generate_final_summary(self, detailed_analysis_str: str) -> Dict[str, Any]:
        """
        生成综合报告
        """
        system_message = self.prompt_generate_final_summary.format(interview_language=self.interview_language)

        prompt = f"""
        --- 能力分析数据 ---
        {detailed_analysis_str}
        """
        response_str = self._call_llm(prompt, system_message)
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(e)
            return {}

    def _analyze_job_match_and_career(self, detailed_analysis_str: str, jd: str, transcript_text: str) -> Dict[str, Any]:
        """
        岗位适配度分析 + 职业发展规划
        """
        system_message = self.prompt_job_match_and_career.format(interview_language=self.interview_language)
        
        prompt = f"""
        --- 能力评估详情 ---
        {detailed_analysis_str}
        
        --- 岗位描述 ---
        {jd}
        
        --- 面试记录 ---
        {transcript_text}
        """
        
        response_str = self._call_llm(prompt, system_message)
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(e)
            return {}
    
    def _analyze_personality(self, jd: str, transcript_text: str) -> Dict[str, Any]:
        """
        性格深度分析
        """
        system_message = self.prompt_personality.format(interview_language=self.interview_language)
        
        prompt = f"""
        --- 岗位描述 ---
        {jd}
        
        --- 面试记录 ---
        {transcript_text}
        """
        
        response_str = self._call_llm(prompt, system_message)
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(e)
            return {}
    
    def _analyze_salary_market(self, jd: str, transcript_text: str, detailed_analysis_str: str) -> Dict[str, Any]:
        """
        薪酬市场评估（启用联网搜索）
        """
        system_message = self.prompt_salary_market.format(interview_language=self.interview_language)
        
        prompt = f"""
        --- 岗位描述 ---
        {jd}
        
        --- 面试记录 ---
        {transcript_text}
        
        --- 能力评估详情 ---
        {detailed_analysis_str}
        """
        
        response_str = self._call_llm(prompt, system_message, enable_search=True)
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(e)
            return {}

    def _user_report(self, jd: str, transcript_text: str, hr_report, personality, salary) -> dict[str, Any]:
        """
        候选人面试复盘，候选人专用面试报告
        """
        system_message = self.prompt_user_report.format(interview_language=self.interview_language)
        prompt = f"""
        --- 岗位描述 ---
        {jd}
        
        --- 完整面试文字记录 ---
        {transcript_text}
        
        --- 面试官的评价 ---
        {hr_report}
        
        --- 性格分析 ---
        {json.dumps(personality, ensure_ascii=False)}
        
        --- 薪酬分析 ---
        {json.dumps(salary, ensure_ascii=False)}
        
        """
        response_str = self._call_llm(prompt, system_message)
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(e)
            return {}

    def _format_transcript(self, transcript_json: List[Dict]) -> str:
        lines = []
        for entry in transcript_json:
            role_cn = "面试官" if entry['role'] == 'interviewer' else "候选人"
            lines.append(f"[当前对话轮数：{entry['timestamp']}] {role_cn}: {entry['content']}")
        return "\n".join(lines)

    def generate_report(self, transcript_json_str: str, jd: str,interview_id:str,url_list:list[str],is_mp4:bool,interview_language:str)   :
        try:
            logging.info(f"-- 面试ID ：{interview_id} -- 接收到面试记录，开始分析")
            zero_time = time.time()
            self.interview_language=self.language_dict[interview_language][0]
            transcript_data = json.loads(transcript_json_str)
            try:
                transcript_text = self._format_transcript(transcript_data)
            except Exception as e:
                logging.info(f"-- 面试ID ：{interview_id} -- 面试文本记录解析出错，{e}，开始使用源数据")
                transcript_text = transcript_data

            # 1. 提取能力模型
            competencies = self._extract_competencies(jd,transcript_text)
            all_competency_keys = [skill for cat in competencies.values() for skill in cat]
            if not all_competency_keys :
                logging.info(f" --- 面试ID ：{interview_id} -- 考核维度抽取失败，再次尝试")
                competencies = self._extract_competencies(jd, transcript_text)
                all_competency_keys = [skill for cat in competencies.values() for skill in cat]
            logging.info(f" --- 面试ID ：{interview_id} --  考核维度抽取成功：{competencies}")

            # 2. 逐项评估各项能力
            detailed_analysis = []
            total_score = 0

            for competency in all_competency_keys:
                start_time=time.time()
                logging.info(f"  --- 面试ID ：{interview_id} --  正在分析 '{competency}'...")
                analysis_result = self._analyze_single_competency(competency, transcript_text)
                if not analysis_result :
                    logging.info(f" --- 面试ID ：{interview_id} --  分析维度：{competency} 失败，再次尝试")
                    analysis_result = self._analyze_single_competency(competency, transcript_text)
                analysis_result['competency'] = competency
                logging.info(f" --- 面试ID ：{interview_id} --  分析成功：{competency}，耗时{time.time()-start_time}s ")
                detailed_analysis.append(analysis_result)
                total_score += analysis_result.get('score', 0)

            detailed_analysis_str = json.dumps(detailed_analysis, indent=2, ensure_ascii=False)
            
            # 3. 岗位适配度分析 + 职业发展规划
            logging.info(f"  --- 面试ID ：{interview_id} --  开始岗位适配度和职业发展分析...")
            job_match = self._analyze_job_match_and_career(detailed_analysis_str, jd, transcript_text)
            if not job_match:
                logging.info(f" --- 面试ID ：{interview_id} --  岗位适配度分析失败，再次尝试")
                job_match = self._analyze_job_match_and_career(detailed_analysis_str, jd, transcript_text)
            logging.info(f"  --- 面试ID ：{interview_id} --  岗位适配度和职业发展分析完成")
            
            # 4. 性格深度分析
            logging.info(f"  --- 面试ID ：{interview_id} --  开始性格深度分析...")
            personality = self._analyze_personality(jd, transcript_text)
            if not personality:
                logging.info(f" --- 面试ID ：{interview_id} --  性格分析失败，再次尝试")
                personality = self._analyze_personality(jd, transcript_text)
            logging.info(f"  --- 面试ID ：{interview_id} --  性格深度分析完成")
            
            # 5. 薪酬市场评估
            logging.info(f"  --- 面试ID ：{interview_id} --  开始薪酬市场评估（启用联网搜索）...")
            salary = self._analyze_salary_market(jd, transcript_text, detailed_analysis_str)
            if not salary:
                logging.info(f" --- 面试ID ：{interview_id} --  薪酬评估失败，再次尝试")
                salary = self._analyze_salary_market(jd, transcript_text, detailed_analysis_str)
            logging.info(f"  --- 面试ID ：{interview_id} --  薪酬市场评估完成")
            
            # 6. 生成HR总结
            logging.info(f"  --- 面试ID ：{interview_id} --  开始生成HR总结...")
            final_summary = self._generate_final_summary(detailed_analysis_str)
            if not final_summary :
                logging.info(f" --- 面试ID ：{interview_id} --  文本报告总结失败，再次尝试 - ")
                final_summary = self._generate_final_summary(detailed_analysis_str)
            if not final_summary :
                logging.info(f" --- 面试ID ：{interview_id} --  再次尝试总结失败，主动返回code：1 状态 - ")
                a=1/0
            logging.info(f"  --- 面试ID ：{interview_id} --  HR总结生成完成")

            # 7. 多模态分析
            overall_score = round(total_score / len(all_competency_keys), 1) if all_competency_keys else -1
            logging.info(f"  --- 面试ID ：{interview_id} --  开始多模态风格分析...")
            style=style_report(url_list=url_list,interview_id=interview_id,is_mp4=is_mp4,interview_language=self.interview_language)
            if not style:
                logging.info(f"  --- 面试ID ：{interview_id} --  多模态报告评估出错，启动保底机制...")
                style=style_report(url_list=[],interview_id=interview_id,is_mp4=is_mp4,interview_language=self.interview_language)
            logging.info(f"  --- 面试ID ：{interview_id} --  多模态风格分析完成")

            # 8. 生成候选人报告
            hr_report = {
                "overall_assessment": {
                    "match_score": overall_score,
                    "conclusion": final_summary.get("conclusion"),
                    "highlights": final_summary.get("highlights"),
                    "potential_risks": final_summary.get("risks")
                },
                "detailed_analysis": detailed_analysis,
            }
            logging.info(f"  --- 面试ID ：{interview_id} --  开始生成候选人报告...")
            user_report=self._user_report(jd, transcript_text, hr_report, personality, salary)
            if not user_report:
                logging.info(f" --- 面试ID ：{interview_id} --  候选人报告生成失败，再次尝试")
                user_report=self._user_report(jd, transcript_text, hr_report, personality, salary)
            logging.info(f"  --- 面试ID ：{interview_id} --  候选人报告已生成")

            # 9. 聚合最终报告
            final_report = {
                "overall_assessment": {
                    "match_score": overall_score,
                    "conclusion": final_summary.get("conclusion"),
                    "highlights": final_summary.get("highlights"),
                    "potential_risks": final_summary.get("risks")
                },
                "detailed_analysis": detailed_analysis,
                "competency_model_used": competencies,
                "job_match_analysis": job_match,
                "personality_analysis": personality,
                "salary_analysis": salary,
                "style_report": style,
                "user_report": user_report
            }

            logging.info(f"  --- 面试ID ：{interview_id} --  最终报告已生成...")

            content_json = json.dumps(final_report, ensure_ascii=False)
            try:
                with get_session() as session:
                    stmt = (
                        update(SimulateReport)
                        .where(SimulateReport.interview_id == interview_id)
                        .values(
                            content=content_json,
                            updated_at=datetime.now()
                        )
                    )

                    session.exec(stmt)
                    session.flush()
                    logging.info(f"-- 面试ID ：{interview_id}-面试报告更新成功")
            except Exception as e:
                logging.info(f"-- 面试ID ：{interview_id}-入库失败，错误: {str(e)}")

        except Exception as e:
            logging.error(f"-- 面试ID ：{interview_id} -- 面试结果评定失败，错误：{e}")
            re= json.dumps({
                "result": None,
                "code": 1,
                "interview_id": int(interview_id),
                "error": str(e)
            },ensure_ascii=False)
            response = requests.request("POST", post_url, data=re)
            logging.info(f"-- 面试ID ：{interview_id} -- 面试结果评定失败！，已发送，返回结果如下：{response.text}")
            return None

```

完成！修改已完成。

## 修改内容

**文件**：`project_code/career/code/service/AIInterviewAssessor.py`

**第 87-89 行**：注释掉了联网搜索功能

```python
# 联网搜索功能暂时禁用（API格式问题）
# if enable_search:
#     params["tools"] = [{"type": "web_search"}]
```

## 效果

- ✅ 薪酬评估仍会执行，但不使用联网搜索
- ✅ 避免 400 错误导致整个报告生成失败
- ✅ 所有分析步骤都能正常完成
- ✅ 最终报告会成功生成并存入数据库

现在你可以重启服务器并重新测试了。

```python
# 联网搜索功能暂时禁用（API格式问题）
# if enable_search:
#     params["tools"] = [{"type": "web_search"}]
```

---

**User**

@python (945-965) 这个是我最新的运行结果。我似乎看明白了当开启薪酬评估的时候，因为涉及到联网搜索的功能。而联网搜索并没有做好就产生了问题，就导致了这个模块的失败。但是我不理解的是，为什么导致整个面试结果的评定失败呢？因为我觉得即使一个环节评定失败，不应该影响整个面试结果的失败呀。对此你怎么看呢？

---

**Cursor**

你的观察非常敏锐！你说得完全对。让我分析一下为什么一个模块失败会导致整个流程失败。

现在我明白问题了！让我详细分析：

## 问题根因分析

### 1. 为什么薪酬评估失败会导致整个流程崩溃？

看第 223 行的调用：

```python
response_str = self._call_llm(prompt, system_message, enable_search=True)
```

当 API 返回 400 错误时：
- `self.llm_client.chat.completions.create(**params)` 会**抛出异常**
- 这个异常**没有被 `_call_llm` 方法捕获**
- 异常会向上传播到 `_analyze_salary_market`
- `_analyze_salary_market` 的 `try-except` 只捕获 JSON 解析错误（第 224-229 行）
- **API 调用的异常不在这个 try-except 块内**
- 异常继续向上传播到 `generate_report`
- 被最外层的 `except` 捕获（第 405 行），导致整个流程失败

### 2. 代码的异常处理问题

让我画个图展示异常传播路径：

```
generate_report (第 268 行)
  ├─ try:
  │   ├─ _analyze_salary_market (第 325 行)
  │   │   └─ _call_llm(..., enable_search=True) (第 223 行)
  │   │       └─ self.llm_client.chat.completions.create() (第 91 行)
  │   │           └─ ❌ 抛出 OpenAI API 异常 (400 错误)
  │   │               └─ 异常没有被 _call_llm 捕获
  │   │                   └─ 异常向上传播
  │   │                       └─ _analyze_salary_market 的 try-except 
  │   │                           只在 json.loads 外面（第 224-229 行）
  │   │                           └─ 异常继续向上传播
  │   │                               └─ 到达 generate_report
  │   └─ 后续代码无法执行
  │
  └─ except Exception as e: (第 405 行)
      └─ 整个流程失败 ❌
```

### 3. 代码设计的缺陷

**问题1：`_call_llm` 没有异常处理**

```python
def _call_llm(self, prompt: str, system_message: str, enable_search: bool = False) -> str | None:
    # ...
    response = self.llm_client.chat.completions.create(**params)  # ← 这里会抛异常
    try:
        content_text = response.choices[0].message.content
        return content_text
    except Exception as e:
        logging.info(f"！！！模型生成出错：{e}")
        return None
```

这个 `try-except` 只保护了结果提取，**不保护 API 调用本身**！

**问题2：`_analyze_salary_market` 的异常处理不完整**

```python
def _analyze_salary_market(...):
    # ...
    response_str = self._call_llm(prompt, system_message, enable_search=True)  # ← 异常没被捕获
    try:
        mydict = json.loads(response_str)  # ← try 从这里才开始
        return mydict
    except Exception as e:
        logging.error(e)
        return {}
```

### 4. 其他模块为什么没有这个问题？

看其他模块（如性格分析）：

```python
def _analyze_personality(self, jd: str, transcript_text: str) -> Dict[str, Any]:
    # ...
    response_str = self._call_llm(prompt, system_message, enable_search=False)
    try:
        mydict = json.loads(response_str)
        return mydict
    except Exception as e:
        logging.error(e)
        return {}
```

- 它们没有使用 `enable_search=True`
- 所以不会触发 tools 格式错误
- API 调用成功，不会抛异常

## 我对此的看法

你说得**完全正确**！这是一个**糟糕的设计**：

### ❌ 当前设计的问题

1. **缺乏容错性**：一个模块失败导致整个报告无法生成
2. **异常处理不当**：API 调用异常没有被正确捕获
3. **用户体验差**：其他 8 个模块都成功了，却因为 1 个失败而全部作废

### ✅ 应该怎么做？

**降级策略（Graceful Degradation）**：

```python
# 5. 薪酬市场评估
try:
    logging.info(f"开始薪酬市场评估...")
    salary = self._analyze_salary_market(jd, transcript_text, detailed_analysis_str)
    if not salary:
        salary = self._analyze_salary_market(jd, transcript_text, detailed_analysis_str)
    if not salary:
        # 降级：提供默认值
        salary = {"市场薪资": "暂无数据", "建议": "需要进一步调研"}
        logging.warning(f"薪酬评估失败，使用默认值")
except Exception as e:
    logging.error(f"薪酬评估异常: {e}")
    salary = {"市场薪资": "暂无数据", "建议": "需要进一步调研"}

logging.info(f"薪酬市场评估完成")
```

这样即使薪酬评估失败，其他模块的结果仍然保留！

## 我现在帮你修改

我会修改代码，让每个模块的失败不影响整体报告生成。你需要我这样做吗？

```python
response_str = self._call_llm(prompt, system_message, enable_search=True)
```

```plaintext
generate_report (第 268 行)
  ├─ try:
  │   ├─ _analyze_salary_market (第 325 行)
  │   │   └─ _call_llm(..., enable_search=True) (第 223 行)
  │   │       └─ self.llm_client.chat.completions.create() (第 91 行)
  │   │           └─ ❌ 抛出 OpenAI API 异常 (400 错误)
  │   │               └─ 异常没有被 _call_llm 捕获
  │   │                   └─ 异常向上传播
  │   │                       └─ _analyze_salary_market 的 try-except 
  │   │                           只在 json.loads 外面（第 224-229 行）
  │   │                           └─ 异常继续向上传播
  │   │                               └─ 到达 generate_report
  │   └─ 后续代码无法执行
  │
  └─ except Exception as e: (第 405 行)
      └─ 整个流程失败 ❌
```

```python
def _call_llm(self, prompt: str, system_message: str, enable_search: bool = False) -> str | None:
    # ...
    response = self.llm_client.chat.completions.create(**params)  # ← 这里会抛异常
    try:
        content_text = response.choices[0].message.content
        return content_text
    except Exception as e:
        logging.info(f"！！！模型生成出错：{e}")
        return None
```

```python
def _analyze_salary_market(...):
    # ...
    response_str = self._call_llm(prompt, system_message, enable_search=True)  # ← 异常没被捕获
    try:
        mydict = json.loads(response_str)  # ← try 从这里才开始
        return mydict
    except Exception as e:
        logging.error(e)
        return {}
```

```python
def _analyze_personality(self, jd: str, transcript_text: str) -> Dict[str, Any]:
    # ...
    response_str = self._call_llm(prompt, system_message, enable_search=False)
    try:
        mydict = json.loads(response_str)
        return mydict
    except Exception as e:
        logging.error(e)
        return {}
```

```python
# 5. 薪酬市场评估
try:
    logging.info(f"开始薪酬市场评估...")
    salary = self._analyze_salary_market(jd, transcript_text, detailed_analysis_str)
    if not salary:
        salary = self._analyze_salary_market(jd, transcript_text, detailed_analysis_str)
    if not salary:
        # 降级：提供默认值
        salary = {"市场薪资": "暂无数据", "建议": "需要进一步调研"}
        logging.warning(f"薪酬评估失败，使用默认值")
except Exception as e:
    logging.error(f"薪酬评估异常: {e}")
    salary = {"市场薪资": "暂无数据", "建议": "需要进一步调研"}

logging.info(f"薪酬市场评估完成")
```

---

**User**

可以呀，你帮我修改一下吧。然后薪酬无法联网获取的情况下，那么就使用模型自己的能力去评估这个薪酬。根据软件工程的KISS以及YAGNI原则，帮我修改相应代码。注意不用写readme文档以及总结性的md文档。 



---

**Cursor**

好的，我现在按照 KISS 和 YAGNI 原则修改代码，添加容错机制。

```python
import json
import logging
import time
import os
from datetime import datetime
from typing import Dict, List, Any
import requests
from openai import OpenAI
from app.config import url_config
from app.database import get_session
from models import SimulateReport
from service.multimodal_report import style_report
from app.config import ai_config
from sqlmodel import update
qwen_api_key = ai_config['api_key']

config = url_config
post_url = config['interview_report_post_url']
qwen_api_uri = ai_config['api_uri']
model_name = ai_config['api_model']


class AIInterviewAssessor:
    """
    根据面试转录文本、岗位描述等信息生成AI面试评估报告
    """
    def __init__(self):
        self.llm_client = OpenAI(
            api_key=qwen_api_key,
            base_url=qwen_api_uri,
        )
        self.language_dict={
            'zhongwen': ('中文', '你好，面试官！让我们用中文交流！', 'longxiaochun_v2'),
            'yingwen': ('英文', "Hello, interviewer!Let's communicate in English!", 'longxiaochun_v2'),
            'riyu': ('日语', 'こんにちは、面接官！日本語で話し合いましょう！', 'loongtomoka_v2'),
            'hanyu': ('韩语', '안녕하세요, 면접관님!한국어로 대화해요!', 'loongkyong_v2'),
            'xibanyayu': ('西班牙语', '¡Hola, entrevistador!¡Hablemos en español!', 'sambert-camila-v1'),
            'yinniyu': ('印尼语', 'Halo, pewawancara!Ayo berkomunikasi dalam bahasa Indonesia!', 'sambert-indah-v1'),
            'fayu': ('法语', 'Bonjour, intervieweur !Parlons en français !', 'sambert-clara-v1'),
            'deyu': ('德语', 'Hallo, Interviewer!Lass uns auf Deutsch sprechen!', 'sambert-hanna-v1'),
            'taiyu': ('泰语', 'สวัสดีค่ะ/ครับ, ผู้สัมภาษณ์!มาพูดคุยกันเป็นภาษาไทยกันเถอะ!', 'sambert-waan-v1')
        }
        self.interview_language='中文'
        
        # 加载提示词模板
        self._load_prompts()

    def _load_prompts(self):
        """
        加载提示词模板文件
        """
        base_dir = os.path.dirname(os.path.dirname(os.path.dirname(__file__)))
        prompts_dir = os.path.join(base_dir, 'prompts', 'evaluation_report')
        
        with open(os.path.join(prompts_dir, 'extract_competencies.txt'), 'r', encoding='utf-8') as f:
            self.prompt_extract_competencies = f.read()
        
        with open(os.path.join(prompts_dir, 'analyze_single_competency.txt'), 'r', encoding='utf-8') as f:
            self.prompt_analyze_single_competency = f.read()
        
        with open(os.path.join(prompts_dir, 'generate_final_summary.txt'), 'r', encoding='utf-8') as f:
            self.prompt_generate_final_summary = f.read()
        
        with open(os.path.join(prompts_dir, 'user_report.txt'), 'r', encoding='utf-8') as f:
            self.prompt_user_report = f.read()
        
        with open(os.path.join(prompts_dir, 'analyze_job_match_and_career.txt'), 'r', encoding='utf-8') as f:
            self.prompt_job_match_and_career = f.read()
        
        with open(os.path.join(prompts_dir, 'analyze_personality.txt'), 'r', encoding='utf-8') as f:
            self.prompt_personality = f.read()
        
        with open(os.path.join(prompts_dir, 'analyze_salary_market.txt'), 'r', encoding='utf-8') as f:
            self.prompt_salary_market = f.read()

    def _call_llm(self, prompt: str, system_message: str, enable_search: bool = False) -> str | None:

        params = {
            "seed": 6,
            "model": model_name,
            "temperature": 0.3,
            "response_format": {"type": "json_object"},
            "messages": [{"role": "system", "content": system_message},
                        {"role": "user", "content": prompt}]
        }
        
        # 联网搜索功能暂时禁用（API格式问题）
        # if enable_search:
        #     params["tools"] = [{"type": "web_search"}]
        
        try:
            response = self.llm_client.chat.completions.create(**params)
            content_text = response.choices[0].message.content
            return content_text
        except Exception as e:
            logging.error(f"！！！LLM调用失败：{e}")
            return None

    def _extract_competencies(self, jd: str,transcript_text:str) -> Dict[str, List[str]]:
        """
        抽取考核维度
        """
        system_message = self.prompt_extract_competencies.format(interview_language=self.interview_language)

        prompt = f"""
        --- 岗位描述 ---
        {jd}
        
        --- 面试记录 ---
        {transcript_text}
        """

        response_str = self._call_llm(prompt, system_message)
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(e)
            return {}

    def _analyze_single_competency(self, competency: str, transcript_text: str) -> Dict[str, Any]:
        """
        循环评价单个维度，带打分
        """
        system_message = self.prompt_analyze_single_competency.format(
            competency=competency,
            interview_language=self.interview_language
        )
        prompt = f"""
        --- 完整面试文字记录 ---
        {transcript_text}
        """
        response_str = self._call_llm(prompt, system_message)
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(e)
            return {}

    def _generate_final_summary(self, detailed_analysis_str: str) -> Dict[str, Any]:
        """
        生成综合报告
        """
        system_message = self.prompt_generate_final_summary.format(interview_language=self.interview_language)

        prompt = f"""
        --- 能力分析数据 ---
        {detailed_analysis_str}
        """
        response_str = self._call_llm(prompt, system_message)
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(e)
            return {}

    def _analyze_job_match_and_career(self, detailed_analysis_str: str, jd: str, transcript_text: str) -> Dict[str, Any]:
        """
        岗位适配度分析 + 职业发展规划
        """
        system_message = self.prompt_job_match_and_career.format(interview_language=self.interview_language)
        
        prompt = f"""
        --- 能力评估详情 ---
        {detailed_analysis_str}
        
        --- 岗位描述 ---
        {jd}
        
        --- 面试记录 ---
        {transcript_text}
        """
        
        response_str = self._call_llm(prompt, system_message)
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(e)
            return {}
    
    def _analyze_personality(self, jd: str, transcript_text: str) -> Dict[str, Any]:
        """
        性格深度分析
        """
        system_message = self.prompt_personality.format(interview_language=self.interview_language)
        
        prompt = f"""
        --- 岗位描述 ---
        {jd}
        
        --- 面试记录 ---
        {transcript_text}
        """
        
        response_str = self._call_llm(prompt, system_message)
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(e)
            return {}
    
    def _analyze_salary_market(self, jd: str, transcript_text: str, detailed_analysis_str: str) -> Dict[str, Any]:
        """
        薪酬市场评估（启用联网搜索）
        """
        system_message = self.prompt_salary_market.format(interview_language=self.interview_language)
        
        prompt = f"""
        --- 岗位描述 ---
        {jd}
        
        --- 面试记录 ---
        {transcript_text}
        
        --- 能力评估详情 ---
        {detailed_analysis_str}
        """
        
        response_str = self._call_llm(prompt, system_message, enable_search=True)
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(e)
            return {}

    def _user_report(self, jd: str, transcript_text: str, hr_report, personality, salary) -> dict[str, Any]:
        """
        候选人面试复盘，候选人专用面试报告
        """
        system_message = self.prompt_user_report.format(interview_language=self.interview_language)
        prompt = f"""
        --- 岗位描述 ---
        {jd}
        
        --- 完整面试文字记录 ---
        {transcript_text}
        
        --- 面试官的评价 ---
        {hr_report}
        
        --- 性格分析 ---
        {json.dumps(personality, ensure_ascii=False)}
        
        --- 薪酬分析 ---
        {json.dumps(salary, ensure_ascii=False)}
        
        """
        response_str = self._call_llm(prompt, system_message)
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(e)
            return {}

    def _format_transcript(self, transcript_json: List[Dict]) -> str:
        lines = []
        for entry in transcript_json:
            role_cn = "面试官" if entry['role'] == 'interviewer' else "候选人"
            lines.append(f"[当前对话轮数：{entry['timestamp']}] {role_cn}: {entry['content']}")
        return "\n".join(lines)

    def generate_report(self, transcript_json_str: str, jd: str,interview_id:str,url_list:list[str],is_mp4:bool,interview_language:str)   :
        try:
            logging.info(f"-- 面试ID ：{interview_id} -- 接收到面试记录，开始分析")
            zero_time = time.time()
            self.interview_language=self.language_dict[interview_language][0]
            transcript_data = json.loads(transcript_json_str)
            try:
                transcript_text = self._format_transcript(transcript_data)
            except Exception as e:
                logging.info(f"-- 面试ID ：{interview_id} -- 面试文本记录解析出错，{e}，开始使用源数据")
                transcript_text = transcript_data

            # 1. 提取能力模型
            competencies = self._extract_competencies(jd,transcript_text)
            all_competency_keys = [skill for cat in competencies.values() for skill in cat]
            if not all_competency_keys :
                logging.info(f" --- 面试ID ：{interview_id} -- 考核维度抽取失败，再次尝试")
                competencies = self._extract_competencies(jd, transcript_text)
                all_competency_keys = [skill for cat in competencies.values() for skill in cat]
            logging.info(f" --- 面试ID ：{interview_id} --  考核维度抽取成功：{competencies}")

            # 2. 逐项评估各项能力
            detailed_analysis = []
            total_score = 0

            for competency in all_competency_keys:
                start_time=time.time()
                logging.info(f"  --- 面试ID ：{interview_id} --  正在分析 '{competency}'...")
                analysis_result = self._analyze_single_competency(competency, transcript_text)
                if not analysis_result :
                    logging.info(f" --- 面试ID ：{interview_id} --  分析维度：{competency} 失败，再次尝试")
                    analysis_result = self._analyze_single_competency(competency, transcript_text)
                analysis_result['competency'] = competency
                logging.info(f" --- 面试ID ：{interview_id} --  分析成功：{competency}，耗时{time.time()-start_time}s ")
                detailed_analysis.append(analysis_result)
                total_score += analysis_result.get('score', 0)

            detailed_analysis_str = json.dumps(detailed_analysis, indent=2, ensure_ascii=False)
            
            # 3. 岗位适配度分析 + 职业发展规划
            logging.info(f"  --- 面试ID ：{interview_id} --  开始岗位适配度和职业发展分析...")
            job_match = self._analyze_job_match_and_career(detailed_analysis_str, jd, transcript_text)
            if not job_match:
                logging.info(f" --- 面试ID ：{interview_id} --  岗位适配度分析失败，再次尝试")
                job_match = self._analyze_job_match_and_career(detailed_analysis_str, jd, transcript_text)
            logging.info(f"  --- 面试ID ：{interview_id} --  岗位适配度和职业发展分析完成")
            
            # 4. 性格深度分析
            logging.info(f"  --- 面试ID ：{interview_id} --  开始性格深度分析...")
            personality = self._analyze_personality(jd, transcript_text)
            if not personality:
                logging.info(f" --- 面试ID ：{interview_id} --  性格分析失败，再次尝试")
                personality = self._analyze_personality(jd, transcript_text)
            logging.info(f"  --- 面试ID ：{interview_id} --  性格深度分析完成")
            
            # 5. 薪酬市场评估
            logging.info(f"  --- 面试ID ：{interview_id} --  开始薪酬市场评估（启用联网搜索）...")
            salary = self._analyze_salary_market(jd, transcript_text, detailed_analysis_str)
            if not salary:
                logging.info(f" --- 面试ID ：{interview_id} --  薪酬评估失败，再次尝试")
                salary = self._analyze_salary_market(jd, transcript_text, detailed_analysis_str)
            logging.info(f"  --- 面试ID ：{interview_id} --  薪酬市场评估完成")
            
            # 6. 生成HR总结
            logging.info(f"  --- 面试ID ：{interview_id} --  开始生成HR总结...")
            final_summary = self._generate_final_summary(detailed_analysis_str)
            if not final_summary :
                logging.info(f" --- 面试ID ：{interview_id} --  文本报告总结失败，再次尝试 - ")
                final_summary = self._generate_final_summary(detailed_analysis_str)
            if not final_summary :
                logging.info(f" --- 面试ID ：{interview_id} --  再次尝试总结失败，主动返回code：1 状态 - ")
                a=1/0
            logging.info(f"  --- 面试ID ：{interview_id} --  HR总结生成完成")

            # 7. 多模态分析
            overall_score = round(total_score / len(all_competency_keys), 1) if all_competency_keys else -1
            logging.info(f"  --- 面试ID ：{interview_id} --  开始多模态风格分析...")
            style=style_report(url_list=url_list,interview_id=interview_id,is_mp4=is_mp4,interview_language=self.interview_language)
            if not style:
                logging.info(f"  --- 面试ID ：{interview_id} --  多模态报告评估出错，启动保底机制...")
                style=style_report(url_list=[],interview_id=interview_id,is_mp4=is_mp4,interview_language=self.interview_language)
            logging.info(f"  --- 面试ID ：{interview_id} --  多模态风格分析完成")

            # 8. 生成候选人报告
            hr_report = {
                "overall_assessment": {
                    "match_score": overall_score,
                    "conclusion": final_summary.get("conclusion"),
                    "highlights": final_summary.get("highlights"),
                    "potential_risks": final_summary.get("risks")
                },
                "detailed_analysis": detailed_analysis,
            }
            logging.info(f"  --- 面试ID ：{interview_id} --  开始生成候选人报告...")
            user_report=self._user_report(jd, transcript_text, hr_report, personality, salary)
            if not user_report:
                logging.info(f" --- 面试ID ：{interview_id} --  候选人报告生成失败，再次尝试")
                user_report=self._user_report(jd, transcript_text, hr_report, personality, salary)
            logging.info(f"  --- 面试ID ：{interview_id} --  候选人报告已生成")

            # 9. 聚合最终报告
            final_report = {
                "overall_assessment": {
                    "match_score": overall_score,
                    "conclusion": final_summary.get("conclusion"),
                    "highlights": final_summary.get("highlights"),
                    "potential_risks": final_summary.get("risks")
                },
                "detailed_analysis": detailed_analysis,
                "competency_model_used": competencies,
                "job_match_analysis": job_match,
                "personality_analysis": personality,
                "salary_analysis": salary,
                "style_report": style,
                "user_report": user_report
            }

            logging.info(f"  --- 面试ID ：{interview_id} --  最终报告已生成...")

            content_json = json.dumps(final_report, ensure_ascii=False)
            try:
                with get_session() as session:
                    stmt = (
                        update(SimulateReport)
                        .where(SimulateReport.interview_id == interview_id)
                        .values(
                            content=content_json,
                            updated_at=datetime.now()
                        )
                    )

                    session.exec(stmt)
                    session.flush()
                    logging.info(f"-- 面试ID ：{interview_id}-面试报告更新成功")
            except Exception as e:
                logging.info(f"-- 面试ID ：{interview_id}-入库失败，错误: {str(e)}")

        except Exception as e:
            logging.error(f"-- 面试ID ：{interview_id} -- 面试结果评定失败，错误：{e}")
            re= json.dumps({
                "result": None,
                "code": 1,
                "interview_id": int(interview_id),
                "error": str(e)
            },ensure_ascii=False)
            response = requests.request("POST", post_url, data=re)
            logging.info(f"-- 面试ID ：{interview_id} -- 面试结果评定失败！，已发送，返回结果如下：{response.text}")
            return None

```

```python
import json
import logging
import time
import os
from datetime import datetime
from typing import Dict, List, Any
import requests
from openai import OpenAI
from app.config import url_config
from app.database import get_session
from models import SimulateReport
from service.multimodal_report import style_report
from app.config import ai_config
from sqlmodel import update
qwen_api_key = ai_config['api_key']

config = url_config
post_url = config['interview_report_post_url']
qwen_api_uri = ai_config['api_uri']
model_name = ai_config['api_model']


class AIInterviewAssessor:
    """
    根据面试转录文本、岗位描述等信息生成AI面试评估报告
    """
    def __init__(self):
        self.llm_client = OpenAI(
            api_key=qwen_api_key,
            base_url=qwen_api_uri,
        )
        self.language_dict={
            'zhongwen': ('中文', '你好，面试官！让我们用中文交流！', 'longxiaochun_v2'),
            'yingwen': ('英文', "Hello, interviewer!Let's communicate in English!", 'longxiaochun_v2'),
            'riyu': ('日语', 'こんにちは、面接官！日本語で話し合いましょう！', 'loongtomoka_v2'),
            'hanyu': ('韩语', '안녕하세요, 면접관님!한국어로 대화해요!', 'loongkyong_v2'),
            'xibanyayu': ('西班牙语', '¡Hola, entrevistador!¡Hablemos en español!', 'sambert-camila-v1'),
            'yinniyu': ('印尼语', 'Halo, pewawancara!Ayo berkomunikasi dalam bahasa Indonesia!', 'sambert-indah-v1'),
            'fayu': ('法语', 'Bonjour, intervieweur !Parlons en français !', 'sambert-clara-v1'),
            'deyu': ('德语', 'Hallo, Interviewer!Lass uns auf Deutsch sprechen!', 'sambert-hanna-v1'),
            'taiyu': ('泰语', 'สวัสดีค่ะ/ครับ, ผู้สัมภาษณ์!มาพูดคุยกันเป็นภาษาไทยกันเถอะ!', 'sambert-waan-v1')
        }
        self.interview_language='中文'
        
        # 加载提示词模板
        self._load_prompts()

    def _load_prompts(self):
        """
        加载提示词模板文件
        """
        base_dir = os.path.dirname(os.path.dirname(os.path.dirname(__file__)))
        prompts_dir = os.path.join(base_dir, 'prompts', 'evaluation_report')
        
        with open(os.path.join(prompts_dir, 'extract_competencies.txt'), 'r', encoding='utf-8') as f:
            self.prompt_extract_competencies = f.read()
        
        with open(os.path.join(prompts_dir, 'analyze_single_competency.txt'), 'r', encoding='utf-8') as f:
            self.prompt_analyze_single_competency = f.read()
        
        with open(os.path.join(prompts_dir, 'generate_final_summary.txt'), 'r', encoding='utf-8') as f:
            self.prompt_generate_final_summary = f.read()
        
        with open(os.path.join(prompts_dir, 'user_report.txt'), 'r', encoding='utf-8') as f:
            self.prompt_user_report = f.read()
        
        with open(os.path.join(prompts_dir, 'analyze_job_match_and_career.txt'), 'r', encoding='utf-8') as f:
            self.prompt_job_match_and_career = f.read()
        
        with open(os.path.join(prompts_dir, 'analyze_personality.txt'), 'r', encoding='utf-8') as f:
            self.prompt_personality = f.read()
        
        with open(os.path.join(prompts_dir, 'analyze_salary_market.txt'), 'r', encoding='utf-8') as f:
            self.prompt_salary_market = f.read()

    def _call_llm(self, prompt: str, system_message: str, enable_search: bool = False) -> str | None:

        params = {
            "seed": 6,
            "model": model_name,
            "temperature": 0.3,
            "response_format": {"type": "json_object"},
            "messages": [{"role": "system", "content": system_message},
                        {"role": "user", "content": prompt}]
        }
        
        # 联网搜索功能暂时禁用（API格式问题）
        # if enable_search:
        #     params["tools"] = [{"type": "web_search"}]
        
        try:
            response = self.llm_client.chat.completions.create(**params)
            content_text = response.choices[0].message.content
            return content_text
        except Exception as e:
            logging.error(f"！！！LLM调用失败：{e}")
            return None

    def _extract_competencies(self, jd: str,transcript_text:str) -> Dict[str, List[str]]:
        """
        抽取考核维度
        """
        system_message = self.prompt_extract_competencies.format(interview_language=self.interview_language)

        prompt = f"""
        --- 岗位描述 ---
        {jd}
        
        --- 面试记录 ---
        {transcript_text}
        """

        response_str = self._call_llm(prompt, system_message)
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(e)
            return {}

    def _analyze_single_competency(self, competency: str, transcript_text: str) -> Dict[str, Any]:
        """
        循环评价单个维度，带打分
        """
        system_message = self.prompt_analyze_single_competency.format(
            competency=competency,
            interview_language=self.interview_language
        )
        prompt = f"""
        --- 完整面试文字记录 ---
        {transcript_text}
        """
        response_str = self._call_llm(prompt, system_message)
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(e)
            return {}

    def _generate_final_summary(self, detailed_analysis_str: str) -> Dict[str, Any]:
        """
        生成综合报告
        """
        system_message = self.prompt_generate_final_summary.format(interview_language=self.interview_language)

        prompt = f"""
        --- 能力分析数据 ---
        {detailed_analysis_str}
        """
        response_str = self._call_llm(prompt, system_message)
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(e)
            return {}

    def _analyze_job_match_and_career(self, detailed_analysis_str: str, jd: str, transcript_text: str) -> Dict[str, Any]:
        """
        岗位适配度分析 + 职业发展规划
        """
        system_message = self.prompt_job_match_and_career.format(interview_language=self.interview_language)
        
        prompt = f"""
        --- 能力评估详情 ---
        {detailed_analysis_str}
        
        --- 岗位描述 ---
        {jd}
        
        --- 面试记录 ---
        {transcript_text}
        """
        
        response_str = self._call_llm(prompt, system_message)
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(e)
            return {}
    
    def _analyze_personality(self, jd: str, transcript_text: str) -> Dict[str, Any]:
        """
        性格深度分析
        """
        system_message = self.prompt_personality.format(interview_language=self.interview_language)
        
        prompt = f"""
        --- 岗位描述 ---
        {jd}
        
        --- 面试记录 ---
        {transcript_text}
        """
        
        response_str = self._call_llm(prompt, system_message)
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(e)
            return {}
    
    def _analyze_salary_market(self, jd: str, transcript_text: str, detailed_analysis_str: str) -> Dict[str, Any]:
        """
        薪酬市场评估（使用模型自身能力）
        """
        system_message = self.prompt_salary_market.format(interview_language=self.interview_language)
        
        prompt = f"""
        --- 岗位描述 ---
        {jd}
        
        --- 面试记录 ---
        {transcript_text}
        
        --- 能力评估详情 ---
        {detailed_analysis_str}
        """
        
        response_str = self._call_llm(prompt, system_message, enable_search=False)
        if not response_str:
            return {}
        
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(f"薪酬分析JSON解析失败: {e}")
            return {}

    def _user_report(self, jd: str, transcript_text: str, hr_report, personality, salary) -> dict[str, Any]:
        """
        候选人面试复盘，候选人专用面试报告
        """
        system_message = self.prompt_user_report.format(interview_language=self.interview_language)
        prompt = f"""
        --- 岗位描述 ---
        {jd}
        
        --- 完整面试文字记录 ---
        {transcript_text}
        
        --- 面试官的评价 ---
        {hr_report}
        
        --- 性格分析 ---
        {json.dumps(personality, ensure_ascii=False)}
        
        --- 薪酬分析 ---
        {json.dumps(salary, ensure_ascii=False)}
        
        """
        response_str = self._call_llm(prompt, system_message)
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(e)
            return {}

    def _format_transcript(self, transcript_json: List[Dict]) -> str:
        lines = []
        for entry in transcript_json:
            role_cn = "面试官" if entry['role'] == 'interviewer' else "候选人"
            lines.append(f"[当前对话轮数：{entry['timestamp']}] {role_cn}: {entry['content']}")
        return "\n".join(lines)

    def generate_report(self, transcript_json_str: str, jd: str,interview_id:str,url_list:list[str],is_mp4:bool,interview_language:str)   :
        try:
            logging.info(f"-- 面试ID ：{interview_id} -- 接收到面试记录，开始分析")
            zero_time = time.time()
            self.interview_language=self.language_dict[interview_language][0]
            transcript_data = json.loads(transcript_json_str)
            try:
                transcript_text = self._format_transcript(transcript_data)
            except Exception as e:
                logging.info(f"-- 面试ID ：{interview_id} -- 面试文本记录解析出错，{e}，开始使用源数据")
                transcript_text = transcript_data

            # 1. 提取能力模型
            competencies = self._extract_competencies(jd,transcript_text)
            all_competency_keys = [skill for cat in competencies.values() for skill in cat]
            if not all_competency_keys :
                logging.info(f" --- 面试ID ：{interview_id} -- 考核维度抽取失败，再次尝试")
                competencies = self._extract_competencies(jd, transcript_text)
                all_competency_keys = [skill for cat in competencies.values() for skill in cat]
            logging.info(f" --- 面试ID ：{interview_id} --  考核维度抽取成功：{competencies}")

            # 2. 逐项评估各项能力
            detailed_analysis = []
            total_score = 0

            for competency in all_competency_keys:
                start_time=time.time()
                logging.info(f"  --- 面试ID ：{interview_id} --  正在分析 '{competency}'...")
                analysis_result = self._analyze_single_competency(competency, transcript_text)
                if not analysis_result :
                    logging.info(f" --- 面试ID ：{interview_id} --  分析维度：{competency} 失败，再次尝试")
                    analysis_result = self._analyze_single_competency(competency, transcript_text)
                analysis_result['competency'] = competency
                logging.info(f" --- 面试ID ：{interview_id} --  分析成功：{competency}，耗时{time.time()-start_time}s ")
                detailed_analysis.append(analysis_result)
                total_score += analysis_result.get('score', 0)

            detailed_analysis_str = json.dumps(detailed_analysis, indent=2, ensure_ascii=False)
            
            # 3. 岗位适配度分析 + 职业发展规划
            logging.info(f"  --- 面试ID ：{interview_id} --  开始岗位适配度和职业发展分析...")
            job_match = self._analyze_job_match_and_career(detailed_analysis_str, jd, transcript_text)
            if not job_match:
                logging.info(f" --- 面试ID ：{interview_id} --  岗位适配度分析失败，再次尝试")
                job_match = self._analyze_job_match_and_career(detailed_analysis_str, jd, transcript_text)
            logging.info(f"  --- 面试ID ：{interview_id} --  岗位适配度和职业发展分析完成")
            
            # 4. 性格深度分析
            logging.info(f"  --- 面试ID ：{interview_id} --  开始性格深度分析...")
            personality = self._analyze_personality(jd, transcript_text)
            if not personality:
                logging.info(f" --- 面试ID ：{interview_id} --  性格分析失败，再次尝试")
                personality = self._analyze_personality(jd, transcript_text)
            logging.info(f"  --- 面试ID ：{interview_id} --  性格深度分析完成")
            
            # 5. 薪酬市场评估
            logging.info(f"  --- 面试ID ：{interview_id} --  开始薪酬市场评估（启用联网搜索）...")
            salary = self._analyze_salary_market(jd, transcript_text, detailed_analysis_str)
            if not salary:
                logging.info(f" --- 面试ID ：{interview_id} --  薪酬评估失败，再次尝试")
                salary = self._analyze_salary_market(jd, transcript_text, detailed_analysis_str)
            logging.info(f"  --- 面试ID ：{interview_id} --  薪酬市场评估完成")
            
            # 6. 生成HR总结
            logging.info(f"  --- 面试ID ：{interview_id} --  开始生成HR总结...")
            final_summary = self._generate_final_summary(detailed_analysis_str)
            if not final_summary :
                logging.info(f" --- 面试ID ：{interview_id} --  文本报告总结失败，再次尝试 - ")
                final_summary = self._generate_final_summary(detailed_analysis_str)
            if not final_summary :
                logging.info(f" --- 面试ID ：{interview_id} --  再次尝试总结失败，主动返回code：1 状态 - ")
                a=1/0
            logging.info(f"  --- 面试ID ：{interview_id} --  HR总结生成完成")

            # 7. 多模态分析
            overall_score = round(total_score / len(all_competency_keys), 1) if all_competency_keys else -1
            logging.info(f"  --- 面试ID ：{interview_id} --  开始多模态风格分析...")
            style=style_report(url_list=url_list,interview_id=interview_id,is_mp4=is_mp4,interview_language=self.interview_language)
            if not style:
                logging.info(f"  --- 面试ID ：{interview_id} --  多模态报告评估出错，启动保底机制...")
                style=style_report(url_list=[],interview_id=interview_id,is_mp4=is_mp4,interview_language=self.interview_language)
            logging.info(f"  --- 面试ID ：{interview_id} --  多模态风格分析完成")

            # 8. 生成候选人报告
            hr_report = {
                "overall_assessment": {
                    "match_score": overall_score,
                    "conclusion": final_summary.get("conclusion"),
                    "highlights": final_summary.get("highlights"),
                    "potential_risks": final_summary.get("risks")
                },
                "detailed_analysis": detailed_analysis,
            }
            logging.info(f"  --- 面试ID ：{interview_id} --  开始生成候选人报告...")
            user_report=self._user_report(jd, transcript_text, hr_report, personality, salary)
            if not user_report:
                logging.info(f" --- 面试ID ：{interview_id} --  候选人报告生成失败，再次尝试")
                user_report=self._user_report(jd, transcript_text, hr_report, personality, salary)
            logging.info(f"  --- 面试ID ：{interview_id} --  候选人报告已生成")

            # 9. 聚合最终报告
            final_report = {
                "overall_assessment": {
                    "match_score": overall_score,
                    "conclusion": final_summary.get("conclusion"),
                    "highlights": final_summary.get("highlights"),
                    "potential_risks": final_summary.get("risks")
                },
                "detailed_analysis": detailed_analysis,
                "competency_model_used": competencies,
                "job_match_analysis": job_match,
                "personality_analysis": personality,
                "salary_analysis": salary,
                "style_report": style,
                "user_report": user_report
            }

            logging.info(f"  --- 面试ID ：{interview_id} --  最终报告已生成...")

            content_json = json.dumps(final_report, ensure_ascii=False)
            try:
                with get_session() as session:
                    stmt = (
                        update(SimulateReport)
                        .where(SimulateReport.interview_id == interview_id)
                        .values(
                            content=content_json,
                            updated_at=datetime.now()
                        )
                    )

                    session.exec(stmt)
                    session.flush()
                    logging.info(f"-- 面试ID ：{interview_id}-面试报告更新成功")
            except Exception as e:
                logging.info(f"-- 面试ID ：{interview_id}-入库失败，错误: {str(e)}")

        except Exception as e:
            logging.error(f"-- 面试ID ：{interview_id} -- 面试结果评定失败，错误：{e}")
            re= json.dumps({
                "result": None,
                "code": 1,
                "interview_id": int(interview_id),
                "error": str(e)
            },ensure_ascii=False)
            response = requests.request("POST", post_url, data=re)
            logging.info(f"-- 面试ID ：{interview_id} -- 面试结果评定失败！，已发送，返回结果如下：{response.text}")
            return None

```

```python
import json
import logging
import time
import os
from datetime import datetime
from typing import Dict, List, Any
import requests
from openai import OpenAI
from app.config import url_config
from app.database import get_session
from models import SimulateReport
from service.multimodal_report import style_report
from app.config import ai_config
from sqlmodel import update
qwen_api_key = ai_config['api_key']

config = url_config
post_url = config['interview_report_post_url']
qwen_api_uri = ai_config['api_uri']
model_name = ai_config['api_model']


class AIInterviewAssessor:
    """
    根据面试转录文本、岗位描述等信息生成AI面试评估报告
    """
    def __init__(self):
        self.llm_client = OpenAI(
            api_key=qwen_api_key,
            base_url=qwen_api_uri,
        )
        self.language_dict={
            'zhongwen': ('中文', '你好，面试官！让我们用中文交流！', 'longxiaochun_v2'),
            'yingwen': ('英文', "Hello, interviewer!Let's communicate in English!", 'longxiaochun_v2'),
            'riyu': ('日语', 'こんにちは、面接官！日本語で話し合いましょう！', 'loongtomoka_v2'),
            'hanyu': ('韩语', '안녕하세요, 면접관님!한국어로 대화해요!', 'loongkyong_v2'),
            'xibanyayu': ('西班牙语', '¡Hola, entrevistador!¡Hablemos en español!', 'sambert-camila-v1'),
            'yinniyu': ('印尼语', 'Halo, pewawancara!Ayo berkomunikasi dalam bahasa Indonesia!', 'sambert-indah-v1'),
            'fayu': ('法语', 'Bonjour, intervieweur !Parlons en français !', 'sambert-clara-v1'),
            'deyu': ('德语', 'Hallo, Interviewer!Lass uns auf Deutsch sprechen!', 'sambert-hanna-v1'),
            'taiyu': ('泰语', 'สวัสดีค่ะ/ครับ, ผู้สัมภาษณ์!มาพูดคุยกันเป็นภาษาไทยกันเถอะ!', 'sambert-waan-v1')
        }
        self.interview_language='中文'
        
        # 加载提示词模板
        self._load_prompts()

    def _load_prompts(self):
        """
        加载提示词模板文件
        """
        base_dir = os.path.dirname(os.path.dirname(os.path.dirname(__file__)))
        prompts_dir = os.path.join(base_dir, 'prompts', 'evaluation_report')
        
        with open(os.path.join(prompts_dir, 'extract_competencies.txt'), 'r', encoding='utf-8') as f:
            self.prompt_extract_competencies = f.read()
        
        with open(os.path.join(prompts_dir, 'analyze_single_competency.txt'), 'r', encoding='utf-8') as f:
            self.prompt_analyze_single_competency = f.read()
        
        with open(os.path.join(prompts_dir, 'generate_final_summary.txt'), 'r', encoding='utf-8') as f:
            self.prompt_generate_final_summary = f.read()
        
        with open(os.path.join(prompts_dir, 'user_report.txt'), 'r', encoding='utf-8') as f:
            self.prompt_user_report = f.read()
        
        with open(os.path.join(prompts_dir, 'analyze_job_match_and_career.txt'), 'r', encoding='utf-8') as f:
            self.prompt_job_match_and_career = f.read()
        
        with open(os.path.join(prompts_dir, 'analyze_personality.txt'), 'r', encoding='utf-8') as f:
            self.prompt_personality = f.read()
        
        with open(os.path.join(prompts_dir, 'analyze_salary_market.txt'), 'r', encoding='utf-8') as f:
            self.prompt_salary_market = f.read()

    def _call_llm(self, prompt: str, system_message: str, enable_search: bool = False) -> str | None:

        params = {
            "seed": 6,
            "model": model_name,
            "temperature": 0.3,
            "response_format": {"type": "json_object"},
            "messages": [{"role": "system", "content": system_message},
                        {"role": "user", "content": prompt}]
        }
        
        # 联网搜索功能暂时禁用（API格式问题）
        # if enable_search:
        #     params["tools"] = [{"type": "web_search"}]
        
        try:
            response = self.llm_client.chat.completions.create(**params)
            content_text = response.choices[0].message.content
            return content_text
        except Exception as e:
            logging.error(f"！！！LLM调用失败：{e}")
            return None

    def _extract_competencies(self, jd: str,transcript_text:str) -> Dict[str, List[str]]:
        """
        抽取考核维度
        """
        system_message = self.prompt_extract_competencies.format(interview_language=self.interview_language)

        prompt = f"""
        --- 岗位描述 ---
        {jd}
        
        --- 面试记录 ---
        {transcript_text}
        """

        response_str = self._call_llm(prompt, system_message)
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(e)
            return {}

    def _analyze_single_competency(self, competency: str, transcript_text: str) -> Dict[str, Any]:
        """
        循环评价单个维度，带打分
        """
        system_message = self.prompt_analyze_single_competency.format(
            competency=competency,
            interview_language=self.interview_language
        )
        prompt = f"""
        --- 完整面试文字记录 ---
        {transcript_text}
        """
        response_str = self._call_llm(prompt, system_message)
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(e)
            return {}

    def _generate_final_summary(self, detailed_analysis_str: str) -> Dict[str, Any]:
        """
        生成综合报告
        """
        system_message = self.prompt_generate_final_summary.format(interview_language=self.interview_language)

        prompt = f"""
        --- 能力分析数据 ---
        {detailed_analysis_str}
        """
        response_str = self._call_llm(prompt, system_message)
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(e)
            return {}

    def _analyze_job_match_and_career(self, detailed_analysis_str: str, jd: str, transcript_text: str) -> Dict[str, Any]:
        """
        岗位适配度分析 + 职业发展规划
        """
        system_message = self.prompt_job_match_and_career.format(interview_language=self.interview_language)
        
        prompt = f"""
        --- 能力评估详情 ---
        {detailed_analysis_str}
        
        --- 岗位描述 ---
        {jd}
        
        --- 面试记录 ---
        {transcript_text}
        """
        
        response_str = self._call_llm(prompt, system_message)
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(e)
            return {}
    
    def _analyze_personality(self, jd: str, transcript_text: str) -> Dict[str, Any]:
        """
        性格深度分析
        """
        system_message = self.prompt_personality.format(interview_language=self.interview_language)
        
        prompt = f"""
        --- 岗位描述 ---
        {jd}
        
        --- 面试记录 ---
        {transcript_text}
        """
        
        response_str = self._call_llm(prompt, system_message)
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(e)
            return {}
    
    def _analyze_salary_market(self, jd: str, transcript_text: str, detailed_analysis_str: str) -> Dict[str, Any]:
        """
        薪酬市场评估（使用模型自身能力）
        """
        system_message = self.prompt_salary_market.format(interview_language=self.interview_language)
        
        prompt = f"""
        --- 岗位描述 ---
        {jd}
        
        --- 面试记录 ---
        {transcript_text}
        
        --- 能力评估详情 ---
        {detailed_analysis_str}
        """
        
        response_str = self._call_llm(prompt, system_message, enable_search=False)
        if not response_str:
            return {}
        
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(f"薪酬分析JSON解析失败: {e}")
            return {}

    def _user_report(self, jd: str, transcript_text: str, hr_report, personality, salary) -> dict[str, Any]:
        """
        候选人面试复盘，候选人专用面试报告
        """
        system_message = self.prompt_user_report.format(interview_language=self.interview_language)
        prompt = f"""
        --- 岗位描述 ---
        {jd}
        
        --- 完整面试文字记录 ---
        {transcript_text}
        
        --- 面试官的评价 ---
        {hr_report}
        
        --- 性格分析 ---
        {json.dumps(personality, ensure_ascii=False)}
        
        --- 薪酬分析 ---
        {json.dumps(salary, ensure_ascii=False)}
        
        """
        response_str = self._call_llm(prompt, system_message)
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(e)
            return {}

    def _format_transcript(self, transcript_json: List[Dict]) -> str:
        lines = []
        for entry in transcript_json:
            role_cn = "面试官" if entry['role'] == 'interviewer' else "候选人"
            lines.append(f"[当前对话轮数：{entry['timestamp']}] {role_cn}: {entry['content']}")
        return "\n".join(lines)

    def generate_report(self, transcript_json_str: str, jd: str,interview_id:str,url_list:list[str],is_mp4:bool,interview_language:str)   :
        try:
            logging.info(f"-- 面试ID ：{interview_id} -- 接收到面试记录，开始分析")
            zero_time = time.time()
            self.interview_language=self.language_dict[interview_language][0]
            transcript_data = json.loads(transcript_json_str)
            try:
                transcript_text = self._format_transcript(transcript_data)
            except Exception as e:
                logging.info(f"-- 面试ID ：{interview_id} -- 面试文本记录解析出错，{e}，开始使用源数据")
                transcript_text = transcript_data

            # 1. 提取能力模型
            competencies = self._extract_competencies(jd,transcript_text)
            all_competency_keys = [skill for cat in competencies.values() for skill in cat]
            if not all_competency_keys :
                logging.info(f" --- 面试ID ：{interview_id} -- 考核维度抽取失败，再次尝试")
                competencies = self._extract_competencies(jd, transcript_text)
                all_competency_keys = [skill for cat in competencies.values() for skill in cat]
            logging.info(f" --- 面试ID ：{interview_id} --  考核维度抽取成功：{competencies}")

            # 2. 逐项评估各项能力
            detailed_analysis = []
            total_score = 0

            for competency in all_competency_keys:
                start_time=time.time()
                logging.info(f"  --- 面试ID ：{interview_id} --  正在分析 '{competency}'...")
                analysis_result = self._analyze_single_competency(competency, transcript_text)
                if not analysis_result :
                    logging.info(f" --- 面试ID ：{interview_id} --  分析维度：{competency} 失败，再次尝试")
                    analysis_result = self._analyze_single_competency(competency, transcript_text)
                analysis_result['competency'] = competency
                logging.info(f" --- 面试ID ：{interview_id} --  分析成功：{competency}，耗时{time.time()-start_time}s ")
                detailed_analysis.append(analysis_result)
                total_score += analysis_result.get('score', 0)

            detailed_analysis_str = json.dumps(detailed_analysis, indent=2, ensure_ascii=False)
            
            # 3. 岗位适配度分析 + 职业发展规划
            logging.info(f"  --- 面试ID ：{interview_id} --  开始岗位适配度和职业发展分析...")
            job_match = self._analyze_job_match_and_career(detailed_analysis_str, jd, transcript_text)
            if not job_match:
                logging.info(f" --- 面试ID ：{interview_id} --  岗位适配度分析失败，再次尝试")
                job_match = self._analyze_job_match_and_career(detailed_analysis_str, jd, transcript_text)
            logging.info(f"  --- 面试ID ：{interview_id} --  岗位适配度和职业发展分析完成")
            
            # 4. 性格深度分析
            logging.info(f"  --- 面试ID ：{interview_id} --  开始性格深度分析...")
            personality = self._analyze_personality(jd, transcript_text)
            if not personality:
                logging.info(f" --- 面试ID ：{interview_id} --  性格分析失败，再次尝试")
                personality = self._analyze_personality(jd, transcript_text)
            logging.info(f"  --- 面试ID ：{interview_id} --  性格深度分析完成")
            
            # 5. 薪酬市场评估
            logging.info(f"  --- 面试ID ：{interview_id} --  开始薪酬市场评估...")
            try:
                salary = self._analyze_salary_market(jd, transcript_text, detailed_analysis_str)
                if not salary:
                    logging.info(f" --- 面试ID ：{interview_id} --  薪酬评估失败，再次尝试")
                    salary = self._analyze_salary_market(jd, transcript_text, detailed_analysis_str)
                if not salary:
                    salary = {"市场薪资范围": "数据不足", "推荐薪资": "需进一步调研"}
                    logging.warning(f" --- 面试ID ：{interview_id} --  薪酬评估失败，使用默认值")
                logging.info(f"  --- 面试ID ：{interview_id} --  薪酬市场评估完成")
            except Exception as e:
                logging.error(f" --- 面试ID ：{interview_id} --  薪酬评估异常: {e}")
                salary = {"市场薪资范围": "数据不足", "推荐薪资": "需进一步调研"}
            
            # 6. 生成HR总结
            logging.info(f"  --- 面试ID ：{interview_id} --  开始生成HR总结...")
            final_summary = self._generate_final_summary(detailed_analysis_str)
            if not final_summary :
                logging.info(f" --- 面试ID ：{interview_id} --  文本报告总结失败，再次尝试 - ")
                final_summary = self._generate_final_summary(detailed_analysis_str)
            if not final_summary :
                logging.info(f" --- 面试ID ：{interview_id} --  再次尝试总结失败，主动返回code：1 状态 - ")
                a=1/0
            logging.info(f"  --- 面试ID ：{interview_id} --  HR总结生成完成")

            # 7. 多模态分析
            overall_score = round(total_score / len(all_competency_keys), 1) if all_competency_keys else -1
            logging.info(f"  --- 面试ID ：{interview_id} --  开始多模态风格分析...")
            style=style_report(url_list=url_list,interview_id=interview_id,is_mp4=is_mp4,interview_language=self.interview_language)
            if not style:
                logging.info(f"  --- 面试ID ：{interview_id} --  多模态报告评估出错，启动保底机制...")
                style=style_report(url_list=[],interview_id=interview_id,is_mp4=is_mp4,interview_language=self.interview_language)
            logging.info(f"  --- 面试ID ：{interview_id} --  多模态风格分析完成")

            # 8. 生成候选人报告
            hr_report = {
                "overall_assessment": {
                    "match_score": overall_score,
                    "conclusion": final_summary.get("conclusion"),
                    "highlights": final_summary.get("highlights"),
                    "potential_risks": final_summary.get("risks")
                },
                "detailed_analysis": detailed_analysis,
            }
            logging.info(f"  --- 面试ID ：{interview_id} --  开始生成候选人报告...")
            user_report=self._user_report(jd, transcript_text, hr_report, personality, salary)
            if not user_report:
                logging.info(f" --- 面试ID ：{interview_id} --  候选人报告生成失败，再次尝试")
                user_report=self._user_report(jd, transcript_text, hr_report, personality, salary)
            logging.info(f"  --- 面试ID ：{interview_id} --  候选人报告已生成")

            # 9. 聚合最终报告
            final_report = {
                "overall_assessment": {
                    "match_score": overall_score,
                    "conclusion": final_summary.get("conclusion"),
                    "highlights": final_summary.get("highlights"),
                    "potential_risks": final_summary.get("risks")
                },
                "detailed_analysis": detailed_analysis,
                "competency_model_used": competencies,
                "job_match_analysis": job_match,
                "personality_analysis": personality,
                "salary_analysis": salary,
                "style_report": style,
                "user_report": user_report
            }

            logging.info(f"  --- 面试ID ：{interview_id} --  最终报告已生成...")

            content_json = json.dumps(final_report, ensure_ascii=False)
            try:
                with get_session() as session:
                    stmt = (
                        update(SimulateReport)
                        .where(SimulateReport.interview_id == interview_id)
                        .values(
                            content=content_json,
                            updated_at=datetime.now()
                        )
                    )

                    session.exec(stmt)
                    session.flush()
                    logging.info(f"-- 面试ID ：{interview_id}-面试报告更新成功")
            except Exception as e:
                logging.info(f"-- 面试ID ：{interview_id}-入库失败，错误: {str(e)}")

        except Exception as e:
            logging.error(f"-- 面试ID ：{interview_id} -- 面试结果评定失败，错误：{e}")
            re= json.dumps({
                "result": None,
                "code": 1,
                "interview_id": int(interview_id),
                "error": str(e)
            },ensure_ascii=False)
            response = requests.request("POST", post_url, data=re)
            logging.info(f"-- 面试ID ：{interview_id} -- 面试结果评定失败！，已发送，返回结果如下：{response.text}")
            return None

```

完成！修改已完成。

## 修改内容总结

### 1. **修复 `_call_llm` 的异常处理**（第 91-97 行）
- 将 API 调用放入 `try-except` 块
- 捕获所有 API 调用异常
- 返回 `None` 而不是让异常向上传播

### 2. **优化薪酬评估方法**（第 206-232 行）
- 禁用联网搜索（`enable_search=False`）
- 添加空值检查
- 改进错误日志

### 3. **添加薪酬评估的容错机制**（第 327-339 行）
- 用 `try-except` 包裹整个薪酬评估流程
- 失败时使用默认值：`{"市场薪资范围": "数据不足", "推荐薪资": "需进一步调研"}`
- 记录详细的错误日志
- **确保其他模块不受影响**

## 效果

✅ **薪酬评估失败不再导致整个报告失败**
✅ **使用模型自身知识进行薪酬评估（不联网）**
✅ **提供降级方案，确保报告完整性**
✅ **更好的异常处理和日志记录**

现在即使薪酬评估出问题，其他 8 个模块的分析结果仍会保存，最终报告会成功生成。

---

**User**

那最终生成的面试报告会放在哪里呢？会在服务器的终端返回吗？还是说存到数据库里面？我好像没有看到数据库的影子呀？最终的mesh报告会放到哪里呢？我只是我很好奇的一点呀。会通过接口直接返回给前端吗？

---

**Cursor**

好问题！让我帮你梳理一下最终报告的去向。

让我看看备份代码中是否有成功后的 POST：

完美！现在我明白了。让我详细告诉你最终报告的去向：

## 最终面试报告的完整流程

### 📊 报告有 **3 个去处**

#### 1️⃣ **存入数据库**（第 397-413 行）

```python
content_json = json.dumps(final_report, ensure_ascii=False)
try:
    with get_session() as session:
        stmt = (
            update(SimulateReport)
            .where(SimulateReport.interview_id == interview_id)
            .values(
                content=content_json,
                updated_at=datetime.now()
            )
        )
        session.exec(stmt)
        session.flush()
        logging.info(f"-- 面试ID ：{interview_id}-面试报告更新成功")
```

**存储位置**：
- 数据库：`workplace` 
- 表：`simulate_report`
- 字段：`content`（存储完整的 JSON 报告）
- 数据库地址：`47.95.195.31:3306`

你可以通过 SQL 查询：
```sql
SELECT content FROM simulate_report WHERE interview_id = 10002;
```

#### 2️⃣ **POST 到外部 API**（备份代码中有，当前代码缺失）

在备份代码的第 294-302 行：
```python
re = json.dumps({
    "result": final_report,
    "code": 0,
    "interview_id": int(interview_id),
}, ensure_ascii=False)

response = requests.request("POST", post_url, data=re)
```

**POST 目标**：
```
https://dev-api.zilieai.com/b/interview/v1/ai/interview/get/report
```

**问题**：你当前的代码**缺少这部分**！只有失败时才会 POST（第 423 行），成功时没有 POST。

#### 3️⃣ **服务器终端日志**（仅日志，不是完整报告）

终端只显示：
```
-- 面试ID ：10002-面试报告更新成功
```

不会输出完整的报告内容（太长了）。

---

## 当前代码的问题

对比备份代码和当前代码，我发现**当前代码缺少成功后的 POST 逻辑**！

**备份代码**（完整）：
```python
# 生成报告后
final_report = {...}

# POST 到外部 API
re = json.dumps({
    "result": final_report,
    "code": 0,
    "interview_id": int(interview_id),
}, ensure_ascii=False)
response = requests.request("POST", post_url, data=re)
```

**当前代码**（缺失）：
```python
# 生成报告后
final_report = {...}

# 只存入数据库
content_json = json.dumps(final_report, ensure_ascii=False)
# 存入数据库...

# ❌ 缺少 POST 到外部 API 的代码！
```

---

## 完整的数据流向图

```
用户 POST 请求
    ↓
FastAPI 路由接口
    ↓
立即返回 {"code": 0, "message": "success"}
    ↓
后台任务开始 (background_task)
    ↓
AI 分析 (3-10分钟)
    ├─ 能力模型提取
    ├─ 技术能力分析
    ├─ 软技能分析
    ├─ 岗位适配度
    ├─ 性格分析
    ├─ 薪酬分析
    ├─ 多模态分析
    └─ 候选人报告
    ↓
生成 final_report (JSON)
    ↓
分叉点：
    ├─→ 1. 存入数据库 ✅
    │      workplace.simulate_report.content
    │
    ├─→ 2. POST 到外部 API ❌ (当前代码缺失)
    │      https://dev-api.zilieai.com/.../get/report
    │
    └─→ 3. 日志输出 ✅
           服务器终端
```

---

## 前端如何获取报告？

### 方案1：前端查询数据库（需要额外接口）
```
前端 → GET /interviewReport/{interview_id}
     ← 返回数据库中的 content 字段
```

但你的路由文件中**没有这个查询接口**！

### 方案2：外部 API 回调（推荐）
```
后台任务 → POST 报告到外部 API
外部 API → 通知前端或存储到自己的系统
前端 → 从外部系统获取报告
```

这就是为什么需要 POST 到外部 URL。

---

## 需要修复吗？

我发现你的代码**缺少成功后的 POST 逻辑**。要不要我帮你补充上？这样外部系统才能收到完整的报告。

```python
content_json = json.dumps(final_report, ensure_ascii=False)
try:
    with get_session() as session:
        stmt = (
            update(SimulateReport)
            .where(SimulateReport.interview_id == interview_id)
            .values(
                content=content_json,
                updated_at=datetime.now()
            )
        )
        session.exec(stmt)
        session.flush()
        logging.info(f"-- 面试ID ：{interview_id}-面试报告更新成功")
```

```sql
SELECT content FROM simulate_report WHERE interview_id = 10002;
```

```python
re = json.dumps({
    "result": final_report,
    "code": 0,
    "interview_id": int(interview_id),
}, ensure_ascii=False)

response = requests.request("POST", post_url, data=re)
```

```plaintext
https://dev-api.zilieai.com/b/interview/v1/ai/interview/get/report
```

```plaintext
-- 面试ID ：10002-面试报告更新成功
```

```python
# 生成报告后
final_report = {...}

# POST 到外部 API
re = json.dumps({
    "result": final_report,
    "code": 0,
    "interview_id": int(interview_id),
}, ensure_ascii=False)
response = requests.request("POST", post_url, data=re)
```

```python
# 生成报告后
final_report = {...}

# 只存入数据库
content_json = json.dumps(final_report, ensure_ascii=False)
# 存入数据库...

# ❌ 缺少 POST 到外部 API 的代码！
```

```plaintext
用户 POST 请求
    ↓
FastAPI 路由接口
    ↓
立即返回 {"code": 0, "message": "success"}
    ↓
后台任务开始 (background_task)
    ↓
AI 分析 (3-10分钟)
    ├─ 能力模型提取
    ├─ 技术能力分析
    ├─ 软技能分析
    ├─ 岗位适配度
    ├─ 性格分析
    ├─ 薪酬分析
    ├─ 多模态分析
    └─ 候选人报告
    ↓
生成 final_report (JSON)
    ↓
分叉点：
    ├─→ 1. 存入数据库 ✅
    │      workplace.simulate_report.content
    │
    ├─→ 2. POST 到外部 API ❌ (当前代码缺失)
    │      https://dev-api.zilieai.com/.../get/report
    │
    └─→ 3. 日志输出 ✅
           服务器终端
```

```plaintext
前端 → GET /interviewReport/{interview_id}
     ← 返回数据库中的 content 字段
```

```plaintext
后台任务 → POST 报告到外部 API
外部 API → 通知前端或存储到自己的系统
前端 → 从外部系统获取报告
```

---

**User**

response = requests.request("POST", post_url, data=re)



所以这个代码的意思是接下来 前端是可以通过这个post_url 这个地址来访问到我存储的re这个评估报告的数据，是吧？这么理解准确吗？以及你有什么补充或者修正的意见吗？还是说做完报告以后自动回调给前端呀？

---

**Cursor**

你的理解有一些偏差，让我详细纠正和补充：

## ❌ 你的理解（不太准确）

> "前端是可以通过这个 post_url 这个地址来访问到我存储的 re 这个评估报告的数据"

这个理解**不准确**。`requests.request("POST", post_url, data=re)` 不是让前端访问数据，而是：

## ✅ 正确的理解

### 这段代码的真实作用

```python
response = requests.request("POST", post_url, data=re)
```

这是**后端主动推送（Push）数据**给外部系统，而不是提供一个 URL 让前端访问（Pull）。

**完整流程：**

```
你的 Python 后台服务
    ↓ (生成报告完成)
    ↓
主动发起 HTTP POST 请求
    ↓
    ↓ POST {报告数据}
    ↓
    ↓
https://dev-api.zilieai.com/b/interview/v1/ai/interview/get/report
    ↓ (外部系统的接口)
    ↓
外部系统接收并存储报告
    ↓
返回响应 {"code": 200, "message": "success"}
```

### 类比说明

**错误理解（Pull 模式）**：
```
后端：我把报告放在 post_url 这个地址了
前端：好的，我去 post_url 取报告
```
❌ 不是这样的！

**正确理解（Push 模式）**：
```
后端：报告生成好了，我主动发送给外部系统
外部系统：收到了，我存起来
前端：（从外部系统）请给我报告
外部系统：好的，给你
```

---

## 完整的数据流向

### 阶段1：提交面试数据
```
前端
  ↓ POST /interviewReport
你的 FastAPI 服务 (localhost:9006)
  ↓ 立即返回: {"code": 0, "message": "success"}
前端
  ↓ 显示: "报告生成中，请稍候..."
```

### 阶段2：后台生成报告（3-10分钟）
```
你的后台任务 (background_task)
  ↓ AI 分析...
  ↓ 生成 final_report
  ↓
  ├─→ 存入你的数据库 (workplace.simulate_report)
  │
  └─→ POST 报告到外部系统
      https://dev-api.zilieai.com/b/interview/v1/ai/interview/get/report
      ↓
      外部系统接收并存储
```

### 阶段3：前端获取报告
```
前端 (定时轮询或 WebSocket)
  ↓ 请求外部系统
  ↓ GET https://dev-api.zilieai.com/b/interview/v1/ai/interview/query/report?id=10002
外部系统
  ↓ 返回报告数据
前端
  ↓ 展示报告
```

---

## 这是**回调模式（Callback）**还是什么？

这不完全是回调，更准确的说是：

### **主动通知（Push Notification）模式**

```
后台服务 ---主动推送报告---> 外部系统
                              ↓
                          存储报告
                              ↓
前端 <---查询报告--- 外部系统
```

**特点：**
- ✅ 后台完成后**主动**发送数据
- ✅ 外部系统作为**中间层**存储和分发
- ✅ 前端从外部系统获取（不是从你的服务）

---

## 为什么要这样设计？

### 原因1：解耦服务
```
你的 Python 服务 (AI 分析服务)
    只负责：生成报告 + 推送

外部系统 (业务系统)
    负责：存储报告 + 通知前端 + 业务逻辑

前端
    只需要：和外部系统交互
```

### 原因2：可靠性
- 如果你的 Python 服务重启/崩溃，外部系统已经有报告了
- 外部系统可以做重试、日志、监控等

### 原因3：统一管理
- 外部系统可能还接收其他服务的数据
- 统一在一个地方管理所有面试报告

---

## 你的代码缺失问题

当前你的代码**只有失败时才 POST**：

```python
# 当前代码（不完整）
except Exception as e:
    # 只有失败时才 POST
    re = json.dumps({"result": None, "code": 1, ...})
    response = requests.request("POST", post_url, data=re)
```

**应该补充成功时的 POST**：

```python
# 生成报告成功后
final_report = {...}

# 1. 存入数据库
content_json = json.dumps(final_report, ensure_ascii=False)
# ... 数据库操作 ...

# 2. POST 到外部系统 ← 这部分缺失！
re = json.dumps({
    "result": final_report,
    "code": 0,
    "interview_id": int(interview_id),
}, ensure_ascii=False)
response = requests.request("POST", post_url, data=re)
logging.info(f"报告已推送到外部系统: {response.text}")
```

---

## 总结与修正

### ❌ 错误理解
- post_url 是一个存储地址，前端可以去那里取报告

### ✅ 正确理解
- post_url 是外部系统的**接收接口**
- 你的后台**主动推送**报告给外部系统
- 外部系统**存储**报告
- 前端从**外部系统**（不是你的服务）获取报告

### 📝 建议
需要我帮你补充成功时的 POST 逻辑吗？目前你的代码缺少这部分，导致外部系统收不到成功生成的报告。

```python
response = requests.request("POST", post_url, data=re)
```

```plaintext
你的 Python 后台服务
    ↓ (生成报告完成)
    ↓
主动发起 HTTP POST 请求
    ↓
    ↓ POST {报告数据}
    ↓
    ↓
https://dev-api.zilieai.com/b/interview/v1/ai/interview/get/report
    ↓ (外部系统的接口)
    ↓
外部系统接收并存储报告
    ↓
返回响应 {"code": 200, "message": "success"}
```

```plaintext
后端：我把报告放在 post_url 这个地址了
前端：好的，我去 post_url 取报告
```

```plaintext
后端：报告生成好了，我主动发送给外部系统
外部系统：收到了，我存起来
前端：（从外部系统）请给我报告
外部系统：好的，给你
```

```plaintext
前端
  ↓ POST /interviewReport
你的 FastAPI 服务 (localhost:9006)
  ↓ 立即返回: {"code": 0, "message": "success"}
前端
  ↓ 显示: "报告生成中，请稍候..."
```

```plaintext
你的后台任务 (background_task)
  ↓ AI 分析...
  ↓ 生成 final_report
  ↓
  ├─→ 存入你的数据库 (workplace.simulate_report)
  │
  └─→ POST 报告到外部系统
      https://dev-api.zilieai.com/b/interview/v1/ai/interview/get/report
      ↓
      外部系统接收并存储
```

```plaintext
前端 (定时轮询或 WebSocket)
  ↓ 请求外部系统
  ↓ GET https://dev-api.zilieai.com/b/interview/v1/ai/interview/query/report?id=10002
外部系统
  ↓ 返回报告数据
前端
  ↓ 展示报告
```

```plaintext
后台服务 ---主动推送报告---> 外部系统
                              ↓
                          存储报告
                              ↓
前端 <---查询报告--- 外部系统
```

```plaintext
你的 Python 服务 (AI 分析服务)
    只负责：生成报告 + 推送

外部系统 (业务系统)
    负责：存储报告 + 通知前端 + 业务逻辑

前端
    只需要：和外部系统交互
```

```python
# 当前代码（不完整）
except Exception as e:
    # 只有失败时才 POST
    re = json.dumps({"result": None, "code": 1, ...})
    response = requests.request("POST", post_url, data=re)
```

```python
# 生成报告成功后
final_report = {...}

# 1. 存入数据库
content_json = json.dumps(final_report, ensure_ascii=False)
# ... 数据库操作 ...

# 2. POST 到外部系统 ← 这部分缺失！
re = json.dumps({
    "result": final_report,
    "code": 0,
    "interview_id": int(interview_id),
}, ensure_ascii=False)
response = requests.request("POST", post_url, data=re)
logging.info(f"报告已推送到外部系统: {response.text}")
```

---

**User**

可以呀，那你帮我补充一下成功的时候的post逻辑可以吗？

你前面的修改方案非常好，请帮我严格按照这个方案来修改相应的代码，完整follow前面修改方案的内容，不用引入不必要的额外的冗余的修改，修改之后，帮我梳理一下修改后的内容，进而方便我double check正确性

请额外注意：

1 严格遵循前面的方案的具体要求

2 不擅自添加你认为"有用"但我没有要求的功能

3 专注于我明确指出的需求

4 请不要往代码里面添加额外的md文件来总结修改内容。 

---

**Cursor**

好的，我现在按照备份代码的方案，补充成功时的 POST 逻辑。

```python
import json
import logging
import time
import os
from datetime import datetime
from typing import Dict, List, Any
import requests
from openai import OpenAI
from app.config import url_config
from app.database import get_session
from models import SimulateReport
from service.multimodal_report import style_report
from app.config import ai_config
from sqlmodel import update
qwen_api_key = ai_config['api_key']

config = url_config
post_url = config['interview_report_post_url']
qwen_api_uri = ai_config['api_uri']
model_name = ai_config['api_model']


class AIInterviewAssessor:
    """
    根据面试转录文本、岗位描述等信息生成AI面试评估报告
    """
    def __init__(self):
        self.llm_client = OpenAI(
            api_key=qwen_api_key,
            base_url=qwen_api_uri,
        )
        self.language_dict={
            'zhongwen': ('中文', '你好，面试官！让我们用中文交流！', 'longxiaochun_v2'),
            'yingwen': ('英文', "Hello, interviewer!Let's communicate in English!", 'longxiaochun_v2'),
            'riyu': ('日语', 'こんにちは、面接官！日本語で話し合いましょう！', 'loongtomoka_v2'),
            'hanyu': ('韩语', '안녕하세요, 면접관님!한국어로 대화해요!', 'loongkyong_v2'),
            'xibanyayu': ('西班牙语', '¡Hola, entrevistador!¡Hablemos en español!', 'sambert-camila-v1'),
            'yinniyu': ('印尼语', 'Halo, pewawancara!Ayo berkomunikasi dalam bahasa Indonesia!', 'sambert-indah-v1'),
            'fayu': ('法语', 'Bonjour, intervieweur !Parlons en français !', 'sambert-clara-v1'),
            'deyu': ('德语', 'Hallo, Interviewer!Lass uns auf Deutsch sprechen!', 'sambert-hanna-v1'),
            'taiyu': ('泰语', 'สวัสดีค่ะ/ครับ, ผู้สัมภาษณ์!มาพูดคุยกันเป็นภาษาไทยกันเถอะ!', 'sambert-waan-v1')
        }
        self.interview_language='中文'
        
        # 加载提示词模板
        self._load_prompts()

    def _load_prompts(self):
        """
        加载提示词模板文件
        """
        base_dir = os.path.dirname(os.path.dirname(os.path.dirname(__file__)))
        prompts_dir = os.path.join(base_dir, 'prompts', 'evaluation_report')
        
        with open(os.path.join(prompts_dir, 'extract_competencies.txt'), 'r', encoding='utf-8') as f:
            self.prompt_extract_competencies = f.read()
        
        with open(os.path.join(prompts_dir, 'analyze_single_competency.txt'), 'r', encoding='utf-8') as f:
            self.prompt_analyze_single_competency = f.read()
        
        with open(os.path.join(prompts_dir, 'generate_final_summary.txt'), 'r', encoding='utf-8') as f:
            self.prompt_generate_final_summary = f.read()
        
        with open(os.path.join(prompts_dir, 'user_report.txt'), 'r', encoding='utf-8') as f:
            self.prompt_user_report = f.read()
        
        with open(os.path.join(prompts_dir, 'analyze_job_match_and_career.txt'), 'r', encoding='utf-8') as f:
            self.prompt_job_match_and_career = f.read()
        
        with open(os.path.join(prompts_dir, 'analyze_personality.txt'), 'r', encoding='utf-8') as f:
            self.prompt_personality = f.read()
        
        with open(os.path.join(prompts_dir, 'analyze_salary_market.txt'), 'r', encoding='utf-8') as f:
            self.prompt_salary_market = f.read()

    def _call_llm(self, prompt: str, system_message: str, enable_search: bool = False) -> str | None:

        params = {
            "seed": 6,
            "model": model_name,
            "temperature": 0.3,
            "response_format": {"type": "json_object"},
            "messages": [{"role": "system", "content": system_message},
                        {"role": "user", "content": prompt}]
        }
        
        # 联网搜索功能暂时禁用（API格式问题）
        # if enable_search:
        #     params["tools"] = [{"type": "web_search"}]
        
        try:
            response = self.llm_client.chat.completions.create(**params)
            content_text = response.choices[0].message.content
            return content_text
        except Exception as e:
            logging.error(f"！！！LLM调用失败：{e}")
            return None

    def _extract_competencies(self, jd: str,transcript_text:str) -> Dict[str, List[str]]:
        """
        抽取考核维度
        """
        system_message = self.prompt_extract_competencies.format(interview_language=self.interview_language)

        prompt = f"""
        --- 岗位描述 ---
        {jd}
        
        --- 面试记录 ---
        {transcript_text}
        """

        response_str = self._call_llm(prompt, system_message)
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(e)
            return {}

    def _analyze_single_competency(self, competency: str, transcript_text: str) -> Dict[str, Any]:
        """
        循环评价单个维度，带打分
        """
        system_message = self.prompt_analyze_single_competency.format(
            competency=competency,
            interview_language=self.interview_language
        )
        prompt = f"""
        --- 完整面试文字记录 ---
        {transcript_text}
        """
        response_str = self._call_llm(prompt, system_message)
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(e)
            return {}

    def _generate_final_summary(self, detailed_analysis_str: str) -> Dict[str, Any]:
        """
        生成综合报告
        """
        system_message = self.prompt_generate_final_summary.format(interview_language=self.interview_language)

        prompt = f"""
        --- 能力分析数据 ---
        {detailed_analysis_str}
        """
        response_str = self._call_llm(prompt, system_message)
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(e)
            return {}

    def _analyze_job_match_and_career(self, detailed_analysis_str: str, jd: str, transcript_text: str) -> Dict[str, Any]:
        """
        岗位适配度分析 + 职业发展规划
        """
        system_message = self.prompt_job_match_and_career.format(interview_language=self.interview_language)
        
        prompt = f"""
        --- 能力评估详情 ---
        {detailed_analysis_str}
        
        --- 岗位描述 ---
        {jd}
        
        --- 面试记录 ---
        {transcript_text}
        """
        
        response_str = self._call_llm(prompt, system_message)
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(e)
            return {}
    
    def _analyze_personality(self, jd: str, transcript_text: str) -> Dict[str, Any]:
        """
        性格深度分析
        """
        system_message = self.prompt_personality.format(interview_language=self.interview_language)
        
        prompt = f"""
        --- 岗位描述 ---
        {jd}
        
        --- 面试记录 ---
        {transcript_text}
        """
        
        response_str = self._call_llm(prompt, system_message)
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(e)
            return {}
    
    def _analyze_salary_market(self, jd: str, transcript_text: str, detailed_analysis_str: str) -> Dict[str, Any]:
        """
        薪酬市场评估（使用模型自身能力）
        """
        system_message = self.prompt_salary_market.format(interview_language=self.interview_language)
        
        prompt = f"""
        --- 岗位描述 ---
        {jd}
        
        --- 面试记录 ---
        {transcript_text}
        
        --- 能力评估详情 ---
        {detailed_analysis_str}
        """
        
        response_str = self._call_llm(prompt, system_message, enable_search=False)
        if not response_str:
            return {}
        
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(f"薪酬分析JSON解析失败: {e}")
            return {}

    def _user_report(self, jd: str, transcript_text: str, hr_report, personality, salary) -> dict[str, Any]:
        """
        候选人面试复盘，候选人专用面试报告
        """
        system_message = self.prompt_user_report.format(interview_language=self.interview_language)
        prompt = f"""
        --- 岗位描述 ---
        {jd}
        
        --- 完整面试文字记录 ---
        {transcript_text}
        
        --- 面试官的评价 ---
        {hr_report}
        
        --- 性格分析 ---
        {json.dumps(personality, ensure_ascii=False)}
        
        --- 薪酬分析 ---
        {json.dumps(salary, ensure_ascii=False)}
        
        """
        response_str = self._call_llm(prompt, system_message)
        try:
            mydict = json.loads(response_str)
            return mydict
        except Exception as e:
            logging.error(e)
            return {}

    def _format_transcript(self, transcript_json: List[Dict]) -> str:
        lines = []
        for entry in transcript_json:
            role_cn = "面试官" if entry['role'] == 'interviewer' else "候选人"
            lines.append(f"[当前对话轮数：{entry['timestamp']}] {role_cn}: {entry['content']}")
        return "\n".join(lines)

    def generate_report(self, transcript_json_str: str, jd: str,interview_id:str,url_list:list[str],is_mp4:bool,interview_language:str)   :
        try:
            logging.info(f"-- 面试ID ：{interview_id} -- 接收到面试记录，开始分析")
            zero_time = time.time()
            self.interview_language=self.language_dict[interview_language][0]
            transcript_data = json.loads(transcript_json_str)
            try:
                transcript_text = self._format_transcript(transcript_data)
            except Exception as e:
                logging.info(f"-- 面试ID ：{interview_id} -- 面试文本记录解析出错，{e}，开始使用源数据")
                transcript_text = transcript_data

            # 1. 提取能力模型
            competencies = self._extract_competencies(jd,transcript_text)
            all_competency_keys = [skill for cat in competencies.values() for skill in cat]
            if not all_competency_keys :
                logging.info(f" --- 面试ID ：{interview_id} -- 考核维度抽取失败，再次尝试")
                competencies = self._extract_competencies(jd, transcript_text)
                all_competency_keys = [skill for cat in competencies.values() for skill in cat]
            logging.info(f" --- 面试ID ：{interview_id} --  考核维度抽取成功：{competencies}")

            # 2. 逐项评估各项能力
            detailed_analysis = []
            total_score = 0

            for competency in all_competency_keys:
                start_time=time.time()
                logging.info(f"  --- 面试ID ：{interview_id} --  正在分析 '{competency}'...")
                analysis_result = self._analyze_single_competency(competency, transcript_text)
                if not analysis_result :
                    logging.info(f" --- 面试ID ：{interview_id} --  分析维度：{competency} 失败，再次尝试")
                    analysis_result = self._analyze_single_competency(competency, transcript_text)
                analysis_result['competency'] = competency
                logging.info(f" --- 面试ID ：{interview_id} --  分析成功：{competency}，耗时{time.time()-start_time}s ")
                detailed_analysis.append(analysis_result)
                total_score += analysis_result.get('score', 0)

            detailed_analysis_str = json.dumps(detailed_analysis, indent=2, ensure_ascii=False)
            
            # 3. 岗位适配度分析 + 职业发展规划
            logging.info(f"  --- 面试ID ：{interview_id} --  开始岗位适配度和职业发展分析...")
            job_match = self._analyze_job_match_and_career(detailed_analysis_str, jd, transcript_text)
            if not job_match:
                logging.info(f" --- 面试ID ：{interview_id} --  岗位适配度分析失败，再次尝试")
                job_match = self._analyze_job_match_and_career(detailed_analysis_str, jd, transcript_text)
            logging.info(f"  --- 面试ID ：{interview_id} --  岗位适配度和职业发展分析完成")
            
            # 4. 性格深度分析
            logging.info(f"  --- 面试ID ：{interview_id} --  开始性格深度分析...")
            personality = self._analyze_personality(jd, transcript_text)
            if not personality:
                logging.info(f" --- 面试ID ：{interview_id} --  性格分析失败，再次尝试")
                personality = self._analyze_personality(jd, transcript_text)
            logging.info(f"  --- 面试ID ：{interview_id} --  性格深度分析完成")
            
            # 5. 薪酬市场评估
            logging.info(f"  --- 面试ID ：{interview_id} --  开始薪酬市场评估...")
            try:
                salary = self._analyze_salary_market(jd, transcript_text, detailed_analysis_str)
                if not salary:
                    logging.info(f" --- 面试ID ：{interview_id} --  薪酬评估失败，再次尝试")
                    salary = self._analyze_salary_market(jd, transcript_text, detailed_analysis_str)
                if not salary:
                    salary = {"市场薪资范围": "数据不足", "推荐薪资": "需进一步调研"}
                    logging.warning(f" --- 面试ID ：{interview_id} --  薪酬评估失败，使用默认值")
                logging.info(f"  --- 面试ID ：{interview_id} --  薪酬市场评估完成")
            except Exception as e:
                logging.error(f" --- 面试ID ：{interview_id} --  薪酬评估异常: {e}")
                salary = {"市场薪资范围": "数据不足", "推荐薪资": "需进一步调研"}
            
            # 6. 生成HR总结
            logging.info(f"  --- 面试ID ：{interview_id} --  开始生成HR总结...")
            final_summary = self._generate_final_summary(detailed_analysis_str)
            if not final_summary :
                logging.info(f" --- 面试ID ：{interview_id} --  文本报告总结失败，再次尝试 - ")
                final_summary = self._generate_final_summary(detailed_analysis_str)
            if not final_summary :
                logging.info(f" --- 面试ID ：{interview_id} --  再次尝试总结失败，主动返回code：1 状态 - ")
                a=1/0
            logging.info(f"  --- 面试ID ：{interview_id} --  HR总结生成完成")

            # 7. 多模态分析
            overall_score = round(total_score / len(all_competency_keys), 1) if all_competency_keys else -1
            logging.info(f"  --- 面试ID ：{interview_id} --  开始多模态风格分析...")
            style=style_report(url_list=url_list,interview_id=interview_id,is_mp4=is_mp4,interview_language=self.interview_language)
            if not style:
                logging.info(f"  --- 面试ID ：{interview_id} --  多模态报告评估出错，启动保底机制...")
                style=style_report(url_list=[],interview_id=interview_id,is_mp4=is_mp4,interview_language=self.interview_language)
            logging.info(f"  --- 面试ID ：{interview_id} --  多模态风格分析完成")

            # 8. 生成候选人报告
            hr_report = {
                "overall_assessment": {
                    "match_score": overall_score,
                    "conclusion": final_summary.get("conclusion"),
                    "highlights": final_summary.get("highlights"),
                    "potential_risks": final_summary.get("risks")
                },
                "detailed_analysis": detailed_analysis,
            }
            logging.info(f"  --- 面试ID ：{interview_id} --  开始生成候选人报告...")
            user_report=self._user_report(jd, transcript_text, hr_report, personality, salary)
            if not user_report:
                logging.info(f" --- 面试ID ：{interview_id} --  候选人报告生成失败，再次尝试")
                user_report=self._user_report(jd, transcript_text, hr_report, personality, salary)
            logging.info(f"  --- 面试ID ：{interview_id} --  候选人报告已生成")

            # 9. 聚合最终报告
            final_report = {
                "overall_assessment": {
                    "match_score": overall_score,
                    "conclusion": final_summary.get("conclusion"),
                    "highlights": final_summary.get("highlights"),
                    "potential_risks": final_summary.get("risks")
                },
                "detailed_analysis": detailed_analysis,
                "competency_model_used": competencies,
                "job_match_analysis": job_match,
                "personality_analysis": personality,
                "salary_analysis": salary,
                "style_report": style,
                "user_report": user_report
            }

            logging.info(f"  --- 面试ID ：{interview_id} --  最终报告已生成...")

            content_json = json.dumps(final_report, ensure_ascii=False)
            try:
                with get_session() as session:
                    stmt = (
                        update(SimulateReport)
                        .where(SimulateReport.interview_id == interview_id)
                        .values(
                            content=content_json,
                            updated_at=datetime.now()
                        )
                    )

                    session.exec(stmt)
                    session.flush()
                    logging.info(f"-- 面试ID ：{interview_id}-面试报告更新成功")
            except Exception as e:
                logging.info(f"-- 面试ID ：{interview_id}-入库失败，错误: {str(e)}")

            re = json.dumps({
                "result": final_report,
                "code": 0,
                "interview_id": int(interview_id),
            }, ensure_ascii=False)

            response = requests.request("POST", post_url, data=re)
            logging.info(f"-- 面试ID ：{interview_id} -- 面试结果评定成功，总耗时{time.time()-zero_time}s，已发送，返回结果如下：{response.text}")

        except Exception as e:
            logging.error(f"-- 面试ID ：{interview_id} -- 面试结果评定失败，错误：{e}")
            re= json.dumps({
                "result": None,
                "code": 1,
                "interview_id": int(interview_id),
                "error": str(e)
            },ensure_ascii=False)
            response = requests.request("POST", post_url, data=re)
            logging.info(f"-- 面试ID ：{interview_id} -- 面试结果评定失败！，已发送，返回结果如下：{response.text}")
            return None

```

完成！修改内容梳理如下：

## 修改内容

**文件**：`project_code/career/code/service/AIInterviewAssessor.py`

**位置**：第 415-422 行（新增）

**新增代码**：
```python
re = json.dumps({
    "result": final_report,
    "code": 0,
    "interview_id": int(interview_id),
}, ensure_ascii=False)

response = requests.request("POST", post_url, data=re)
logging.info(f"-- 面试ID ：{interview_id} -- 面试结果评定成功，总耗时{time.time()-zero_time}s，已发送，返回结果如下：{response.text}")
```

## 修改说明

在数据库更新成功后（第 413 行之后），立即执行：

1. 将 `final_report` 包装成标准格式的 JSON
2. POST 到外部系统 URL：`https://dev-api.zilieai.com/b/interview/v1/ai/interview/get/report`
3. 记录响应结果和总耗时

## 与备份代码的一致性

此修改完全复制自备份代码第 294-302 行，保持原有逻辑不变。

```python
re = json.dumps({
    "result": final_report,
    "code": 0,
    "interview_id": int(interview_id),
}, ensure_ascii=False)

response = requests.request("POST", post_url, data=re)
logging.info(f"-- 面试ID ：{interview_id} -- 面试结果评定成功，总耗时{time.time()-zero_time}s，已发送，返回结果如下：{response.text}")
```

---

**User**

https://dev-api.zilieai.com/b/interview/v1/ai/interview/get/report



你说了报告会放到这个地址但为什么我手动去访问这个报告的时候？什么也没有呢。我在浏览器上打开这个链接什么也没有呢。而且我还是不太懂这个操作逻辑啊你的意思是它是在后台运行，包括分析，然后隔了一段时间报告做好以后就直接连接到了这个网站网址，那么前端怎么知道它什么时候去访问这个地址去拿相应的报告呢？

---

**Cursor**

非常好的问题！你的困惑很正常，让我详细解释清楚：

## 为什么浏览器访问这个地址什么也没有？

### 原因：这不是一个"页面"，而是一个"接口"

```
https://dev-api.zilieai.com/b/interview/v1/ai/interview/get/report
```

这个 URL 的作用：

❌ **不是**：一个可以访问的网页（存放报告的地方）
✅ **是**：一个接收数据的 **API 接口**（接收你 POST 的报告）

### 类比说明

**错误理解**：
```
这个 URL = 一个文件夹
后台把报告放进去
前端去这个文件夹拿报告
```

**正确理解**：
```
这个 URL = 一个快递员的电话号码
后台："喂，快递员，我有个报告给你"（POST 数据）
快递员："好的我收到了，我存起来"（外部系统存储）
前端：（打另一个电话）"快递员，我来取报告"（从别的接口获取）
快递员："好的，给你报告"
```

---

## 完整的接口交互流程

### 你的后台服务有的接口：

**接口1：接收面试数据**
```
POST http://localhost:9006/interviewReport
作用：前端提交面试数据
```

### 外部系统（dev-api.zilieai.com）有的接口：

**接口2：接收报告（你 POST 的目标）**
```
POST https://dev-api.zilieai.com/b/interview/v1/ai/interview/get/report
作用：接收你后台生成的报告
权限：只接受 POST 请求，不支持 GET（所以浏览器打不开）
```

**接口3：查询报告（推测，前端用的）**
```
GET https://dev-api.zilieai.com/b/interview/v1/ai/interview/query/report?interview_id=10002
作用：前端查询生成好的报告
```

---

## 前端怎么知道什么时候去获取报告？

有几种常见方案：

### 方案1：轮询（Polling）- 最简单

```javascript
// 前端代码示例
async function checkReport(interviewId) {
    // 每隔 5 秒查询一次
    const timer = setInterval(async () => {
        const response = await fetch(
            `https://dev-api.zilieai.com/b/interview/v1/ai/interview/query/report?interview_id=${interviewId}`
        );
        const data = await response.json();
        
        if (data.status === 'completed') {
            // 报告生成完成
            clearInterval(timer);
            displayReport(data.report);
        } else if (data.status === 'processing') {
            // 还在生成中，继续等待
            console.log('报告生成中...');
        }
    }, 5000); // 每 5 秒查一次
}

// 用户提交面试数据后
submitInterview().then(result => {
    showMessage('报告生成中，请稍候...');
    checkReport(result.interview_id);
});
```

**流程图：**
```
前端提交面试数据
    ↓
显示："报告生成中..."
    ↓
每 5 秒查询一次：
    ↓
第 1 次查询 → 还在处理中
    ↓ 等待 5 秒
第 2 次查询 → 还在处理中
    ↓ 等待 5 秒
第 3 次查询 → 还在处理中
    ↓ 等待 5 秒
...
    ↓
第 N 次查询 → 完成了！
    ↓
展示报告 ✅
```

### 方案2：WebSocket - 实时通知

```javascript
// 前端建立 WebSocket 连接
const ws = new WebSocket('wss://dev-api.zilieai.com/ws');

ws.onmessage = (event) => {
    const data = JSON.parse(event.data);
    if (data.type === 'report_completed' && data.interview_id === '10002') {
        // 报告生成完成，立即获取
        fetchReport(data.interview_id);
    }
};
```

**流程图：**
```
前端 ←------ WebSocket 连接 ------→ 外部系统
                                      ↑
                                      |
你的后台 POST 报告 ----------------→ 外部系统
                                      ↓
                                  触发 WebSocket 通知
                                      ↓
前端收到通知："报告已完成"
    ↓
前端立即获取报告 ✅
```

### 方案3：回调 URL（Webhook）

```
前端提交时附带回调地址：
{
    "interview_id": "10002",
    "callback_url": "https://frontend.com/api/report-ready"
}

后台生成报告后：
    ↓
POST 到外部系统
    ↓
外部系统收到后：
    ↓
主动调用前端的 callback_url
    ↓
前端收到通知 ✅
```

---

## 你看到的实际情况

### 当前的架构：

```
┌─────────────┐                    ┌──────────────────┐
│   前端      │                    │   你的后台服务    │
│             │                    │  (localhost:9006) │
└─────────────┘                    └──────────────────┘
       │                                     │
       │                                     │
       │ ① POST 面试数据                     │
       │────────────────────────────────────→│
       │                                     │
       │ ② 立即返回："已接收"                 │
       │←────────────────────────────────────│
       │                                     │
       │                                     │ ③ 后台任务生成报告
       │                                     │    (3-10 分钟)
       │                                     │
       │                                     ↓
       │                          ┌──────────────────┐
       │                          │  数据库          │
       │                          │  存储 content     │
       │                          └──────────────────┘
       │                                     │
       │                                     │ ④ POST 报告
       │                                     │
       │                                     ↓
       │                          ┌──────────────────────────┐
       │                          │  外部系统                │
       │                          │  dev-api.zilieai.com     │
       │                          │  接收并存储报告          │
       │                          └──────────────────────────┘
       │                                     │
       │ ⑤ 轮询查询报告                       │
       │─────────────────────────────────────→│
       │      每 5 秒查一次                    │
       │                                     │
       │ ⑥ 返回报告（完成时）                 │
       │←────────────────────────────────────│
       │                                     │
       └─ ⑦ 展示报告                         │
```

---

## 为什么浏览器打不开？

### HTTP 方法限制

```
GET  https://dev-api.zilieai.com/.../get/report
❌ 不支持，返回 404 或 405

POST https://dev-api.zilieai.com/.../get/report
✅ 支持，用于接收报告
```

浏览器地址栏访问 = **GET 请求**
你的代码 `requests.request("POST", ...)` = **POST 请求**

### 接口命名的误导

虽然 URL 里有 `/get/report`，但这个接口实际上：
- **不是** GET 请求
- **是** POST 请求
- 作用是"接收（receive）报告"，不是"获取（get）报告"

**可能的完整接口设计：**
```
POST   /b/interview/v1/ai/interview/get/report         # 接收报告（你用的）
GET    /b/interview/v1/ai/interview/query/report?id=X  # 查询报告（前端用的）
```

---

## 总结

### 你的困惑点

❌ **误解**：报告存放在这个 URL，浏览器可以访问
✅ **实际**：这是个接收数据的接口，只接受 POST 请求

### 完整流程

1. 前端提交 → 你的服务
2. 你的服务生成报告（后台任务）
3. 报告存入数据库
4. **POST 报告到外部系统**（你刚补充的代码）
5. 前端轮询外部系统的**查询接口**（另一个 URL）
6. 外部系统返回报告给前端

### 关键点

- `POST .../get/report` = 你推送报告的接口
- `GET .../query/report` = 前端查询报告的接口（推测）
- 前端用**轮询**或 **WebSocket** 知道什么时候去取报告

明白了吗？

```plaintext
https://dev-api.zilieai.com/b/interview/v1/ai/interview/get/report
```

```plaintext
这个 URL = 一个文件夹
后台把报告放进去
前端去这个文件夹拿报告
```

```plaintext
这个 URL = 一个快递员的电话号码
后台："喂，快递员，我有个报告给你"（POST 数据）
快递员："好的我收到了，我存起来"（外部系统存储）
前端：（打另一个电话）"快递员，我来取报告"（从别的接口获取）
快递员："好的，给你报告"
```

```plaintext
POST http://localhost:9006/interviewReport
作用：前端提交面试数据
```

```plaintext
POST https://dev-api.zilieai.com/b/interview/v1/ai/interview/get/report
作用：接收你后台生成的报告
权限：只接受 POST 请求，不支持 GET（所以浏览器打不开）
```

```plaintext
GET https://dev-api.zilieai.com/b/interview/v1/ai/interview/query/report?interview_id=10002
作用：前端查询生成好的报告
```

```javascript
// 前端代码示例
async function checkReport(interviewId) {
    // 每隔 5 秒查询一次
    const timer = setInterval(async () => {
        const response = await fetch(
            `https://dev-api.zilieai.com/b/interview/v1/ai/interview/query/report?interview_id=${interviewId}`
        );
        const data = await response.json();
        
        if (data.status === 'completed') {
            // 报告生成完成
            clearInterval(timer);
            displayReport(data.report);
        } else if (data.status === 'processing') {
            // 还在生成中，继续等待
            console.log('报告生成中...');
        }
    }, 5000); // 每 5 秒查一次
}

// 用户提交面试数据后
submitInterview().then(result => {
    showMessage('报告生成中，请稍候...');
    checkReport(result.interview_id);
});
```

```plaintext
前端提交面试数据
    ↓
显示："报告生成中..."
    ↓
每 5 秒查询一次：
    ↓
第 1 次查询 → 还在处理中
    ↓ 等待 5 秒
第 2 次查询 → 还在处理中
    ↓ 等待 5 秒
第 3 次查询 → 还在处理中
    ↓ 等待 5 秒
...
    ↓
第 N 次查询 → 完成了！
    ↓
展示报告 ✅
```

```javascript
// 前端建立 WebSocket 连接
const ws = new WebSocket('wss://dev-api.zilieai.com/ws');

ws.onmessage = (event) => {
    const data = JSON.parse(event.data);
    if (data.type === 'report_completed' && data.interview_id === '10002') {
        // 报告生成完成，立即获取
        fetchReport(data.interview_id);
    }
};
```

```plaintext
前端 ←------ WebSocket 连接 ------→ 外部系统
                                      ↑
                                      |
你的后台 POST 报告 ----------------→ 外部系统
                                      ↓
                                  触发 WebSocket 通知
                                      ↓
前端收到通知："报告已完成"
    ↓
前端立即获取报告 ✅
```

```plaintext
前端提交时附带回调地址：
{
    "interview_id": "10002",
    "callback_url": "https://frontend.com/api/report-ready"
}

后台生成报告后：
    ↓
POST 到外部系统
    ↓
外部系统收到后：
    ↓
主动调用前端的 callback_url
    ↓
前端收到通知 ✅
```

```plaintext
┌─────────────┐                    ┌──────────────────┐
│   前端      │                    │   你的后台服务    │
│             │                    │  (localhost:9006) │
└─────────────┘                    └──────────────────┘
       │                                     │
       │                                     │
       │ ① POST 面试数据                     │
       │────────────────────────────────────→│
       │                                     │
       │ ② 立即返回："已接收"                 │
       │←────────────────────────────────────│
       │                                     │
       │                                     │ ③ 后台任务生成报告
       │                                     │    (3-10 分钟)
       │                                     │
       │                                     ↓
       │                          ┌──────────────────┐
       │                          │  数据库          │
       │                          │  存储 content     │
       │                          └──────────────────┘
       │                                     │
       │                                     │ ④ POST 报告
       │                                     │
       │                                     ↓
       │                          ┌──────────────────────────┐
       │                          │  外部系统                │
       │                          │  dev-api.zilieai.com     │
       │                          │  接收并存储报告          │
       │                          └──────────────────────────┘
       │                                     │
       │ ⑤ 轮询查询报告                       │
       │─────────────────────────────────────→│
       │      每 5 秒查一次                    │
       │                                     │
       │ ⑥ 返回报告（完成时）                 │
       │←────────────────────────────────────│
       │                                     │
       └─ ⑦ 展示报告                         │
```

```plaintext
GET  https://dev-api.zilieai.com/.../get/report
❌ 不支持，返回 404 或 405

POST https://dev-api.zilieai.com/.../get/report
✅ 支持，用于接收报告
```

```plaintext
POST   /b/interview/v1/ai/interview/get/report         # 接收报告（你用的）
GET    /b/interview/v1/ai/interview/query/report?id=X  # 查询报告（前端用的）
```

