---
title: Electronで雑にアプリを作ってWindowsのポータブルアプリとして動くまでやる。 2020/02/28追記
tags: 
- JavaScript
- Electron
author: takusan_23
slide: false
---
久しぶりにElectronについて**お話します。**

# exeファイルにして動かしたい。
しかし2015年とか2016年の記事がおおい。つらい。~~令和だぞ。~~

というわけで今回はElectronで適当にアプリを作ってポータブルアプリとしてすぐ使えるようにするところを目標に作っていこうと思います。==インストール不要で動かす！

# 何作る。
marqueeタグで🍣を流すだけのアプリ。かんたん。

# 作り方
## セットアップ npm init
セットアップは自分が書いた過去の記事と同じことしてます。[ここ](https://qiita.com/takusan_23/items/eca19b4111109616bbfa)。


適当にフォルダを作成する。

作ったフォルダの中で```Shift+右クリック```で```PowerShell ウィンドウをここに開く```か```コマンド ウィンドウをここで開く```を押します。（Win10は前者。それ以外は後者）。

開いたら中で以下の文を入力します。

```terminal
npm init -y
````

これで```package.json```が作成できていれば成功です。

そしたらpackage.jsonを開いて、少し書き換えます。

```"main" : "index.js",```
↓
```"main": "./src/main.js",```

```
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
```
↓

```
 "scripts": {
    "start": "electron ."
  },
```

## セットアップ npm install --save-dev electron

PowerShellまたはコマンドプロンプトの画面はそのまま、次の文を入力しましょう。

```console
npm install --save-dev electron
```

## セットアップ 好きなエディタを開いて

最初に作ったフォルダの中にsrcフォルダを作成してください。（スクリーンショットにicon.icoがありますが気にせず。）
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/50f257ab-2fac-e705-a5ca-235e207c2496.png)

作成したら中に。
```package.json```
```index.html```
```main.js```

それぞれ

```json:package.json
{
    "main": "main.js"
}
```

```html:index.html
<!DOCTYPE html>
<html lang="ja">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>寿司が流れるだけのアプリ</title>
</head>

<body style="-webkit-app-region: drag;background-color: rgba(157, 204, 224	, .7)">
    <!-- 寿司が流れるだけ -->
    <div style="padding: 20px" class="center">
        <marquee id="marquee" scrollamount="25">
            <font id="text" size="7">🍣</font>
        </marquee>
    </div>
</body>

</html>
```

```javascript:main.js
// Modules to control application life and create native browser window
const { app, BrowserWindow, Menu } = require('electron')

// Keep a global reference of the window object, if you don't, the window will
// be closed automatically when the JavaScript object is garbage collected.
let mainWindow

function createWindow() {
    // Create the browser window.
    mainWindow = new BrowserWindow({
        width: 300,         //横
        height: 150,        //縦
        frame: false,       //フレームなくす
        transparent: true,   //背景透明化
        alwaysOnTop: true,         //最前面
        webPreferences: {
            nodeIntegration: true   //これ書く。
        }
    })

    // and load the index.html of the app.
    mainWindow.loadFile('./src/index.html')
    //メニューバー削除
    Menu.setApplicationMenu(null)

    // Open the DevTools.
    // mainWindow.webContents.openDevTools()

    // Emitted when the window is closed.
    mainWindow.on('closed', function () {
        // Dereference the window object, usually you would store windows
        // in an array if your app supports multi windows, this is the time
        // when you should delete the corresponding element.
        mainWindow = null
    })
}

// This method will be called when Electron has finished
// initialization and is ready to create browser windows.
// Some APIs can only be used after this event occurs.
app.on('ready', createWindow)

// Quit when all windows are closed.
app.on('window-all-closed', function () {
    // On macOS it is common for applications and their menu bar
    // to stay active until the user quits explicitly with Cmd + Q
    if (process.platform !== 'darwin') app.quit()
})

app.on('activate', function () {
    // On macOS it's common to re-create a window in the app when the
    // dock icon is clicked and there are no other windows open.
    if (mainWindow === null) createWindow()
})

// In this file you can include the rest of your app's specific main process
// code. You can also put them in separate files and require them here.
```

このまま実行すると半透明で小さくてどのウィンドウより前に出るただ🍣が流れるウィンドウができてるはずです。
![SnapCrab_NoName_2019-8-22_0-1-26_No-00.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/d2ef021e-e0b8-de7c-38fb-74ca80adf223.png)

ちなみに右クリックすることで閉じたり最大化できます。
![SnapCrab_寿司が流れるだけのアプリ_2019-8-21_23-58-10_No-00.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/01f43431-50e5-9aba-9fbe-0322af15944a.png)

とってもいらないアプリが完成しました。

## electron-builderをいれる
これはyarnをインストールする必要があります。インストーラーに沿ってやればできます。
ちゃんとインストールできたかどうかは以下の文をいれてバージョンが出ればおｋです。

```console
yarn -v
```

そしたら以下の文をいれて```electron-builder```をインストールします。

```console
yarn global add electron-builder
```

## package.jsonに書き足す
どっちのpackage.jsonか？srcじゃない方。npm initで作成したほう。
開いてみて明らかに下の中身と違う場合は開くの間違えてます。

```json:package.json
{
  "name": "sushi_portable",
  "version": "1.0.0",
  "description": "寿司の絵文字を眺めるアプリ。",
  "main": "./src/main.js",
  "scripts": {
    "start": "electron ."
  },
  "keywords": [],
  "author": "sushi",
  "license": "ISC",
  "devDependencies": {
    "electron": "^6.0.3"
  }
}
```

そしてすこし書き足します。buildから増えました。

```json:package.json
{
  "name": "sushi_portable",
  "version": "1.0.0",
  "description": "寿司の絵文字を眺めるアプリ。",
  "main": "./src/main.js",
  "scripts": {
    "start": "electron ."
  },
  "keywords": [],
  "author": "sushi",
  "license": "ISC",
  "devDependencies": {
    "electron": "^6.0.3"
  },
  "build": {
    "productName": "寿司の絵文字眺めるやつ",
    "appId": "sushi.emoji",
    "win": {
      "target": "portable",
      "icon": "./src/icon.ico"
    }
  }
}
```

productNameが名前、appIdはアプリケーションID（Application User Model ID）らしいです？
```target```には```portable```にします。これでexeファイルダブルクリックで起動できるアプリになります。ポータブルアプリ。
```nsis```にすればインストール形式になるそうです？。（要検証）
```icon```はアイコン画像のパスです。srcの中に入れればいいのですが、拡張子がicoなので画像ファイルを何らかの方法でicoに変換する必要があります。ただし、一つ条件があって画像サイズを256×256にする必要があるようです。

## ポータブルアプリ作成
ターミナル（PowerShell・コマンドプロンプト）で以下の文を入力。

```console
electron-builder build --win
```

あとは終わるまで待ちましょう。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/50abd5ca-91a3-2152-e577-5b82e9b24296.png)

おわると```dist```という名前のフォルダができてるのでその中のにあるexeファイルをダブルクリックして少し待てばウィンドウが出てきます。

![SnapCrab_NoName_2019-8-22_0-30-47_No-00.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/9fdcb99d-00ab-b83d-71d7-1f1d9d7d90bb.png)


**完成です！！！**

## 追記

![SnapCrab_NoName_2019-8-22_0-39-35_No-00.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/f0d114e3-99ed-fb82-441e-19ac8a9097f8.png)

ソースコードです→https://github.com/takusan23/SushiPortable

## 更に追記　2020/02/28

コメントでelectron-builderが見つからんってあったので調べたら[公式サイト](https://www.electron.build/)と導入方法が違ったので追記しておきます。

ちなみにyarnのバージョンは1.22.0です。

とりあえずelectron-builderを消しましょう。

```console
yarn global remove electron-builder
```

消せたら以下の文を入力してなにもないことを確認します

```console
yarn global list --depth=0
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/226b19ef-9f95-0116-fb30-471e25eb2d0f.png)

次に公式と同じ方法でelectron-builderを入れます。

```console
yarn add electron-builder --dev
```

成功すれば```package.json```のdevDependenciesの中に
```"electron-builder": "^22.3.2"```が追加されているはずです。

ちなみにelectron-builderと入力しても無いって言われるだけです。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/b0192e15-4f4b-915d-4bef-0bfaa01386e6.png)

次にpackage.jsonに書き足してなければ、
[package.jsonに書き足す](https://qiita.com/takusan_23/items/0ae82e0a4a1ea6469bbd#packagejson%E3%81%AB%E6%9B%B8%E3%81%8D%E8%B6%B3%E3%81%99)で書き足してきてください。ここは同じ。

最後に以下のコマンドを入力すれば処理が始まるはずです。

```console
yarn electron-builder --win
```

electron-builderがうまく動かない場合は試してみてください。（私はどちらでも動いたのですが；；）

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/b41f1f15-2800-b2ce-b128-7b24c7a9c839.png)

あと一応```package.json```置いときますね

```json:package.json
{
  "name": "ElectronBuilderSample",
  "version": "1.0.0",
  "description": "",
  "main": "src/main.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "takusan23",
  "license": "ISC",
  "devDependencies": {
    "electron": "^8.0.2",
    "electron-builder": "^22.3.2"
  },
  "build": {
    "productName": "てすと",
    "appId": "aiueo.test",
    "win": {
      "target": "portable",
      "icon": "src/icon.ico"
    }
  }
}
```

### おわりに
exeダブルクリックから数秒～数十秒かかるのは仕様？わからん！

## 参考にしました。
https://qiita.com/SallyAcolyte/items/94ed26ab62b8b32b1b2c
http://var.blog.jp/archives/78877702.html
