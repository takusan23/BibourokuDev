---
title: 【Electron / TypeScript】 ElectronでTypeScript
tags:
- Electron
- JavaScript
- TypeScript
- Node.js
author: takusan_23
slide: false
---
次Electronでなにか作る時はTypeScript使おっかなー。

# 本題
TypeScriptってのをあんまり触ったこと無いけど型が決められるとかなんとか。

# 参考にしました
https://github.com/electron/electron-quick-start-typescript

ここを真似してやる。

# package.jsonつくる

Node.js入れておいてね。
npm versionは6.4.1です。

適当にフォルダを作成し、ターミナルで以下のコードを。

```terminal
npm init -y
```

```package.json``` が作成されていれば成功です。

# Electron入れる

ターミナルで

```terminal
npm install --save electron
```

# TypeScript入れる

ターミナルで

```terminal
npm install -g typescript
```

# package.jsonを書き換える

package.jsonを開いて、"scripts"{}を書き換えます。

```json:package.json
  "scripts": {
    "build": "tsc",
    "start": "npm run build && electron ./js/main.js"
  }
```

あとmainのところも変えます

```json:package.json
  "main": "js/main.js",
```


# tsconfig.json 作成

ターミナルで

```terminal
tsc --init
```

と入力してください。
tsconfig.jsonが生成されていれば成功です。

ファイルの中身は公式通りに書き換えておきましょう。

```json:tsconfig.json
{
  "compilerOptions": {
    "module": "commonjs",
    "noImplicitAny": true,
    "sourceMap": true,
    "outDir": "js",
    "baseUrl": ".",
    "paths": {
      "*": ["node_modules/*"]
    }
  },
  "include": [
    "src/**/*"
  ]
}
```

本家ではoutDirがdistになってますが、Electron Builderの出力と被りそうなので変えときました。(ドキュメント見れば変えられそう。)

# HTMLとTypeScriptかく

srcフォルダを作成して、

index.html と main.ts を作成してください。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/185fc1f6-72ec-00aa-b75d-503663a4696c.png)

```html:index.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>TypeScript</title>
</head>
<body>
    <h1>TypeScriptですー</h1>
</body>
</html>
```

```typescript:main.ts
import { app, BrowserWindow } from "electron";
import * as path from "path";

let mainWindow: Electron.BrowserWindow;

function createWindow() {
    // Create the browser window.
    mainWindow = new BrowserWindow({
        height: 600,
        webPreferences: {
            nodeIntegration: true //trueにしておく。preload使ってもいいけど今回はパス。
        },
        width: 800,
    });

    // and load the index.html of the app.
    mainWindow.loadFile(path.join(__dirname, "../src/index.html"));　//index.htmlはsrcフォルダ（main.jsはjsフォルダ）なのでパス気をつけて。

    // Open the DevTools.
    mainWindow.webContents.openDevTools();

    // Emitted when the window is closed.
    mainWindow.on("closed", () => {
        // Dereference the window object, usually you would store windows
        // in an array if your app supports multi windows, this is the time
        // when you should delete the corresponding element.
        mainWindow = null;
    });
}

// This method will be called when Electron has finished
// initialization and is ready to create browser windows.
// Some APIs can only be used after this event occurs.
app.on("ready", createWindow);

// Quit when all windows are closed.
app.on("window-all-closed", () => {
    // On OS X it is common for applications and their menu bar
    // to stay active until the user quits explicitly with Cmd + Q
    if (process.platform !== "darwin") {
        app.quit();
    }
});

app.on("activate", () => {
    // On OS X it"s common to re-create a window in the app when the
    // dock icon is clicked and there are no other windows open.
    if (mainWindow === null) {
        createWindow();
    }
});

// In this file you can include the rest of your app"s specific main process
// code. You can also put them in separate files and require them here.
```

main.tsは参考どおり（ちょっと変えたけど）です→https://github.com/electron/electron-quick-start-typescript/blob/master/src/main.ts

# 実行してみる

ターミナルで

```terminal
npm start
```

と入力すると起動するはずです。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/9c327da2-1e15-6c66-b1fd-226fb39458c9.png)

実行するとTypeScriptがJavaScriptに変換されて、jsフォルダの中に入ってると思います。

# レンダラープロセスもTypeScriptで

レンダラープロセスとメインプロセスがよくわらんって方、
console.log()がデベロッパーツールの方に出ればレンダラープロセス、
ターミナルの方に出力されればメインプロセスです。
あとはalert()はレンダラープロセスからしか使えないのでそんな感じで。


## まずはHTMLを変えて

```html:index.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>TypeScript</title>
</head>
<body>
    <h1>TypeScriptですー</h1>
    <input type="button" value="ダイアログ" id="button">
    <!-- srcには変換後のJS指定しておく -->
    <script src="../js/renderer.js"></script>
</body>
</html>
```

## renderer.ts 作成
今回はダイアログを出してみましょう。
申し訳程度のTypeScript要素

```typescript:renderer.ts
//レンダラープロセスなのでremoteつける
const { dialog } = require('electron').remote

const button: HTMLElement = document.getElementById('button')

//ダイアログの選択肢とか
const dialogList: string[] = ["🍜", "🍣", "🥞"]

//押した時
button.onclick = function () {
    dialog.showMessageBox(null, {
        type: "info",
        message: "TypeScriptだぞ",
        buttons: dialogList
    })
}
```

これで実行してボタンを押すとダイアログが表示されると思います。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/41d52c79-08b5-8177-1df2-3616c4d74a7c.png)

以上です。

### ソースコード
https://github.com/takusan23/ElectronTypeScriptSample
