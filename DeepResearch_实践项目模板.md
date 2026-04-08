# DeepResearch 实践项目模板

## 项目概述

本模板提供了一个标准化的项目结构，帮助你基于 DeepResearch 框架快速开发自定义应用。模板包含了所有必要的组件和配置，让你可以专注于业务逻辑的实现。

## 项目结构

```
my_deepresearch_project/
├── config/                    # 配置文件
│   ├── __init__.py
│   ├── settings.py           # 主配置
│   ├── agents.yaml           # 智能体配置
│   └── models.yaml           # 模型配置
├── src/                      # 源代码
│   ├── __init__.py
│   ├── agents/              # 自定义智能体
│   │   ├── __init__.py
│   │   ├── base.py          # 智能体基类
│   │   ├── custom_agent.py  # 自定义智能体
│   │   └── coordinator.py   # 协调器
│   ├── tools/               # 自定义工具
│   │   ├── __init__.py
│   │   ├── base.py          # 工具基类
│   │   ├── web_tools.py     # 网络工具
│   │   └── data_tools.py    # 数据处理工具
│   ├── inference/           # 推理服务
│   │   ├── __init__.py
│   │   ├── engine.py        # 推理引擎
│   │   ├── models.py        # 模型管理
│   │   └── prompts.py       # 提示词模板
│   ├── evaluation/          # 评估系统
│   │   ├── __init__.py
│   │   ├── evaluator.py     # 评估器
│   │   ├── metrics.py       # 评估指标
│   │   └── pipeline.py      # 评估流水线
│   └── utils/               # 工具函数
│       ├── __init__.py
│       ├── logging.py       # 日志配置
│       ├── data_loader.py   # 数据加载
│       └── validator.py     # 数据验证
├── tests/                   # 测试代码
│   ├── __init__.py
│   ├── test_agents.py
│   ├── test_tools.py
│   └── test_integration.py
├── data/                    # 数据文件
│   ├── raw/                # 原始数据
│   ├── processed/          # 处理后的数据
│   └── results/            # 结果文件
├── notebooks/              # Jupyter 笔记本
│   ├── exploration.ipynb   # 数据探索
│   ├── training.ipynb      # 训练笔记本
│   └── evaluation.ipynb    # 评估笔记本
├── scripts/                # 脚本文件
│   ├── setup.py           # 环境设置
│   ├── train.py           # 训练脚本
│   └── serve.py           # 服务启动脚本
├── docs/                   # 文档
│   ├── api.md             # API 文档
│   ├── architecture.md    # 架构文档
│   └── deployment.md      # 部署指南
├── .env.example           # 环境变量示例
├── requirements.txt       # Python 依赖
├── pyproject.toml         # 项目配置
├── README.md              # 项目说明
└── Dockerfile             # Docker 配置
```

## 快速开始

### 1. 项目初始化

```bash
# 1. 创建项目目录
mkdir my_deepresearch_project
cd my_deepresearch_project

# 2. 创建虚拟环境
python -m venv venv
source venv/bin/activate  # Linux/Mac
# 或
venv\Scripts\activate     # Windows

# 3. 安装基础依赖
pip install torch transformers aiohttp fastapi pydantic
```

### 2. 基础配置

```python
# config/settings.py
import os
from typing import Dict, Any
from dataclasses import dataclass

@dataclass
class ProjectSettings:
    """项目设置"""
    
    # 项目信息
    project_name: str = "My DeepResearch Project"
    version: str = "1.0.0"
    
    # 路径配置
    data_dir: str = "data"
    logs_dir: str = "logs"
    models_dir: str = "models"
    
    # 模型配置
    default_model: str = "gpt-3.5-turbo"
    embedding_model: str = "text-embedding-ada-002"
    
    # 性能配置
    max_workers: int = 10
    timeout_seconds: int = 30
    retry_attempts: int = 3
    
    # 日志配置
    log_level: str = "INFO"
    log_format: str = "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
    
    @classmethod
    def from_env(cls) -> 'ProjectSettings':
        """从环境变量加载配置"""
        return cls(
            project_name=os.getenv("PROJECT_NAME", cls.project_name),
            default_model=os.getenv("DEFAULT_MODEL", cls.default_model),
            max_workers=int(os.getenv("MAX_WORKERS", cls.max_workers)),
            log_level=os.getenv("LOG_LEVEL", cls.log_level)
        )

# 全局配置实例
settings = ProjectSettings.from_env()
```

### 3. 基础智能体模板

```python
# src/agents/base.py
import asyncio
from typing import Dict, Any, List, Optional
from abc import ABC, abstractmethod
from dataclasses import dataclass

@dataclass
class AgentState:
    """智能体状态"""
    memory: List[Dict[str, Any]] = None
    context: Dict[str, Any] = None
    tools: List[Any] = None
    
    def __post_init__(self):
        if self.memory is None:
            self.memory = []
        if self.context is None:
            self.context = {}
        if self.tools is None:
            self.tools = []

class BaseAgent(ABC):
    """智能体基类"""
    
    def __init__(self, name: str, description: str = ""):
        self.name = name
        self.description = description
        self.state = AgentState()
        
    @abstractmethod
    async def think(self, input_data: Dict[str, Any]) -> Dict[str, Any]:
        """思考阶段：分析输入，制定计划"""
        pass
    
    @abstractmethod
    async def act(self, plan: Dict[str, Any]) -> Dict[str, Any]:
        """行动阶段：执行计划，使用工具"""
        pass
    
    @abstractmethod
    async def reflect(self, result: Dict[str, Any]) -> Dict[str, Any]:
        """反思阶段：评估结果，更新记忆"""
        pass
    
    async def run(self, input_data: Dict[str, Any]) -> Dict[str, Any]:
        """运行完整的 React 循环"""
        
        # 1. 思考
        print(f"[{self.name}] 思考中...")
        plan = await self.think(input_data)
        
        # 2. 行动
        print(f"[{self.name}] 执行中...")
        result = await self.act(plan)
        
        # 3. 反思
        print(f"[{self.name}] 反思中...")
        final_result = await self.reflect(result)
        
        # 4. 更新记忆
        self.state.memory.append({
            "input": input_data,
            "plan": plan,
            "result": result,
            "final_result": final_result,
            "timestamp": asyncio.get_event_loop().time()
        })
        
        return final_result
    
    def add_tool(self, tool: Any):
        """添加工具"""
        self.state.tools.append(tool)
        
    def get_memory_summary(self) -> str:
        """获取记忆摘要"""
        return f"记忆条目数: {len(self.state.memory)}"
```

### 4. 自定义智能体示例

```python
# src/agents/custom_agent.py
from typing import Dict, Any
from .base import BaseAgent, AgentState

class ResearchAgent(BaseAgent):
    """研究智能体"""
    
    def __init__(self, name: str = "ResearchAgent", specialization: str = "general"):
        super().__init__(name, f"专注于 {specialization} 领域的研究智能体")
        self.specialization = specialization
        
    async def think(self, input_data: Dict[str, Any]) -> Dict[str, Any]:
        """制定研究计划"""
        query = input_data.get("query", "")
        
        plan = {
            "query": query,
            "steps": [
                {"action": "search", "target": "general_info"},
                {"action": "analyze", "target": "key_points"},
                {"action": "synthesize", "target": "final_report"}
            ],
            "expected_output": "包含关键发现的研究报告",
            "specialization": self.specialization
        }
        
        return plan
    
    async def act(self, plan: Dict[str, Any]) -> Dict[str, Any]:
        """执行研究计划"""
        results = []
        
        for step in plan["steps"]:
            action = step["action"]
            target = step["target"]
            
            # 使用工具执行步骤
            tool_result = await self._use_tool(action, target, plan["query"])
            results.append({
                "step": step,
                "result": tool_result
            })
            
        return {
            "plan": plan,
            "step_results": results,
            "status": "completed"
        }
    
    async def reflect(self, result: Dict[str, Any]) -> Dict[str, Any]:
        """反思研究结果"""
        # 评估结果质量
        quality_score = self._evaluate_quality(result)
        
        # 提取关键信息
        key_findings = self._extract_key_findings(result)
        
        final_result = {
            "query": result["plan"]["query"],
            "key_findings": key_findings,
            "quality_score": quality_score,
            "specialization": self.specialization,
            "timestamp": asyncio.get_event_loop().time()
        }
        
        return final_result
    
    async def _use_tool(self, action: str, target: str, query: str) -> Dict[str, Any]:
        """使用工具执行操作"""
        # 这里应该调用具体的工具
        # 示例实现
        return {
            "action": action,
            "target": target,
            "query": query,
            "result": f"执行 {action} 于 {target}，查询: {query}"
        }
    
    def _evaluate_quality(self, result: Dict[str, Any]) -> float:
        """评估结果质量"""
        # 简单的质量评估逻辑
        steps_completed = len(result.get("step_results", []))
        total_steps = len(result.get("plan", {}).get("steps", []))
        
        if total_steps == 0:
            return 0.0
            
        completion_rate = steps_completed / total_steps
        return min(completion_rate * 1.2, 1.0)  # 最高1.0
    
    def _extract_key_findings(self, result: Dict[str, Any]) -> List[str]:
        """提取关键发现"""
        findings = []
        
        for step_result in result.get("step_results", []):
            result_text = step_result.get("result", {}).get("result", "")
            if result_text:
                findings.append(f"步骤 {step_result['step']['action']}: {result_text}")
                
        return findings
```

### 5. 推理服务模板

```python
# src/inference/engine.py
import asyncio
from typing import Dict, Any, List, Optional
from dataclasses import dataclass
import aiohttp

@dataclass
class InferenceRequest:
    """推理请求"""
    prompt: str
    model: str = "gpt-3.5-turbo"
    temperature: float = 0.7
    max_tokens: int = 1000
    system_prompt: Optional[str] = None

@dataclass
class InferenceResponse:
    """推理响应"""
    content: str
    model: str
    usage: Dict[str, int]
    latency: float

class InferenceEngine:
    """推理引擎"""
    
    def __init__(self, api_key: str = None, api_base: str = None):
        self.api_key = api_key
        self.api_base = api_base or "https://api.openai.com/v1"
        self.session = None
        
    async def __aenter__(self):
        await self.start()
        return self
        
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        await self.stop()
        
    async def start(self):
        """启动引擎"""
        self.session = aiohttp.ClientSession(
            headers={
                "Authorization": f"Bearer {self.api_key}",
                "Content-Type": "application/json"
            }
        )
        
    async def stop(self):
        """停止引擎"""
        if self.session:
            await self.session.close()
            
    async def infer(self, request: InferenceRequest) -> InferenceResponse:
        """执行推理"""
        start_time = asyncio.get_event_loop().time()
        
        messages = []
        if request.system_prompt:
            messages.append({"role": "system", "content": request.system_prompt})
        messages.append({"role": "user", "content": request.prompt})
        
        payload = {
            "model": request.model,
            "messages": messages,
            "temperature": request.temperature,
            "max_tokens": request.max_tokens
        }
        
        try:
            async with self.session.post(
                f"{self.api_base}/chat/completions",
                json=payload
            ) as response:
                data = await response.json()
                
                latency = asyncio.get_event_loop().time() - start_time
                
                return InferenceResponse(
                    content=data["choices"][0]["message"]["content"],
                    model=request.model,
                    usage=data.get("usage", {}),
                    latency=latency
                )
                
        except Exception as e:
            raise InferenceError(f"推理失败: {str(e)}")

class InferenceError(Exception):
    """推理错误"""
    pass
```

### 6. 评估系统模板

```python
# src/evaluation/evaluator.py
import asyncio
from typing import Dict, Any, List
from dataclasses import dataclass
from enum import Enum

class EvaluationMetric(Enum):
    """评估指标"""
    ACCURACY = "accuracy"
    RELEVANCE = "relevance"
    COMPLETENESS = "completeness"
    COHERENCE = "coherence"

@dataclass
class EvaluationResult:
    """评估结果"""
    metric: EvaluationMetric
    score: float
    explanation: str
    confidence: float = 1.0

class Evaluator:
    """评估器"""
    
    def __init__(self, inference_engine = None):
        self.inference_engine = inference_engine
        
    async def evaluate(self, 
                      query: str, 
                      response: str, 
                      expected: str = None,
                      metrics: List[EvaluationMetric] = None) -> List[EvaluationResult]:
        """评估响应"""
        
        if metrics is None:
            metrics = [EvaluationMetric.ACCURACY, EvaluationMetric.RELEVANCE]
            
        results = []
        
        for metric in metrics:
            score, explanation = await self._evaluate_metric(
                metric, query, response, expected
            )
            
            results.append(EvaluationResult(
                metric=metric,
                score=score,
                explanation=explanation
            ))
            
        return results
    
    async def _evaluate_metric(self, 
                             metric: EvaluationMetric,
                             query: str,
                             response: str,
                             expected: str = None) -> tuple[float, str]:
        """评估单个指标"""
        
        if metric == EvaluationMetric.ACCURACY:
            return await self._evaluate_accuracy(query, response, expected)
        elif metric == EvaluationMetric.RELEVANCE:
            return await self._evaluate_relevance(query, response)
        elif metric == EvaluationMetric.COMPLETENESS:
            return await self._evaluate_completeness(query, response)
        else:
            return 0.5, "未实现的评估指标"
    
    async def _evaluate_accuracy(self, query: str, response: str, expected: str) -> tuple[float, str]:
        """评估准确性"""
        if not expected:
            return 0.5, "缺少预期答案，无法评估准确性"
            
        # 使用推理引擎评估
        prompt = f"""请评估以下回答的准确性：

问题：{query}
预期答案：{expected}
实际回答：{response}

请给出0-1的分数和简要解释。"""
        
        if self.inference_engine:
            inference_response = await self.inference_engine.infer(
                InferenceRequest(prompt=prompt)
            )
            
            # 解析响应
            content = inference_response.content
            # 这里应该解析分数和解释
            score = 0.7  # 示例
            explanation = content[:100] + "..."
        else:
            # 简单实现
            score = 0.8 if expected.lower() in response.lower() else 0.3
            explanation = "基于关键词匹配的简单评估"
            
        return score, explanation
```

### 7. 完整示例应用

```python
# examples/complete_example.py
import asyncio
import os
from src.agents.custom_agent import ResearchAgent
from src.inference.engine import InferenceEngine, InferenceRequest
from src.evaluation.evaluator import Evaluator, EvaluationMetric

async def main():
    """完整示例"""
    
    # 1. 初始化组件
    print("初始化组件...")
    
    # 推理引擎
    inference = InferenceEngine(
        api_key=os.getenv("OPENAI_API_KEY")
    )
    await inference.start()
    
    # 研究智能体
    researcher = ResearchAgent(
        name="ScienceResearcher",
        specialization="科学研究"
    )
    
    # 评估器
    evaluator = Evaluator(inference_engine=inference)
    
    # 2. 执行研究任务
    print("\n执行研究任务...")
    
    research_query = {
        "query": "人工智能在医疗诊断中的应用",
        "depth": "detailed",
        "sources": ["academic", "recent"]
    }
    
    result = await researcher.run(research_query)
    
    print(f"\n研究完成！")
    print(f"查询: {research_query['query']}")
    print(f"关键发现: {result['key_findings'][:2]}...")
    print(f"质量评分: {result['quality_score']:.2f}")
    
    # 3. 评估结果
    print("\n评估结果...")
    
    evaluation_results = await evaluator.evaluate(
        query=research_query["query"],
        response=str(result["key_findings"]),
        expected="人工智能在医疗影像分析、疾病预测、药物发现等方面有广泛应用",
        metrics=[EvaluationMetric.ACCURACY, EvaluationMetric.RELEVANCE]
    )
    
    for eval_result in evaluation_results:
        print(f"{eval_result.metric.value}: {eval_result.score:.2f}")
        print(f"解释: {eval_result.explanation[:50]}...")
    
    # 4. 清理
    await inference.stop()
    
    print("\n示例完成！")

if __name__ == "__main__":
    # 设置环境变量
    os.environ["OPENAI_API_KEY"] = "your-api-key-here"
    
    asyncio.run(main())
```

## 项目配置

### 环境变量配置

```bash
# .env.example
# API 配置
OPENAI_API_KEY=your-openai-api-key
ANTHROPIC_API_KEY=your-anthropic-api-key

# 项目配置
PROJECT_NAME=My DeepResearch Project
LOG_LEVEL=INFO
MAX_WORKERS=10

# 模型配置
DEFAULT_MODEL=gpt-3.5-turbo
EMBEDDING_MODEL=text-embedding-ada-002

# 数据库配置（可选）
DATABASE_URL=postgresql://user:password@localhost/dbname
REDIS_URL=redis://localhost:6379
```

### 依赖管理

```txt
# requirements.txt
# 核心依赖
torch>=2.0.0
transformers>=4.30.0
aiohttp>=3.9.0
fastapi>=0.104.0
pydantic>=2.5.0

# 工具依赖
beautifulsoup4>=4.12.0
requests>=2.31.0
python-dotenv>=1.0.0

# 开发依赖
pytest>=7.4.0
black>=23.0.0
mypy>=1.7.0
jupyter>=1.0.0
```

## 部署指南

### 本地部署

```bash
# 1. 克隆项目
git clone <your-project-repo>
cd my_deepresearch_project

# 2. 设置环境
python -m venv venv
source venv/bin/activate  # Linux/Mac
venv\Scripts\activate     # Windows

# 3. 安装依赖
pip install -r requirements.txt

# 4. 配置环境变量
cp .env.example .env
# 编辑 .env 文件，填入你的 API 密钥

# 5. 运行测试
pytest tests/

# 6. 启动服务
python scripts/serve.py
```

### Docker 部署

```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

# 安装系统依赖
RUN apt-get update && apt-get install -y \
    gcc \
    g++ \
    && rm -rf /var/lib/apt/lists/*

# 复制依赖文件
COPY requirements.txt .

# 安装 Python 依赖
RUN pip install --no-cache-dir -r requirements.txt

# 复制应用代码
COPY . .

# 创建非 root 用户
RUN useradd -m -u 1000 appuser && chown -R appuser:appuser /app
USER appuser

# 暴露端口
EXPOSE 8000

# 启动命令
CMD ["python", "scripts/serve.py"]
```

## 开发指南

### 代码规范

1. **命名规范**：
   - 类名：大驼峰，如 `ResearchAgent`
   - 函数名：小写加下划线，如 `extract_key_findings`
   - 变量名：小写加下划线，如 `research_query`

2. **类型提示**：
   ```python
   def process_data(data: List[Dict[str, Any]]) -> pd.DataFrame:
       """处理数据并返回 DataFrame"""
       ...
   ```

3. **文档字符串**：
   ```python
   class ResearchAgent(BaseAgent):
       """研究智能体
       
       用于执行深度研究任务，支持多步骤分析和综合。
       
       Attributes:
           name: 智能体名称
           specialization: 专业领域
       """
   ```

### 测试指南

```python
# tests/test_agents.py
import pytest
import asyncio
from src.agents.custom_agent import ResearchAgent

class TestResearchAgent:
    """研究智能体测试"""
    
    @pytest.fixture
    def researcher(self):
        return ResearchAgent(name="TestResearcher")
    
    @pytest.mark.asyncio
    async def test_think_phase(self, researcher):
        """测试思考阶段"""
        input_data = {"query": "测试查询"}
        plan = await researcher.think(input_data)
        
        assert "query" in plan
        assert "steps" in plan
        assert len(plan["steps"]) > 0
    
    @pytest.mark.asyncio
    async def test_full_cycle(self, researcher):
        """测试完整循环"""
        input_data = {"query": "简单测试"}
        result = await researcher.run(input_data)
        
        assert "key_findings" in result
        assert "quality_score" in result
        assert 0 <= result["quality_score"] <= 1
```

## 故障排除

### 常见问题

1. **API 密钥错误**：
   ```bash
   # 检查环境变量
   echo $OPENAI_API_KEY
   
   # 或在 Python 中检查
   import os
   print(os.getenv("OPENAI_API_KEY"))
   ```

2. **依赖冲突**：
   ```bash
   # 创建干净的虚拟环境
   rm -rf venv
   python -m venv venv
   source venv/bin/activate
   pip install -r requirements.txt
   ```

3. **内存不足**：
   ```python
   # 减少工作线程数
   settings.max_workers = 5
   
   # 使用较小的模型
   settings.default_model = "gpt-3.5-turbo"
   ```

### 调试技巧

1. **启用详细日志**：
   ```python
   import logging
   logging.basicConfig(level=logging.DEBUG)
   ```

2. **使用调试器**：
   ```python
   import pdb
   
   async def debug_function():
       pdb.set_trace()  # 设置断点
       # 你的代码
   ```

3. **性能分析**：
   ```python
   import cProfile
   
   profiler = cProfile.Profile()
   profiler.enable()
   
   # 运行代码
   asyncio.run(main())
   
   profiler.disable()
   profiler.print_stats(sort='time')
   ```

## 扩展指南

### 添加新工具

1. **创建工具类**：
   ```python
   # src/tools/new_tool.py
   from .base import BaseTool
   
   class NewTool(BaseTool):
       """新工具"""
       
       def __init__(self, name: str = "new_tool"):
           super().__init__(name)
           
       async def execute(self, params: Dict[str, Any]) -> Dict[str, Any]:
           """执行工具"""
           # 实现工具逻辑
           return {"result": "工具执行结果"}
   ```

2. **注册工具**：
   ```python
   # 在智能体中使用
   researcher.add_tool(NewTool())
   ```

### 添加新评估指标

1. **定义新指标**：
   ```python
   # 在 EvaluationMetric 枚举中添加
   class EvaluationMetric(Enum):
       CUSTOM_METRIC = "custom_metric"
   ```

2. **实现评估逻辑**：
   ```python
   async def _evaluate_custom_metric(self, query, response):
       # 实现自定义评估逻辑
       return 0.8, "自定义评估结果"
   ```

## 贡献指南

欢迎贡献代码！请遵循以下步骤：

1. Fork 项目
2. 创建特性分支
3. 提交更改
4. 推送到分支
5. 创建 Pull Request

## 许可证

本项目基于 MIT 许可证发布。

---

**开始你的 DeepResearch 项目吧！**