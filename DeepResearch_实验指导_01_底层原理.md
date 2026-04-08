# DeepResearch 实验指导系列

## 实验一：理解 DeepResearch 底层运行原理

### 实验目的
1. 理解 DeepResearch 项目的整体架构设计
2. 掌握多智能体系统的核心运行机制
3. 了解 React（Reasoning and Acting）模式的基本原理
4. 熟悉模块间的通信和协作方式

### 实验原理

#### 1. 系统架构概述
DeepResearch 是一个基于多智能体架构的深度研究系统，其核心设计理念是：

```
用户请求 → 智能体调度 → 任务分解 → 并行执行 → 结果聚合 → 反馈学习
```

#### 2. 核心组件
- **WebAgent 模块**：包含多个 specialized agents，每个负责特定网络任务
- **Inference 模块**：提供模型推理服务，支持多种 AI 模型
- **Evaluation 模块**：监控和评估智能体性能
- **Agent 框架**：提供智能体的基础架构和工具

#### 3. React 模式原理
React（Reasoning and Acting）模式是系统的核心思想：
- **Reasoning**：智能体分析任务，制定计划
- **Acting**：执行具体行动，获取结果
- **反思**：评估结果，调整策略

### 实验内容与步骤

#### 步骤1：环境准备
```bash
# 1. 创建虚拟环境
python -m venv deepresearch_env
source deepresearch_env/bin/activate  # Linux/Mac
# 或
deepresearch_env\Scripts\activate     # Windows

# 2. 安装基础依赖
pip install fastapi==0.116.1
pip install aiohttp==3.12.15
pip install transformers==4.40.0
pip install torch==2.3.0

# 3. 验证安装
python -c "import fastapi; import aiohttp; print('环境准备完成')"
```

#### 步骤2：创建最简单的智能体框架
创建 `basic_agent.py`：
```python
"""
基础智能体框架 - 理解 React 模式
"""
from abc import ABC, abstractmethod
from typing import Dict, Any, List
import asyncio

class Agent(ABC):
    """智能体基类"""
    
    def __init__(self, name: str):
        self.name = name
        self.memory = []  # 记忆存储
        self.tools = {}   # 可用工具
    
    async def run(self, task: str) -> Dict[str, Any]:
        """运行智能体 - React 模式的核心"""
        print(f"[{self.name}] 开始处理任务: {task}")
        
        # 1. Reasoning: 分析任务
        plan = await self.reason(task)
        print(f"[{self.name}] 制定计划: {plan}")
        
        # 2. Acting: 执行计划
        result = await self.act(plan)
        print(f"[{self.name}] 执行结果: {result}")
        
        # 3. 反思和学习
        await self.reflect(task, plan, result)
        
        return {
            "agent": self.name,
            "task": task,
            "plan": plan,
            "result": result
        }
    
    @abstractmethod
    async def reason(self, task: str) -> List[str]:
        """推理阶段：分析任务并制定计划"""
        pass
    
    @abstractmethod
    async def act(self, plan: List[str]) -> Dict[str, Any]:
        """行动阶段：执行计划"""
        pass
    
    async def reflect(self, task: str, plan: List[str], result: Dict[str, Any]):
        """反思阶段：评估结果并学习"""
        self.memory.append({
            "task": task,
            "plan": plan,
            "result": result,
            "timestamp": asyncio.get_event_loop().time()
        })
        print(f"[{self.name}] 已记录本次经验，记忆长度: {len(self.memory)}")

class SimpleResearchAgent(Agent):
    """简单研究智能体"""
    
    def __init__(self):
        super().__init__("SimpleResearchAgent")
        # 初始化工具
        self.tools = {
            "search": self.tool_search,
            "analyze": self.tool_analyze,
            "summarize": self.tool_summarize
        }
    
    async def reason(self, task: str) -> List[str]:
        """分析研究任务"""
        # 简单的规则-based 推理
        if "总结" in task:
            return ["search", "analyze", "summarize"]
        elif "搜索" in task:
            return ["search", "analyze"]
        else:
            return ["search", "analyze", "summarize"]
    
    async def act(self, plan: List[str]) -> Dict[str, Any]:
        """执行研究计划"""
        results = {}
        for step in plan:
            if step in self.tools:
                result = await self.tools[step](f"执行{step}")
                results[step] = result
        return results
    
    async def tool_search(self, query: str):
        """模拟搜索工具"""
        await asyncio.sleep(0.5)  # 模拟网络延迟
        return f"搜索结果: {query}"
    
    async def tool_analyze(self, content: str):
        """模拟分析工具"""
        await asyncio.sleep(0.3)
        return f"分析结果: 找到关键信息"
    
    async def tool_summarize(self, content: str):
        """模拟总结工具"""
        await asyncio.sleep(0.2)
        return f"总结: 这是重要发现"

async def main():
    """主函数 - 演示智能体工作流程"""
    # 创建智能体
    agent = SimpleResearchAgent()
    
    # 运行任务
    tasks = [
        "总结人工智能的最新发展",
        "搜索机器学习的最佳实践",
        "分析深度学习框架比较"
    ]
    
    for task in tasks:
        result = await agent.run(task)
        print(f"\n任务完成: {result}\n")
        print("-" * 50)

if __name__ == "__main__":
    asyncio.run(main())
```

#### 步骤3：运行和观察
```bash
# 运行基础智能体
python basic_agent.py

# 预期输出：
# [SimpleResearchAgent] 开始处理任务: 总结人工智能的最新发展
# [SimpleResearchAgent] 制定计划: ['search', 'analyze', 'summarize']
# [SimpleResearchAgent] 执行结果: {'search': '搜索结果: 执行search', ...}
# [SimpleResearchAgent] 已记录本次经验，记忆长度: 1
```

#### 步骤4：扩展理解 - 多智能体协作
创建 `multi_agent_system.py`：
```python
"""
多智能体系统 - 理解 DeepResearch 的协作机制
"""
import asyncio
from typing import Dict, Any, List
from dataclasses import dataclass

@dataclass
class Task:
    """任务定义"""
    id: str
    description: str
    priority: int = 1

class AgentCoordinator:
    """智能体协调器"""
    
    def __init__(self):
        self.agents = {}
        self.task_queue = asyncio.Queue()
        self.results = {}
    
    def register_agent(self, agent):
        """注册智能体"""
        self.agents[agent.name] = agent
        print(f"注册智能体: {agent.name}")
    
    async def submit_task(self, task: Task):
        """提交任务"""
        await self.task_queue.put(task)
        print(f"提交任务: {task.description}")
    
    async def run(self):
        """运行协调器"""
        print("智能体协调器开始运行...")
        
        while True:
            # 获取任务
            task = await self.task_queue.get()
            
            # 分配任务给合适的智能体
            assigned_agent = self.select_agent(task)
            
            if assigned_agent:
                # 异步执行任务
                asyncio.create_task(
                    self.execute_task(assigned_agent, task)
                )
            else:
                print(f"没有找到合适的智能体处理任务: {task.description}")
            
            self.task_queue.task_done()
    
    def select_agent(self, task: Task):
        """选择最合适的智能体"""
        # 简单的基于名称的匹配
        for agent_name, agent in self.agents.items():
            if any(keyword in task.description for keyword in agent.keywords):
                return agent
        return None
    
    async def execute_task(self, agent, task: Task):
        """执行任务"""
        try:
            result = await agent.run(task.description)
            self.results[task.id] = result
            print(f"任务 {task.id} 完成: {result}")
        except Exception as e:
            print(f"任务 {task.id} 失败: {e}")

# 扩展智能体类
class WebResearcher(Agent):
    """网络研究员智能体"""
    
    def __init__(self):
        super().__init__("WebResearcher")
        self.keywords = ["研究", "搜索", "调查"]
    
    async def reason(self, task: str):
        return ["web_search", "extract_info", "organize_data"]
    
    async def act(self, plan: List[str]):
        return {"status": "completed", "data": "研究数据"}

class DataAnalyzer(Agent):
    """数据分析师智能体"""
    
    def __init__(self):
        super().__init__("DataAnalyzer")
        self.keywords = ["分析", "统计", "处理"]
    
    async def reason(self, task: str):
        return ["load_data", "analyze_patterns", "generate_report"]
    
    async def act(self, plan: List[str]):
        return {"status": "completed", "insights": "分析洞察"}

async def demo_multi_agent():
    """演示多智能体协作"""
    coordinator = AgentCoordinator()
    
    # 注册智能体
    coordinator.register_agent(WebResearcher())
    coordinator.register_agent(DataAnalyzer())
    
    # 提交任务
    tasks = [
        Task("1", "研究人工智能在医疗领域的应用"),
        Task("2", "分析用户行为数据"),
        Task("3", "调查气候变化对农业的影响")
    ]
    
    for task in tasks:
        await coordinator.submit_task(task)
    
    # 运行协调器（有限时间）
    await asyncio.sleep(3)
    
    print(f"\n任务完成情况: {len(coordinator.results)}/{len(tasks)}")

if __name__ == "__main__":
    asyncio.run(demo_multi_agent())
```

### 实验总结

#### 1. 核心收获
- **理解了 React 模式**：Reasoning → Acting → Reflect 的循环
- **掌握了智能体架构**：基类设计、工具集成、记忆机制
- **了解了多智能体协作**：任务分配、协调机制、结果聚合

#### 2. DeepResearch 项目特点
- **模块化设计**：每个智能体专注特定任务
- **异步处理**：提高系统吞吐量和响应速度
- **可扩展性**：易于添加新的智能体和工具
- **学习能力**：通过记忆和反思不断优化

#### 3. 下一步学习方向
1. 深入学习具体的 WebAgent 实现（WebWatcher、WebWeaver 等）
2. 理解 inference 模块的模型调用机制
3. 掌握 evaluation 模块的性能评估方法
4. 探索实际部署和优化策略

#### 4. 关键思考问题
1. React 模式与传统编程模式有何不同？
2. 如何设计智能体之间的通信协议？
3. 记忆机制如何影响智能体的长期表现？
4. 多智能体系统如何避免冲突和重复工作？

### 实验扩展

#### 扩展任务1：添加新的工具
尝试为 `SimpleResearchAgent` 添加新的工具，如：
- `tool_translate`: 翻译工具
- `tool_validate`: 数据验证工具
- `tool_visualize`: 可视化工具

#### 扩展任务2：实现优先级调度
修改 `AgentCoordinator`，实现基于任务优先级的调度算法。

#### 扩展任务3：添加持久化存储
将智能体的记忆保存到数据库或文件中，实现长期学习。

---

**实验完成标准**：
- ✅ 成功运行基础智能体框架
- ✅ 理解 React 模式的工作流程
- ✅ 实现多智能体协作演示
- ✅ 能够回答关键思考问题

通过本实验，你已经掌握了 DeepResearch 项目的底层运行原理，为后续深入学习具体模块打下了坚实基础。