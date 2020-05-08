---
title: OkHttpでダウンロード進捗が知りたい！
date: 2020-05-09 00:35:36
tags:
- Android
- OkHttp
- Kotlin
---
どうもこんばんわ。  
Android 11 Betaが6/3に来るそうですよ。  
それとは関係ないんだけどインターネットが遅い。今はそこそこ出てるけど。
# 本題
Androidアプリでファイルダウンロードする時に使える**DownloadManager**ってのがあるんですけど、使えない理由が発生したのでOkHttp代替しようって話です。

## DownloadManagerとは
ファイルダウンロード時に使える。特に難しいこと(`inputStream`とか)しなくても保存してくれる。  
一行コード書けば通知領域に進捗をだせる。

## なんで使えないの
Android 10から7日過ぎたファイルを勝手に消すようになった。というか7日前に消えてる気がする。  
ちなみにファイル名を変えれば削除を回避できるけどなんかなあ。

# 環境
|なまえ|あたい|
|---|---|
|Android|11 DP 4|
|言語|Kotlin|

# 代替案：OkHttp
(API叩く時に使う)OkHttpで保存できればいいのでは！

## ただダウンロードするだけなら
- urlの中にDLしたいファイルのURL貼り付けてね。  
- 保存場所は`Android/data/{PackageName}/files/test.mp4`です。
```kotlin
fun get() {
    val request = Request.Builder().apply {
        url("")
        get()
    }.build()
    val okHttpClient = OkHttpClient()
    okHttpClient.newCall(request).enqueue(object : Callback {
        override fun onFailure(call: Call, e: IOException) {
        }
        override fun onResponse(call: Call, response: Response) {
            val file = File("${getExternalFilesDir(null)}/test.mp4")
            val byteArray = response.body?.bytes()
            if (byteArray != null) {
                file.writeBytes(byteArray)
            }
        }
    })
}
```

## ダウンロードの進捗の計算
計算とか言ったけどパーセントを求める式を使えばいいんだけど、  
`File#writeBytes()`で保存部分省略してるので進捗もなにもない。

# OkHttpにダウンロード進捗（パーセント）付ける！！

## gradle.build
dependenciesに一行足す
```gradle
dependencies {
    implementation("com.squareup.okhttp3:okhttp:4.6.0")
}
```

## activity_main.xml(レイアウト)
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center"
    android:orientation="vertical"
    tools:context=".MainActivity">


    <ProgressBar
        android:id="@+id/progressBar"
        style="?android:attr/progressBarStyleHorizontal"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_margin="10dp" />


    <TextView
        android:id="@+id/progressTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="10dp"
        android:text="0%" />

    <Button
        android:id="@+id/button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="データ取得" />

</LinearLayout>
```

## AndroidManifest.xml
インターネット接続パーミッション付けといて。

## MainActivity.kt
```kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        button.setOnClickListener {
            getData()
        }

    }

    fun getData() {
        val request = Request.Builder().apply {
            url("")
            get()
        }.build()
        val okHttpClient = OkHttpClient()
        okHttpClient.newCall(request).enqueue(object : Callback {
            override fun onFailure(call: Call, e: IOException) {

            }

            override fun onResponse(call: Call, response: Response) {
                val file = File("${getExternalFilesDir(null)}/test.mp4")
                val inputStream = response.body?.byteStream()
                val outputStream = file.outputStream()
                val buff = ByteArray(1024 * 4)
                val target = response.body?.contentLength() // 合計サイズ
                var progress = 0L
                // プログレスバーの最大値設定
                runOnUiThread {
                    progressBar.max = target?.toInt()!!
                }
                while (true) {
                    val read = inputStream?.read(buff)
                    if (read == -1 || read == null || target == null) {
                        break
                    }
                    progress += read
                    outputStream.write(buff, 0, read)
                    // UI更新
                    runOnUiThread {
                        // プログレスバー進める
                        progressBar.progress = progress.toInt()
                        // パーセントの式。
                        progressTextView.text =
                            "${((progress.toFloat() / target.toFloat()) * 100).toInt()} %"
                    }
                }
                inputStream?.close()
                outputStream.close()
            }
        })
    }
}
```

パーセントはこの部分で計算してます。~~(プログラミング要素っていうか数学かな？)~~
```kotlin
// パーセントの式。
progressTextView.text =
    "${((progress.toFloat() / target.toFloat()) * 100).toInt()} %"
```

これでOkHttpで進捗付きでダウンロードができるようになりました。わーい
	
{% asset_img download.png download %}

# おわりに
**DownloadManagerをAndroid 10以降でも使いたい人はダウンロード後にファイル名変更しないと消されます。**  
あと成功してるかどうか`response.isSuccessful`で成功時のみ保存するようにしたほうがいいと思った。(だったら書き直せ)  
あとJava読めるマンはこれ読んだほうがいいと思う→ https://github.com/square/okhttp/blob/master/samples/guide/src/main/java/okhttp3/recipes/Progress.java
# ソースコード
https://github.com/takusan23/OkHttpProgress
# 参考にしました
https://stackoverflow.com/questions/26114299/squares-okhttp-download-progress  
https://medium.com/@mkaflowski/disappearing-files-downloaded-by-downloadmanager-7c9ee5c6a66a