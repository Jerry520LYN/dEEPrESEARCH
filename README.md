# DeepResearch - 从0到1搭建深度研究系统

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![Status](https://img.shields.io/badge/status-learning%20guide-green.svg)]()

## 📖 项目简介

**DeepResearch** 是一个完整的多智能体深度研究系统学习与实践项目。本项目提供从零开始理解、复现和扩展 DeepResearch 框架的完整实验指导体系，帮助你掌握现代 AI 驱动的研究系统架构。

### 核心特性

- 🧠 **React 模式实现**: Reasoning（推理）→ Acting（行动）→ Reflecting（反思）循环
- 🤖 **多智能体架构**: 10+ 专用 WebAgents 协同工作
- ⚡ **异步高性能**: 基于 asyncio 的高并发处理
- 🔧 **模块化设计**: Agent、WebAgent、Inference、Evaluation 四大核心模块
- 📊 **完整评估系统**: 自动化性能监控和质量评估
- 🎯 **实践导向**: 包含完整的学习路线图和实践项目模板

## 🎯 学习目标

完成本项目的学习后，你将能够：

1. ✅ 深入理解多智能体系统的底层原理和 React 模式
2. ✅ 掌握 WebAgent 模块的设计与实现（10+ specialized agents）
3. ✅ 熟练使用 Inference 模块集成多种 AI 模型
4. ✅ 构建完整的评估系统监控和优化性能
5. ✅ 独立开发基于 DeepResearch 框架的实际应用
6. ✅ 部署生产级的多智能体研究系统

## 📚 文档结构

本项目采用渐进式学习方法，包含以下核心文档：

### 实验指导系列

| 文档 | 内容 | 预计时间 | 难度 |
|------|------|----------|------|
| [实验01：底层运行原理](DeepResearch_实验指导_01_底层原理.md) | React 模式、异步架构、智能体基础 | 2-3小时 | ⭐⭐ |
| [实验02：WebAgent 模块](DeepResearch_实验指导_02_WebAgent模块.md) | 10+ specialized agents、协调器、工具链 | 4-6小时 | ⭐⭐⭐ |
| [实验03：Inference 模块](DeepResearch_实验指导_03_Inference模块.md) | 模型管理、推理引擎、提示词工程 | 3-4小时 | ⭐⭐⭐ |
| [实验04：Evaluation 模块](DeepResearch_实验指导_04_Evaluation模块.md) | 评估系统、答案提取、性能指标 | 3-4小时 | ⭐⭐⭐ |

### 支持文档

| 文档 | 描述 |
|------|------|
| [完整实验指导体系](DeepResearch_完整实验指导体系.md) | 学习路径图、模块依赖关系、快速上手指南 |
| [实践项目模板](DeepResearch_实践项目模板.md) | 标准化项目结构、代码模板、部署指南 |
| [学习路线图与总结](DeepResearch_学习路线图与总结.md) | 分阶段学习计划、常见问题解答、资源汇总 |

## 🚀 快速开始

### 环境要求

- Python 3.8+
- 8GB+ RAM（推荐 16GB）
- 网络连接（用于 API 调用）

### 安装步骤

```bash
# 1. 克隆或下载项目
cd "D:\PROJECTS\从0到1搭建DeepResearch"

# 2. 创建虚拟环境
python -m venv venv
venv\Scripts\activate  # Windows
# source venv/bin/activate  # Linux/Mac

# 3. 安装核心依赖
pip install fastapi==0.116.1
pip install aiohttp==3.12.15
pip install transformers==4.40.0
pip install torch==2.3.0
pip install pydantic>=2.5.0
pip install python-dotenv>=1.0.0

# 4. 验证安装
python -c "import fastapi; import aiohttp; print('✅ 环境准备完成')"
```

### 运行第一个示例

```python
# test_basic_agent.py
import asyncio
from basic_agent import SimpleResearchAgent

async def main():
    # 创建智能体
    agent = SimpleResearchAgent()
    
    # 运行任务
    result = await agent.run("总结人工智能的最新发展")
    print(f"结果: {result}")

if __name__ == "__main__":
    asyncio.run(main())
```

## 🏗️ 系统架构

### 整体架构图

```
┌─────────────────────────────────────────────────────┐
│                  用户请求层                           │
└──────────────────┬──────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────┐
│              WebAgent 协调器                         │
│  ┌──────────┬──────────┬──────────┬──────────┐     │
│  │WebWatcher│WebWeaver │WebResear.│  Others  │     │
│  └──────────┴──────────┴──────────┴──────────┘     │
└──────────────────┬──────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────┐
│               工具链层                               │
│  ┌────────┬────────┬────────┬────────┬────────┐    │
│  │搜索工具│浏览工具│分析工具│总结工具│其他工具│    │
│  └────────┴────────┴────────┴────────┴────────┘    │
└──────────────────┬──────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────┐
│             Inference 服务层                         │
│  ┌──────────────┬──────────────┬──────────────┐    │
│  │ ModelManager │InferenceEng. │PromptEngine  │    │
│  └──────────────┴──────────────┴──────────────┘    │
└──────────────────┬──────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────┐
│            Evaluation 评估层                         │
│  ┌──────────────┬──────────────┬──────────────┐    │
│  │AnswerExtract.│  Evaluator   │  Pipeline    │    │
│  └──────────────┴──────────────┴──────────────┘    │
└──────────────────┬──────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────┐
│                 最终结果                             │
└─────────────────────────────────────────────────────┘
```

### 核心模块说明

#### 1. Agent 基础框架
- **功能**: 提供智能体的基础架构和 React 模式实现
- **关键组件**: `BaseAgent`, `AgentState`, 工具基类
- **核心思想**: Think → Act → Reflect 循环

#### 2. WebAgent 模块
- **功能**: 处理网络相关的研究和信息收集任务
- **Specialized Agents**:
  - `WebWatcher`: 监控网页变化
  - `WebWeaver`: 编织和整合信息
  - `WebResearcher`: 执行深度研究
  - 等 7+ 其他专用智能体
- **协调器**: 管理多个智能体的协作

#### 3. Inference 模块
- **功能**: 提供统一的 AI 模型推理服务
- **关键组件**:
  - `ModelManager`: 模型注册和管理
  - `InferenceEngine`: 推理执行引擎
  - `PromptEngine`: 提示词模板管理
- **支持的模型**: OpenAI GPT、HuggingFace Transformers、本地模型等

#### 4. Evaluation 模块
- **功能**: 系统性能评估和质量监控
- **关键组件**:
  - `AnswerExtractor`: 从响应中提取答案
  - `Evaluator`: 多维度评估器
  - `EvaluationPipeline`: 完整评估流水线
- **评估指标**: 准确性、相关性、完整性、连贯性等

## 📖 学习路线

### 阶段一：基础掌握（1-2周）

**目标**: 理解核心概念，运行基础示例

```
第1-2天  → 阅读实验01，理解 React 模式和异步架构
第3-4天  → 运行所有基础示例代码，修改参数观察变化
第5-7天  → 创建简单智能体，实现基础功能
```

**检查点**:
- [ ] 理解智能体的三个思考阶段（Think-Act-Reflect）
- [ ] 能够创建简单的异步任务
- [ ] 实现基础的工具类

### 阶段二：模块深入（2-3周）

**目标**: 掌握每个核心模块的实现

```
第1周  → 学习 WebAgent 模块，理解各种 specialized agents
第2周  → 学习 Inference 模块，掌握模型集成方法
第3周  → 学习 Evaluation 模块，实现性能评估系统
```

**检查点**:
- [ ] 能够创建自定义 WebAgent
- [ ] 集成新的 AI 模型到推理服务
- [ ] 实现自定义评估指标

### 阶段三：系统集成（1-2周）

**目标**: 整合所有模块，构建完整系统

```
第1周  → 按照模板创建完整项目
第2周  → 优化系统性能，添加监控和日志
```

**检查点**:
- [ ] 成功运行完整的多智能体系统
- [ ] 系统能够处理复杂的研究任务
- [ ] 评估系统提供准确的性能报告

### 阶段四：高级应用（持续）

**目标**: 基于框架开发实际应用

- 选择应用领域：新闻研究、学术分析、市场情报等
- 定制开发：根据需求扩展智能体和工具
- 生产部署：优化性能，部署到生产环境

## 💡 实践项目建议

### 项目1：新闻研究助手
自动收集、分析和总结新闻，生成每日简报。

**涉及模块**: WebWatcher, WebResearcher, Inference  
**关键功能**:
- 监控新闻源
- 自动分类和总结
- 生成每日简报

### 项目2：学术论文分析器
分析学术论文并提取关键信息。

**涉及模块**: WebWeaver, Inference, Evaluation  
**关键功能**:
- 论文内容提取
- 关键发现总结
- 相关性评估

### 项目3：市场情报系统
收集和分析市场信息，提供竞争情报。

**涉及模块**: 所有 WebAgents, Evaluation  
**关键功能**:
- 竞品监控
- 趋势分析
- 风险评估

## 🔧 配置说明

### 环境变量配置

创建 `.env` 文件：

```bash
# API 密钥
OPENAI_API_KEY=your-openai-api-key
ANTHROPIC_API_KEY=your-anthropic-api-key

# 项目配置
PROJECT_NAME=My DeepResearch Project
LOG_LEVEL=INFO
MAX_WORKERS=10

# 模型配置
DEFAULT_MODEL=gpt-3.5-turbo
EMBEDDING_MODEL=text-embedding-ada-002
```

### 配置文件示例

```python
# config/settings.py
from dataclasses import dataclass

@dataclass
class ProjectSettings:
    project_name: str = "DeepResearch"
    max_workers: int = 10
    timeout_seconds: int = 30
    default_model: str = "gpt-3.5-turbo"
    log_level: str = "INFO"
```

## 🧪 测试

```bash
# 运行单元测试
pytest tests/

# 运行集成测试
pytest tests/test_integration.py -v

# 性能测试
python scripts/benchmark.py
```

## 📊 性能优化建议

### 1. 并发控制
```python
# 调整工作线程数
settings.max_workers = 20  # 根据硬件调整

# 使用信号量限制并发
semaphore = asyncio.Semaphore(10)
```

### 2. 缓存策略
```python
# 实现结果缓存
from functools import lru_cache

@lru_cache(maxsize=1000)
def cached_search(query: str):
    return search_engine.search(query)
```

### 3. 批量处理
```python
# 批量执行任务
tasks = [agent.run(query) for query in queries]
results = await asyncio.gather(*tasks)
```

## ❓ 常见问题

### Q1: 如何选择合适的实验顺序？
**A**: 建议按编号顺序进行（实验01 → 04），每个实验都建立在前一个的基础上。

### Q2: 需要多少编程经验？
**A**: 需要 Python 中级水平，熟悉异步编程（asyncio）和面向对象设计。

### Q3: 硬件要求是什么？
**A**: 
- 最低：8GB RAM，支持 Python 3.8+
- 推荐：16GB RAM，GPU（可选）
- 生产：32GB+ RAM，多核 CPU，GPU 加速

### Q4: 如何调试智能体行为？
**A**:
1. 启用详细日志：`logging.basicConfig(level=logging.DEBUG)`
2. 使用评估模块监控性能
3. 逐步测试每个工具

### Q5: 如何扩展系统？
**A**:
1. 添加新的工具类（继承 `BaseTool`）
2. 实现新的智能体类型（继承 `BaseAgent`）
3. 集成新的模型服务（扩展 `ModelManager`）

## 🛠️ 故障排除

### 问题1: API 密钥错误
```bash
# 检查环境变量
echo $OPENAI_API_KEY  # Linux/Mac
echo %OPENAI_API_KEY%  # Windows

# 或在 Python 中检查
import os
print(os.getenv("OPENAI_API_KEY"))
```

### 问题2: 依赖冲突
```bash
# 创建干净的虚拟环境
rm -rf venv
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### 问题3: 内存不足
```python
# 减少工作线程数
settings.max_workers = 5

# 使用较小的模型
settings.default_model = "gpt-3.5-turbo"
```

## 📝 代码规范

### 命名规范
- **类名**: 大驼峰，如 `ResearchAgent`
- **函数名**: 小写加下划线，如 `extract_key_findings`
- **变量名**: 小写加下划线，如 `research_query`

### 类型提示
```python
from typing import Dict, Any, List

def process_data(data: List[Dict[str, Any]]) -> pd.DataFrame:
    """处理数据并返回 DataFrame"""
    ...
```

### 文档字符串
```python
class ResearchAgent(BaseAgent):
    """研究智能体
    
    用于执行深度研究任务，支持多步骤分析和综合。
    
    Attributes:
        name: 智能体名称
        specialization: 专业领域
    """
```

## 🤝 贡献指南

欢迎贡献代码、文档或提出建议！

1. Fork 项目
2. 创建特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 创建 Pull Request

## 📄 许可证

本项目基于 MIT 许可证发布 - 详见 [LICENSE](LICENSE) 文件

## 🙏 致谢

- DeepResearch 开源项目提供的灵感和参考
- 所有参与学习和改进本项目的社区成员
- HuggingFace、OpenAI 等提供的优秀 AI 工具和模型

## 📧 联系方式

- 项目问题: [GitHub Issues](https://github.com/your-repo/issues)
- 邮件联系: 19166910919@163.com

## 🌟 Star History

如果这个项目对你有帮助，请给它一个 ⭐ Star！

---

**开始你的 DeepResearch 学习之旅吧！** 🚀

从 [实验01：底层运行原理](DeepResearch_实验指导_01_底层原理.md) 开始，逐步掌握多智能体深度研究系统的核心技术。
