# DeepResearch 实验指导 04：Evaluation 模块

## 实验目的

1. **理解评估系统的工作原理**：学习如何评估多智能体系统的性能
2. **掌握评估指标设计**：了解准确率、置信度、一致性等评估维度
3. **实现自动评估流程**：构建完整的评估流水线
4. **学习评估结果分析**：掌握如何解读评估结果并改进系统

## 实验原理

### 1. 评估系统架构

DeepResearch 的评估系统采用三层架构：

```
原始输出 → 答案提取 → 评估判断 → 结果统计
    ↓           ↓           ↓         ↓
智能体响应 → 结构化答案 → 正确性判断 → 性能报告
```

### 2. 核心评估流程

1. **答案提取阶段**：从智能体的自然语言响应中提取结构化答案
2. **评估判断阶段**：使用评估模型（Judge Model）判断答案正确性
3. **结果统计阶段**：计算各项性能指标并生成报告

### 3. 关键技术组件

- **提示词工程**：设计评估专用的系统提示词
- **结构化输出**：定义标准化的答案格式
- **并发评估**：支持大规模并行评估
- **置信度评分**：为每个评估结果提供置信度

## 实验内容和步骤

### 步骤 1：环境准备

```python
# evaluation/environment.py
import os
from typing import Dict, Any
from dataclasses import dataclass

@dataclass
class EvaluationConfig:
    """评估系统配置"""
    judge_model: str = "openai/o3-mini"
    max_workers: int = 10
    api_key: str = None
    api_base: str = None
    
    def __post_init__(self):
        """从环境变量加载配置"""
        self.api_key = os.getenv("OPENAI_API_KEY", self.api_key)
        self.api_base = os.getenv("OPENAI_API_BASE", self.api_base)
        
class EvaluationEnvironment:
    """评估环境管理器"""
    
    def __init__(self, config: EvaluationConfig = None):
        self.config = config or EvaluationConfig()
        self._setup_environment()
    
    def _setup_environment(self):
        """设置评估环境"""
        # 设置必要的环境变量
        os.environ.setdefault("EVALUATION_MODE", "official")
        os.environ.setdefault("MAX_WORKERS", str(self.config.max_workers))
        
    def validate(self) -> bool:
        """验证环境配置"""
        required_vars = ["OPENAI_API_KEY"]
        missing = [var for var in required_vars if not os.getenv(var)]
        
        if missing:
            print(f"缺少环境变量: {missing}")
            return False
        return True
```

### 步骤 2：提示词系统实现

```python
# evaluation/prompts.py
from typing import Dict, Any

class EvaluationPrompts:
    """评估提示词系统"""
    
    @staticmethod
    def get_system_prompt() -> str:
        """获取评估系统提示词"""
        return """You are a Web Information Seeking Master. Your task is to thoroughly seek the internet for information and provide accurate answers to questions. No matter how complex the query, you will not give up until you find the corresponding information.

As you proceed, adhere to the following principles:

1. **Persistent Actions for Answers**: You will engage in many interactions, delving deeply into the topic to explore all possible aspects until a satisfactory answer is found.

2. **Repeated Verification**: Before presenting a Final Answer, you will **cross-check** and **validate the information** you've gathered to confirm its accuracy and reliability.

3. **Attention to Detail**: You will carefully analyze each information source to ensure that all data is current, relevant, and from credible origins."""

    @staticmethod
    def get_extractor_prompt() -> str:
        """获取答案提取提示词"""
        return """## **Task Guidelines**
1. **Content Scanning for Rational**: Locate the **specific sections/data** directly related to the user's goal within the webpage content
2. **Key Extraction for Evidence**: Identify and extract the **most relevant information** from the content, you never miss any important information, output the **full original context** of the content as far as possible, it can be more than three paragraphs.
3. **Summary Output for Summary**: Organize into a concise paragraph with logical flow, prioritizing clarity and judge the contribution of the information to the goal.

## **Output Format**
You must output a JSON object with the following structure:
{
  "rational": "Detailed reasoning about how the content relates to the goal",
  "evidence": "Full original context extracted from the content",
  "summary": "Concise summary of the information's contribution to the goal"
}"""

    @staticmethod
    def get_judge_prompt(question: str, response: str, correct_answer: str) -> str:
        """获取评估判断提示词"""
        return f"""Judge whether the following [response] to [question] is correct or not based on the precise and unambiguous [correct_answer] below.

[question]: {question}

[response]: {response}

Your judgement must be in the format and criteria specified below:

extracted_final_answer: The final exact answer extracted from the [response]. Put the extracted answer as 'None' if there is no exact, final answer to extract from the response.

[correct_answer]: {correct_answer}

reasoning: Explain why the extracted_final_answer is correct or incorrect based on [correct_answer], focusing only on if there are meaningful differences between [correct_answer] and the extracted_final_answer. Do not comment on any background to the problem, do not attempt to solve the problem, do not argue for any answer different than [correct_answer], focus only on whether the answers match.

correct: Answer 'yes' if extracted_final_answer matches the [correct_answer] given above, or is within a small margin of error for numerical problems, otherwise answer 'no'."""
```

### 步骤 3：答案提取器实现

```python
# evaluation/extractor.py
import json
from typing import Dict, Any, Optional
from pydantic import BaseModel, Field

class ExtractedAnswer(BaseModel):
    """提取的答案结构"""
    rational: str = Field(description="详细推理过程")
    evidence: str = Field(description="原始证据内容")
    summary: str = Field(description="信息总结")
    
class ExtractedAnswerForConfidence(BaseModel):
    """带置信度的答案结构"""
    rational: str = Field(description="详细推理过程")
    evidence: str = Field(description="原始证据内容")
    summary: str = Field(description="信息总结")
    confidence: float = Field(description="置信度分数", ge=0.0, le=1.0)

class AnswerExtractor:
    """答案提取器"""
    
    def __init__(self, client):
        self.client = client
        
    async def extract_answer(self, response: str, format_type: str = "standard") -> Dict[str, Any]:
        """从响应中提取结构化答案"""
        
        if format_type == "confidence":
            schema = ExtractedAnswerForConfidence
        elif format_type == "xbench":
            # 中文格式
            schema = type("XBenchAnswer", (BaseModel,), {
                "推理过程": (str, Field(description="详细推理过程")),
                "证据内容": (str, Field(description="原始证据内容")),
                "总结": (str, Field(description="信息总结"))
            })
        else:
            schema = ExtractedAnswer
        
        # 调用模型进行提取
        prompt = EvaluationPrompts.get_extractor_prompt()
        
        try:
            completion = await self.client.chat.completions.create(
                model="gpt-4",
                messages=[
                    {"role": "system", "content": prompt},
                    {"role": "user", "content": response}
                ],
                response_format={"type": "json_object"}
            )
            
            result = json.loads(completion.choices[0].message.content)
            return schema(**result).dict()
            
        except Exception as e:
            print(f"提取答案失败: {e}")
            return {}
```

### 步骤 4：评估器实现

```python
# evaluation/evaluator.py
import asyncio
from typing import List, Dict, Any, Optional
from concurrent.futures import ThreadPoolExecutor
import json

class EvaluationResult(BaseModel):
    """评估结果"""
    question: str
    response: str
    extracted_answer: Dict[str, Any]
    correct_answer: str
    is_correct: bool
    reasoning: str
    confidence: Optional[float] = None
    
class Evaluator:
    """评估器"""
    
    def __init__(self, config: EvaluationConfig):
        self.config = config
        self.executor = ThreadPoolExecutor(max_workers=config.max_workers)
        
    async def evaluate_single(self, 
                            question: str, 
                            response: str, 
                            correct_answer: str) -> EvaluationResult:
        """评估单个问答对"""
        
        # 1. 提取答案
        extractor = AnswerExtractor(self._get_client())
        extracted = await extractor.extract_answer(response)
        
        # 2. 判断正确性
        is_correct, reasoning = await self._judge_correctness(
            question, response, correct_answer
        )
        
        return EvaluationResult(
            question=question,
            response=response,
            extracted_answer=extracted,
            correct_answer=correct_answer,
            is_correct=is_correct,
            reasoning=reasoning
        )
    
    async def evaluate_batch(self, 
                           data: List[Dict[str, str]]) -> List[EvaluationResult]:
        """批量评估"""
        tasks = []
        for item in data:
            task = self.evaluate_single(
                item["question"],
                item["response"],
                item["correct_answer"]
            )
            tasks.append(task)
        
        return await asyncio.gather(*tasks)
    
    async def _judge_correctness(self, 
                               question: str, 
                               response: str, 
                               correct_answer: str) -> tuple[bool, str]:
        """判断答案正确性"""
        
        prompt = EvaluationPrompts.get_judge_prompt(
            question, response, correct_answer
        )
        
        try:
            completion = await self._get_client().chat.completions.create(
                model=self.config.judge_model,
                messages=[
                    {"role": "system", "content": "You are an impartial judge."},
                    {"role": "user", "content": prompt}
                ]
            )
            
            result = completion.choices[0].message.content
            # 解析结果
            lines = result.strip().split('\n')
            is_correct = "yes" in lines[-1].lower() if lines else False
            reasoning = "\n".join(lines[:-1]) if len(lines) > 1 else result
            
            return is_correct, reasoning
            
        except Exception as e:
            print(f"评估失败: {e}")
            return False, f"评估错误: {str(e)}"
    
    def _get_client(self):
        """获取评估客户端"""
        # 这里应该返回配置好的客户端
        # 实际实现中会使用 OpenAI 或其他模型客户端
        pass
```

### 步骤 5：评估流水线

```python
# evaluation/pipeline.py
import asyncio
from typing import List, Dict, Any
from dataclasses import dataclass
import json
from pathlib import Path

@dataclass
class EvaluationMetrics:
    """评估指标"""
    total: int = 0
    correct: int = 0
    accuracy: float = 0.0
    avg_confidence: float = 0.0
    details: List[Dict[str, Any]] = None
    
    def __post_init__(self):
        if self.details is None:
            self.details = []
    
    def update(self, result: Dict[str, Any]):
        """更新指标"""
        self.total += 1
        if result.get("is_correct", False):
            self.correct += 1
        self.details.append(result)
        self.accuracy = self.correct / self.total if self.total > 0 else 0.0
        
    def to_dict(self) -> Dict[str, Any]:
        """转换为字典"""
        return {
            "total": self.total,
            "correct": self.correct,
            "accuracy": self.accuracy,
            "avg_confidence": self.avg_confidence,
            "details": self.details
        }

class EvaluationPipeline:
    """评估流水线"""
    
    def __init__(self, config_path: str = None):
        self.config = self._load_config(config_path)
        self.evaluator = Evaluator(self.config)
        self.metrics = EvaluationMetrics()
        
    def _load_config(self, config_path: str) -> EvaluationConfig:
        """加载配置"""
        if config_path and Path(config_path).exists():
            with open(config_path, 'r', encoding='utf-8') as f:
                config_data = json.load(f)
                return EvaluationConfig(**config_data)
        return EvaluationConfig()
    
    async def run(self, 
                 input_data: List[Dict[str, str]],
                 output_path: str = None) -> EvaluationMetrics:
        """运行评估流水线"""
        
        print(f"开始评估 {len(input_data)} 个样本...")
        
        # 1. 批量评估
        results = await self.evaluator.evaluate_batch(input_data)
        
        # 2. 计算指标
        for result in results:
            self.metrics.update(result.dict())
        
        # 3. 保存结果
        if output_path:
            self._save_results(output_path)
        
        print(f"评估完成！准确率: {self.metrics.accuracy:.2%}")
        return self.metrics
    
    def _save_results(self, output_path: str):
        """保存评估结果"""
        output_dir = Path(output_path).parent
        output_dir.mkdir(parents=True, exist_ok=True)
        
        # 保存详细结果
        with open(output_path, 'w', encoding='utf-8') as f:
            json.dump(self.metrics.to_dict(), f, ensure_ascii=False, indent=2)
        
        # 保存摘要报告
        summary_path = output_dir / "evaluation_summary.txt"
        with open(summary_path, 'w', encoding='utf-8') as f:
            f.write(self._generate_summary())
    
    def _generate_summary(self) -> str:
        """生成摘要报告"""
        return f"""评估报告
==========

总样本数: {self.metrics.total}
正确数: {self.metrics.correct}
准确率: {self.metrics.accuracy:.2%}

详细结果已保存至 JSON 文件。
"""
```

### 步骤 6：完整示例

```python
# evaluation/example.py
import asyncio
from evaluation.pipeline import EvaluationPipeline

async def main():
    """评估示例"""
    
    # 准备测试数据
    test_data = [
        {
            "question": "Python 中如何读取文件？",
            "response": "可以使用 open() 函数读取文件，例如：with open('file.txt', 'r') as f: content = f.read()",
            "correct_answer": "使用 open() 函数，模式为 'r'，配合 with 语句"
        },
        {
            "question": "什么是异步编程？",
            "response": "异步编程允许程序在等待 I/O 操作时执行其他任务，提高效率",
            "correct_answer": "一种编程模式，允许非阻塞执行，提高 I/O 密集型应用性能"
        }
    ]
    
    # 创建评估流水线
    pipeline = EvaluationPipeline()
    
    # 运行评估
    metrics = await pipeline.run(
        input_data=test_data,
        output_path="results/evaluation_results.json"
    )
    
    # 打印结果
    print(f"评估完成！")
    print(f"总样本: {metrics.total}")
    print(f"正确数: {metrics.correct}")
    print(f"准确率: {metrics.accuracy:.2%}")

if __name__ == "__main__":
    asyncio.run(main())
```

## 实验总结

### 1. 核心收获

**评估系统架构理解**：
- 掌握了三层评估架构：提取 → 判断 → 统计
- 理解了提示词在评估中的关键作用
- 学会了如何设计结构化输出格式

**技术实现能力**：
- 实现了完整的答案提取器
- 构建了基于模型的评估判断系统
- 掌握了批量并发评估技术

**系统集成经验**：
- 学会了如何将评估系统集成到多智能体框架中
- 理解了评估结果如何反馈改进智能体性能
- 掌握了评估指标的设计和计算方法

### 2. 关键技术要点

**提示词工程**：
- 评估系统提示词需要明确、具体
- 结构化输出格式必须严格定义
- 评估标准需要客观、可量化

**并发处理**：
- 使用 ThreadPoolExecutor 实现并发评估
- 合理设置最大工作线程数
- 处理并发中的异常和超时

**结果分析**：
- 不仅要看准确率，还要分析错误原因
- 置信度评分可以提供更多信息
- 详细日志有助于问题诊断

### 3. 实际应用建议

**生产环境优化**：
- 添加缓存机制减少重复评估
- 实现增量评估支持
- 添加监控和告警功能

**扩展性考虑**：
- 支持多种评估模型
- 可配置的评估标准
- 模块化的评估组件

**性能优化**：
- 批量处理优化
- 异步 I/O 优化
- 内存使用优化

### 4. 与其他模块的集成

**与 WebAgent 集成**：
```python
# 在 WebAgent 训练后自动评估
async def train_and_evaluate(agent, training_data, test_data):
    # 1. 训练智能体
    await agent.train(training_data)
    
    # 2. 生成测试响应
    responses = await agent.batch_predict(test_data["questions"])
    
    # 3. 评估性能
    evaluator = Evaluator()
    results = await evaluator.evaluate_batch(
        zip(test_data["questions"], responses, test_data["answers"])
    )
    
    return results
```

**与 Inference 模块集成**：
```python
# 评估推理服务的准确性
async def evaluate_inference_service(service, test_cases):
    evaluator = Evaluator()
    
    for test_case in test_cases:
        # 调用推理服务
        response = await service.infer(test_case["input"])
        
        # 评估结果
        result = await evaluator.evaluate_single(
            test_case["question"],
            response,
            test_case["expected"]
        )
        
        yield result
```

### 5. 下一步学习方向

**高级评估技术**：
- 多维度评估（准确性、相关性、完整性）
- 基于人类反馈的评估
- A/B 测试框架

**自动化优化**：
- 基于评估结果的自动调参
- 在线学习系统
- 自适应评估标准

**大规模部署**：
- 分布式评估系统
- 实时评估流水线
- 评估结果可视化

通过本实验，你已经掌握了 DeepResearch 评估系统的核心原理和实现方法。评估系统是确保多智能体系统质量的关键组件，也是持续改进的基础。在实际应用中，需要根据具体需求调整评估标准和实现细节。