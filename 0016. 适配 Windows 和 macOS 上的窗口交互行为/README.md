# 0016. 适配 Windows 和 macOS 上的窗口交互行为

## 📝 概述

- 理解 Windows 和 macOS 的桌面应用，在窗口交互行为上的一些差异。
  - 处理逻辑很简单，重点在于理解两种系统中窗口交互的一些差异点。

## 🔗 links

- https://www.electronjs.org/zh/docs/latest/api/app#%E4%BA%8B%E4%BB%B6-activate-macos
  - 官方文档，查看主进程的 app 模块上的 activate 事件 app.on('activate', func) 的相关描述。
- https://www.electronjs.org/zh/docs/latest/api/app#%E4%BA%8B%E4%BB%B6-window-all-closed
  - 官方文档，查看主进程的 app 模块上的 window-all-closed 事件的相关描述。

## 📒 notes

- 我们知道在桌面端，macOS 和 Windows 有着特别多的操作差异性，比如在窗口管理上，Windows 用户习惯使用最大化、最小化和关闭窗口的按钮，而 macOS 用户则通常使用红、黄、绿色的按钮分别表示关闭、最小化和全屏。
- 除此之外，在 macOS 中，即使所有窗口关闭了，应用仍然在底部的菜单栏中保持活动状态。因此，通常需要特殊处理以确保用户主动退出应用程序。而在其他平台（如 Windows 或 Linux）中，通常情况下关闭最后一个窗口也意味着退出应用程序是合理的行为。所以为了实现这个操作习惯，我们也可以增加一个情况判断。核心逻辑如下：

```js
// 当窗口都被关闭了
app.on('window-all-closed', () => {
  // 如果不是macOS
  if (process.platform !== 'darwin') {
    // 应用退出
    app.quit();
  }
});

// macOS 特定行为：当没有窗口打开时点击应用图标打开新窗口
app.on('activate', () => {
  if (BrowserWindow.getAllWindows().length === 0) {
    createWindow();
  }
});
```

## 💻 demo

```js
// index.js
const {app, BrowserWindow} = require('electron');

let win;

function createWindow() {
  win = new BrowserWindow();

  win.loadFile('./index.html');

  // 当窗口关闭时清除 win 对象
  win.on('closed', () => win = null);
}

app.on('window-all-closed', () => {
  // macOS 的常规行为是应用及其菜单栏继续激活，直到用户使用 Cmd + Q 明确退出
  if (process.platform !== 'darwin') {
    app.quit();
  }
});

app.whenReady().then(createWindow);

// macOS 特定行为：当没有窗口打开时点击应用图标打开新窗口
app.on('activate', () => {
  if (BrowserWindow.getAllWindows().length === 0) {
    createWindow();
  }
});
```