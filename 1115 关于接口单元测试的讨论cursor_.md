
```python
"""
端到端测试 - 真实调用 OpenAI API
这些测试会产生实际费用，请谨慎运行
"""
import pytest
import sys
import os

# 添加项目路径
sys.path.insert(0, os.path.join(os.path.dirname(__file__), '..', '..', 'code'))

from service.virtual_interview import CallInterview


@pytest.mark.e2e
@pytest.mark.asyncio
class TestRealInterviewFlow:
    """真实的面试流程测试（会调用真实 OpenAI API）"""
    
    async def test_first_call_generates_question(self):
        """测试首次调用能生成第一个问题"""
        
        print("\n" + "="*60)
        print("🎯 测试场景：首次调用，应该生成面试计划并返回第一个问题")
        print("="*60)
        
        # 创建真实的面试服务（不 Mock）
        service = CallInterview(interview_id="e2e_test_001")
        service.user_portrait = "Python工程师，3年工作经验，熟悉Django和FastAPI框架"
        service.jd_title = "高级后端工程师"
        service.jd_portrait = "要求5年以上Python开发经验，熟悉微服务架构"
        service.interview_type = 1
        service.interview_language = "zhongwen"
        
        print(f"\n📋 输入信息：")
        print(f"   候选人: {service.user_portrait[:30]}...")
        print(f"   岗位: {service.jd_title}")
        
        # 真实调用（首次调用，空的对话历史）
        print(f"\n🚀 开始调用真实 API...")
        
        chunks = []
        async for chunk in service.get_ai_response([]):
            chunks.append(chunk)
            # 实时显示返回内容
            if 'content' in chunk:
                import json
                try:
                    data = json.loads(chunk.split('data: ')[1])
                    if data.get('type') == 'chunk':
                        print(data.get('content', ''), end='', flush=True)
                except:
                    pass
        
        response = ''.join(chunks)
        
        print(f"\n\n✅ API 调用完成")
        print(f"\n📊 验证结果：")
        
        # 验证 1：有返回内容
        assert len(response) > 0, "应该有返回内容"
        print(f"   ✓ 返回内容长度: {len(response)} 字符")
        
        # 验证 2：包含 SSE 格式
        assert "data:" in response, "应该是 SSE 格式"
        print(f"   ✓ 返回格式正确（SSE）")
        
        # 验证 3：包含 DONE 标记
        assert "[DONE]" in response, "应该有结束标记"
        print(f"   ✓ 包含结束标记")
        
        # 验证 4：面试计划已生成
        assert service.interview_plan is not None, "应该生成了面试计划"
        print(f"   ✓ 面试计划已生成")
        
        if service.interview_plan:
            question_count = len(service.interview_plan.get('questions', []))
            print(f"   ✓ 计划包含 {question_count} 个问题")
        
        # 验证 5：当前问题索引已更新
        assert service.current_question_index >= 1, "问题索引应该递增"
        print(f"   ✓ 当前问题索引: {service.current_question_index}")
        
        print(f"\n{'='*60}")
        print("🎉 测试通过！接口运行正常")
        print(f"{'='*60}\n")
    
    async def test_response_to_incomplete_answer(self):
        """测试对不完整回答的响应（可能触发追问）"""
        
        print("\n" + "="*60)
        print("🎯 测试场景：候选人给出不完整回答")
        print("="*60)
        
        service = CallInterview(interview_id="e2e_test_002")
        service.user_portrait = "Python工程师，2年经验"
        service.jd_title = "后端工程师"
        service.jd_portrait = "要求Python、微服务经验"
        service.interview_type = 1
        service.interview_language = "zhongwen"
        
        # 模拟一轮对话（候选人回答不完整）
        dialog_history = [
            {"role": "interviewer", "content": "请介绍一下你的微服务开发经验"},
            {"role": "candidate", "content": "我用过一点微服务..."}
        ]
        
        print(f"\n📋 对话历史：")
        print(f"   面试官: {dialog_history[0]['content']}")
        print(f"   候选人: {dialog_history[1]['content']}")
        
        print(f"\n🚀 开始调用真实 API...")
        
        chunks = []
        async for chunk in service.get_ai_response(dialog_history):
            chunks.append(chunk)
            # 实时显示
            if 'content' in chunk:
                import json
                try:
                    data = json.loads(chunk.split('data: ')[1])
                    if data.get('type') == 'chunk':
                        print(data.get('content', ''), end='', flush=True)
                except:
                    pass
        
        response = ''.join(chunks)
        
        print(f"\n\n✅ API 调用完成")
        print(f"\n📊 验证结果：")
        
        # 验证基本功能
        assert len(response) > 0, "应该有返回内容"
        print(f"   ✓ 返回内容长度: {len(response)} 字符")
        
        assert "data:" in response, "应该是 SSE 格式"
        print(f"   ✓ 返回格式正确")
        
        # 这里不强制验证是否追问，因为 LLM 输出不确定
        # 只验证接口能正常工作
        print(f"   ✓ 接口正常响应")
        
        print(f"\n{'='*60}")
        print("🎉 测试通过！接口能处理对话历史")
        print(f"{'='*60}\n")


if __name__ == "__main__":
    # 可以直接运行这个文件
    import asyncio
    
    print("\n" + "🔥"*30)
    print("端到端测试 - 真实调用 OpenAI API")
    print("⚠️  警告：这将产生实际的 API 调用费用")
    print("🔥"*30 + "\n")
    
    test = TestRealInterviewFlow()
    
    print("运行测试 1: 首次调用...")
    asyncio.run(test.test_first_call_generates_question())
    
    print("\n\n运行测试 2: 响应不完整回答...")
    asyncio.run(test.test_response_to_incomplete_answer())



```

好了！这次应该对了！我的错误是：

**问题所在**：
- ❌ 我之前试图导入 `from code.service.virtual_interview`，但 `code` 作为顶层包名可能有问题
- ✅ 正确做法：把路径直接指向 `code` 目录，然后导入 `from service.virtual_interview`

**修改内容**：
1. 路径改为：`sys.path.insert(0, os.path.join(os.path.dirname(__file__), '..', '..', 'code'))`
2. 导入改为：`from service.virtual_interview import CallInterview`

这样和 `manual_interview.py` 的做法完全一致了。

现在再试试运行：

```bash
cd "E:\Python Code\cv-project"
pytest project_code/career/tests/e2e/test_real_interview_flow.py -v -s
```

这次应该能正常导入了！真的抱歉刚才没仔细看 `manual_interview.py` 的实现方式 🙏

```shellscript
cd "E:\Python Code\cv-project"
pytest project_code/career/tests/e2e/test_real_interview_flow.py -v -s
```


