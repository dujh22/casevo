# Casevo 使用指导手册

## 目录
1. [简介](#1-简介)
2. [安装指南](#2-安装指南)
3. [快速开始](#3-快速开始)
4. [核心概念](#4-核心概念)
5. [详细使用说明](#5-详细使用说明)
6. [常见问题](#6-常见问题)

## 1. 简介

Casevo（认知代理与社会演化模拟器）是一个基于Python的框架，专门用于构建基于复杂网络的社会模拟多智能体实验。本手册将指导您如何使用Casevo进行社会模拟实验。

### 1.1 主要特点
- 基于Mesa框架构建
- 支持复杂网络结构
- 集成大语言模型接口
- 提供记忆机制
- 支持思维链处理
- 灵活的提示模板系统

## 2. 安装指南

### 2.1 环境要求
- Python 3.11或更高版本
- 虚拟环境（推荐使用venv或conda）

### 2.2 安装步骤
1. 创建并激活虚拟环境
2. 下载Casevo的whl文件：`casevo-0.3.18-py3-none-any.whl`
3. 使用pip安装：`pip install casevo-0.3.18-py3-none-any.whl`

## 3. 快速开始

### 3.1 创建项目结构
```
your_project/
├── content/          # 存放实验内容
├── prompt/          # 提示模板
├── log/             # 日志输出
├── memory/          # 记忆向量数据库
├── election_model.py # 模型文件
├── election_agent.py # 代理文件
├── baichuan.py      # 大模型接口
└── run.py           # 运行入口
```

### 3.2 基本使用流程
1. 准备配置文件（case_lite.json）
2. 实现大模型接口
3. 创建代理类
4. 创建模型类
5. 编写运行脚本
6. 执行模拟实验

## 4. 核心概念

### 4.1 模型（Model）
- 定义全局场景信息
- 管理调度方法
- 控制模拟流程

### 4.2 代理（Agent）
- 代表模拟中的个体
- 具有自主决策能力
- 可以与其他代理交互

### 4.3 思维链（Thought Chain）
- 定义代理的思考过程
- 包含多个处理步骤
- 支持复杂决策逻辑

### 4.4 记忆机制（Memory）
- 短期记忆：临时存储信息
- 长期记忆：持久化重要信息
- 支持信息检索和反思

## 5. 详细使用说明

### 5.1 配置文件构建
```python
import networkx as nx
from networkx.readwrite import json_graph
import json

# 生成图结构
node_num = 3
graph = nx.complete_graph(node_num)
graph_data = json_graph.node_link_data(graph)

# 配置代理画像
with open('person.json') as f:
    person_data = json.load(f)

# 生成配置文件
output_item = {
    "graph": graph_data,
    "person": person_data[:node_num]
}

with open('case_lite.json', 'w') as f:
    json.dump(output_item, f, ensure_ascii=False)
```

### 5.2 代理实现
```python
from casevo import AgentBase, BaseStep, JsonStep

class ElectionAgent(AgentBase):
    def __init__(self, unique_id, model, description, context):
        super().__init__(unique_id, model, description, context)
        # 加载提示模板
        issue_prompt = self.model.prompt_factory.get_template("issue.txt")
        opinion_prompt = self.model.prompt_factory.get_template("opinion.txt")
        # 设置思维链
        issue_step = BaseStep(0, issue_prompt)
        opinion_step = OpinionStep(1, opinion_prompt)
        chain_dict = {
            'listen': [issue_step, opinion_step],
            'talk': [talk_step]
        }
        self.setup_chain(chain_dict)
```

### 5.3 模型实现
```python
from casevo import ModelBase

class ElectionModel(ModelBase):
    def __init__(self, tar_graph, person_list, llm):
        super().__init__(tar_graph, llm)
        # 设置代理
        for cur_id in range(len(person_list)):
            cur_person = person_list[cur_id]
            cur_agent = ElectionAgent(cur_id, self, cur_person, None)
            self.add_agent(cur_agent, cur_id)
```

### 5.4 运行实验
```python
# 在命令行中执行
python run.py case_lite.json 6
```

## 6. 常见问题

### 6.1 环境配置问题
- 确保Python版本正确（3.11+）
- 检查虚拟环境是否正确激活
- 验证所有依赖包是否安装

### 6.2 运行问题
- 检查配置文件格式是否正确
- 确保大模型API密钥正确配置
- 验证文件路径是否正确

### 6.3 性能优化
- 合理设置代理数量
- 优化记忆存储策略
- 调整思维链复杂度

## 7. 获取帮助

如果您在使用过程中遇到问题，可以通过以下方式获取帮助：
1. 查阅官方文档
2. 查看GitHub Issues
3. 联系开发团队

## 8. 更新日志

请关注GitHub仓库获取最新版本和更新信息。

## 9. 贡献指南

欢迎贡献代码、报告问题或提出建议。请遵循以下步骤：
1. Fork项目
2. 创建特性分支
3. 提交更改
4. 发起Pull Request

## 10. 许可证

本项目采用MIT许可证。详情请参阅LICENSE文件。 