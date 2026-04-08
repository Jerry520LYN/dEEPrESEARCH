# DeepResearch 实验指导系列

## 实验二：深入理解 WebAgent 模块

### 实验目的
1. 理解 WebAgent 模块的架构设计和职责划分
2. 掌握 specialized agents 的工作原理和实现方式
3. 学习网络智能体的工具集成和任务执行机制
4. 了解异步处理和并发控制的最佳实践

### 实验原理

#### 1. WebAgent 模块架构
WebAgent 是 DeepResearch 的核心模块，包含多个 specialized agents：

```
WebAgent/
├── WebWatcher/      # 监控和观察智能体
├── WebWeaver/       # 信息编织智能体  
├── WebWalker/       # 网络爬行智能体
├── WebResearcher/   # 研究搜索智能体
├── WebResummer/     # 总结归纳智能体
├── WebSailor/       # 导航浏览智能体
└── ... (其他智能体)
```

#### 2. 智能体专业化设计
每个智能体都有特定的职责：
- **WebWatcher**：监控网络变化，实时跟踪信息
- **WebWeaver**：整合多源信息，生成连贯报告
- **WebWalker**：系统化浏览网页，收集结构化数据
- **WebResearcher**：深度研究特定主题
- **WebResummer**：自动总结长文档和对话

#### 3. 工具链集成
每个智能体都集成了特定的工具：
- **浏览器工具**：模拟用户浏览行为
- **搜索工具**：调用搜索引擎 API
- **解析工具**：提取网页结构化信息
- **分析工具**：数据分析和模式识别

### 实验内容与步骤

#### 步骤1：创建基础的 WebAgent 框架
创建 `web_agent_framework.py`：

```python
"""
WebAgent 框架 - 理解 specialized agents 设计
"""
import asyncio
import aiohttp
from abc import ABC, abstractmethod
from typing import Dict, Any, List, Optional
from dataclasses import dataclass
import json
import logging

# 配置日志
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

@dataclass
class WebPage:
    """网页数据结构"""
    url: str
    title: str
    content: str
    metadata: Dict[str, Any]
    timestamp: float

class WebTool(ABC):
    """网络工具基类"""
    
    def __init__(self, name: str):
        self.name = name
        self.session: Optional[aiohttp.ClientSession] = None
    
    async def initialize(self):
        """初始化工具"""
        if not self.session:
            timeout = aiohttp.ClientTimeout(total=30)
            self.session = aiohttp.ClientSession(timeout=timeout)
    
    async def cleanup(self):
        """清理资源"""
        if self.session:
            await self.session.close()
    
    @abstractmethod
    async def execute(self, **kwargs) -> Any:
        """执行工具操作"""
        pass

class BrowserTool(WebTool):
    """浏览器工具 - 模拟网页浏览"""
    
    def __init__(self):
        super().__init__("browser")
        self.headers = {
            "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"
        }
    
    async def execute(self, url: str, method: str = "GET", **kwargs) -> WebPage:
        """浏览网页"""
        await self.initialize()
        
        try:
            async with self.session.request(
                method=method,
                url=url,
                headers=self.headers,
                **kwargs
            ) as response:
                content = await response.text()
                
                return WebPage(
                    url=url,
                    title=self._extract_title(content),
                    content=content[:5000],  # 限制内容长度
                    metadata={
                        "status": response.status,
                        "headers": dict(response.headers),
                        "method": method
                    },
                    timestamp=asyncio.get_event_loop().time()
                )
        except Exception as e:
            logger.error(f"浏览失败 {url}: {e}")
            raise
    
    def _extract_title(self, html: str) -> str:
        """提取网页标题"""
        import re
        match = re.search(r'<title>(.*?)</title>', html, re.IGNORECASE)
        return match.group(1) if match else "无标题"

class SearchTool(WebTool):
    """搜索工具 - 模拟搜索引擎"""
    
    def __init__(self):
        super().__init__("search")
        # 模拟搜索API（实际项目中会调用真实API）
        self.search_engines = ["google", "bing", "duckduckgo"]
    
    async def execute(self, query: str, limit: int = 5, **kwargs) -> List[Dict[str, Any]]:
        """执行搜索"""
        await self.initialize()
        
        # 模拟搜索过程
        results = []
        for i in range(limit):
            results.append({
                "title": f"{query} - 结果 {i+1}",
                "url": f"https://example.com/result/{i}",
                "snippet": f"这是关于 {query} 的第 {i+1} 个搜索结果",
                "engine": self.search_engines[i % len(self.search_engines)]
            })
        
        await asyncio.sleep(0.5)  # 模拟网络延迟
        return results

class BaseWebAgent(ABC):
    """WebAgent 基类"""
    
    def __init__(self, name: str):
        self.name = name
        self.tools: Dict[str, WebTool] = {}
        self.memory: List[Dict[str, Any]] = []
        self.max_memory_size = 1000
        
        # 初始化基础工具
        self._init_base_tools()
    
    def _init_base_tools(self):
        """初始化基础工具"""
        self.tools["browser"] = BrowserTool()
        self.tools["search"] = SearchTool()
    
    async def initialize(self):
        """初始化智能体"""
        logger.info(f"初始化智能体: {self.name}")
        for tool in self.tools.values():
            await tool.initialize()
    
    async def cleanup(self):
        """清理资源"""
        for tool in self.tools.values():
            await tool.cleanup()
        logger.info(f"清理智能体: {self.name}")
    
    def add_tool(self, tool: WebTool):
        """添加自定义工具"""
        self.tools[tool.name] = tool
        logger.info(f"添加工具: {tool.name}")
    
    async def run(self, task: str) -> Dict[str, Any]:
        """运行智能体 - 完整的 React 流程"""
        logger.info(f"[{self.name}] 开始任务: {task}")
        
        # 1. 推理阶段
        plan = await self.reason(task)
        logger.info(f"[{self.name}] 制定计划: {plan}")
        
        # 2. 行动阶段
        results = []
        for step in plan:
            try:
                result = await self.execute_step(step, task)
                results.append(result)
                logger.info(f"[{self.name}] 完成步骤: {step}")
            except Exception as e:
                logger.error(f"[{self.name}] 步骤失败 {step}: {e}")
                results.append({"step": step, "error": str(e)})
        
        # 3. 整合结果
        final_result = await self.integrate_results(results)
        
        # 4. 反思和学习
        await self.reflect(task, plan, final_result)
        
        return {
            "agent": self.name,
            "task": task,
            "plan": plan,
            "results": results,
            "final_result": final_result
        }
    
    @abstractmethod
    async def reason(self, task: str) -> List[str]:
        """推理：分析任务并制定计划"""
        pass
    
    async def execute_step(self, step: str, context: str) -> Dict[str, Any]:
        """执行单个步骤"""
        # 根据步骤类型选择工具
        if step.startswith("browse_"):
            url = step.replace("browse_", "")
            page = await self.tools["browser"].execute(url=url)
            return {"type": "browse", "url": url, "page": page}
        
        elif step.startswith("search_"):
            query = step.replace("search_", "")
            results = await self.tools["search"].execute(query=query)
            return {"type": "search", "query": query, "results": results}
        
        else:
            # 默认处理
            return {"type": "custom", "step": step, "context": context}
    
    @abstractmethod
    async def integrate_results(self, results: List[Dict[str, Any]]) -> Dict[str, Any]:
        """整合多个步骤的结果"""
        pass
    
    async def reflect(self, task: str, plan: List[str], result: Dict[str, Any]):
        """反思和学习"""
        experience = {
            "task": task,
            "plan": plan,
            "result": result,
            "timestamp": asyncio.get_event_loop().time(),
            "agent": self.name
        }
        
        self.memory.append(experience)
        if len(self.memory) > self.max_memory_size:
            self.memory = self.memory[-self.max_memory_size:]
        
        logger.info(f"[{self.name}] 记录经验，记忆大小: {len(self.memory)}")
```

#### 步骤2：实现具体的 WebAgent
创建 `specialized_agents.py`：

```python
"""
具体化的 WebAgents - 理解 specialized 设计
"""
import asyncio
from typing import Dict, Any, List
import re
from web_agent_framework import BaseWebAgent, WebTool

class WebWatcherAgent(BaseWebAgent):
    """WebWatcher - 监控智能体"""
    
    def __init__(self):
        super().__init__("WebWatcher")
        self.watch_list = []
        self.change_threshold = 0.8
    
    async def reason(self, task: str) -> List[str]:
        """监控任务的推理逻辑"""
        if "监控" in task or "跟踪" in task:
            # 提取监控目标
            urls = self._extract_urls(task)
            if urls:
                steps = []
                for url in urls:
                    steps.append(f"browse_{url}")
                    steps.append("analyze_changes")
                return steps
        
        # 默认计划
        return ["search_相关主题", "browse_主要网站", "analyze_trends"]
    
    async def integrate_results(self, results: List[Dict[str, Any]]) -> Dict[str, Any]:
        """整合监控结果"""
        changes = []
        trends = []
        
        for result in results:
            if result["type"] == "browse":
                change_analysis = await self._analyze_changes(result["page"])
                if change_analysis["has_changes"]:
                    changes.append({
                        "url": result["url"],
                        "changes": change_analysis
                    })
            
            elif result["type"] == "search":
                trends.extend(result["results"])
        
        return {
            "changes_detected": len(changes),
            "changes": changes,
            "trends": trends[:10],  # 只返回前10个趋势
            "summary": f"检测到 {len(changes)} 个变化，发现 {len(trends)} 个相关趋势"
        }
    
    async def _analyze_changes(self, page) -> Dict[str, Any]:
        """分析网页变化"""
        # 模拟变化分析
        await asyncio.sleep(0.2)
        import random
        has_changes = random.random() > 0.5
        
        return {
            "has_changes": has_changes,
            "change_score": random.random(),
            "key_elements": ["标题", "内容", "图片"] if has_changes else [],
            "timestamp": asyncio.get_event_loop().time()
        }
    
    def _extract_urls(self, text: str) -> List[str]:
        """从文本中提取URL"""
        url_pattern = r'https?://[^\s]+'
        return re.findall(url_pattern, text)

class WebWeaverAgent(BaseWebAgent):
    """WebWeaver - 信息编织智能体"""
    
    def __init__(self):
        super().__init__("WebWeaver")
        self.sources = []
        self.weaving_strategy = "hierarchical"
    
    async def reason(self, task: str) -> List[str]:
        """信息编织的推理逻辑"""
        if "整合" in task or "编织" in task or "报告" in task:
            # 提取信息源
            sources = self._identify_sources(task)
            steps = []
            
            for source in sources:
                if source.startswith("http"):
                    steps.append(f"browse_{source}")
                else:
                    steps.append(f"search_{source}")
            
            steps.append("extract_keypoints")
            steps.append("organize_structure")
            steps.append("generate_report")
            
            return steps
        
        return ["search_背景信息", "browse_相关资源", "synthesize_content"]
    
    async def integrate_results(self, results: List[Dict[str, Any]]) -> Dict[str, Any]:
        """编织多个信息源"""
        contents = []
        keypoints = []
        
        for result in results:
            if result["type"] == "browse":
                content = self._extract_content(result["page"])
                contents.append(content)
                keypoints.extend(self._extract_keypoints(content))
            
            elif result["type"] == "search":
                for item in result["results"]:
                    keypoints.append(item["snippet"])
        
        # 生成结构化报告
        report = await self._generate_report(contents, keypoints)
        
        return {
            "sources_processed": len(contents),
            "keypoints_found": len(keypoints),
            "report_structure": report["structure"],
            "summary": report["summary"],
            "recommendations": report["recommendations"]
        }
    
    def _identify_sources(self, task: str) -> List[str]:
        """识别信息源"""
        # 简单的关键词匹配
        sources = []
        keywords = ["网站", "文章", "论文", "新闻", "博客"]
        
        for keyword in keywords:
            if keyword in task:
                sources.append(f"相关{keyword}")
        
        # 添加默认源
        if not sources:
            sources = ["权威网站", "学术资源", "行业报告"]
        
        return sources
    
    def _extract_content(self, page) -> str:
        """提取网页主要内容"""
        return f"{page.title}\n\n{page.content[:1000]}"
    
    def _extract_keypoints(self, content: str) -> List[str]:
        """提取关键点"""
        sentences = content.split('。')
        return [s.strip() for s in sentences[:5] if len(s.strip()) > 10]
    
    async def _generate_report(self, contents: List[str], keypoints: List[str]) -> Dict[str, Any]:
        """生成报告"""
        await asyncio.sleep(0.3)
        
        return {
            "structure": {
                "introduction": "背景介绍",
                "body": "主要内容分析",
                "conclusion": "总结和建议"
            },
            "summary": f"基于 {len(contents)} 个信息源，提取了 {len(keypoints)} 个关键点",
            "recommendations": [
                "进一步研究相关领域",
                "关注最新发展趋势",
                "验证信息准确性"
            ]
        }

class WebResearcherAgent(BaseWebAgent):
    """WebResearcher - 研究智能体"""
    
    def __init__(self):
        super().__init__("WebResearcher")
        self.research_depth = "deep"
        self.citation_style = "APA"
    
    async def reason(self, task: str) -> List[str]:
        """研究任务的推理逻辑"""
        # 深度研究流程
        steps = [
            "search_background",
            "browse_key_resources",
            "extract_data",
            "analyze_patterns",
            "validate_sources",
            "synthesize_findings"
        ]
        
        # 根据任务调整
        if "快速" in task:
            steps = steps[:3]
        elif "深度" in task:
            steps.append("peer_review_check")
        
        return steps
    
    async def integrate_results(self, results: List[Dict[str, Any]]) -> Dict[str, Any]:
        """整合研究结果"""
        data_points = []
        insights = []
        citations = []
        
        for result in results:
            if result["type"] == "search":
                for item in result["results"]:
                    data_points.append({
                        "source": item["engine"],
                        "content": item["snippet"],
                        "relevance": 0.8  # 模拟相关性评分
                    })
            
            elif result["type"] == "browse":
                insights.append({
                    "source": result["url"],
                    "insight": f"从 {result['url']} 发现重要信息",
                    "confidence": 0.9
                })
                citations.append(result["url"])
        
        # 生成研究报告
        research_paper = await self._write_research_paper(data_points, insights)
        
        return {
            "data_points": len(data_points),
            "insights": len(insights),
            "citations": citations[:5],
            "research_paper": research_paper,
            "conclusion": "研究完成，发现多个重要模式和趋势"
        }
    
    async def _write_research_paper(self, data_points: List, insights: List) -> Dict[str, Any]:
        """撰写研究报告"""
        await asyncio.sleep(0.5)
        
        return {
            "title": "深度研究报告",
            "abstract": "本研究通过多源数据分析，发现了重要模式和趋势",
            "methodology": "采用系统性的网络研究方法",
            "findings": f"基于 {len(data_points)} 个数据点和 {len(insights)} 个洞察",
            "discussion": "结果具有重要理论和实践意义",
            "references": [dp["source"] for dp in data_points[:3]]
        }
```

#### 步骤3：创建智能体协调系统
创建 `agent_coordinator.py`：

```python
"""
智能体协调系统 - 管理多个 WebAgents
"""
import asyncio
from typing import Dict, List, Any, Optional
from dataclasses import dataclass
import logging
from specialized_agents import WebWatcherAgent, WebWeaverAgent, WebResearcherAgent

logger = logging.getLogger(__name__)

@dataclass
class AgentTask:
    """智能体任务"""
    id: str
    description: str
    agent_type: str  # watcher, weaver, researcher, etc.
    priority: int = 1
    timeout: int = 30

class WebAgentCoordinator:
    """WebAgent 协调器"""
    
    def __init__(self):
        self.agents = {}
        self.task_queue = asyncio.Queue()
        self.results = {}
        self.agent_pool = {}
        
        # 初始化智能体池
        self._init_agent_pool()
    
    def _init_agent_pool(self):
        """初始化智能体池"""
        self.agent_pool = {
            "watcher": WebWatcherAgent(),
            "weaver": WebWeaverAgent(),
            "researcher": WebResearcherAgent()
        }
        
        logger.info("智能体池初始化完成")
    
    async def initialize(self):
        """初始化所有智能体"""
        logger.info("初始化所有智能体...")
        
        init_tasks = []
        for agent in self.agent_pool.values():
            init_tasks.append(agent.initialize())
        
        await asyncio.gather(*init_tasks)
        logger.info("所有智能体初始化完成")
    
    async def cleanup(self):
        """清理所有资源"""
        logger.info("清理智能体资源...")
        
        cleanup_tasks = []
        for agent in self.agent_pool.values():
            cleanup_tasks.append(agent.cleanup())
        
        await asyncio.gather(*cleanup_tasks)
        logger.info("资源清理完成")
    
    async def submit_task(self, task: AgentTask):
        """提交任务"""
        await self.task_queue.put(task)
        logger.info(f"提交任务: {task.id} - {task.description}")
    
    async def process_tasks(self, max_tasks: int = 10):
        """处理任务队列"""
        logger.info(f"开始处理任务，最大数量: {max_tasks}")
        
        processed = 0
        while processed < max_tasks and not self.task_queue.empty():
            try:
                task = await asyncio.wait_for(
                    self.task_queue.get(),
                    timeout=1.0
                )
                
                # 选择合适的智能体
                agent = self._select_agent(task)
                if agent:
                    # 异步执行任务
                    asyncio.create_task(
                        self._execute_with_timeout(agent, task)
                    )
                else:
                    logger.warning(f"没有合适的智能体处理任务: {task.id}")
                
                self.task_queue.task_done()
                processed += 1
                
            except asyncio.TimeoutError:
                break
            except Exception as e:
                logger.error(f"处理任务出错: {e}")
        
        logger.info(f"已调度 {processed} 个任务")
    
    def _select_agent(self, task: AgentTask) -> Optional[BaseWebAgent]:
        """选择智能体"""
        # 根据任务类型选择
        agent_type = task.agent_type.lower()
        
        if agent_type in self.agent_pool:
            return self.agent_pool[agent_type]
        
        # 默认选择
        logger.info(f"使用默认智能体处理任务: {task.id}")
        return self.agent_pool.get("researcher")
    
    async def _execute_with_timeout(self, agent: BaseWebAgent, task: AgentTask):
        """带超时的任务执行"""
        try:
            result = await asyncio.wait_for(
                agent.run(task.description),
                timeout=task.timeout
            )
            
            self.results[task.id] = {
                "status": "success",
                "result": result,
                "agent": agent.name
            }
            
            logger.info(f"任务完成: {task.id}")
            
        except asyncio.TimeoutError:
            self.results[task.id] = {
                "status": "timeout",
                "error": f"任务超时 ({task.timeout}s)",
                "agent": agent.name
            }
            logger.warning(f"任务超时: {task.id}")
            
        except Exception as e:
            self.results[task.id] = {
                "status": "error",
                "error": str(e),
                "agent": agent.name
            }
            logger.error(f"任务失败 {task.id}: {e}")
    
    def get_results(self) -> Dict[str, Any]:
        """获取所有结果"""
        return {
            "total_tasks": len(self.results),
            "successful": sum(1 for r in self.results.values() if r["status"] == "success"),
            "failed": sum(1 for r in self.results.values() if r["status"] == "error"),
            "timeout": sum(1 for r in self.results.values() if r["status"] == "timeout"),
            "results": self.results
        }

async def demo_webagent_system():
    """演示完整的 WebAgent 系统"""
    coordinator = WebAgentCoordinator()
    
    try:
        # 1. 初始化
        await coordinator.initialize()
        
        # 2. 提交任务
        tasks = [
            AgentTask("1", "监控科技新闻网站的变化", "watcher", priority=2),
            AgentTask("2", "整合人工智能最新发展报告", "weaver", priority=1),
            AgentTask("3", "研究机器学习在医疗领域的应用", "researcher", priority=1),
            AgentTask("4", "快速搜索Python编程技巧", "researcher", priority=3),
            AgentTask("5", "跟踪气候变化相关研究", "watcher", priority=2)
        ]
        
        for task in tasks:
            await coordinator.submit_task(task)
        
        # 3. 处理任务
        await coordinator.process_tasks(max_tasks=5)
        
        # 4. 等待任务完成
        await asyncio.sleep(5)
        
        # 5. 查看结果
        results = coordinator.get_results()
        print("\n" + "="*60)
        print("任务执行结果:")
        print("="*60)
        
        for task_id, result in results["results"].items():
            status = result["status"]
            agent = result["agent"]
            print(f"任务 {task_id}: {status} (智能体: {agent})")
        
        print(f"\n统计: 成功 {results['successful']}, 失败 {results['failed']}, 超时 {results['timeout']}")
        
    finally:
        # 6. 清理资源
        await coordinator.cleanup()

if __name__ == "__main__":
    asyncio.run(demo_webagent_system())
```

#### 步骤4：运行和测试
```bash
# 运行完整的 WebAgent 系统演示
python agent_coordinator.py

# 预期输出：
# 初始化所有智能体...
# 所有智能体初始化完成
# 提交任务: 1 - 监控科技新闻网站的变化
# 开始处理任务，最大数量: 5
# 已调度 5 个任务
# [WebWatcher] 开始任务: 监控科技新闻网站的变化
# [WebWeaver] 开始任务: 整合人工智能最新发展报告
# ...
# 任务执行结果:
# ============================================================
# 任务 1: success (智能体: WebWatcher)
# 任务 2: success (智能体: WebWeaver)
# ...
# 统计: 成功 5, 失败 0, 超时 0
# 清理智能体资源...
# 资源清理完成
```

### 实验总结

#### 1. WebAgent 模块核心理解
- **专业化设计**：每个智能体专注特定领域，提高效率和质量
- **工具链集成**：统一的工具接口，便于扩展和维护
- **异步架构**：充分利用现代硬件，提高并发处理能力
- **记忆和学习**：通过经验积累不断优化表现

#### 2. 关键技术要点
1. **aiohttp 异步HTTP客户端**：高效处理网络请求
2. **asyncio 并发框架**：管理多个智能体和任务
3. **抽象基类设计**：确保代码一致性和可扩展性
4. **配置化工具链**：灵活调整智能体能力

#### 3. DeepResearch 项目中的实际应用
- **WebWatcher**：实际用于监控学术网站、新闻源、社交媒体
- **WebWeaver**：整合多源信息生成综合报告
- **WebResearcher**：执行深度学术研究和文献综述
- **协调系统**：智能分配任务，优化资源利用

#### 4. 性能优化考虑
- **连接池管理**：重用HTTP连接，减少开销
- **超时控制**：防止任务阻塞系统
- **内存限制**：控制经验记忆大小
- **错误处理**：优雅处理网络异常和工具故障

### 实验扩展

#### 扩展任务1：实现新的 specialized agent
创建一个 `WebSummarizerAgent`，专门用于：
- 自动总结长文档
- 提取关键信息
- 生成简洁摘要

#### 扩展任务2：添加工具缓存机制
为工具添加缓存层，减少重复请求：
- 缓存网页内容
- 缓存搜索结果
- 实现缓存过期策略

#### 扩展任务3：实现智能体性能监控
添加监控功能：
- 记录每个智能体的执行时间
- 统计任务成功率
- 分析工具使用频率
- 生成性能报告

#### 扩展任务4：集成真实API
替换模拟工具为真实API：
- 使用真实的搜索引擎API
- 集成网页解析库（BeautifulSoup, lxml）
- 添加反爬虫策略

---

**实验完成标准**：
- ✅ 理解 WebAgent 模块的架构设计
- ✅ 实现多个 specialized agents
- ✅ 掌握异步处理和工具集成
- ✅ 运行完整的智能体协调系统
- ✅ 能够解释各智能体的职责和协作方式

通过本实验，你已经深入理解了 DeepResearch 项目中 WebAgent 模块的设计思想和实现方式，为后续学习其他模块和实际应用打下了坚实基础。