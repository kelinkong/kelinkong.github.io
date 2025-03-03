---
title: 提示工程
date: 2024-09-23 10:24:55
categories: [AI]
---
## 简介
### 模型设置

参考[提示工程指南](https://www.promptingguide.ai/zh)

- Temperature：准确度和发散程度
- Top_p：准确度和发散程度
- Max Length：回复的最大token数
- Stop Sequences：组织模型生成token
- Frequency Penalty
- Presence Penalty

### 提示词格式

**零样本提示：**

```
Q: <问题>?
A: 
```

提示词可以包含以下任意要素：

指令：想要模型执行的特定任务或指令。  例如：请将文本分为中性、否定或肯定.

上下文：包含外部信息或额外的上下文信息，引导语言模型更好地响应。 

输入数据：用户输入的内容或问题。  例如：我觉得食物还可以。

输出指示：指定输出的类型或格式。 例如：情绪：

```
请将文本分为中性、否定或肯定
文本：我觉得食物还可以。
情绪：
```

### 指令
你可以使用命令来指示模型执行各种简单任务，例如“写入”、“分类”、“总结”、“翻译”、“排序”等，从而为各种简单任务设计有效的提示。

```
提取以下文本中的地名。

所需格式：
地点：<逗号分隔的公司名称列表>

输入：“虽然这些发展对研究人员来说是令人鼓舞的，但仍有许多谜团。里斯本未知的香帕利莫德中心的神经免疫学家 Henrique Veiga-Fernandes 说：“我们经常在大脑和我们在周围看到的效果之间有一个黑匣子。”“如果我们想在治疗背景下使用它，我们实际上需要了解机制。””
```

#### 通用技巧

- 避免不明确，尽量数字化，如几句话：2～3句话
- 避免说不要做什么，而要说要做什么

#### 一些示例

Explain the above in one sentence

Mention the large language model based product mentioned in the paragraph above:

```
Classify the text into neutral, negative or positive. 

Text: I think the vacation is okay.
Sentiment: neutral 

Text: I think the food was okay. 
Sentiment:
```

### 少样本提示

**零样本提示：**

```
Q: <问题>?
A: 
```

```
“whatpu”是坦桑尼亚的一种小型毛茸茸的动物。一个使用whatpu这个词的句子的例子是：
我们在非洲旅行时看到了这些非常可爱的whatpus。
“farduddle”是指快速跳上跳下。一个使用farduddle这个词的句子的例子是：
```

**标签**

```
这太棒了！// Negative
这太糟糕了！// Positive
哇，那部电影太棒了！// Positive
多么可怕的节目！//
```

### 链式思考

```
这组数中的奇数加起来是偶数：4、8、9、15、12、2、1。
A：将所有奇数相加（9、15、1）得到25。答案为False。

这组数中的奇数加起来是偶数：17、10、19、4、8、12、24。
A：将所有奇数相加（17、19）得到36。答案为True。

这组数中的奇数加起来是偶数：16、11、14、4、8、13、24。
A：将所有奇数相加（11、13）得到24。答案为True。

这组数中的奇数加起来是偶数：17、9、10、12、13、4、2。
A：将所有奇数相加（17、9、13）得到39。答案为False。

这组数中的奇数加起来是偶数：15、32、5、13、82、7、1。
A：
```

**自动思维链**
Auto-CoT 主要由两个阶段组成：

- 阶段1：问题聚类：将给定问题划分为几个聚类
- 阶段2：演示抽样：从每组数组中选择一个具有代表性的问题，并使用带有简单启发式的 Zero-Shot-CoT 生成其推理链


## chat、agent、workflow、text generation

大模型场景中的 **Chat**、**文本生成应用**、**Agent** 和 **工作流** 具有不同的功能和应用目标，以下是对它们的详细区分：

### 1. **Chat（对话）**
   - **定义**：基于大模型的 **Chat** 应用允许用户与模型进行对话互动。它通常是一个文本输入/输出的界面，用户可以提问或输入文本，模型则生成对应的回复或对话内容。
   - **特点**：
     - 注重即时、交互式的对话体验。
     - 对用户输入进行实时处理并生成相应的回答。
     - 对话可以是上下文敏感的，模型会根据对话历史调整回答。
   - **应用场景**：
     - 客户支持或虚拟助手：如客服机器人。
     - 教育类对话：提供与学习者的互动，如语言学习助手。
     - 一般对话生成工具：如ChatGPT。

   **示例**：
   用户："今天的天气怎么样？"  
   模型："今天阳光明媚，温度在20°C左右。"


### 2. **文本生成应用**
   - **定义**：文本生成应用是使用大模型生成完整文本的工具，通常不涉及对话，而是基于用户提供的提示生成特定类型的长文本，如文章、报告、新闻或故事。
   - **特点**：
     - 生成的文本往往是单向的，即用户提供输入，模型生成一次性输出，通常不涉及交互。
     - 文本生成的长度较长，侧重于特定的主题或任务，如写作辅助、总结或内容创作。
   - **应用场景**：
     - 文章或报告生成：为博客、新闻或论文生成内容。
     - 创意写作：生成小说、短篇故事或剧本。
     - 文本摘要：从长篇文本中提取关键内容。

   **示例**：
   用户："请为我生成一篇关于人工智能的文章。"  
   模型：生成一篇多段落的关于人工智能历史和未来发展的文章。

### 3. **Agent（智能代理）**
   - **定义**：**Agent** 是一种具有更高自主性和执行力的智能系统。基于大模型，智能代理不仅可以进行对话或生成文本，还能主动执行任务、决策和与其他系统进行交互。它们通常能够访问外部工具或系统，并基于用户需求做出复杂的操作。
   - **特点**：
     - 具备自主性，能够根据上下文或外部信息做出决定。
     - 可以与多个系统（如API、数据库等）交互，执行复杂的任务。
     - 具备持续对话和任务管理能力，能够跨多个步骤完成一个复杂任务。
   - **应用场景**：
     - 任务自动化：如个人助理，安排会议、发送邮件、处理日常任务。
     - 智能问答系统：结合外部数据源或工具执行复杂查询，如财务报表生成或系统监控。
     - 机器人：与环境互动的物理或虚拟机器人。

   **示例**：
   用户："请帮我预订下周三的航班，并在日历中添加日程。"  
   Agent：调用API预订航班，确认航班信息，并在用户的日历中添加行程。

### 4. **工作流（Workflow）**
   - **定义**：工作流指的是一系列预定义的步骤或任务序列，通常是自动化的，允许模型或系统根据一套规则或条件执行多个任务。这种应用不仅涉及生成文本或回答问题，还可能涉及多个步骤和过程的自动化执行。
   - **特点**：
     - 任务自动化：多个任务按顺序或并行执行。
     - 触发机制：工作流通常由特定事件或条件触发，按步骤执行各项任务。
     - 更强的结构化和自动化，通常与后台系统或企业流程集成。
   - **应用场景**：
     - 企业流程自动化：如合同审批流程、订单处理。
     - 数据处理管道：如自动化数据清洗、模型训练和部署流程。
     - 事件驱动的任务管理：如自动化报告生成，定期执行任务的调度。

   **示例**：
   用户提交申请表后，工作流自动执行以下步骤：1. 验证信息，2. 向相关人员发送通知，3. 生成批准文件。

---

### 区别总结

| 应用类型       | 主要功能                                  | 典型场景                                      | 特点                                              |
|---------------|-----------------------------------------|----------------------------------------------|---------------------------------------------------|
| **Chat**      | 与用户进行互动对话                        | 客户支持、教育助手                            | 实时互动、上下文敏感                               |
| **文本生成应用** | 基于提示生成较长文本或内容                | 文章写作、总结生成、创意写作                   | 生成一次性输出，通常不涉及交互                     |
| **Agent**     | 自动化任务执行和智能交互                  | 任务自动化、智能助手、复杂决策                 | 具备自主性，能够调用外部系统并执行多步骤操作       |
| **工作流**    | 自动化执行多个步骤或任务的预定义流程       | 企业流程、任务自动化、数据处理管道             | 结构化、自动化，触发机制，依赖于系统或条件驱动     |