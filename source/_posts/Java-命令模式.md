---
title: Java-命令模式
date: 2025-11-18 09:43:49
categories: [Java]
---

**命令模式（Command Pattern）和大模型调用中的 Chain（Chain of Thought / Chain of Actions / Agent Chain / Pipeline Chain）本质上是同一种思想：将复杂逻辑拆成可插拔、可组合、可扩展的步骤链式执行。**

**大模型调用流程场景的命令链（Chain）简化 Demo**

> 场景：对话请求 → 经过步骤链：**内容安全检查 → Prompt 优化 → 调用模型 → 结果润色 → 结果输出**


如果是我来设计这个 Chain 模式，我会怎么写？

将每一种工具调用都封装为一个函数or一个类，然后按照步骤一个个调用。

如果是使用命令模式，要怎么实现？

## 目录结构

```
chain
 ├─ step // 各步骤实现
 │   ├─ CheckSafetyStep.java
 │   ├─ EnhancePromptStep.java
 │   ├─ CallLLMStep.java
 │   ├─ PolishOutputStep.java
 ├─ ChainStep.java // 统一接口
 ├─ ChainContext.java // 上下文对象
 ├─ ChainExecutor.java // 执行器
```

## ① 统一接口：ChainStep

```java
public interface ChainStep {
    void execute(ChainContext ctx);
    String stepName(); // 用于日志、调试、动态开关
}
```
每一个step都需要实现这个统一接口，包含执行方法execute和步骤名称stepName。

调用execute方法，进行执行。

## ② 上下文对象：ChainContext

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class ChainContext {
    private String userInput;
    private String optimizedPrompt;
    private String llmResult;
    private String finalOutput;
}
```

## ③ 各步骤示例

### 内容安全检测

```java
@Component
public class CheckSafetyStep implements ChainStep {
    @Override
    public void execute(ChainContext ctx) {
        if (ctx.getUserInput().contains("违法")) {
            throw new RuntimeException("内容违规");
        }
        System.out.println("安全检查通过");
    }
    public String stepName() { return "check_safety"; }
}
```

### Prompt 优化

```java
@Component
public class EnhancePromptStep implements ChainStep {
    @Override
    public void execute(ChainContext ctx) {
        ctx.setOptimizedPrompt("请专业、克制地回答：" + ctx.getUserInput());
        System.out.println("Prompt 优化完成");
    }
    public String stepName() { return "enhance_prompt"; }
}
```

### 调用大模型

```java
@Component
public class CallLLMStep implements ChainStep {
    @Override
    public void execute(ChainContext ctx) {
        // 假装调用大模型
        ctx.setLlmResult("【大模型回答】" + ctx.getOptimizedPrompt());
        System.out.println("模型调用完成");
    }
    public String stepName() { return "call_llm"; }
}
```

### 输出润色

```java
@Component
public class PolishOutputStep implements ChainStep {
    @Override
    public void execute(ChainContext ctx) {
        ctx.setFinalOutput(ctx.getLlmResult().replace("【大模型回答】", ""));
        System.out.println("输出润色完成");
    }
    public String stepName() { return "polish_output"; }
}
```

## ④ Chain 执行器（最核心部分）

结合 Spring 自动装载所有 ChainStep：

```java
@Component
public class ChainExecutor {

    // 存放所有需要执行的步骤
    private final List<ChainStep> steps;

    @Autowired
    public ChainExecutor(List<ChainStep> stepList) {
        // 可按注解排序、配置化排序
        this.steps = stepList;
    }

    // 执行
    public ChainContext run(ChainContext ctx) {
        for (ChainStep step : steps) {
            step.execute(ctx);
        }
        return ctx;
    }
}
```


## ⑤ 调用链执行

```java
@RestController
@RequiredArgsConstructor
public class ChatController {

    private final ChainExecutor chainExecutor;

    @PostMapping("/chat")
    public String chat(@RequestBody String input) {
        ChainContext ctx = new ChainContext();
        ctx.setUserInput(input);

        chainExecutor.run(ctx);
        return ctx.getFinalOutput();
    }
}
```

## 测试输出结果

请求：

```
hello
```

控制台：

```
安全检查通过
Prompt 优化完成
模型调用完成
输出润色完成
```

返回给前端：

```
请专业、克制地回答：hello
```

##  优点（非常适用于大模型开发）

| 传统做法       | Chain 做法      |
| ---------- | ------------- |
| 所有逻辑写在一个方法 | 每一步一个 Step    |
| 无法扩展       | 新增 Step 即插拔   |
| 难调试        | 日志基于 stepName |
| 逻辑顺序静态     | steps 可动态配置   |
| 不支持灰度      | Step 可开关、版本化  |

## 高阶玩法（真实上生产的）

| 增强    | 用法                             |
| ----- | ------------------------------ |
| 动态规则链 | steps 从 DB / Redis / YAML 读取排序 |
| 灰度开关  | Step 加注解决定开启关闭                 |
| 多链共存  | ChainExecutor 根据 Chain ID 选择链  |
| 回滚    | Step 支持 reverse() 实现补偿         |
| 并行链   | CompletableFuture 执行部分 step    |

如果是要支持回滚，在step接口中增加一个reverse方法即可：

```java
public interface ChainStep {
    void execute(ChainContext ctx);
    void reverse(ChainContext ctx); // 回滚方法
    String stepName();
}
```
回滚方法中，可以进行一些清理工作，比如删除临时文件、数据库记录等。