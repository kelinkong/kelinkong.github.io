---
title: input-translation开发日志（下）
date: 2024-03-07 15:03:28
categories: [Frontend]
---

## 需求整理
**popup.html：**
- 是否开启划词翻译
- 选择划词翻译的默认目标语言
- 是否开启输入翻译
- 输入框翻译的简介
- 项目地址

**popup.js**
- 从popup界面接收信息，如果用户更新设置，就更新存储在浏览器中的值，同时发送消息给content脚本，让其监听
- 监听用户是否点击项目地址
- 每次点开popup界面都重新从浏览器中加载存储的设置

**content.js**
- 从浏览器中获取设置
- 监听用户的输入
- 监听用户是否复制文本
- 将信息发送给background脚本
- 显示翻译框
  
**background.js**
- 接收content脚本发来的信息
- 发送请求给API，同时接收应答信息
- 将翻译后的内容发送给content脚本

翻译展示框：
- 上方：左边显示logo，右边显示关闭按钮
- 中间显示译文
- 当用户点击别的地方时关闭翻译框，同时取消选中
## 具体实现

### popup

在上一篇有写过popup脚本支持跨域，所以我最初的想法是，在划词翻译这个功能中，content脚本只负责发送、接收信息，并且将信息显示出来，而向百度翻译发送请求由popup脚本完成。

但是**popup 脚本（popup.js）只在用户点击扩展图标并打开 popup 页面时才会运行。**

所以最终改为popup脚本只负责接收popup界面的信息，并传递给content脚本，与外界的交互还是
由background脚本完成，这样子增加了代码复用性，也更符合逻辑。

**需要保存用户的设置，所以当用户点击插件时都需要从storage中获取保存的设置**

```js
window.onload = function () {
	chrome.storage.sync.get(['autoTranslate', 'targetLanguage', 'inputTranslate'], function (result) {
	document.getElementById('auto-translate').checked = result.autoTranslate;
	document.getElementById('target-language').value = result.targetLanguage;
	document.getElementById('input-translate').checked = result.inputTranslate;
	});
}
```

**当用户更新设置时，也需要将设置保存下来，同时发给content脚本。**

这里为什么要发送给content脚本呢？而不是要content脚本自己去获取？

这里我更想用通知的方法，这样假设用户开启或关闭某一个功能，content脚本可以第一时间得到反馈。

```js
function saveSettings() {
	let autoTranslate = document.getElementById('auto-translate').checked;
	let targetLanguage = document.getElementById('target-language').value;
	let inputTranslate = document.getElementById('input-translate').checked;
	chrome.storage.sync.set({ autoTranslate: autoTranslate, targetLanguage: targetLanguage, inputTranslate: inputTranslate }, function () {
	sendMessageToContent(autoTranslate, targetLanguage, inputTranslate);
	});
}
```

由于扩展不支持直接从popup界面跳转链接，所以要在popup脚本去跳转。

```js
document.getElementById('project-link').addEventListener('click', function () {
	chrome.tabs.create({ url: 'https://github.com/kelinkong/Input-Translation.git' });
});
```

### content

这里真的是折磨我好几个小时，主要是调试弹出的翻译框。

和popup一样，在每次加载界面时需要从storage中获取用户之前的设置。

然后就是监听用户是否进行选中文本。这里除了记录选中的文本，还需要记录选中的位置，因为我想要直接在选中文本下方弹出翻译框。

```js
// 获取文本
var selectedText = window.getSelection().toString();
// 获取位置
var rect = window.getSelection().getRangeAt(0).getBoundingClientRect();
```

和background交互方式与上一篇相同，此处不再赘述。

接收到饭回来的译文后，这里可以直接调用`panel = document.createElement('div');`来创建一个视图。

```js
panel.innerHTML = innerHTMLContent; // 可以使用内嵌html
panel.style.position = 'fixed';  // 可以直接调整样式
panel.style.top = rect.bottom + 10 + 'px';
panel.style.left = rect.left + 'px';
```

其实这里的逻辑，应该是有一个单独的HTML文件来控制翻译框。但是我对前端太不了解，折腾了好久也没有折腾成功，只能将就着写进content脚本中。

#### 开启和关闭功能的实现

对于输入翻译来说，`if (event.target.tagName.toLowerCase() === 'input' && inputTranslate)`，检测到输入且inputTranslate为真才会去分析用户的输入是否含有目标语言前缀。

对于划词翻译，和输入翻译不同的是，这里还涉及到弹出框的开启与关闭。
- 当检测到用户更新设置时，先移除所有的鼠标监听，如果划词翻译被开启，则开始监听鼠标。
- 当用户点了弹出框的关闭，或者点击其他地方，关闭翻译框，同时取消选中。

这里取消选中其实有个坑。一开始我没有设置关闭翻译框后取消选中，结果就是，不断弹出翻译框。

当我设置了取消选中后，我一开始担心，也许用户选中并不是想要翻译，而是想要复制。所以我又给用户在翻译框下添加了两个选项：复制原文和复制译文。作为一个前端小白，调试界面的过程真的太难了，这些元素我怎么摆放都不好看。好不容易调试好了，我突然发现其实我不需要给用户提供这个功能，因为弹出翻译框并不影响用户的复制。

所以最终我没有给复制的选项。

### 调试

popup脚本--->右键点击插件，选择检查，就可以打开调试控制台
content脚本--->打开浏览器开发者面板，就可以看到
background脚本--->在插件管理中，点击服务工作进程，可以打开后台的调试控制台

### 理解前端

前端就类似做IDesign设计，由各种框组成，大框套小框，前后层叠关系类似图层的概念。每一对尖括号就是一个框。

那么**先把所有的框显示出来，就可以看到各个页面的包含关系**。

同样，在前端调试界面，可以看到每一个框的代码。如果呈现出来的画面不是自己想要的那样，可以去界面上调试，来查看是哪一块的代码出现了问题。

#### 如何去设计这些框？

```html
<div> 新框</div>

<div class="">属于哪一类</div>  class可以被css和js用来访问和操作元素

<label>可以有不同的标签</label>

<input type="checkbox" id="auto-translate" checked>可以交互
```

`.css`文件可以定义框的属性，框内元素的呈现。比如：
- padding：内部元素与框的距离。padding-left
- margin：内部框与外部框的距离。margin-left
#### 如何与用户交互

当用户改变前端界面时，发生了什么？

比如当用户点击了一个复选框，那么这条信息传到哪里？对后台有什么影响？

这些交互都可以在js脚本中定义。写js脚本和写其他的编程语言更像，都是写一个又一个的函数去实现各种各样的功能。

## 尾声

整个插件开发的有效工作时间大概是4天，这个过程AI帮我写了大部份代码，我只是在努力debug。之前的开发debug就是打断点，看变量信息，而在浏览器上调试就是用笨办法，在每一个函数入口都输出日志，然后去看是哪里出现了问题。

整个过程耗费时间的点：
- 各个脚本之间的通信
- 浏览器API
- 界面调试

在界面调试时，遇到了几个我实在是解决不了的bug，请外援帮忙解决的。易用性和UI上同门们也都给过建议。

如果觉得有用，可以在GitHub上帮忙点个star。感谢。https://github.com/kelinkong/Input-Translation.git
