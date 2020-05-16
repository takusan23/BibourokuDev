---
title: AndroidStudioで変に改行されるのを直す
date: 2020-05-17 00:54:03
tags:
- Android
- AndroidStudio
---
自分用メモ

# 引数の部分改行しすぎ問題
```kotlin
override fun onCreateView(
    inflater: LayoutInflater,
    container: ViewGroup?,
    savedInstanceState: Bundle?
): View? {
    return inflater.inflate(R.layout.fragment_screen_roration, container, false)
}
```
引数の部分が改行されててきつい

# 設定
設定を開いて
Editor > Code Style > Kotlin と進んで  
`Wapping and Brances`の`warp if long`と`Chop down if long`を全部`Do not warp`にします。  
`Warp always`のところはやんなくていいと思う

{%asset_img setting.png setting%}

終わり