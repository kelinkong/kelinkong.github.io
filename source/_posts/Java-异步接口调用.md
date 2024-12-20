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

**推荐**：使用 **`CompletableFuture`**，它功能强大、非阻塞，并且易于维护和扩展。

