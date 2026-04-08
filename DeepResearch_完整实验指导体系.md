# DeepResearch 完整实验指导体系

## 体系总览

本实验指导体系旨在帮助你从零开始理解和复现 DeepResearch 多智能体深度研究系统。体系采用渐进式学习方法，从底层原理到具体实现，逐步构建完整的系统理解。

### 学习路径图

```
┌─────────────────────────────────────────────────────────────┐
│                    DeepResearch 实验指导体系                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ 阶段一：基础原理理解                                         │
│   ├── 实验01：底层运行原理                                   │
│   └── 核心：React 模式、异步架构、智能体基础                   │
│                                                             │
│ 阶段二：核心模块深入                                         │
│   ├── 实验02：WebAgent 模块                                  │
│   ├── 实验03：Inference 模块                                 │
│   └── 实验04：Evaluation 模块                                │
│                                                             │
│ 阶段三：系统集成实践                                         │
│   ├── 实验05：完整系统集成                                   │
│   └── 实验06：性能优化与扩展                                 │
│                                                             │
│ 阶段四：高级应用开发                                         │
│   ├── 实验07：自定义智能体开发                               │
│   └── 实验08：生产环境部署                                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 各实验指导摘要

### 实验01：底层运行原理
**核心内容**：React 模式、异步架构、智能体基础框架
**学习目标**：理解多智能体系统的核心设计思想
**关键代码**：基础智能体类、工具基类、异步任务执行器
**预计时间**：2-3小时

### 实验02：WebAgent 模块
**核心内容**：10+ specialized agents、协调器、工具链
**学习目标**：掌握网络智能体的工作原理和实现
**关键代码**：WebWatcher、WebWeaver、WebResearcher 等智能体
**预计时间**：4-6小时

### 实验03：Inference 模块
**核心内容**：模型管理、推理引擎、提示词工程
**学习目标**：理解 AI 模型在系统中的集成和使用
**关键代码**：ModelManager、InferenceEngine、PromptEngine
**预计时间**：3-4小时

### 实验04：Evaluation 模块
**核心内容**：评估系统、答案提取、性能指标
**学习目标**：掌握系统性能评估和优化方法
**关键代码**：答案提取器、评估器、评估流水线
**预计时间**：3-4小时

## 模块间依赖关系

### 数据流图

```
用户请求
    ↓
WebAgent 协调器
    ├── WebWatcher (监控)
    ├── WebWeaver (信息编织)
    ├── WebResearcher (研究)
    └── 其他智能体...
    ↓
工具链执行
    ├── 搜索工具
    ├── 浏览工具  
    ├── 分析工具
    └── 其他工具...
    ↓
Inference 服务
    ├── 模型推理
    ├── 结果处理
    └── 响应生成
    ↓
Evaluation 系统
    ├── 答案提取
    ├── 正确性判断
    └── 性能统计
    ↓
最终结果
```

### 模块依赖矩阵

| 模块 | 依赖模块 | 提供功能 | 使用场景 |
|------|----------|----------|----------|
| WebAgent | Inference | 模型推理 | 智能体决策、内容分析 |
| Inference | - | 模型服务 | 所有需要 AI 能力的场景 |
| Evaluation | WebAgent, Inference | 性能评估 | 训练后评估、在线监控 |
| Agent 基础 | - | 基础框架 | 所有智能体的基类 |

## 快速上手指南

### 1. 环境准备

```bash
# 1. 克隆项目
git clone <deepresearch-repo>
cd DeepResearch

# 2. 安装依赖
pip install -r requirements.txt

# 3. 设置环境变量
export OPENAI_API_KEY="your-api-key"
export MAX_WORKERS=10

# 4. 验证安装
python -c "import torch; print('PyTorch:', torch.__version__)"
```

### 2. 基础运行示例

```python
# quick_start.py
import asyncio
from agent.base import Agent
from inference.engine import InferenceEngine
from evaluation.pipeline import EvaluationPipeline

async def main():
    """快速启动示例"""
    
    # 1. 初始化基础组件
    agent = Agent(name="test_agent")
    inference = InferenceEngine()
    evaluator = EvaluationPipeline()
    
    # 2. 简单任务执行
    question = "Python 中如何实现异步编程？"
    response = await agent.think_and_act(question)
    
    print(f"问题: {question}")
    print(f"回答: {response}")
    
    # 3. 评估结果
    metrics = await evaluator.run_single(
        question=question,
        response=response,
        expected_answer="使用 asyncio 库，配合 async/await 语法"
    )
    
    print(f"评估结果: {metrics}")

if __name__ == "__main__":
    asyncio.run(main())
```

### 3. 分阶段学习建议

**第一阶段（1-2天）**：
1. 完成实验01，理解基础原理
2. 运行所有示例代码
3. 尝试修改基础智能体行为

**第二阶段（3-5天）**：
1. 完成实验02-04，深入核心模块
2. 实现自定义工具
3. 构建简单的研究任务

**第三阶段（1-2周）**：
1. 集成所有模块
2. 优化系统性能
3. 添加新功能

## 实践项目建议

### 项目1：新闻研究助手
**目标**：自动收集、分析和总结新闻
**涉及模块**：WebWatcher、WebResearcher、Inference
**关键功能**：
- 监控新闻源
- 自动分类和总结
- 生成每日简报

### 项目2：学术论文分析器
**目标**：分析学术论文并提取关键信息
**涉及模块**：WebWeaver、Inference、Evaluation
**关键功能**：
- 论文内容提取
- 关键发现总结
- 相关性评估

### 项目3：市场情报系统
**目标**：收集和分析市场信息
**涉及模块**：所有 WebAgents、Evaluation
**关键功能**：
- 竞品监控
- 趋势分析
- 风险评估

## 常见问题解答

### Q1：如何选择合适的实验顺序？
**A**：建议按编号顺序进行，每个实验都建立在前一个实验的基础上。

### Q2：需要多少编程经验？
**A**：需要 Python 中级水平，熟悉异步编程和面向对象设计。

### Q3：硬件要求是什么？
**A**：
- 最低：8GB RAM，支持 Python 3.8+
- 推荐：16GB RAM，GPU（可选）
- 生产：32GB+ RAM，多核 CPU，GPU 加速

### Q4：如何调试智能体行为？
**A**：
1. 启用详细日志
2. 使用评估模块监控性能
3. 逐步测试每个工具

### Q5：如何扩展系统？
**A**：
1. 添加新的工具类
2. 实现新的智能体类型
3. 集成新的模型服务

## 进阶学习资源

### 官方文档
- DeepResearch 项目 README
- 技术报告 (Tech_Report.pdf)
- 模块化实验方案报告

### 相关技术
- **异步编程**：asyncio 官方文档
- **AI 模型**：HuggingFace Transformers
- **Web 爬虫**：aiohttp、BeautifulSoup
- **评估系统**：MLflow、Weights & Biases

### 社区资源
- GitHub Issues 和 Discussions
- 相关论文和博客
- 在线课程和教程

## 评估与认证

### 学习成果评估

完成本实验指导体系后，你应该能够：

1. ✅ **理解多智能体系统架构**
2. ✅ **实现基础智能体框架**
3. ✅ **集成 AI 推理服务**
4. ✅ **构建评估系统**
5. ✅ **优化系统性能**
6. ✅ **扩展系统功能**

### 项目完成检查清单

- [ ] 环境配置完成
- [ ] 所有示例代码可运行
- [ ] 理解模块间依赖关系
- [ ] 完成至少一个实践项目
- [ ] 能够调试和优化系统
- [ ] 掌握扩展系统的方法

## 更新与维护

### 版本说明
- **v1.0**：基础实验指导体系
- **计划更新**：更多实践项目、性能优化指南

### 反馈与贡献
欢迎通过以下方式提供反馈：
1. GitHub Issues
2. 邮件反馈
3. 社区讨论

### 许可证
本实验指导体系基于 MIT 许可证发布。

---

## 附录：完整代码结构参考

```
deepresearch_experiments/
├── experiment_01_basics/          # 实验01：基础原理
│   ├── agent_framework.py
│   ├── react_pattern.py
│   └── async_executor.py
├── experiment_02_webagent/        # 实验02：WebAgent
│   ├── web_watcher.py
│   ├── web_weaver.py
│   └── coordinator.py
├── experiment_03_inference/       # 实验03：Inference
│   ├── model_manager.py
│   ├── inference_engine.py
│   └── prompt_engine.py
├── experiment_04_evaluation/      # 实验04：Evaluation
│   ├── answer_extractor.py
│   ├── evaluator.py
│   └── pipeline.py
├── experiment_05_integration/     # 实验05：集成
│   ├── full_system.py
│   └── workflow_orchestrator.py
├── experiment_06_optimization/    # 实验06：优化
│   ├── performance_tuning.py
│   └── scaling_strategies.py
├── projects/                      # 实践项目
│   ├── news_research/
│   ├── paper_analyzer/
│   └── market_intelligence/
└── utils/                         # 工具函数
    ├── logging_config.py
    └── data_loader.py
```

## 开始学习

现在你已经了解了完整的实验指导体系。建议从实验01开始，逐步深入。每个实验都包含完整的代码示例和详细的解释，确保你能够真正理解和掌握每个概念。

**学习建议**：
1. 不要跳过任何实验
2. 动手运行所有代码
3. 尝试修改和扩展
4. 记录学习笔记
5. 参与社区讨论

祝你学习顺利！