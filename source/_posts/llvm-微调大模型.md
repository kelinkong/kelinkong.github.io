---
title: llvm-微调大模型
date: 2024-11-01 14:35:34
cagegories: [LLVM]
---

## 微调大模型

源代码来自:[老牛同学](https://www.cnblogs.com/obullxl/p/18312594/NTopic2024071801)

使用开源大模型 Qwen2-0.5B 的示例，实现了一个基于微调和 RAG（Retrieval-Augmented Generation）的文本分类助手。以下是各部分的详细解释：

### 1. 引入必要库
```python
import json
import pandas as pd
import torch
from datasets import Dataset
from modelscope import AutoTokenizer
from swanlab.integration.huggingface import SwanLabCallback
from peft import LoraConfig, TaskType, get_peft_model
from transformers import AutoModelForCausalLM, TrainingArguments, Trainer, DataCollatorForSeq2Seq
import os
import swanlab
```

这部分代码引入了主要用于微调、训练和生成文本的库，包括 `transformers`、`peft`（主要用于 LoRA 微调）、`datasets`（用于处理数据集），以及 `swanlab` 用于回调和日志记录。

### 2. 设置路径和设备
```python
BASE_DIR = 'D:\\ModelSpace\\Qwen2'
device = 'cuda' if torch.cuda.is_available() else 'cpu'
```
设置模型的根目录和设备名称。设备名称判断系统是否支持 CUDA（GPU 加速），如果不支持则使用 CPU。

### 3. 数据集格式转换函数
```python
def dataset_jsonl_transfer(origin_path, new_path):
    messages = []
    with open(origin_path, "r", encoding="utf-8") as file:
        for line in file:
            data = json.loads(line)
            text = data["text"]
            catagory = data["category"]
            output = data["output"]
            message = {
                "input": f"文本:{text},分类选项列表:{catagory}",
                "output": output,
            }
            messages.append(message)
    with open(new_path, "w", encoding="utf-8") as file:
        for message in messages:
            file.write(json.dumps(message, ensure_ascii=False) + "\n")
```
`dataset_jsonl_transfer` 函数用于将原始 JSON 数据转换成微调所需的数据格式。原始文件的每行包含一个 JSON 对象，函数将其读取、重构并保存成新的 JSONL 格式文件，每行包含一个示例数据。

### 4. 数据预处理函数
```python
def process_func(example):
    MAX_LENGTH = 384
    instruction = tokenizer(f"<|im_start|>system\n你是一个文本分类领域的专家...{example['input']}<|im_end|>\n<|im_start|>assistant\n", add_special_tokens=False)
    response = tokenizer(f"{example['output']}", add_special_tokens=False)
    input_ids = instruction["input_ids"] + response["input_ids"] + [tokenizer.pad_token_id]
    attention_mask = instruction["attention_mask"] + response["attention_mask"] + [1]
    labels = [-100] * len(instruction["input_ids"]) + response["input_ids"] + [tokenizer.pad_token_id]
    if len(input_ids) > MAX_LENGTH:
        input_ids, attention_mask, labels = input_ids[:MAX_LENGTH], attention_mask[:MAX_LENGTH], labels[:MAX_LENGTH]
    return {"input_ids": input_ids, "attention_mask": attention_mask, "labels": labels}
```
`process_func` 函数将数据处理成大模型可以接受的格式，包含 `input_ids`、`attention_mask` 和 `labels`。这里模拟了一个对话输入，用户提问，助手返回分类输出。如果序列长度超出最大限制 `MAX_LENGTH`，进行截断。

### 5. 加载模型和分词器
```python
model_dir = os.path.join(BASE_DIR, 'Qwen2-0.5B')
tokenizer = AutoTokenizer.from_pretrained(model_dir, use_fast=False, trust_remote_code=True)
model = AutoModelForCausalLM.from_pretrained(model_dir, device_map=device, torch_dtype=torch.bfloat16)
model.enable_input_require_grads()
```
加载模型和分词器，将 `bfloat16` 用于精度，以减少 GPU 占用量。`model.enable_input_require_grads()` 开启梯度检查点支持，以节省内存。

### 6. 加载和处理数据集
```python
train_jsonl_new_path = os.path.join(BASE_DIR, 'train.jsonl')
test_jsonl_new_path = os.path.join(BASE_DIR, 'test.jsonl')

if not os.path.exists(train_jsonl_new_path):
    dataset_jsonl_transfer(train_dataset_path, train_jsonl_new_path)
if not os.path.exists(test_jsonl_new_path):
    dataset_jsonl_transfer(test_dataset_path, test_jsonl_new_path)

train_df = pd.read_json(train_jsonl_new_path, lines=True)
train_ds = Dataset.from_pandas(train_df)
train_dataset = train_ds.map(process_func, remove_columns=train_ds.column_names)
```
检查并转换数据集，将其加载为 `Dataset` 格式，并通过 `process_func` 处理成可用于训练的数据格式。

### 7. LoRA 配置与应用
```python
config = LoraConfig(
    task_type=TaskType.CAUSAL_LM,
    target_modules=["q_proj", "k_proj", ...],
    inference_mode=False,
    r=8,
    lora_alpha=32,
    lora_dropout=0.1,
)

model = get_peft_model(model, config)
```
设置并应用 LoRA（Low-Rank Adaptation）配置，用于高效微调。`LoRA` 能通过添加低秩矩阵在不改变原模型参数的情况下更新模型，适合大模型的微调。

### 8. 训练参数与 Trainer 初始化
```python
args = TrainingArguments(
    output_dir=os.path.join(BASE_DIR, 'output', 'Qwen2-0.5B'),
    per_device_train_batch_size=4,
    gradient_accumulation_steps=4,
    logging_steps=10,
    num_train_epochs=2,
    save_steps=100,
    learning_rate=1e-4,
    save_on_each_node=True,
    gradient_checkpointing=True,
    report_to="none",
)

swanlab_callback = SwanLabCallback(project="Qwen2-FineTuning", experiment_name="Qwen2-0.5B")

trainer = Trainer(
    model=model,
    args=args,
    train_dataset=train_dataset,
    data_collator=DataCollatorForSeq2Seq(tokenizer=tokenizer, padding=True),
    callbacks=[swanlab_callback],
)
```
配置训练参数并创建 `Trainer` 实例，指定保存路径、batch 大小、梯度累积步数、日志记录频率、学习率等。`SwanLabCallback` 用于将训练过程发送至 `SwanLab` 进行实时监控。

### 9. 训练模型
```python
trainer.train()
```
调用 `.train()` 开始训练。

### 10. 模型推理与评估
```python
def predict(messages, model, tokenizer):
    text = tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
    model_inputs = tokenizer([text], return_tensors="pt").to(device)
    generated_ids = model.generate(model_inputs.input_ids, max_new_tokens=512)
    generated_ids = [output_ids[len(input_ids):] for input_ids, output_ids in zip(model_inputs.input_ids, generated_ids)]
    return tokenizer.batch_decode(generated_ids, skip_special_tokens=True)[0]
```
`predict` 函数实现了模型推理功能，将输入转成模型格式后生成输出文本。

### 11. 测试集上的推理
```python
test_df = pd.read_json(test_jsonl_new_path, lines=True)[:10]
test_text_list = []

for index, row in test_df.iterrows():
    instruction = row['你是一个文本分类领域的专家，你会接收到一段文本和几个潜在的分类选项列表，请输出文本内容的正确分类']
    input_value = row['input']

    messages = [{"role": "system", "content": f"{instruction}"}, {"role": "user", "content": f"{input_value}"}]
    response = predict(messages, model, tokenizer)
    messages.append({"role": "assistant", "content": f"{response}"})

    result_text = f"{messages[0]}\n\n{messages[1]}\n\n{messages[2]}"
    test_text_list.append(swanlab.Text(result_text, caption=response))

swanlab.log({"Prediction": test_text_list})
swanlab.finish()
```
在测试集上对模型进行评估，将预测结果和输入对比，并将输出文本记录到 `SwanLab`。