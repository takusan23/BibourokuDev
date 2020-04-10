---
title: AkashicEngineで作ったゲームをHexoで遊ぶ
date: 2020-04-10 23:42:48
tags:
---

どうもこんばんわ。    


# ここで遊べます
https://takusan23.github.io/Bibouroku/omake/

### ソースです
https://github.com/takusan23/AkashicEngine-FlappyBird

## Hexoの固定ページにAkashicEngineで作ったゲーム入れる

## AkashicEngine　#とは
ニコ生ゲーム作るときに使うやつ。  
**J**ava**S**criptかTypeScriptのどっちかで書ける。TSがおすすめですが。

## HTML形式で書き出す
```sh
akashic export html --bundle --minify --output export 
```
`--minify`を指定してJSを短くしてます。これしないとVSCodeのスクロールがおもすぎて使えないと思う（多分）

## Hexoに固定ページを追加
```sh
hexo new page 名前
```
これで中にmdファイルのあるフォルダが生成されます。  
ちなみにindex.mdは別にHTML(index.html)でも良いらしいです。（というかできなったから書いてない）

## でも動かない・・
{% asset_img error.png えらー %}

これはエラーの出てる部分を見るとなんかパーセントエンコーディングされた文字があるんですよね。

{% asset_img json.png パーセントエンコーディング %}

これはパーセントエンコーディングを戻してくれるサイトを使って直すことで解決します。

{% asset_img json_fix.png JSON直した %}

これで遊べるようになりました。おしまい。