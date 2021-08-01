# Electron开发入门

## 一、Electron是什么

`Electron`是`GitHub`开发的一个开源框架。它允许使用`Node.js`（作为后端）和`Chromium`（作为前端）完成桌面`GUI`应用程序的开发。`Electron`现已被多个开源`Web`应用程序用于前端与后端的开发，著名项目包括`GitHub`的`Atom`和微软的`Visual Studio Code`。

与传统跨平台技术相比（如`QT`...），有以下特点：

- 优点
  - 跨平台：可以打包为`Mac`、`Windows`、`Linux`系统下的应用
  - 上手简单，开发成本较低：主要采用`HTML、CSS、JavaScript、Node.js`进行开发
- 缺点
  - 应用打包体积较大
  - 性能较差，打开窗体存在白屏

### 1.1 Electron应用架构

![electron应用架构](../asserts/electron架构.webp)

核心组成是由 `Chromium` + `Node.js` + `Native APIs` 组成的。其中 `Chromium` 提供了 `UI` 的能力，`Node.js` 让 `Electron` 有了底层操作的能力，`Navtive APIs` 则解决了跨平台的一些问题。

![electron进程模型](../asserts/electron-architecture.png)

`Electron` 继承了来自 `Chromium`的多进程架构，这使得此框架在架构上非常相似于一个现代的网页浏览器。

每个 `Electron` 应用都有一个单一的主进程，作为应用程序的入口点。通过 `Electron` 的 `app` 模块来控制应用程序的生命周期。每打开一个`BrowserWindow`(窗口)，就会生成一个单独的渲染进程。主进程与渲染进程之间可以通过预加载脚本实现数据共享，可以暴露出主进程`API`供渲染进程使用。

### 1.2 常见的Electron应用

- Atom
- VScode

## 二、快速实现一个Electron应用

- 初始化项目

```JavaScript
npm install electron -D
```

- 设置启动脚本

```JavaScript
// package.json
{
  "script”: {
    "start": "electron ."
  }
}
```

- 创建HTML模板文件

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self'">
    <meta http-equiv="X-Content-Security-Policy" content="default-src 'self'; script-src 'self'">
    <title>Hello World!</title>
  </head>
  <body>
    <h1>Hello World!</h1>
    We are using Node.js <span id="node-version"></span>,
    Chromium <span id="chrome-version"></span>,
    and Electron <span id="electron-version"></span>.
  </body>
</html>

```

- 设置启动脚本

```JavaScript
const { app, BrowserWindow } = require('electron')

function createWindow () {
  const win = new BrowserWindow({
    width: 800,
    height: 600
  })

  win.loadFile('index.html')
}
// 只有在 app 模块的 ready 事件被激发后才能创建浏览器窗口
app.whenReady().then(() => {
  createWindow()
})

```

- 启动应用

```JavaScript
npm run start
```

## 三、窗口与展现

业务开发中，使用最多的就是`BrowserWindow`。

### 3.1 创建窗口

```JavaScript
const win = new BrowserWindow({
  width: 800, // 窗口宽度
  height: 600, // 窗口高度
  resizable: true, // 窗口大小是否可调整
  fullscreen: false, // 是否全屏
  title: '测试', // 窗口名称
  icon: 'logo.ico', // 窗口图标
  frame: false, // 是否是无边框窗口
  parent: null, // 父窗口
  backgroundColor: '#fff', // 窗口背景颜色
  modal: false, // 是否是模态窗口
  webPreferences: {
    devTools: false, // 是否开启DevTools
    nodeIntegration: true, // 允许渲染进程使用node，electron
    enableRemoteModule: true, // 可以启用webRTC
    webSecurity: false, // 禁用浏览器安全协议，否则打不开本地文件
    preload: path.resolve(__dirname, './preload.js') // 预加载脚本
  }
})


```

### 3.2 加载窗口资源文件

- loadFile
- loadURL

```JavaScript
win.loadFile('index.html')
win.loadURL('http://localhost:8080')
```

### 3.3 窗口事件

- show
- hide
- blur
- ready-to-show
- resize
- move
- restore
- minimize
- maximize
// ...

```JavaScript
win.on('ready-to-show', e => {
  console.log('ready-to-show')
})
// 阻止窗口关闭事件
window.onbeforeunload = (e) => {
  console.log('I do not want to be closed')
  //返回非空值将默认取消关闭
  e.returnValue = false
}

```

### 3.4 窗口实例方法

- close
- focus
- show
- hide
- minimize
- maximize
- setFullScreen
// ...

## 四、应用扩展功能

### 4.1 app

控制应用程序的生命周期事件。

#### 4.1.1 事件

- `window-all-closed`(所有的窗口都被关闭时触发)
- `before-quit`(在程序关闭窗口前触发,`event.preventDefault()`阻止窗口关闭)
- `activate`(应用程序被激活)
- `web-contents-created`(在创建新的 `webContents` 时发出)
- `render-process-gone`(渲染器进程意外消失时触发。 这种情况通常因为进程崩溃或被杀死。)
- `second-instance`(当第二个实例被执行并且调用 `app.requestSingleInstanceLock()` 时，这个事件将在你的应用程序的首个实例中触发)
// ...

#### 4.1.2 方法

- `whenReady`(`electron` 初始化完成)
- `quit`(关闭所有窗口)
- `exit`(立即退出)
- `relaunch`(重启应用)
- `disableHardwareAcceleration`(禁用当前应用程序的硬件加速。)
// ...

### 4.2 对话框

#### 4.2.1 基本使用

```JavaScript
const { dialog } = require('electron')
console.log(dialog.showOpenDialog({ properties: ['openFile', 'multiSelections'] }))
```

#### 4.2.2 模态框种类

- `showOpenDialog`
- `showSaveDialog`
- `showMessageBox`
- `showErrorBox`

### 4.3 菜单

#### 4.3.1 方法

- 静态方法

  - `setApplicationMenu(menu)`(设置窗口顶部菜单)
  - `buildFromTemplate`(设置应用内菜单)

- 实例方法

  - `popup`(显示菜单)
  - `closePopup`(关闭菜单)
  - `destroy`(销毁菜单)

#### 4.3.2 基本使用

```JavaScript
const remote = require('electron').remote;
const Menu = remote.Menu;
const MenuItem = remote.MenuItem;

var menu = new Menu();
menu.append(new MenuItem({ label: 'MenuItem1', click: function() { console.log('item 1 clicked'); } }));
menu.append(new MenuItem({ type: 'separator' }));
menu.append(new MenuItem({ label: 'MenuItem2', type: 'checkbox', checked: true }));

window.addEventListener('contextmenu', function (e) {
  e.preventDefault();
  menu.popup(remote.getCurrentWindow());
}, false);
```

### 4.4 快捷键

#### 4.4.1 静态方法

- `register`(注册全局快捷键)
- `registerAll`
- `isRegistered`
- `unregister`
- `unregisterAll`

#### 4.4.2 基本使用

```JavaScript
const { app, globalShortcut } = require('electron')

app.whenReady().then(() => {
  // Register a 'CommandOrControl+X' shortcut listener.
  const ret = globalShortcut.register('CommandOrControl+X', () => {
    console.log('CommandOrControl+X is pressed')
  })
  if (!ret) {
    console.log('registration failed')
  }
  // 检查快捷键是否注册成功
  console.log(globalShortcut.isRegistered('CommandOrControl+X'))
})

app.on('will-quit', () => {
  // 注销快捷键
  globalShortcut.unregister('CommandOrControl+X')
  // 注销所有快捷键
  globalShortcut.unregisterAll()
})

```

## 五、进程间通信

## 六、打包（electron-builder）

## 七、踩过的坑

- 内存泄漏

  - 发布-订阅消息及时销毁
  - 第三方引用及时销毁

- 性能优化

  - `Node Addon`扩展
  - 文件等`IO`使用异步方式
  - `Fork`子进程进行计算
