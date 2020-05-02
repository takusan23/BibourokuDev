---
title: AndroidのRippleのアニメーションがかくかくになるのを直す
date: 2020-05-03 02:42:37
tags:
- Android
---

# 本題
Ripple（ボタン押したときにぶあーってなるあれ）のアニメーションが固まる。  
↓これスクショだから止まってるけど、実際に押してアニメーションさせてもなんか中途半端にアニメーションが止まるんだよね。

{% asset_img ripple.png ripple %}

# 原因
`addOnGlobalLayoutListener`の中で`LayoutParams`を使ってサイズ変更するとなんかRippleがうまく動かなくなる。

これが実際の部分↓（レイアウトのIDとかgetAspectHeightFromWidthとかないけどあるってことで読んで）

```kotlin
fragment_nicovideo_framelayout.viewTreeObserver.addOnGlobalLayoutListener {
    val width = fragment_nicovideo_framelayout.width
    val height = getAspectHeightFromWidth(width)
    val params = LinearLayout.LayoutParams(width, height)
    fragment_nicovideo_framelayout.layoutParams = params
}
```

これは横の大きさが分かっているので後は計算で縦を出してサイズ変更をしているんですが、`View#getWidth()`が「0」を返すんですよ。（0以上のサイズなのに！）  
そこで`addOnGlobalLayoutListener`の登場です。  
この中で`View#getWidth()`を呼ぶことでちゃんとした横の長さが帰ってきます。

で、なぜかは知りませんがこの方法でサイズ変更をするとRippleが掛かりません。はー🤔

# 解決
今回欲しかったのは**Viewの横の長さ**ではなく**画面の横の長さ**の値なので以下の方法で画面の横の長さを取得できます。

```kotlin
val display = activity?.windowManager?.defaultDisplay
val point = Point()
display?.getSize(point)
// 横の長さ
val width = point.x
```

# 以上です
原因がなかなかわからなくてつらかった。