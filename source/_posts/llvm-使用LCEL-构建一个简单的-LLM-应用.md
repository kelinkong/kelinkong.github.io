---
title: 使用 LCEL 构建一个简单的 LLM 应用
date: 2024-11-06 09:43:04
categories: [LLVM]
---

## 简单教程
教程来源于：[LangChain](https://www.langchain.com.cn/docs/tutorials/llm_chain/)

## 使用 LCEL 构建一个简单的 LLM 应用

langchain的简单使用。

### 什么是LangChain？
langchain是一个用于构建自然语言处理（NLP）应用的工具包。它提供了一种简单的方法来构建NLP应用，无需编写复杂的代码。langchain的核心是一个称为“链”的概念，链是一系列处理步骤，每个步骤都接收输入并生成输出。链可以包含各种处理步骤，例如模型、解析器和提示。

在本教程中，会创建一个简单的LLM（Large Language Model）应用。

主要分为：prompt、model、parser


## 代码示例

```python
from fastapi import FastAPI
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langserve import add_routes
from langchain_groq import ChatGroq
import getpass

import os

os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
os.environ["GROQ_API_KEY"] = getpass.getpass()

# Create prompt template
system_template = "Translate the following into {language}:"
prompt_template = ChatPromptTemplate.from_messages([
    "system", system_template,
    "user", "{text}"
])

# Create model
model = ChatGroq(model="llama3-8b-8192")

# Create output parser
parser = StrOutputParser()

# Create chain
chain = prompt_template | model | parser

# Create FastAPI app
app = FastAPI(
    title="LangChain OpenAI API",
    description="A FastAPI app that uses LangChain and OpenAI to translate text.",
    version="0.1.0"
)

# Add routes
add_routes(
    app,
    chain,
    path="/chain"
)

@app.get("/")
async def read_root():
    return {"message": "Welcome to the LangChain OpenAI API"}

# Run FastAPI app
if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="localhost", port=8000)
```