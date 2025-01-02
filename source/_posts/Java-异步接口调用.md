---
title: Java-异步接口调用
date: 2024-12-17 19:16:30
categories: [Java]
---
当**异步方法**提供服务时，调用方通常需要一种机制来**知道异步方法什么时候返回结果**以及**是否执行成功**。在 **Spring Boot** 中，这可以通过返回 `Future`、`CompletableFuture` 或使用回调机制来实现。

## **1. 使用 `Future` 接口**
`@Async` 方法可以返回一个 `Future` 对象，调用方可以通过 `Future` 的方法查询执行状态和结果。

### **示例**
```java
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;

import java.util.concurrent.Future;
import java.util.concurrent.CompletableFuture;

@Service
public class MyService {

    @Async
    public Future<String> asyncMethodWithFuture() {
        try {
            Thread.sleep(3000); // 模拟耗时任务
            return CompletableFuture.completedFuture("异步方法执行成功");
        } catch (InterruptedException e) {
            return CompletableFuture.failedFuture(e);
        }
    }
}
```

### **调用方示例**
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.concurrent.Future;

@RestController
public class MyController {

    @Autowired
    private MyService myService;

    @GetMapping("/testAsync")
    public String testAsync() {
        try {
            Future<String> result = myService.asyncMethodWithFuture();

            while (!result.isDone()) { // 轮询检查是否完成
                Thread.sleep(500);
                System.out.println("异步方法未完成...");
            }

            return result.get(); // 获取结果，阻塞直到异步任务完成
        } catch (Exception e) {
            return "异步调用失败：" + e.getMessage();
        }
    }
}
```

### **结果分析**
1. `Future` 提供方法如：
   - `isDone()`：检查任务是否完成。
   - `get()`：获取结果（会阻塞当前线程，直到任务完成）。
2. 调用方可以轮询 `isDone()` 判断异步方法的状态。

**缺点**：轮询消耗资源，且 `get()` 方法是**阻塞**的，不够高效。

---

## **2. 使用 `CompletableFuture`**
`CompletableFuture` 提供更灵活的非阻塞异步处理，支持链式操作和回调。

### **示例**
```java
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;

import java.util.concurrent.CompletableFuture;

@Service
public class MyService {

    @Async
    public CompletableFuture<String> asyncMethodWithCompletableFuture() {
        try {
            Thread.sleep(3000); // 模拟耗时任务
            return CompletableFuture.completedFuture("异步方法执行成功");
        } catch (InterruptedException e) {
            return CompletableFuture.failedFuture(e);
        }
    }
}
```

### **调用方示例**
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.concurrent.CompletableFuture;

@RestController
public class MyController {

    @Autowired
    private MyService myService;

    @GetMapping("/testCompletableFuture")
    public CompletableFuture<String> testCompletableFuture() {
        return myService.asyncMethodWithCompletableFuture()
                .thenApply(result -> "异步任务返回结果：" + result)
                .exceptionally(ex -> "异步任务失败：" + ex.getMessage());
    }
}
```

### **结果分析**
1. **非阻塞**：调用方无需等待，可以直接返回 `CompletableFuture` 供调用方使用。
2. `thenApply`：任务成功时执行的回调操作。
3. `exceptionally`：任务失败时的回调处理。

**优点**：
- 支持链式调用，代码清晰。
- 无需主动轮询，任务完成时自动触发回调。

---

## **3. 使用回调机制**
如果你不想依赖 `Future` 或 `CompletableFuture`，可以手动实现回调逻辑，通过 **自定义回调接口** 来通知调用方。

### **示例**
```java
public interface Callback {
    void onSuccess(String result);
    void onFailure(Throwable t);
}

import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;

@Service
public class MyService {

    @Async
    public void asyncMethodWithCallback(Callback callback) {
        try {
            Thread.sleep(3000); // 模拟耗时任务
            callback.onSuccess("异步方法执行成功");
        } catch (Exception e) {
            callback.onFailure(e);
        }
    }
}
```

### **调用方示例**
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class MyController {

    @Autowired
    private MyService myService;

    @GetMapping("/testCallback")
    public String testCallback() {
        myService.asyncMethodWithCallback(new Callback() {
            @Override
            public void onSuccess(String result) {
                System.out.println("回调成功：" + result);
            }

            @Override
            public void onFailure(Throwable t) {
                System.out.println("回调失败：" + t.getMessage());
            }
        });

        return "异步方法已触发，等待回调结果...";
    }
}
```

### **结果分析**
- `Callback` 接口定义了成功和失败时的回调方法。
- 异步任务执行后，通过回调接口通知调用方结果。

---

## **总结**
| 方法                        | 特点                           | 是否阻塞   | 适用场景              |
|-----------------------------|--------------------------------|------------|-----------------------|
| `Future`                    | 简单轮询和阻塞获取结果         | 阻塞       | 小型任务，简单异步    |
| `CompletableFuture`         | 支持链式调用、非阻塞和回调     | 非阻塞     | 高性能异步处理        |
| 自定义回调接口              | 完全自定义回调逻辑             | 非阻塞     | 灵活控制回调和通知    |

**推荐**：使用 `CompletableFuture`，它功能强大、非阻塞，并且易于维护和扩展。

## 异步编程+多并发示例
为了确保外系统调用接口 A 时不会因接口 B 的处理耗时而超时，可以设计接口 A 为 **异步返回机制**，即外系统调用接口 A 时，接口 A 立即返回一个请求 ID，随后在后台完成调用接口 B 的多次处理任务，外系统可以通过请求 ID 查询最终结果。

假设我现在有一个接口A，调用了接口B，但是接口B每次需要耗时20s左右才能处理完成，且我需要多次调用接口B，但是我又不能大规模并发的调用接口B（有并发数限制），我的接口A和B都是一个微应用接口，现在我需要把微应用接口A暴露给外系统调用，我要如何高效实现？

以下是基于已有线程池的实现方案：

---

### **解决方案：立即响应，异步处理**

1. **外部接口 A 的处理逻辑：**
   - 接收外部请求。
   - 立即生成一个唯一请求 ID，并存储初始状态。
   - 使用线程池异步处理任务。
   - 立即返回请求 ID。

2. **后台任务逻辑：**
   - 在线程池中分解任务。
   - 按并发限制逐个调用接口 B。
   - 将结果存储到缓存或数据库中。

3. **结果查询接口：**
   - 外系统通过查询接口，使用请求 ID 检索处理结果。

---

### **实现步骤**

#### **1. 定义接口 A**
通过线程池异步处理任务，立即返回请求 ID：

```java
@RestController
@RequestMapping("/api")
public class InterfaceAController {

    private final ExecutorService executorService; // 现有线程池
    private final Semaphore semaphore = new Semaphore(3); // 并发限制
    private final Map<String, String> results = new ConcurrentHashMap<>(); // 模拟存储结果

    // 构造函数注入线程池
    public InterfaceAController(ExecutorService executorService) {
        this.executorService = executorService;
    }

    @PostMapping("/process")
    public ResponseEntity<String> processRequest(@RequestBody RequestData requestData) {
        String requestId = UUID.randomUUID().toString(); // 生成唯一请求 ID
        results.put(requestId, "Processing"); // 初始化状态

        // 异步处理任务
        executorService.submit(() -> handleRequest(requestId, requestData));

        // 立即返回请求 ID
        return ResponseEntity.ok(requestId);
    }

    private void handleRequest(String requestId, RequestData requestData) {
        try {
            List<String> tasks = requestData.getSubTasks(); // 分解请求
            List<String> taskResults = new ArrayList<>();

            for (String task : tasks) {
                semaphore.acquire(); // 控制并发数
                try {
                    String result = callInterfaceB(task); // 调用接口 B
                    taskResults.add(result);
                } finally {
                    semaphore.release(); // 释放许可
                }
            }

            // 合并结果并存储
            results.put(requestId, String.join(", ", taskResults));
        } catch (Exception e) {
            results.put(requestId, "Error: " + e.getMessage());
        }
    }

    private String callInterfaceB(String task) throws InterruptedException {
        // 模拟接口 B 的调用
        Thread.sleep(20000); // 假设接口 B 耗时 20 秒
        return "Processed " + task;
    }

    @GetMapping("/result/{id}")
    public ResponseEntity<String> getResult(@PathVariable String id) {
        String result = results.get(id);
        if (result == null) {
            return ResponseEntity.status(HttpStatus.NOT_FOUND).body("Result not found");
        } else if ("Processing".equals(result)) {
            return ResponseEntity.status(HttpStatus.PROCESSING).body("Still processing");
        }
        return ResponseEntity.ok(result);
    }
}
```

---

### **2. 外系统调用流程**
1. 调用 `/api/process`：
   - **请求：**发送业务数据。
   - **响应：**立即返回一个唯一的请求 ID。

2. 调用 `/api/result/{id}`：
   - **请求：**查询对应请求 ID 的处理结果。
   - **响应：**
     - 如果结果尚未处理完成，返回状态码 `HTTP 102 Processing`。
     - 如果处理完成，返回最终结果。

---

### **3. 优化与增强**

#### **结果存储**
- 如果任务结果较大，建议存储到 Redis 或数据库中，以减少内存压力。

#### **结果通知**
- 除轮询机制外，可以提供**回调机制**，通知外系统任务完成。
- 例如，外系统调用接口 A 时提供回调 URL，任务完成后主动通知：
  ```java
  private void notifyExternalSystem(String callbackUrl, String result) {
      RestTemplate restTemplate = new RestTemplate();
      restTemplate.postForEntity(callbackUrl, result, String.class);
  }
  ```

#### **超时监控**
- 为每个任务设置超时时间，确保长时间未完成的任务能正确标记为失败。

#### **负载调控**
- 如果任务量较大，可结合消息队列（如 Kafka）进一步解耦任务提交和处理。

---

### **优势**

1. **外系统不超时：**接口 A 立即响应，外系统只需查询结果或接收回调。
2. **高效调用接口 B：**通过线程池和信号量控制并发，避免超出接口 B 的限制。
3. **扩展性好：**可通过增加线程池大小或优化队列管理提升性能。

---

### **完整调用示例**
1. 外系统调用：
   ```bash
   curl -X POST -H "Content-Type: application/json" -d '{"subTasks": ["task1", "task2"]}' http://localhost:8080/api/process
   ```
   返回：
   ```json
   {
       "requestId": "abcd1234"
   }
   ```

2. 查询结果：
   ```bash
   curl http://localhost:8080/api/result/abcd1234
   ```
   - 如果正在处理中：
     ```json
     {
         "status": "Processing"
     }
     ```
   - 如果处理完成：
     ```json
     {
         "status": "Completed",
         "result": "Processed task1, Processed task2"
     }
     ```

## 示例二

Java并发编程：

```java
import java.util.List;
import java.util.concurrent.*;
import java.util.ArrayList;

public class YourService {

    private final ExecutorService executorService;

    // 创建固定大小为 5 的线程池
    public YourService() {
        this.executorService = Executors.newFixedThreadPool(5);  // 设置最大并发数为 5
    }

    public void processConcurrently(List<String> prompts, User user, List<QuestionGenerationContext> questionsContext, Vo vo) {
        List<CompletableFuture<Void>> futures = new ArrayList<>();

        for (int i = 0; i < prompts.size(); ++i) {
            final int index = i;
            // 使用 CompletableFuture 结合线程池来并发执行任务
            CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
                // 调用 send 方法
                LinkedHashMap<String, String> response = send(prompts.get(index), user);

                // 格式化结果
                List<QuestionGeneration> questions = QuestionGenerationUtil.formartText(response, questionsContext.get(index));

                // 线程安全地加入 vo
                synchronized (vo) {
                    vo.addQuestions(questions);
                }
            }, executorService);
            futures.add(future);
        }

        // 等待所有任务完成
        CompletableFuture.allOf(futures.toArray(new CompletableFuture[0])).join();
        // 关闭线程池
        executorService.shutdown();
    }

    public LinkedHashMap<String, String> send(String prompt, User user) {
        // 模拟 send 请求
        return new LinkedHashMap<>();
    }

    // 关闭线程池
    public void shutdown() {
        executorService.shutdown();
    }
}
```

一个长期运行的接口，如果调用关闭线程池，那么接下来就会拒绝服务。解决方法：
1. 全局线程池，避免每一个接口都创建一个线程池。
2. 在接口调用时去创建一个线程池，然后调用完接口后关闭（不推荐）。

## 示例三

```java
ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
executor.setCorePoolSize(5);  // 设置核心线程池大小
executor.setMaxPoolSize(10);  // 设置最大线程池大小
executor.setQueueCapacity(25);  // 设置任务队列的容量
executor.setThreadNamePrefix("custom-thread-");  // 设置线程名前缀
executor.initialize();  // 初始化线程池
```
`ThreadPoolTaskExecutor`（以及其他线程池实现，如 `ThreadPoolExecutor`）提供了以下三项核心配置：**核心线程池大小（core pool size）**、**最大线程池大小（maximum pool size）** 和 **任务队列容量（queue capacity）**。这三者共同决定了线程池的行为、任务的调度方式以及线程池的资源使用情况。

让我们逐一解释它们的作用：

### 1. **核心线程池大小（Core Pool Size）**：
   - **作用**：核心线程池大小指定了线程池中始终保持的最小线程数。这些核心线程会在没有任务可执行时持续存活，直到线程池被关闭。即使线程池中的线程没有正在执行的任务，它们也不会被销毁，直到线程池关闭或线程池中的任务完全完成。
   - **默认行为**：线程池会尽可能维持这些核心线程，不会因为任务队列为空而减少到小于 `corePoolSize` 的数量。
   - **影响**：如果你的应用有一定的并发需求并且需要快速响应任务，可以增加核心线程池的大小。否则，默认的大小可能已经足够。

   ```java
   executor.setCorePoolSize(5);  // 核心线程数为5
   ```

### 2. **最大线程池大小（Maximum Pool Size）**：
   - **作用**：最大线程池大小指定了线程池中最多可以同时容纳多少线程。当任务数超过核心线程池大小，并且队列已满时，线程池会创建更多的线程来处理任务，但总线程数不会超过 `maximumPoolSize`。
   - **默认行为**：如果没有设置最大线程池大小，线程池的最大线程数默认为 `Integer.MAX_VALUE`，这意味着它可以根据系统资源随时增加线程数，直到达到系统限制。
   - **影响**：通过设置 `maximumPoolSize`，你可以控制线程池的最大并发数，避免线程池消耗过多系统资源。

   ```java
   executor.setMaxPoolSize(10);  // 最大线程数为10
   ```

### 3. **任务队列容量（Queue Capacity）**：
   - **作用**：任务队列用于存储等待执行的任务。当线程池的核心线程池处于满负荷状态时，新的任务会被放入队列中等待执行。队列的容量决定了能存储多少个等待任务。
   - **默认行为**：如果没有设置队列容量，通常线程池使用的是一个 **无界队列**（`LinkedBlockingQueue`），这意味着队列容量几乎是无限的，所有等待的任务会被排入队列直到有线程空闲。如果队列有界（有限容量），则会导致任务被拒绝执行（例如，抛出 `RejectedExecutionException`）。
   - **影响**：通过设置 `queueCapacity`，你可以控制任务队列的大小。当队列满了且没有空闲线程时，新的任务会被拒绝，或者在有界队列时，会创建新线程直到达到 `maximumPoolSize`。

   ```java
   executor.setQueueCapacity(25);  // 任务队列容量为25
   ```

### 线程池行为总结：
- **当有任务到达时**，线程池首先会尝试使用核心线程池中的线程来处理任务。如果核心线程池中的线程都在忙，且队列未满，那么任务会被放入队列等待执行。
- **当队列已满**，如果线程池的线程数还没有达到最大线程数，线程池会创建新的线程来处理任务。此时，线程池的线程数会增大，但不会超过 `maximumPoolSize`。
- **当线程池达到最大线程数且队列已满时**，新的任务会被拒绝执行，除非你配置了拒绝策略来处理这些任务（如 `AbortPolicy`, `CallerRunsPolicy` 等）。

### 典型的线程池配置：
- **核心线程池大小**：适合处理常规并发任务的最小线程数。
- **最大线程池大小**：适合应对任务爆发期或高并发的最大线程数。
- **任务队列容量**：决定了线程池如何排队等待任务执行。根据应用的需求，队列可以选择有界或无界，具体取决于任务的性质。

### 示例配置的具体作用：
```java
executor.setCorePoolSize(5);  // 最小线程数为 5
executor.setMaxPoolSize(10);  // 最大线程数为 10
executor.setQueueCapacity(25);  // 队列容量为 25
```
- 线程池最开始会有 5 个线程（核心线程数）。如果有更多的任务到达，而没有线程可用，它会将任务放入队列中等待执行。
- 当队列中的任务满了（队列最大容量为 25），线程池会新增线程来处理任务，但线程池的最大线程数不能超过 10。
- 如果任务数超过了最大线程数，且队列也已满，新的任务将会被拒绝（除非你配置了拒绝策略）。

### 总结：
- **核心线程池大小**：控制最小线程数，任务会先被这些线程处理。
- **最大线程池大小**：控制最大线程数，超出这个数量的任务会等待或被拒绝。
- **任务队列容量**：决定任务等待队列的大小，当线程池的核心线程都在工作且队列满时，线程池才会创建新线程。