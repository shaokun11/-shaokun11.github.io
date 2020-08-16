---
title: create-react-app + electron开发桌面版应用
date: 2019-09-07 19:26:50
---

##### 目标
使用create-react-app 来配置electron 并打包,当然不想折腾环境的可以直接使用 
[模板](https://github.com/electron-react-boilerplate/electron-react-boilerplate)来搭建环境呢.我选择了折腾,这里记录其中的关键点

#### 步骤

1. 使用create-react-app 创建项目

```javascript
npx create-react-app react-electron
cd react-electron
npm install electron electron-builder image-size url --save-dev
// electron基础包
// electron-builder打包工具
// image-size 提供桌面图标的lib
// url electron的文件路径需要的东西
```

2. 在public里创建electron.js文件(可以copy官网的,下面是我的最终版本),一定要放在public目录下,因为cra会把这个文件夹的文件复制到build目录下的呢

```javascript
const electron = require('electron');
const platform = require('os').platform(); // 获取平台：https://nodejs.org/api/os.html#os_os_platform
// 控制app生命周期.
const app = electron.app;
// 浏览器窗口.
const BrowserWindow = electron.BrowserWindow;
const Menu = electron.Menu;

const path = require('path');
const url = require('url');
const ipc = electron.ipcMain
let mainWindow;

ipc.on('app close window', (sys, msg) => {
    console.log(sys, msg)
    mainWindow.close()
})

console.log(platform);

const setupMenu = () => {
    const menu = new Menu();
    mainWindow.setMenu(menu);

    const template = [{
        label: "Application",
        submenu: [{
            label: "About Application",
            selector: "orderFrontStandardAboutPanel:"
        },
            {
                type: "separator"
            },
            {
                label: "Quit",
                accelerator: "Command+Q",
                click: () => {
                    quit();
                }
            }
        ]
    }, {
        label: "Edit",
        submenu: [{
            label: "Undo",
            accelerator: "CmdOrCtrl+Z",
            selector: "undo:"
        },
            {
                label: "Redo",
                accelerator: "Shift+CmdOrCtrl+Z",
                selector: "redo:"
            },
            {
                type: "separator"
            },
            {
                label: "Cut",
                accelerator: "CmdOrCtrl+X",
                selector: "cut:"
            },
            {
                label: "Copy",
                accelerator: "CmdOrCtrl+C",
                selector: "copy:"
            },
            {
                label: "Paste",
                accelerator: "CmdOrCtrl+V",
                selector: "paste:"
            },
            {
                label: "Select All",
                accelerator: "CmdOrCtrl+A",
                selector: "selectAll:"
            }
        ]
    }];

    Menu.setApplicationMenu(Menu.buildFromTemplate(template));
};

function createWindow() {
    // 创建一个浏览器窗口.
    mainWindow = new BrowserWindow({
        width: 800,
        height: 600,
        webPreferences: {
            nodeIntegration: true,  //允许使用node的方式引入
            webSecurity: false, // 允许使用本地资源
        },
        backgroundColor: '#B1FF9D',
    });

    // 这里要注意一下，这里是让浏览器窗口加载网页。
    // 如果是开发环境，则url为http://localhost:3000（package.json中配置）
    // 如果是生产环境，则url为build/index.html
    const startUrl = process.env.ELECTRON_START_URL || url.format({
        pathname: path.join(__dirname, '/../build/index.html'),
        protocol: 'file:',
        slashes: true
    });
    setupMenu()
    // 加载网页之后，会创建`渲染进程`
    mainWindow.loadURL(startUrl);

    // 打开chrome浏览器开发者工具.
    if (startUrl.startsWith('http')) {
        mainWindow.webContents.openDevTools();
    }
    mainWindow.webContents.openDevTools();
    mainWindow.on('closed', function () {
        mainWindow = null
    });
}

app.on('ready', createWindow);

app.on('window-all-closed', function () {
    if (process.platform !== 'darwin') {
        app.quit();
    }
});

app.on('activate', function () {
    if (mainWindow === null) {
        createWindow();
    }
});
```

3. 修改package.json  main字段修改的是程序的入口, build为electron-builder的打包的一些配置

```javascript
  "main": "./public/electron.js",
  "homepage": ".",
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject",
    "electron": "electron .",
    "electron-dev": "ELECTRON_START_URL=http://localhost:3000/ electron .",
    "packager": "npm run build && rm -rf dist && electron-builder --mac"
  },
  "build": {
    "productName": "example",
    "appId": "com.example.server",
    "files": [
      "build/**/*",
      "package.json"
    ],
    "directories": {
      "buildResources": "public"
    }
  }
```

4. npm start 启动react项目
5. npm run electron-dev 启动electron项目  
6. npm run build 打包react项目
7. npm run electron 查看react打包的结果
8. npm run package 打包mac应用(win需要再配置一下)

##### 结果演示
[源码](https://github.com/shaokun11/react-electron)  
![material-ui self host font](/react/react-electron.gif) 

##### 关于我
区块链技术痴迷的程序猿一枚，如果你喜欢我的文章，可以加上微信共同学习，共同进步。  



