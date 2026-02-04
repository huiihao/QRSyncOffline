# 打包 QRSyncOffline 为可执行文件

本文档介绍如何将 QRSyncOffline 打包为独立的可执行文件，方便在没有浏览器的设备上使用。

## 方案一：使用 Nativefier（推荐）

[Nativefier](https://github.com/nativefier/nativefier) 是一个将网页应用打包为桌面应用的工具。

### 安装 Nativefier

```bash
# 需要 Node.js 环境
npm install -g nativefier
```

### 打包命令

```bash
# 打包发送端
nativefier \
  --name "QRSyncOffline-Sender" \
  --icon "../icon.png" \
  --width 1200 \
  --height 800 \
  --min-width 800 \
  --min-height 600 \
  --file-download-options '{"saveAs": true}' \
  --single-instance \
  --tray \
  "file:///path/to/QRSyncOffline-Fixed/send/index.html"

# 打包接收端
nativefier \
  --name "QRSyncOffline-Receiver" \
  --icon "../icon.png" \
  --width 900 \
  --height 800 \
  --min-width 600 \
  --min-height 600 \
  --file-download-options '{"saveAs": true}' \
  --single-instance \
  --tray \
  "file:///path/to/QRSyncOffline-Fixed/receiver/index.html"
```

### Linux 系统打包

```bash
# 安装依赖
sudo apt-get install imagemagick  # Debian/Ubuntu
sudo yum install ImageMagick        # CentOS/RHEL

# 打包 Linux 版本
nativefier \
  --platform linux \
  --arch x64 \
  --name "QRSyncOffline-Sender" \
  --width 1200 \
  --height 800 \
  "file:///path/to/QRSyncOffline-Fixed/send/index.html"
```

## 方案二：使用 Electron

### 1. 创建 Electron 项目

```bash
mkdir qrsync-electron && cd qrsync-electron
npm init -y
npm install electron --save-dev
```

### 2. 创建主进程文件 (main.js)

```javascript
const { app, BrowserWindow } = require('electron');
const path = require('path');

function createWindow () {
  const mainWindow = new BrowserWindow({
    width: 1200,
    height: 800,
    minWidth: 800,
    minHeight: 600,
    webPreferences: {
      nodeIntegration: true,
      contextIsolation: false
    },
    icon: path.join(__dirname, 'icon.png')
  });

  // 加载本地文件
  mainWindow.loadFile('index.html');
  
  // 打开开发者工具（调试用）
  // mainWindow.webContents.openDevTools();
}

app.whenReady().then(() => {
  createWindow();
  
  app.on('activate', function () {
    if (BrowserWindow.getAllWindows().length === 0) createWindow();
  });
});

app.on('window-all-closed', function () {
  if (process.platform !== 'darwin') app.quit();
});
```

### 3. 创建 package.json

```json
{
  "name": "QRSyncOffline",
  "version": "2.0.0",
  "description": "Offline QR Code File Transfer Tool",
  "main": "main.js",
  "scripts": {
    "start": "electron .",
    "build": "electron-builder",
    "build:win": "electron-builder --win",
    "build:mac": "electron-builder --mac",
    "build:linux": "electron-builder --linux"
  },
  "devDependencies": {
    "electron": "^28.0.0",
    "electron-builder": "^24.0.0"
  },
  "build": {
    "appId": "com.qrsyncoffline.app",
    "productName": "QRSyncOffline",
    "directories": {
      "output": "dist"
    },
    "files": [
      "main.js",
      "index.html",
      "send/**/*",
      "receiver/**/*",
      "js/**/*"
    ],
    "mac": {
      "target": "dmg"
    },
    "win": {
      "target": "nsis"
    },
    "linux": {
      "target": "AppImage"
    }
  }
}
```

### 4. 打包

```bash
# 调试运行
npm start

# 打包所有平台
npm run build

# 打包特定平台
npm run build:win    # Windows
npm run build:mac    # macOS
npm run build:linux  # Linux
```

## 方案三：使用 Tauri（Rust，体积更小）

### 1. 安装依赖

```bash
# 安装 Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# 安装 Tauri CLI
cargo install tauri-cli
```

### 2. 创建项目

```bash
cargo create-tauri-app qr-sync-offline
cd qr-sync-offline
```

### 3. 配置 tauri.conf.json

```json
{
  "build": {
    "beforeBuildCommand": "",
    "beforeDevCommand": "",
    "devPath": "../send/index.html",
    "distDir": "../"
  },
  "tauri": {
    "allowlist": {
      "all": false,
      "fs": {
        "all": true
      },
      "dialog": {
        "all": true
      }
    },
    "windows": [
      {
        "title": "QRSyncOffline - Sender",
        "width": 1200,
        "height": 800,
        "resizable": true,
        "fullscreen": false
      }
    ],
    "security": {
      "csp": null
    },
    "bundle": {
      "active": true,
      "targets": ["deb", "rpm", "appimage", "msi", "dmg"],
      "identifier": "com.qrsyncoffline.app",
      "icon": ["icons/32x32.png", "icons/128x128.png", "icons/128x128@2x.png", "icons/icon.icns", "icons/icon.ico"]
    }
  }
}
```

### 4. 打包

```bash
cargo tauri build
```

## 方案四：使用 PyWebView（Python）

### 1. 安装依赖

```bash
pip install pywebview pyinstaller
```

### 2. 创建启动脚本 (app.py)

```python
import webview
import os
import sys

def get_resource_path(relative_path):
    """获取资源路径"""
    if hasattr(sys, '_MEIPASS'):
        return os.path.join(sys._MEIPASS, relative_path)
    return os.path.join(os.path.abspath("."), relative_path)

def main():
    # 创建发送端窗口
    sender_window = webview.create_window(
        'QRSyncOffline - 发送端',
        get_resource_path('send/index.html'),
        width=1200,
        height=800,
        min_size=(800, 600)
    )
    
    # 创建接收端窗口
    receiver_window = webview.create_window(
        'QRSyncOffline - 接收端',
        get_resource_path('receiver/index.html'),
        width=900,
        height=800,
        min_size=(600, 600)
    )
    
    webview.start()

if __name__ == '__main__':
    main()
```

### 3. 使用 PyInstaller 打包

```bash
# 创建 spec 文件
pyi-makespec --windowed --onefile --add-data "send:send" --add-data "receiver:receiver" --add-data "js:js" --icon=icon.ico app.py

# 打包
pyinstaller app.spec
```

### Linux 系统打包

```bash
# 安装依赖
sudo apt-get install python3-pip python3-dev
pip3 install pywebview pyinstaller

# 对于 GTK 后端
sudo apt-get install python3-gi python3-gi-cairo gir1.2-gtk-3.0 gir1.2-webkit2-4.0

# 打包
pyinstaller --windowed --onefile \
  --add-data "send:send" \
  --add-data "receiver:receiver" \
  --add-data "js:js" \
  app.py
```

## 方案五：使用 Docker + Nginx（服务器部署）

```dockerfile
FROM nginx:alpine

COPY . /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

```bash
# 构建镜像
docker build -t qrsyncoffline .

# 运行容器
docker run -d -p 8080:80 qrsyncoffline

# 访问 http://localhost:8080
```

## 推荐方案总结

| 方案 | 优点 | 缺点 | 适用场景 |
|------|------|------|----------|
| Nativefier | 简单快速 | 包体积较大 | 快速打包 |
| Electron | 功能强大 | 包体积大 | 复杂应用 |
| Tauri | 体积小、性能好 | 需要Rust | 追求小体积 |
| PyWebView | Python生态 | 依赖Python | Python开发者 |

## 调试技巧

### 打开开发者工具

大多数方案都支持打开 Chromium 开发者工具：

- **Electron**: 取消注释 `mainWindow.webContents.openDevTools()`
- **Nativefier**: 打包时添加 `--inject` 参数注入调试脚本
- **Tauri**: 按 `Ctrl+Shift+I` 或 `F12`
- **PyWebView**: 设置 `debug=True`

### 查看控制台日志

```javascript
// 在 HTML 中添加调试日志
console.log('Debug info:', variable);
```

## 常见问题

### Q: 打包后文件路径错误？

A: 确保使用相对路径，并在打包配置中正确设置资源目录。

### Q: 摄像头无法使用？

A: 需要在打包配置中添加摄像头权限，或使用 HTTPS/localhost。

### Q: 文件下载失败？

A: 检查打包配置中的文件下载选项，确保 `saveAs` 设置为 `true`。

---

如有问题，请提交 Issue 或查看各工具的官方文档。
