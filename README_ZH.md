# Casevo：认知代理与社会演化模拟器

![Casevo Logo](logo_casevo.svg)

## 1. 简介

**Casevo（认知代理与社会演化模拟器）**是一个专门用于构建基于复杂网络的社会模拟多智能体实验或应用的Python框架。本教程旨在帮助您入门。通过完成本教程，您将发现Casevo的核心功能。在整个教程中，您将逐步创建一个基础级别的模型，随着进展，功能将不断增加。

本教程的目标是帮助您构建一个简单的基础级别模型，并能够对结果进行简单分析。

我们的论文可在[此链接](https://arxiv.org/abs/2412.19498)获取。欢迎引用。

```bibtex
@misc{jiang2024casevocognitiveagentssocial,
      title={Casevo: A Cognitive Agents and Social Evolution Simulator}, 
      author={Zexun Jiang and Yafang Shi and Maoxu Li and Hongjiang Xiao and Yunxiao Qin and Qinglan Wei and Ye Wang and Yuan Zhang},
      year={2024},
      eprint={2412.19498},
      archivePrefix={arXiv},
      primaryClass={cs.SI},
      url={https://arxiv.org/abs/2412.19498}, 
}
```

## 2. 背景信息

Casevo基于[Mesa](https://github.com/projectmesa/mesa)构建，因此在基本元素和操作逻辑上相似。关键相关信息：

- **Mesa**：一个基于Python的基于代理的建模（[**agent-based model，ABM**](https://zh.wikipedia.org/wiki/%E4%B8%AA%E4%BD%93%E4%B8%BA%E6%9C%AC%E6%A8%A1%E5%9E%8B)）工具，逻辑简单，易于二次开发。
  - [GitHub链接](https://github.com/projectmesa/mesa)
- Mesa中模拟实验的组成部分：
  - Model：类似于场景的概念，定义实验中的全局场景信息，包括调度方法和建模方式。
  - Agent：场景中的代理。
  - 入口文件：模拟实验的入口文件，通常命名为 `run.py`。

模拟实验的基本操作逻辑：

- 入口文件重复调用模型中定义的 `step`函数，代表模拟的每一轮。
- 模型中的 `step`函数调用调度器，调度器根据调度安排调用相应代理的 `step`函数。

总结来说，**Model定义全局信息和事件，而Agent定义自己的局部信息和事件**。

如果您想进一步了解Mesa的操作逻辑，请仔细阅读[文档](https://mesa.readthedocs.io/stable/tutorials/intro_tutorial.html)。

## 3. 工具安装

创建并激活Python[虚拟环境](https://docs.python-guide.org/dev/virtualenvs/)或[conda环境](https://docs.conda.org.cn/projects/conda/en/stable/user-guide/index.html)。需要Python 3.11以上版本。

- 下载工具的whl文件 `casevo-0.3.18-py3-none-any.whl`。
- 使用 `pip install casevo-0.3.18-py3-none-any.whl`安装工具。

## 4. 构建示例模拟实验

### 4.1 模拟场景描述与配置文件构建

目标是构建一个入门级的社会模拟实验。在美国总统选举中，候选人组织辩论，通过电视转播，让候选人表达观点并吸引选票。然而，辩论的效果不容易量化或衡量。为了解决这个问题，我们可以创建一个虚拟选民池，模拟选民观看辩论内容、相互讨论并最终投票的过程，从而评估候选人的辩论效果。这个设计不仅确保代理做出有组织的决策，还通过记忆机制和反思过程反映了信息交换和个人经验对最终决策的影响。

模拟实验中的关键信息包括：

- 辩论内容：使用2020年美国总统电视辩论的文本[链接1](https://www.debates.org/voter-education/debate-transcripts/october-22-2020-debate-transcript/) [链接2](https://www.presidency.ucsb.edu/documents/presidential-debate-belmont-university-nashville-tennessee-0)
  - 分为6个部分，放在 `content`文件夹中。
- 选民画像：来自[政治选民研究文章](https://www.pewresearch.org/politics/2021/11/09/beyond-red-vs-blue-the-political-typology-2/)
  - 共9个选民画像，选择3个在 `person.json`文件中以JSON格式配置。
- 网络结构：
  - 9个节点的随机网络，使用完全网络。

以下代码在 `build_case.py`中可用于构建模拟场景配置：

```python
import networkx as nx
from networkx.readwrite import json_graph
import matplotlib.pyplot as plt
import json
# 生成图
# 节点数量
node_num = 3
graph = nx.complete_graph(node_num)
graph_data = json_graph.node_link_data(graph)

# 配置画像
with open('person.json') as f:
    person_data = json.load(f)

output_item = {
    "graph": graph_data,
    "person": person_data[:node_num]
}

# 输出实验配置文件
with open('case_lite.json', 'w') as f:
    json.dump(output_item, f, ensure_ascii=False)
```

输出结果为 `case_lite.json`：

```json
{
    "graph": {
        "directed": false,
        "multigraph": false,
        "graph": {},
        "nodes": [
            {
                "id": 0
            },
            {
                "id": 1
            },
            {
                "id": 2
            }
        ],
        "links": [
            {
                "source": 0,
                "target": 1
            },
            {
                "source": 0,
                "target": 2
            },
            {
                "source": 1,
                "target": 2
            }
        ]
    },
    "person": [
        {
            "general": "一位55岁的保守派白人男性，生活在农村社区。",
            "character": "1) 虔诚的基督徒，认为宗教信仰非常重要。支持宗教在公共生活中发挥更大作用，认为政府政策应该支持宗教价值观。\n 2) 关注政府和公共事务，希望减少政府在社会中的角色，对堕胎和同性婚姻持限制性观点，认为同性婚姻合法化对国家有害，认为白人不会因为种族而在社会中获得优势，更可能认为白人今天面临重大歧视。\n 3) 认为强大的国家军事存在在国际事务中至关重要，确保和平的最佳方式是通过军事力量而不是外交。支持军事扩张。",
            "issue": "a) 认为国家是世界上最强大的。\n b) 认为非法移民是一个非常严重的国家问题。\n c) 认为政府应该减少对弱势群体的公共援助。\n d) 认为参与宗教的人口比例下降对社会有害。\n e) 支持允许城镇在公共财产上放置宗教符号。\n f) 认为保守派不能自由表达观点。"
        },
        {
            "general": "一位50岁以上的白人男性，经济稳定，受过良好教育。",
            "character": "1) 持有亲商观点，对国际贸易持积极态度，主张限制政府角色。他们的国际关系方法以与美国盟友合作和维护美国军事力量为中心，在外交政策中考虑盟友利益。\n 2) 对移民和种族问题持更温和立场，但总体上保持保守态度。3) 积极参与政治活动。",
            "issue": "a) 认同国会中共和党成员的行动。\n b) 认为政府对贫困人口的援助导致过度依赖政府援助，弊大于利。\n c) 认为每个人都有能力靠自己成功。\n d) 部分同意疫苗是保护人们免受COVID-19的最佳方式。"
        },
        {
            "general": "一位生活在农村地区的白人女性，收入和受教育水平较低。",
            "character": "1) 对移民持强硬立场，对经济体系持批评态度，认为非法移民是一个非常严重的国家问题，认为应该减少美国的合法移民，认为美国人口中白人比例下降对社会有害。\n 2) 对政府和经济体系持批评和怀疑态度，主张增加对大公司的税收，认为娱乐行业、科技公司和工会对国家发展产生了负面影响。",
            "issue": "a) 认为政府几乎总是效率低下和浪费。\n b) 认为政府过多干预个人和商业事务。\n c) 认为非法移民经常使他们的社区变得更糟。\n d) 认为国家的经济体系不公平地偏袒强大的利益集团。\n e) 主张增加对收入超过40万美元家庭的税收。"
        }
    ]
}
```

整体模拟过程可描述如下：

- 模拟分为6轮，每轮对应一部分辩论内容，包括以下事件：
  - 公开辩论：代理在此阶段接收总统候选人辩论的信息，主要是理解和记录候选人的陈述和政策立场。每个代理根据自己的观点和偏好做出初步判断。
  - 讨论：辩论后，代理与其他选民讨论。这是代理之间信息交换和观点冲突的关键阶段。讨论主题可以包括对辩论的看法、对候选人政策的讨论或个人经验的分享。
  - 反思：讨论后，每个代理进行自我反思。在此阶段，代理根据讨论内容、对辩论的理解和记忆中的信息进行综合思考。反思过程引入了记忆机制，允许代理根据过去的经验和记忆调整对候选人的看法。
  - 投票：每个代理根据辩论、讨论和反思的结果做出投票决定。这一步是整个过程的最终输出，所有代理的决定都反映在投票结果中。

### 4.2 初始化目录和文件

- 创建 `election_example`项目文件夹。
- 在项目文件夹中创建主要目录：
  - `content`：辩论内容的文本，按轮次命名为 `1.txt, 2.txt, ...`
  - `prompt`：所有提示的模板
  - `log`：日志输出目录
  - `memory`：记忆向量数据库目录
- 创建主要项目文件：
  - `election_model.py`：模拟场景文件
  - `election_agent.py`：代理文件
  - `baichuan.py`：大模型接口文件，本示例中实现百川智能API
  - `run.py`：模拟入口点

*代码将在教程的后续部分补充。*

### 4.3 实现大模型接口文件

本示例集成了[百川智能API](https://platform.baichuan-ai.com/docs/api)，具体实现参考[LLM_INTERFACE](#user-content-51-大模型接口-llm_interfacepy-模块)。

### 4.4 创建代理（`election_agent.py`）

#### 4.4.1 代理初始化

使用[casevo.AgentBase](#user-content-56-代理基类-agentbase-agent_basepy)初始化代理，主要步骤包括：

- 初始化父类
- 加载提示
- 设置思维链（CoT）

```python
from casevo import AgentBase
from casevo import BaseStep, JsonStep
from casevo import TotLog
# 定义思维链中的步骤
class OpinionStep(BaseStep):
    def pre_process(self, input, agent=None, model=None):
        cur_input = input['input']
        cur_input['issue'] = input['last_response']
        return cur_input

class AgreeStep(JsonStep):
    def pre_process(self, input, agent=None, model=None):
        cur_input = input['input']
        cur_input['opinion'] = input['last_response']
        return cur_input
      
class ElectionAgent(AgentBase):
    def __init__(self, unique_id, model, description, context):
        # 初始化父类
        super().__init__(unique_id, model, description, context)
        # 加载提示
        issue_prompt = self.model.prompt_factory.get_template("issue.txt")
        opinion_prompt = self.model.prompt_factory.get_template("opinion.txt")
        agree_prompt = self.model.prompt_factory.get_template("agree.txt")
        talk_prompt = self.model.prompt_factory.get_template("talk.txt")
        # 设置思维链
        issue_step = BaseStep(0, issue_prompt)
        opinion_step = OpinionStep(1, opinion_prompt)
        agree_step = AgreeStep(2, agree_prompt)
        talk_step = BaseStep(3, talk_prompt)
        chain_dict = {
            'listen': [issue_step, opinion_step, agree_step],
            'talk': [talk_step]
        }
        self.setup_chain(chain_dict)
```

#### 4.4.2 定义代理行为函数

通过 `ElectionAgent`类的成员函数定义代理的行为函数。代理的行为逻辑通过思维链和多轮输入-处理-输出来管理，决定代理如何接收信息、处理信息、反思和与其他代理交互。

###### (1) **`listen`**函数

- **功能**：模拟代理的倾听行为。此函数接收来自候选人辩论或其他代理讨论的内容，通过思维链处理，记录代理的理解和反应。
  - 代理将处理后的意见、同意程度等内容存储在记忆模块中，并记录相关信息。

###### (2) **`talk`**函数

- **功能**：实现代理之间的对话行为。代理根据听到的辩论信息、反思结果和长期记忆，与目标代理进行互动，交换意见。
  - 函数首先通过思维链处理信息，然后生成对话内容并记录。对话后，目标代理将对话内容传递给自己的 `listen`函数进行进一步处理。

###### (3) **`reflect`**函数

- **功能**：代理对长期意见进行反思，更新长期记忆。通过比较反思前后意见的变化，代理可以学习和适应新信息，同时记录反思过程。

#### 4.4.3 定义代理调度函数

通过重写 `casevo.AgentBase.step`函数，实现模型对代理的调度。在本示例中，代理以一定概率与其邻居中的另一个代理交互，通过 `talk`函数进行对话和信息交换。

### 4.5 创建模型（`election_model.py`）

#### 4.5.1 模型初始化

使用[casevo.ModelBase](#user-content-57-模型基类-modelbase-model_basepy)初始化模型，主要步骤包括：

- 初始化父类
- 生成代理并添加到调度序列

```python
from election_agent import ElectionAgent
from casevo import ModelBase
from casevo import TotLog

# 测试示例模型
class ElectionModel(ModelBase):

    # 根据配置生成模型
    def __init__(self, tar_graph, person_list, llm):
        """
        初始化对话系统中的每个人及其对话过程。
        :param tar_graph: 表示对话系统结构的目标图。
        :param person_list: 包含对话中所有参与者信息的人员列表。
        :param llm: 用于生成对话内容的语言模型。
        """
        super().__init__(tar_graph, llm)
        # 设置代理
        for cur_id in range(len(person_list)):
            cur_person = person_list[cur_id]
            cur_agent = ElectionAgent(cur_id, self, cur_person, None)
            self.add_agent(cur_agent, cur_id)
```

#### 4.5.2 定义全局事件函数

通过 `ElectionModel`类的成员函数定义全局事件函数。在本示例中，包括以下两个全局函数：

###### (1) **`public_debate`**函数

- **功能**：此函数模拟公开辩论的过程，让每个代理倾听辩论内容，并将事件添加到日志中。

###### (2) **`reflect`**函数

- **功能**：让所有代理进行反思，执行各自的 `reflect`成员函数。

#### 4.5.3 定义模拟入口函数

通过重写 `casevo.ModelBase.step`函数，实现一轮模拟实验。在本示例中，包括倾听辩论、代理自由讨论和形成意见。

```python
# 整体模型的step函数
    def step(self):
        # 倾听辩论内容
        self.public_debate()
        # 代理自由讨论
        self.schedule.step()
        # 代理反思
        self.reflect()
        return 0
```

### 4.6 编写模拟入口文件并运行（`run.py`）

编写模拟入口文件，主要功能包括：

- 初始化大模型接口
- 读取模拟配置
- 初始化模型
- 为每一轮调用 `ElectionModel.step`函数

通过在命令行执行以下命令运行实验：

```cmd
python run.py case_lite.json 6
```

其中 `case_lite.json`是配置文件，`6`是模拟轮数。

PS：在示例中，您需要将 `run.py`中的 `API_KEY`替换为有效的百川大模型API_KEY。

### 4.7 分析结果

结果将输出在 `log`文件夹中，包括以下文件：

- `agent_id.json`：对应ID代理的事件日志，包含代理在各个阶段的意见变化。
- `event.json`：所有事件的日志。
- `model.json`：模型的全局事件日志。

## 5. 模块和API概述

### 5.1 大模型接口：`llm_interface.py`模块

此模块定义了与大语言模型（LLM）交互的接口基类。
该接口提供了集成不同LLM实现的标准方法，使其易于扩展和集成。

#### 5.1.1 类：`LLM_INTERFACE`

这是一个**抽象基类**，定义了与LLM交互所需的基本方法，必须在子类中**重写**。

- `send_message(prompt, json_flag=False)`:
  - **参数**：
    - `prompt`：发送给LLM的提示文本。
    - `json_flag`：布尔值，指示返回的数据是否应为JSON格式。默认为 `False`。
  - **返回**：此方法应返回LLM对提示的响应。
- `send_embedding(text_list)`:
  - **参数**：
    - `text_list`：表示要生成嵌入的文本的字符串列表。
  - **返回**：此方法应返回与输入文本对应的嵌入。
- `get_lang_embedding()`:
  - **返回**：此方法应返回用于生成LangChain嵌入的工具类实例。

### 5.2 提示模板：Prompt + PromptFactory（`prompt.py`）

此模块定义了用于生成和发送提示信息的类，集成了模板渲染功能。

- 目的：
  - 使用模板定义提示
  - 使用Jinja2进行模板化
- 模型共享工厂类PromptFactory，
  - 标准化LLM调用接口
- 每个代理使用自己的 `Prompt`类

#### 5.2.1 类：`Prompt`

处理提示信息的生成和发送。

**属性**：

- `template`：提示模板对象。
- `factory`：用于发送生成的提示信息的工厂对象。

**方法**：

- `init(tar_template, tar_factory)`：初始化 `Prompt`实例。此构造函数通过提供模板和工厂参数为后续操作准备必要的属性。

  - **参数**
    - `tar_template`：用于生成目标文件的模板对象。模板对象应具有特定的结构和规则来指导目标文件的创建。
    - `tar_factory`：用于基于模板生成目标文件的工厂对象。工厂对象应具有基于模板生成实际目标文件的方法和逻辑。
  - **返回**：无返回值。此方法主要初始化类的实例属性。
- `get_prompt(tar_dict)`：通过使用提供的字典渲染模板来生成提示文本。
- `send_prompt(ertra=None, agent=None, model=None)`：生成并发送提示消息。此方法基于提供的参数生成并发送提示消息。它支持使用代理和模型自定义提示内容。额外参数（ertra）可以提供进一步的自定义。

  - **参数**
    - `ertra`：用于提供额外自定义信息的额外参数，默认为None。
    - `agent`：代理对象，如果提供，将使用代理的描述和上下文来自定义提示消息。
    - `model`：模型对象，如果提供，将使用模型的上下文来自定义提示消息。
  - **返回**：发送的提示消息的响应结果。

#### 5.2.2 类：`PromptFactory`

管理提示模板并生成 `Prompt`对象。

**属性**：

- `prompt_folder`：模板文件夹的路径。
- `env`：用于加载模板的Jinja2环境对象。
- `llm`：语言模型实例。

**方法**：

- `init(tar_folder, llm)`：初始化 `PromptFactory`实例，负责设置模板文件夹路径和语言模型，以及验证模板文件夹的存在。

  - **参数**：
    - `param tar_folder`：模板文件夹的路径。必须是现有目录。
    - `param llm`：用于自然语言处理的语言模型实例。
- `get_template(tar_temp)`：根据给定的模板名称从特定文件夹中检索模板文件。此方法首先构建模板文件的完整路径，然后检查文件是否存在。

  - 如果文件不存在，则抛出异常。
  - 如果文件存在，则使用环境变量加载模板，并返回使用加载的模板和当前对象初始化的 `Prompt`对象。
  - **参数**：
    - `tar_temp (str)`：模板文件的名称。
  - **返回**：
    - `Prompt`：使用加载的模板和当前对象初始化的 `Prompt`对象。
  - **抛出**：
    - `Exception`：如果指定的模板文件不存在，则抛出异常。
- `send_message(prompt_text)`：将提示文本发送到语言模型并返回响应结果。

### 5.3 思维链：Step + ThoughtChain（`chain.py`）

此模块定义了用于处理步骤的链式结构，包括基本步骤、选择步骤和评分步骤，以及用于执行这些步骤的ThoughtChain类。这些类允许用户创建和管理复杂的交互过程。

目的：快速定义和调用思维链过程。

#### 5.3.1 步骤类：`BaseStep`

`BaseStep`是所有步骤的**基类**，提供预处理输入、执行操作和处理后处理任务的基本方法。

- 构造函数 `def __init__(self, step_id, tar_prompt)`
  - **参数**
    - `step_id`：步骤的唯一标识符。
    - `tar_prompt`：需要用户回答的问题或提示。
- **方法**
  - `pre_process(input, agent=None, model=None)`：对输入数据执行**预处理**并返回处理后的数据。*（目前，函数直接返回输入，但随着功能扩展可能会添加额外的预处理步骤。）*
    - **参数**
      - `input`：需要预处理的输入数据。
      - `agent`：（可选）用于特定预处理任务的代理对象。
      - `model`：（可选）用于特定预处理任务的模型对象。
    - **返回**：处理后的输入数据。
  - `action(input, agent=None, model=None)`：根据输入和上下文执行特定操作。此函数的主要职责是通过调用提示模块中的 `send_prompt`方法发送提示并获取响应，考虑输入、代理和模型信息。此方法通常用于在对话系统中生成回复。
    - **参数**
      - `input (str)`：用户的输入，作为生成响应的基础。
      - `agent (Agent, optional)`：用于传递上下文信息的代理对象。默认为None。
      - `model (Model, optional)`：用于处理输入和生成响应的模型对象。默认为None。
    - **返回**：生成的响应文本。
  - `after_process(input, response, agent=None, model=None)`：这是对话后的回调函数。它收集并返回关键信息，例如原始用户输入和最终响应，以字典形式。
    - **参数**
      - `input`：用户的原始输入，用于记录或进一步处理。
      - `response`：机器人对用户输入的响应，用于分析或记录。
      - `agent`：代理对象，通常用于访问对话管理相关功能。默认为None，表示不使用。
      - `model`：模型对象，通常用于访问自然语言处理相关功能。默认为None，表示不使用。
    - **返回**：包含原始用户输入和系统生成的最后一个响应的字典。

#### 5.3.2 思维链的三种常见步骤类型：选择、评分、JSON

1. **选择步骤类**：`ChoiceStep`用于需要模型做出选择的交互步骤，添加了处理模型选择响应的逻辑。
2. **评分步骤类**：`ScoreStep`用于根据给定的步骤ID、目标提示和评分模板评估和生成评分响应。
3. **JSON步骤类**：`JsonStep`用于处理JSON格式的数据。

#### 5.3.3 思维链：`ThoughtChain`

此类扩展 `BaseAgentComponent`，表示由一系列步骤组成的操作链。

- **构造函数：`def __init__(self, agent, step_list)`**：此构造函数用于创建一个表示由多个步骤组成的操作链的实例。它继承自基类，并使用特定参数自定义实例。
  - **参数**：
    - `agent`：负责执行链中步骤的代理。
    - `step_list`：定义链的序列和内容的步骤列表。
- **方法**：
  - `set_input(input)`：设置输入内容并更新状态。
  - `run_step()`：执行思维链中的步骤，依次调用步骤类中的三个函数并更新步骤历史和输出。
  - `get_output()`：获取输出内容。状态必须为 `finish`。
  - `get_history()`：获取步骤历史。状态必须为 `finish`。

### 5.4 模块执行逻辑

1. **定义大模型接口**：用户根据自己的需求定制大模型接口，确保可以发送和接收消息。
2. **定义提示模板**：创建包含要发送到大模型的信息结构和格式的提示模板。
3. **自动调用大模型接口**：一旦设置了提示模板，系统可以自动调用大模型接口生成适当的响应。
4. **与思维链步骤结合**：将生成的提示与思维链中的特定步骤集成，形成完整的思维过程。
5. **通过思维链调用大模型**：最后，使用思维链的逻辑调用大模型，完成信息处理和响应生成。

### 5.5 记忆机制 Memory + MemoryFactory（`memory.py`）

此模块实现代理的记忆系统，管理短期和长期记忆，通过与ChromaDB集成实现持久化。

- 目的
  - 实现记忆机制
  - 检索
  - 反思
- 共享工厂类MemoryFactory
  - 提供统一的LLM接口。
- 每个代理的独立记忆类
- 所有记忆项存储在向量数据库表中
- RAG
  - 背景

#### 5.5.1 记忆元素：`MemoryItem`类

表示单个记忆项。

- **属性**：

  - `id`：记忆项的唯一标识符，默认为-1。
  - `ts`：时间戳。
  - `source`：源代理。
  - `target`：目标代理。
  - `action`：事件类型。
  - `content`：记忆内容。
- **方法**：

  - `init(ts, source, target, action, content)`：初始化记忆项。
  - `toDict()`：将记忆项转换为字典。
  - `toList(memory_list, start_id)`：将记忆项列表转换为内容、元数据和ID的列表。

#### 5.5.2 记忆模块：`Memory`类

代理的记忆模块，负责管理短期和长期记忆。

- **方法**：
  - `__init__(component_id, agent, tar_factory)`：初始化Memory实例。
  - `add_short_memory(source, target, action, content, ts=None)`：将记忆项添加到短期记忆中。
  - `search_short_memory_by_doc(content_list: List[str])`：根据文档内容列表搜索短期记忆。
  - `reflect_memory()`：更新长期记忆并检索最新的记忆ID。
  - `get_long_memory()`：检索长期记忆。

#### 5.5.3 全局记忆工厂：`MemoryFactory`类

全局记忆工厂模块，负责为代理创建记忆实体和管理记忆存储。

- **方法**：
  - `__init__(tar_llm: LLM_INTERFACE, memory_num, prompt, model, tar_path=None)`：初始化记忆模块。
  - `create_memory(agent)`：为指定代理创建Memory实例。
  - `add_short_memory(tar_memory: List[MemoryItem])`：将目标记忆项添加到短期记忆中。
  - `search_short_memory_by_doc(content_list: List[str], tar_agent)`：根据内容搜索短期记忆。
  - `reflect_memory(tar_agent, tar_pos, tar_long_opinion)`：反思短期记忆以更新长期记忆。

#### 5.5.4 外部记忆：`BackgroundItem/Background/BackgroundFactory`类

处理外部记忆的RAG功能，参考 `MemoryItem/Memory/MemoryFactory`类设计。

### 5.6 代理基类 AgentBase（`agent_base.py`）

此模块定义了代理的基类 `AgentBase`，为其他代理类提供基本功能和结构。`AgentBase`类扩展 `mesa.Agent`，负责初始化和管理代理的基本属性和行为。

- **构造函数**：`def __init__(self, unique_id, model, description, context)：`

  - **参数**：
    - `unique_id`：代理的唯一标识符。
    - `model`：代理所处的模型环境。
    - `description`：代理角色的描述。
    - `context`：代理的上下文信息（用于提示生成）。
  - **功能**：
    - 调用父类的初始化方法，设置代理的唯一ID和模型。
    - 生成代理特定的组件ID。
    - 初始化日志对象（可选，用于记录代理的操作）。
    - 设置代理的描述和上下文。
    - 使用模型的记忆工厂创建记忆对象来存储代理的状态信息。
- **方法**：

  - `setup_chain`：**初始化代理的思维链集合。**它为给定字典中的每个思维链创建 `ThoughtChain`实例，并将它们存储在 `self.chains`中以供后续操作。
    - **参数**
      - `chain_dict` (dict)：包含思维链标识符和数据的字典。
    - **返回**：无
  - **抽象方法**：`step`
    - 定义一个抽象方法，要求所有子类实现特定的代理行为。
    - 允许直接设置相应的思维链和调度函数，必须实现。

### 5.7 模型基类 ModelBase（`model_base.py`）

`model_base.py`模块定义了一个基于Mesa框架的模型类 `ModelBase`，它扩展 `mesa.Model`用于创建和管理代理模型。此模型支持网络结构、调度机制、记忆管理和提示生成。主要功能包括：

- **网络设置**：使用 `NetworkGrid`创建网络结构。
- **代理调度**：通过 `RandomActivation`调度器管理代理活动。
- **上下文管理**：支持上下文信息的传递。
- **提示工厂**：集成提示模板，便于与大模型接口交互。
- **记忆工厂**：管理短期和长期记忆。
- **属性**：

  - `grid`：代理的网络空间。
  - `schedule`：代理调度器。
  - `context`：上下文信息。
  - `llm`：语言模型。
  - `prompt_factory`：用于生成提示的提示工厂。
  - `memory_factory`：用于管理记忆项的记忆工厂。
  - `agent_list`：存储代理对象的列表。
- **方法**：

  - `init(tar_graph, llm, context=None, prompt_path='./prompt/', memory_path=None, memory_num=10, reflect_file='reflect.txt')`：初始化模型及其相关组件。
  - `add_agent(tar_agent, node_id)`：将新代理添加到模型并将其放置在指定节点上。
    - **参数**：
      - `tar_agent`：要添加的代理对象。
      - `node_id`：代理将被放置的节点的ID。
  - `step()`：执行模拟步骤，推进模拟时间并管理所有代理活动。此方法将模拟推进一步并更新所有调度对象。它不接受任何参数或返回任何有意义的值，主要用于触发模拟的进展。
    - **返回**：始终返回0作为步骤执行结果的指示。

### 5.8 日志类 TotLog（`util/tot_log.py`）

`TotLog`类用于**记录和管理日志数据**，支持将日志信息保存到文件并管理时间偏移。此类提供添加日志、设置日志和将日志写入文件的功能。

## 6 致谢

在Casevo的开发过程中，我们很幸运地得到了一群优秀的代码贡献者的支持。

- [Yafang Shi](https://github.com/Freya236)
- [Maoxu Li](https://github.com/limaoSure)
- [Hang Su](https://github.com/suhangha)
