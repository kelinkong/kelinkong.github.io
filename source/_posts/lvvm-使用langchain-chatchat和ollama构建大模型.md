---
title: llvm-使用langchain-chatchat和ollama构建大模型
date: 2024-11-07 19:14:13
cagegories: [LLVM]
---

## 工具

**github地址：**

langchina-chatchat地址：[Langchain-Chatchat
](https://github.com/chatchat-space/Langchain-Chatchat)

ollama地址：[ollama](https://github.com/ollama/ollama)

## 使用langchain-chatchat和ollama构建大模型

### 安装

**langchain-chatchat**

langchain-chatchat在0.3版本之后支持使用pip安装，这里推荐创建一个新的虚拟环境来安装。

```bash
# 创建环境
conda create -n langchain python=3.10
conda activate langchain

# 安装
pip install langchain-chatchat -U
```

**ollama**

ollama的安装比较简单，只需要使用pip安装即可。

```bash
pip install ollama
```

### 初始化

**langchain-chatchat**

**初始化**，建议创建一个空的文件夹来进行初始化。

```bash
chatchat init
```

该命令会执行以下操作：

- 创建所有需要的数据目录
- 复制 samples 知识库内容
- 生成默认 yaml 配置文件

**修改配置文件**

具体每个配置文件的作用可以到github仓库查看，这里只列出需要修改的配置文件。

在文件`model_settings.yaml`中，修改:

```yaml
  - platform_name: ollama
    platform_type: ollama
    api_base_url: http://127.0.0.1:11434/v1
    api_key: EMPTY
    api_proxy: ''
    api_concurrencies: 5
    auto_detect_model: false
    llm_models: # 本地部署的模型
      - qwen2
    embed_models:
      - quentinz/bge-large-zh-v1.5 # 嵌入模型
    text2image_models: []
    image2text_models: []
    rerank_models: []
    speech2text_models: []
    text2speech_models: []
```

前面的默认模型可以不修改，开启自动检测后，会自动检测ollama中的模型。

**ollama**

加载大语言模型：
```bash
ollama run qwen2
```
如果本地没有部署，会自动下载模型。打开浏览器查看是否正确启动：http://localhost:11434

加载嵌入模型：
```bash
ollama pull quentinz/bge-large-zh-v1.5:latest
```

嵌入模型不需要运行，只需要下载即可。

> 嵌入模型（例如 quentinz/bge-large-zh-v1.5）的主要作用是将文本转换成向量表示，这种向量表示也叫做“嵌入” (embedding)。这些嵌入可以捕捉到文本的语义信息，以便在不同的 NLP 任务中进行高效的相似度计算和语义搜索。
> 
> quentinz/bge-large-zh-v1.5 可以用于生成中文文本的嵌入。例如，当输入一句中文文本时，该模型会输出一个向量（embedding），我们可以利用这个向量来完成以下操作

### 生成知识库

在生成知识库之前，要先确保ollama已经正确启动，使用默认的向量数据库，需要下载驱动，在pycharm中点击info.db，根据提示下载驱动即可。

使用`chatchat`命令生成知识库：

```bash
chatchat kb -r
```

这里可能会遇到各种bug，自行到社区issue中查找解决方案。如果没有解决方案，可以给社区提issue。

出现以下提示表示生成成功：

```bash

----------------------------------------------------------------------------------------------------
知识库名称      ：samples
知识库类型      ：faiss
向量模型：      ：bge-large-zh-v1.5
知识库路径      ：/root/anaconda3/envs/chatchat/lib/python3.11/site-packages/chatchat/data/knowledge_base/samples
文件总数量      ：47
入库文件数      ：42
知识条目数      ：740
用时            ：0:02:29.701002
----------------------------------------------------------------------------------------------------

总计用时        ：0:02:33.414425
```

### 运行

```bash
chatchat start -a
```

运行成功之后，可以在网页上进行知识库配置、对话等。

其余功能有待摸索。