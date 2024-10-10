---
title: Java学习笔记-gradle
date: 2024-10-10 11:30:04
categories: [Java]
---

## 背景介绍

### 什么是Gradle

Gradle是一个强大的、基于JVM的构建自动化工具。它使用Groovy或Kotlin语言来编写构建脚本，相较于传统的XML配置方式，更加灵活和易于阅读。

### Gradle核心概念

- 项目（Project）：一个Gradle构建的最小单位。
- 任务（Task）：构建过程中的具体操作，如编译、测试、打包等。
- 构建文件（build.gradle）：定义项目配置和任务的脚本文件。
- 插件（Plugin）：扩展Gradle功能的模块，如Java插件、Android插件等。

Gradle 里的任何东西都是基于这两个基础概念: `projects` ( 项目 ) 和 `tasks` ( 任务 ) 。每一个构建都是由一个或多个 `projects` 构成的，每一个 `project` 是由一个或多个 `tasks` 构成的。一个 `project` 可以代表一个 JAR ，一个网页应用，一个发布的 ZIP 压缩包等。一个`tasks`就是一段可执行的代码，比如编译代码、运行测试、打包等。

gradle命令行基本格式：`gradle 任务名称` ，比如 `gradle clean`（清空所有编译、打包生成的文件） ，`gradle build -x test`（跳过测试构建构建）

`build.gradle`文件是 Gradle 构建脚本的核心，它可以用来定义项目的构建逻辑、依赖管理、插件管理等内容。 该文件通常位于项目根目录下。
执行Gradle命令的时候，会默认加载当前目录下的`build.gradle`脚本文件，你也可以通过 -b 参数指定想要加载执行的文件。

`Gradle插件`是一种可重用的构建逻辑，它可以提供各种功能来简化构建过程。Gradle中有丰富的插件，例如：application 插件可以打包可执行的Java应用程序,war 插件可以打包Web应用程序。除了官方插件之外，还有很多第三方插件。

`Gradle Wrapper`是Gradle的一个特性，它能够让我们在不安装Gradle的情况下运行Gradle构建。它是一个shell脚本和一个二进制文件，可以自动下载指定版本的Gradle，并使用该版本运行Gradle构建。


## 组成

### Project（项目）
定义: Gradle构建的最小单位，代表一个需要构建的软件系统。

组成:
- build.gradle文件: 定义项目配置和任务的脚本文件。
- 子项目: 一个项目可以包含多个子项目。

作用:
- 组织构建逻辑：将整个构建过程划分成不同的项目，方便管理。
- 定义依赖关系：不同项目之间可以存在依赖关系。
### Task（任务）
定义: 构建过程中的具体操作，是构建的原子单位。

作用:
- 编译源代码
- 运行测试
- 生成jar包
- 打包war包
- 自定义任务

特点:
- 有序执行：任务之间可以有依赖关系，确保执行顺序。
- 可配置：可以通过参数来配置任务的行为。
- 可复用：可以将任务定义为公共任务，在多个项目中共享。
###  Plugin（插件）
定义: 扩展Gradle功能的模块，提供特定领域的构建支持。

作用:
- 添加新的任务：比如Java插件添加了compileJava、test等任务。
- 提供新的配置项：比如Java插件提供了sourceSets配置。
- 定义新的约定：比如Java插件定义了源代码和资源的默认目录结构。

## 示例

```groovy
// build.gradle
plugins {
    id 'java' //  通过应用这个插件，Gradle知道这是一个Java项目，并会自动配置一些默认的任务和约定。
}

repositories { // 配置仓库。
    mavenCentral() // mavenCentral() 表示使用Maven中央仓库。
}

dependencies { // 声明依赖。
    implementation 'junit:junit:4.13.2'
}

// 自定义任务
task hello {
    doLast {
        println 'Hello, Gradle!'
    }
}
```

### 多项目

```
my-project
├── settings.gradle
├── core
│   ├── build.gradle
│   └── src
├── web
│   ├── build.gradle
│   └── src
└── build.gradle
```
类似cmake的多项目构建，每个项目都有自己的`build.gradle`文件，根目录下的`settings.gradle`文件用来定义项目的结构。
    
```groovy
// settings.gradle
include 'core', 'web'
```

### 多个脚本

当依赖项过多时，可以将依赖项提取到一个单独的文件中，然后在`build.gradle`中引入（`dependencies.gradle`）。

```groovy
// build.gradle
apply from: 'dependencies.gradle'
```
- 脚本包含: apply from语句告诉Gradle去包含另一个脚本文件，就好像把那个脚本文件的内容直接复制粘贴到当前脚本中一样。
- 执行顺序: apply from语句通常放在 plugins 块之后，这样被包含的脚本就可以访问插件提供的功能。

在build脚本中引用 `gradle.properties`
- 直接使用属性: Gradle会自动将 `gradle.properties` 中定义的属性加载到当前构建脚本的上下文中，可以直接使用 `$propertyName` 的方式引用。
- 使用 `ext` 对象: Gradle提供了一个 `ext` 对象，可以用来存储自定义的属性。可以在 `build.gradle` 中将 g`radle.properties` 中的属性赋值给 `ext` 对象，然后通过 `ext.propertyName` 的方式引用。

```groovy
// gradle.properties
version=1.0.0
myProperty=value

// build.gradle
println "Project version: $version"

ext.myCustomProperty = "custom value"
println "Custom property: ${ext.myCustomProperty}"
```
### buildscript

在Gradle构建脚本中，buildscript 块主要用于配置构建脚本本身的依赖和环境。它定义了构建脚本在执行过程中所需要的资源。

ext {} 和 repositories {} 是 buildscript 块中的两个重要的配置块：

- ext {}： 用于定义扩展属性。这些属性可以在整个构建脚本中被引用，提供了一种灵活的方式来存储和共享配置信息。
- repositories {}： 用于配置仓库地址。这些仓库是 Gradle 下载插件和依赖的来源。
