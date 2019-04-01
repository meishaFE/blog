---
title: Electron 自动更新
date: 2019-03-24 05:55:43
author: Troy
tags:
  - Electron
---

## 1 前言

开发客户端一定要做的就是自动更新模块，否则每次迭代将会出现很多问题。electron 的打包和更新都有多种方式，其中的坑也很多，下面将介绍使用 electron-builder 配合 electron-updater 实现自动更新的解决方案。在开发中也临时了解了下几种不同的方案，挑选了一个相对较方便的。

特别注意，mac 上的应用自动更新需要代码签名。

另外，这个工具更新下载完成后，只需要重启应用就能完成更新（体验参考 vscode）

<!-- more -->

## 2 mac 安装包代码签名

1. 安装 Xcode
2. 打开菜单栏 Xcode -> Preferences -> Accounts -> 登录开发者账号 -> Manage certificates
3. 新增一个 `Developer ID Application` 密钥
4. 打开 `钥匙串访问` -> 我的证书 可以看的刚才创建的密钥
5. 导出密钥，这时会得到一个文件
6. `sudo vim ~/.bash_profile` 打开配置环境变量的文件
7. 加上密钥文件的路径 `export CSC_LINK=密钥文件的路径`
8. 配置完成

## 3 打包配置

electron 打包的配置在`package.json`中

```javascript
{
  ...
  ...
  "build": {
    "productName": "appname",
    "appId": "com.name.name",
    "publish": [
      {
        "provider": "generic",
        "url": "https://www.test.com/release"
      }
    ],
    "directories": {
      "buildResources": "static",
      "output": "release"
    },
    "files": [
      "appBuild/**/*",
      "node_modules/**/*",
      "package.json",
      "./static/electron.js"
    ],
    "win": {
      "verifyUpdateCodeSignature": false,
      "icon": "./static/icon.ico",
      "artifactName": "mscode_setup.${ext}"
    },
    "mac": {
      "icon": "./static/icon.icns",
      "artifactName": "mscode_setup.${ext}"
    },
    "dmg": {
      "contents": [
        {
          "x": 140,
          "y": 200
        },
        {
          "x": 400,
          "y": 200,
          "type": "link",
          "path": "/Applications"
        }
      ]
    }
  }
  ...
  ...
}
```

注意点：

- 配置了 publish 才会生成 latest.yml 文件，用于自动更新的配置信息；latest.yml 文件是打包过程生成的文件，为避免自动更新出错，打包后禁止对 latest.yml 文件做任何修改
- 可以在 mac 上打包 windows 的安装包，打包过程中会下载一些依赖包，非常耗时
- windows 下不需要代码签名，但是因为配置了代码签名，打包 windows 的安装包时会将 mac 的代码签名打包进去，这时 windows 下自动更新会出问题。将`verifyUpdateCodeSignature`配置为`false`，windows 将不签名
- windows 需要准备不同规格的图标文件（特别注意：一个 ico 文件可以包含多个尺寸的图标）

## 4 主进程配置

直接上代码+注释，下面是自动更新的一些**主要**的配置（不要直接复制哦）

```javascript
const electron = require('electron');
const app = electron.app;
const BrowserWindow = electron.BrowserWindow;
const autoUpdater = require('electron-updater').autoUpdater;
const ipcMain = electron.ipcMain;
const path = require('path');

let mainWindow;

function createWindow() {
  mainWindow = new BrowserWindow({ width: 1280, height: 720 });
  mainWindow.loadURL(isDev ? 'http://localhost' : `file://${path.join(__dirname, '../appBuild/index.html')}`);
  mainWindow.on('closed', () => (mainWindow = null));
}

app.on('ready', createWindow);

// 在这里检查更新
app.on('will-finish-launching', () => {
  updateHandle(mainWindow);
  mainWindow.webContents.send('onReady');
});

// 检测更新，在你想要检查更新的时候执行，renderer事件触发后的操作自行编写
function updateHandle(mainWindow) {
  // 监测到新版本后是否自动下载
  autoUpdater.autoDownload = false;

  // 安装包的地址（把打包后的文件扔这个地址）
  autoUpdater.setFeedURL('https://www.test.com/release');

  // 下面是监听一些事件，向渲染进程发送事件，注意这里不会处理更新相关的交互
  // 错误
  autoUpdater.on('error', function(error) {
    mainWindow.webContents.send('onError', '错误信息');
  });

  // 检查更新
  autoUpdater.on('checking-for-update', function() {
    mainWindow.webContents.send('message', '检查更新');
  });

  // 检测到新版本
  autoUpdater.on('update-available', function(info) {
    // info 为新版本的信息
    mainWindow.webContents.send('onHasNewVersion', info);
  });

  // 没有监测到新版本
  autoUpdater.on('update-not-available', function(info) {
    mainWindow.webContents.send('message', '没有监测到新版本');
  });

  // 更新下载进度事件
  autoUpdater.on('download-progress', function(progressObj) {
    mainWindow.webContents.send('downloadProgress', progressObj);
  });

  // 下载完成
  autoUpdater.on('update-downloaded', function(
    event,
    releaseNotes,
    releaseName,
    releaseDate,
    updateUrl,
    quitAndUpdate
  ) {
    mainWindow.webContents.send('isUpdateNow');
  });

  // 下面是监听渲染进程的事件
  // 执行自动更新检查
  ipcMain.on('checkForUpdate', () => {
    autoUpdater.checkForUpdates();
  });

  // 下载更新
  ipcMain.on('onDoDownload', async () => {
    autoUpdater.downloadUpdate();
  });

  // 重启并更新
  ipcMain.on('isUpdateNow', (e, arg) => {
    autoUpdater.quitAndInstall();
  });
}
```

## 5 view 层配置

自动更新的交互逻辑和界面是需要自己写的，你可以在某个页面中监听到上面代码中主进程发送的事件。下面要做的事情就很明了了，直接上代码。

```javascript
// 这是一个自动更新的组件 update.jsx
// 仅作参考，交互自由发挥

// 这个方法用来监听主进送发送的事件/发送事件
const ipcRenderer = require('electron').ipcRenderer;

// 打开桌面端事件，检查更新
ipcRenderer.on('onReady', (event, text) => {
  ipcRenderer.send('checkForUpdate');
});

// 检测到新版本
ipcRenderer.on('onHasNewVersion', (event, info) => {
  // 提示用户是否立即更新
  Modal.confirm({
    ...
  });
});

// 下载中
ipcRenderer.on('downloadProgress', (event, progressObj) => {
  // 展示进度条
  this.setState({
    downloading: true,
    percent: parseInt(progressObj.percent)
  });
});

// 下载完成事件
ipcRenderer.on('isUpdateNow', () => {
  // 提示是否重启，重启应用就能完成更新
  Modal.confirm({
    ...
  });
});

// 更新失败
ipcRenderer.on('onError', (event, err) => {

});
```

## 6 更新

当有新版本要发布时，注意需要更改 package.json 中的版本号
