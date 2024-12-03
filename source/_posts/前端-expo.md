---
title: 前端-expo
date: 2024-11-22 11:38:47
cagegories: [Frontend]
---

## expo是什么？

Expo 是一个用于构建原生应用的工具集合，它提供了一组工具和框架，用于开发跨平台应用。Expo 提供了简化原生应用的开发过程的工具，如自动构建和发布，以及集成各种第三方库。

### **Expo 的介绍**
Expo 是一个用于构建 React Native 应用的工具链，提供了一种快速搭建、开发、测试和发布移动应用的方法。它简化了 React Native 开发流程，使开发者可以更专注于业务逻辑，而无需担心底层的原生配置。

#### **特点**
1. **易用性**：快速启动，无需配置复杂的环境。
2. **跨平台支持**：一次开发，适配 iOS 和 Android。
3. **丰富的 API**：内置支持摄像头、位置、通知等原生功能。
4. **强大的工具链**：
   - **Expo Go**：一个移动应用，用于实时查看开发中的项目。
   - **Expo CLI**：命令行工具，管理项目的开发、打包和部署。
5. **托管服务**：提供打包、发布和更新服务。

---

### **Expo 的安装**
Expo 的安装分为工具安装和项目初始化。

#### **前置要求**
1. **Node.js**：安装最新的 LTS 版本（推荐使用 [nvm](https://github.com/nvm-sh/nvm) 管理 Node.js）。
   ```bash
   node -v    # 检查 Node.js 版本
   ```
2. **npm 或 yarn**：Node.js 安装后会自带 `npm`，也可以选择安装 `yarn`。
   ```bash
   npm -v     # 检查 npm 版本
   yarn -v    # 如果使用 yarn，检查其版本
   ```

现在推荐通过 `npx expo` 直接执行命令，而不是全局安装 `expo-cli`。这是为了减少全局依赖，确保使用的是最新版本的 Expo 命令行工具。

---

### **使用 npx expo 的流程**

#### **1. 初始化项目**
你可以直接通过 `npx expo` 命令初始化一个新的 Expo 项目：

```bash
npx expo init my-new-project
```

- **选择模板**：执行命令后，你可以选择以下模板之一：
  - **blank**：空白项目。
  - **blank (TypeScript)**：支持 TypeScript 的空白项目。
  - **tabs (TypeScript)**：带底部导航栏的 TypeScript 项目。
  
- **进入项目目录**：
  ```bash
  cd my-new-project
  ```

---

#### **2. 启动开发服务器**
在项目目录中运行以下命令：
```bash
npx expo start
```

- 打开浏览器后，你会看到一个开发面板：
  - **二维码**：可以用手机上的 **Expo Go** 应用扫描来查看项目。
  - **选择平台**：可以选择在 iOS 模拟器、Android 模拟器或 Web 浏览器中运行。

---

#### **3. 安装依赖**
Expo 提供了一个专用的 `expo install` 命令，用来安装与当前 Expo SDK 版本兼容的依赖。

示例：
```bash
npx expo install expo-camera expo-location
```

---

#### **4. 构建和发布应用**
- **构建应用**：
  - 使用 `EAS Build`（推荐）来构建应用：
    ```bash
    npx expo install eas-cli
    npx eas build
    ```
  - Expo 已不再推荐直接使用 `expo build`，因为 EAS 提供了更现代的构建方式，支持自定义原生代码。

- **发布应用**：
  ```bash
  npx expo publish
  ```

---

### **为什么推荐 npx？**
1. **确保使用最新版本**：通过 `npx expo`，每次执行时都会自动下载最新的命令行工具。
2. **减少全局安装依赖**：无需全局安装 `expo-cli`，避免版本冲突。
3. **简单易用**：直接通过项目本地的工具执行命令，无需额外配置。

---

### **常用 npx expo 命令**
| 命令                     | 作用                                      |
|--------------------------|-------------------------------------------|
| `npx expo init`          | 初始化一个新的 Expo 项目                  |
| `npx expo start`         | 启动开发服务器，支持 Web 和移动设备测试     |
| `npx expo install`       | 安装兼容 Expo 的依赖包                    |
| `npx expo upgrade`       | 升级 Expo 项目到最新版本                  |
| `npx eas build`          | 构建 iOS 和 Android 的生产版本             |
| `npx expo export:web`    | 导出为静态 Web 应用                       |

---

### **总结**
- 使用 `npx expo` 是当前最推荐的方式。
- 无需全局安装 `expo-cli`，保持工具链轻量化。
- 支持快速开发、调试和构建跨平台应用。

如果需要详细指导，比如环境配置或特定功能开发，可以继续告诉我！