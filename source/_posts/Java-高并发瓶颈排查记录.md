---
title: Java-高并发瓶颈排查记录
date: 2026-04-02 16:41:52
categories: [Java]
---

## 压测的服务器性能

腾讯云服务器：
- CPU：4核
- 内存：4G
- 带宽：3Mbps

## k6

k6 是一个现代化的负载测试工具，专为测试 Web 应用程序和 API 设计。它使用 JavaScript 编写测试脚本，提供了丰富的功能来模拟用户行为、生成负载并分析性能数据。

一个简单的示例:

```javascript
import http from 'k6/http';
import { sleep } from 'k6';

export let options = {
    stages: [ // 分别代表：第一阶段、第二阶段、第三阶段
        { duration: '30s', target: 10 }, // 逐渐加压
        { duration: '1m', target: 20 },  // 保持一个稳定的量，不要太猛
        { duration: '30s', target: 0 }, 
    ],
};

export default function () {
  http.get('https://test.k6.io');
  sleep(1);
}
```
在这个示例中，我们定义了一个负载测试，分为三个阶段：逐渐增加到 10 个虚拟用户，保持 20 个用户的负载，然后逐渐减少到 0。每个虚拟用户会每秒发送一个 HTTP GET 请求到指定的 URL，并在请求之间休眠 1 秒。

## arthas
Arthas 是一个强大的 Java 诊断工具，提供了丰富的功能来帮助开发者排查 Java 应用程序中的性能问题和瓶颈。它支持在线调试、监控和分析 Java 应用程序的运行状态。
Arthas 的主要功能包括：
1.	在线调试：可以在不停止应用程序的情况下进行调试，支持设置断点、查看变量值和调用栈等功能。
2.	性能分析：可以监控应用程序的 CPU 使用率、内存使用率和线程状态，帮助开发者识别性能瓶颈。
3.	类加载分析：可以查看类的加载情况，帮助开发者了解类的使用和加载过程。
4.	方法调用分析：可以分析方法的调用情况，帮助开发者了解方法的执行时间和调用关系。
5.	内存分析：可以分析对象的内存使用情况，帮助开发者识别内存泄漏和优化内存使用。
Arthas 提供了一个命令行界面，开发者可以通过命令行输入各种命令来进行诊断和分析。它还支持远程连接，可以在生产环境中使用，帮助开发者快速定位和解决性能问题。

```bash
arthas
docker exec -it backend java -jar /app/arthas-boot.jar
```
在命令行中输入 `arthas` 命令后，您可以连接到正在运行的 Java 应用程序，并使用各种命令来进行诊断和分析。例如，您可以使用 `dashboard` 命令来查看应用程序的性能指标，使用 `threads` 命令来查看线程状态，使用 `heap` 命令来分析内存使用情况，等等。

## 测试过程

1. 测试登录接口：发现错误：
```text
ERRO[0085] GoError: the body is null so we can't transform it to JSON - this likely was because of a request error getting the response
```
2. 查看nginx日志：
```text
2026/04/02 08:37:49 [warn] 30#30: *3001 an upstream response is buffered to a temporary file /var/cache/nginx/proxy_temp/2/05/0000000052 while reading upstream, client: xxxx, server: xxx, request: "GET /api/auth/me HTTP/1.1", upstream: "http://xxx/api/auth/me", host: "xxxx"
```
👉 后端返回的数据太大

👉 Nginx 内存 buffer 放不下

👉 Nginx 开始写临时文件（磁盘）

3. 经过排查，是登录的时候，会返回用户头像的base64编码字符串，导致数据过大。
4. 解决方案：数据库不在存储用户头像的base64字符串，而是存储一个URL地址，前端根据URL地址去获取用户头像。

## 寻找瓶颈

### 如何寻找瓶颈

```
1. 看 Body： 只要超时报错里带了部分内容，就是传输瓶颈。
2. 看 Nginx Warn： 只要出现 buffered to a temporary file，就是响应体过大。
3. 看 Arthas thread： 如果线程都在 WAITING 而不占 CPU，说明后端很闲，压力全在 网络 I/O 或 磁盘 I/O 上。
```

看service层的耗时：
```bash
watch com.goalflow.api.service.GoalService getGoalsByUser '#cost, #params' -x 10
```

接着看controller层的耗时：
```bash
watch com.goalflow.api.controller.GoalController getGoalsByUser '#cost, #params' -x 10
```

查看数据库查询的耗时：
```bash
watch com.goalflow.api.mapper.GoalMapper selectList '#params, #returnObj' -x 10
watch com.goalflow.api.mapper.GoalPlanItemMapper selectList '#params, #returnObj' -x 10
```

### 瓶颈
- k6 失败类型不是 500，而是 request timeout
- 超时时，k6 已经收到了大量 body 内容，说明请求不是卡在连接建立，也不是卡在后端执行业务，而是“响应太大，20 秒内没传完”
- 服务器后端日志同时显示：
   - GET /api/goals - Status: 200 - Duration: 22ms ~ 31ms
- 服务器资源同时显示：
   - backend CPU 3.79%
   - mysql CPU 0.25%

这三个事实放在一起，结论只有一个：

/api/goals 现在把完整 taskPlan 一起返回了，重度用户 50 个 goals，每个 30 天、每天 5 个任务，响应 JSON 过大，公网传输和序列化后的整体返回时间才是真正瓶颈。

```text
   1 INFO[0108] Request failed! status=0, error=request timeout, ... body=[{"id":"23","name":"备考英语四级", ...
     "taskPlan":[["浏览一套真题目录...
   * 分析： status=0 且 error=request timeout 说明请求没完成。
   * 破绽： 但在 body= 后面，竟然打印出了一大段 JSON 内容，里面全是详细的 taskPlan。
   * 推论： 如果是卡在数据库查询，body 应该是空的；既然 body
     已经有内容了，说明后端已经查完数据库并开始往外吐数据了，只是因为数据量太大，k6 在 20
     秒内还没接完剩下的数据包，由于超时时间到了，k6 强行断开了连接。
```

### 解决办法

  1. GET /api/goals 改成轻量列表 DTO，不返回 taskPlan
  2. 详情页需要任务计划时，再走 GET /api/goals/{id} 或 GET /api/goals/{id}/timeline

本质上是：列表查询只做列表查询，详情查询才做详情查询。不要在列表查询里把所有数据都返回了。

### 继续寻找瓶颈

在500个并发的时候，查询又出现了传输瓶颈。分析原因：这里前后端没做分页查询，导致一次性返回了过多数据，传输时间过长。服务器的带宽为3Mbps，500个并发时，数据量过大，导致传输时间超过了k6的默认超时时间。

### 数据库并发

k6测试时出现：
```text
INFO[0112] habit checkin failed: status=500, error=none, error_code=1500, body={"status":500,"error":"Internal Server Error","message":"服务器开小差了: \n### Error updating database.  Cause: java.sql.SQLIntegrityConstraintViolationException: Duplicate entry '3-2026-04-02' for key 'habit_checkins.uniq_habit_date'\n### The error may exist in com/goalflow/api/mapper/HabitCheckinMapper.java (best guess)\n### The error may involve com.goalflow.api.mapper.HabitCheckinMapper.insert-Inline\n### The error occurred while setting parameters\n### SQL: INSERT INTO habit_checkins  ( habit_id, user_id, date, is_done, created_at, updated_at )  VALUES (  ?, ?, ?, ?, ?, ?  )\n### Cause: java.sql.SQLIntegrityConstraintViolationException: Duplicate entry '3-2026-04-02' for key 'habit_checkins.uniq_habit_date'\n; Duplicate entry '3-2026-04-02' for key 'habit_checkins.uniq_habit_date'"}  source=console
```
分析：这个错误是因为在高并发情况下，多个请求同时尝试插入同一条记录，导致违反了数据库的唯一约束。

```Java
if (!isDone) {
    if (existing != null) {
        habitCheckinMapper.deleteById(existing.getId());
    }
} else if (existing == null) {
    habitCheckinMapper.insert(HabitCheckin.builder()
            .habitId(habitId)
            .userId(user.getId())
            .date(date)
            .isDone(true)
            .createdAt(now)
            .updatedAt(now)
            .build());
} else {
    habitCheckinMapper.update(null, new LambdaUpdateWrapper<HabitCheckin>()
            .eq(HabitCheckin::getId, existing.getId())
            .set(HabitCheckin::getIsDone, true)
            .set(HabitCheckin::getUpdatedAt, now));
}
```

这个地方的逻辑是：如果用户要取消打卡（isDone=false），就删除记录；如果用户要打卡（isDone=true），但之前没有记录，就插入新记录；如果用户要打卡，但之前已经有记录了，就更新记录。

但是在高并发情况下，可能会出现以下情况：
1. 两个请求同时检查到 existing == null，认为没有记录了，都尝试插入新记录，导致违反唯一约束。
2. 两个请求同时检查到 existing != null，认为有记录了，都尝试更新记录，但其中一个请求可能已经删除了记录，导致另一个请求更新失败。

这个地方改为：数据库原子 upsert。这个 SQL 在数据库层做了“原子性判断 + 插入/更新”，一次 SQL 就能搞定，无需先 select 再 insert。

```sql
INSERT INTO habit_checkins (habit_id, user_id, date, is_done, created_at, updated_at)
VALUES (?, ?, ?, ?, ?, ?)
ON DUPLICATE KEY UPDATE  // 如果违反唯一约束了，就执行更新而不是插入
is_done = VALUES(is_done),  
updated_at = VALUES(updated_at)
```
这样就能保证在高并发情况下，不会出现违反唯一约束的错误了。

原子操作：数据库内部会在同一条 SQL 上做锁处理（InnoDB 行锁），不会让两个请求同时插入同一行，避免了之前的竞态条件。

**这个地方MP 自带 saveOrUpdate 依旧是先 select 再 insert/update，所以并不能解决这个问题，必须写原生 SQL。**

### 调用AI没有使用异步处理

后端调用AI接口时，没有使用异步处理，导致在高并发情况下，线程被AI接口调用阻塞，无法及时响应其他请求。

为什么会被阻塞：因为Spring Boot中的默认线程池是有限的，当所有线程都被AI接口调用占用时，新的请求就无法获得线程来处理，导致请求积压和超时。