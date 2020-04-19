---
title: Android11(DP2)でファイル保存まとめ
date: 2020-04-19 23:42:49
tags:
- Android
- ScopedStorage
- Kotlin
---

どうもこんばんわ。  
超会議とても良かったです。１押す  
ちなみに私は超パーティー再放送とアニメ一挙放送を主に見てました。Vはわからんのでな。  
らららコッペパンってらき☆すただったんだ。

今年のGoogle IOが中止になったどころかAndroid 11に関してもドキュメントが全然更新されなくなっちゃって大丈夫なんかこれ。

# ほんぺん 

ところでAndroid10から自由にフォルダにアクセスできなくなりました。これにより自由に`Downloadフォルダ`とか`Picturesフォルダ`等へアクセスしたりファイル作成とかができなくなりました。

じゃあどこに保存すればいいんだって話ですが、`Scoped Storage（日本語：対象範囲別外部ストレージ）`という仕組みが作られ、アプリごとに**権限無し**で書き込むことができるフォルダが作られるようになりました。

ドキュメント：https://developer.android.com/training/data-storage/files/external-scoped?hl=ja

# 環境

| あたい  | なまえ                                        |
|---------|-----------------------------------------------|
| 端末    | Pixel 3 XL                                    |
| Android | 11 Developer Preview 2                        |
| 言語    | Kotlin                                        |
| SDCard  | 知らんわ（Pixelに刺さらないのでわからない。） |

# Android 11 から（保存ではない）
`Downloadフォルダ`とか`Picturesフォルダ`等に読み取り専用でならアクセスできるようになったっぽい？  

```kotlin
File("/storage/emulated/0/").listFiles()?.forEach {
    println(it.name)
}
```

↓実行結果
```console
I/System.out: Android
I/System.out: Music
I/System.out: Podcasts
I/System.out: Ringtones
I/System.out: Alarms
I/System.out: Notifications
I/System.out: Pictures
I/System.out: Movies
I/System.out: Download
I/System.out: DCIM
```

# 一番いい？→Intent.ACTION_OPEN_DOCUMENT_TREE
## #これはなに？
自由にはアクセスできない代わりにユーザーが指定したフォルダにはアクセスできるよってやつ。  
写真アプリなんだけどScoped Storageに保存するのではなくPicturesフォルダに保存したいんだって時に使う。
## これを使うと？
内部ストレージを指定すると`Downloadフォルダ`とか`Picturesフォルダ`にアクセスできるちょっとやばめ。  
**~~一度許可を貰えればこっちのもんです。~~**

# 実装
## ライブラリ入れる
appフォルダにある方の`build.gradle`です。
```gradle
// Preference
implementation "androidx.preference:preference:1.1.0"
// ↓一行書き足す
implementation "androidx.documentfile:documentfile:1.0.1"
```
あと楽するためにJavaのバージョンを8にします。  
これとこれってなってるところね。
```gradle
android {
    compileSdkVersion 29
    buildToolsVersion "29.0.3"

    defaultConfig {
        applicationId "io.github.takusan23.actionopendocumenttreesample"
        minSdkVersion 21
        targetSdkVersion 29
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
    // これと
    compileOptions {
        targetCompatibility 1.8
        sourceCompatibility 1.8
    }
    // これ
    kotlinOptions {
        jvmTarget = '1.8'
    }
}
```
## レイアウト
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <EditText
        android:id="@+id/editText"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:ems="10"
        android:inputType="textPersonName"
        android:text="" />

    <Button
        android:id="@+id/path_button"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="保存先指定" />

    <Button
        android:id="@+id/save_button"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="保存" />

    <Button
        android:id="@+id/read_button"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="読み込み" />
</LinearLayout>
```

## 許可をもらう
takePersistableUriPermission()が大事？
```kotlin

val REQUEST_CODE = 816
lateinit var prefSetting: SharedPreferences

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    setContentView(R.layout.activity_main)
    prefSetting = PreferenceManager.getDefaultSharedPreferences(this)
    path_button.setOnClickListener {
        // ユーザーにアプリで使っていいフォルダを選んでもらう。
        val intent = Intent(Intent.ACTION_OPEN_DOCUMENT_TREE)
        startActivityForResult(intent, REQUEST_CODE)
    }

}
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    super.onActivityResult(requestCode, resultCode, data)
    if (requestCode == REQUEST_CODE && resultCode == Activity.RESULT_OK) {
        // リクエストコードが一致&成功のとき
        val uri = data?.data ?: return
        // Uriは再起すると使えなくなるので対策
        contentResolver.takePersistableUriPermission(
            uri,
            Intent.FLAG_GRANT_READ_URI_PERMISSION or Intent.FLAG_GRANT_WRITE_URI_PERMISSION
        )
        // Uri保存。これでアプリ再起動後も使えます。
        prefSetting.edit {
            putString("uri", uri.toString())
        }
    }
}

```

これで実行して「保存先指定」ボタンを押して保存先を選びます。  

{% asset_img kyoka_1.png 許可 %}  
許可します  
{% asset_img kyoka_2.png ダイアログ %}  

## 保存、読み込みを実装する
最終的の`MainActivity.kt`はこうです！

```kotlin
class MainActivity : AppCompatActivity() {

    val REQUEST_CODE = 816
    lateinit var prefSetting: SharedPreferences

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        prefSetting = PreferenceManager.getDefaultSharedPreferences(this)

        path_button.setOnClickListener {
            // ユーザーにアプリで使っていいフォルダを選んでもらう。
            val intent = Intent(Intent.ACTION_OPEN_DOCUMENT_TREE)
            startActivityForResult(intent, REQUEST_CODE)
        }

        // 保存ボタン
        save_button.setOnClickListener {
            saveFile()
        }

        // 読み込みボタン
        read_button.setOnClickListener {
            readFile()
        }

    }

    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        super.onActivityResult(requestCode, resultCode, data)
        if (requestCode == REQUEST_CODE && resultCode == Activity.RESULT_OK) {
            // リクエストコードが一致&成功のとき
            val uri = data?.data ?: return
            // Uriは再起すると使えなくなるので対策
            contentResolver.takePersistableUriPermission(
                uri,
                Intent.FLAG_GRANT_READ_URI_PERMISSION or Intent.FLAG_GRANT_WRITE_URI_PERMISSION
            )
            // Uri保存。これでアプリ再起動後も使えます。
            prefSetting.edit {
                putString("uri", uri.toString())
            }
        }
    }

    // ファイル保存
    private fun saveFile() {
        val uri = prefSetting.getString("uri", "")?.toUri() ?: return
        // ファイル操作
        DocumentFile.fromTreeUri(this, uri)?.apply {
            // テキストファイル
            val textFile = if (findFile("test.txt")?.exists() == true) {
                // すでに作成済み
                findFile("test.txt") ?: return@apply
            } else {
                // まだないので新規作成
                createFile("text/plain", "test.txt") ?: return@apply
            }
            // 書き込む
            contentResolver.openOutputStream(textFile.uri)?.apply {
                // Activityに置いたTextEditのテキスト取得
                write(editText.text?.toString()?.toByteArray())
                close()
            }
        }
    }

    // ファイル読み込み
    private fun readFile() {
        val uri = prefSetting.getString("uri", "")?.toUri() ?: return
        // ファイル操作
        DocumentFile.fromTreeUri(this, uri)?.apply {
            // テキストファイル取り出し
            if (findFile("test.txt")?.exists() == false) {
                // なければ終了
                return@apply
            }
            val textFile = findFile("test.txt") ?: return@apply
            // テキスト取り出す
            val text = contentResolver.openInputStream(textFile.uri)?.bufferedReader()?.readLine()
            editText.setText(text)
        }
    }

}

```

これで保存、読み込みができるようになりました。やったー  
{% asset_img filer.png ふぁいらー %}  

## これで内部ストレージのパスを指定すると・・？
スクリーンショットのフォルダにもアクセスできます。
```kotlin
val uri = prefSetting.getString("uri", "")?.toUri() ?: return
// スクリーンショットの名前を表示させる
DocumentFile.fromTreeUri(this, uri)?.apply {
findFile("Pictures")?.findFile("Screenshots")?.listFiles()?.forEach {
            println(it.name)
        }
    }
```

## Android 11 DP2 の仕様？
SDカードに保存する手段だったらしいけどAndroid 11からこの方法でSDカードの場所を指定するのはできなくなるらしいぞ。


一応ソースコード置いておきますね↓
https://github.com/takusan23/ActionOpenDocumentTreeSample

# Scoped Storageで保存

Scoped Storageのパスは以下の関数で取れます。（キャッシュ系は省くぜ）  
この２つはScoped Storageなフォルダのパスなので権限無しでFileクラスで読み書きできます。（Kotlinだと拡張関数で幸せになれる。）
- `getExternalFilesDir(null)?.path`
    - `/storage/emulated/0/Android/data/io.github.takusan23.scopedstoragesample/files`
    - データはアンインストール時に削除されます。
- `externalMediaDirs[0].path`
    - `/storage/emulated/0/Android/media/io.github.takusan23.scopedstoragesample`
    - データはアンインストール時に削除されます。
    - MediaStoreのスキャンに対応してるので他のアプリでも利用できる？
        - ~~Googleフォトの新しいフォルダ見つけたよ！バックアップする？のやつに認知されるのはこれだっけ。~~
        - Android 11 DP2で検証できなかった。ごめん。
    - **Android R（11）から非推奨になりました。でも使えるっぽい。~~余計なことするなよ~~**


両者共にファイルパスが長い/アンインストール時にデータを消すのが特徴。  

以下書き込みサンプルです。  
Kotlinの拡張関数（writeText()とか）でらくらくテキスト保存。

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <EditText
        android:id="@+id/editText"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:ems="10"
        android:inputType="textPersonName"
        android:text="" />

    <Button
        android:id="@+id/save_button"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="保存" />

    <Button
        android:id="@+id/read_button"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="読み込み" />
</LinearLayout>
```

```kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        save_button.setOnClickListener {
            saveFile()
        }

        read_button.setOnClickListener {
            readFile()
        }

    }

    private fun saveFile() {
        File("${externalMediaDirs[0].path}/test.txt").apply {
            createNewFile()
            writeText(editText.text.toString())
        }
    }

    private fun readFile() {
        File("${externalMediaDirs[0].path}/test.txt").apply {
            if (exists()) {
                val text = readText()
                editText.setText(text)
            }
        }
    }

}
```

## Android 11 DP2 の仕様？
ファイルマネージャー（Filesアプリとか）から見れなくなった。  
USB接続してパソコンで見るかAndroid StudioのDevice Explorerを使うしかない？

## ちなみに
ドキュメントとか関数名とかにExternal（日本語：外部）っていう文字列があると外部ストレージのSDカードのことだと思っちゃうけど実は違って、  
アプリ自身とroot権限のある環境？でしかアクセスできないパスを返す関数`getFilesDir()`の場所のことを**内部ストレージ**としているらしくて、  
それ以外が外部扱いということでExternalって文字列が使われているんだって。   
わからんわ。

# Storage Access Framework で保存
好きな場所に保存したいときはこれを使えって  
ファイル選択画面みたいなUIでファイルの保存先を選んで保存したあと、`onActivityResult`でUriを受け取り`openOutputStream`を使って書き込むらしいです。

以下コード

レイアウトは上のScoped Storageを使い回す。

```kotlin
class MainActivity : AppCompatActivity() {
    val REQUEST_CODE = 810

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        save_button.setOnClickListener {
            openSAF()
        }

    }

    private fun openSAF() {
        val intent = Intent(Intent.ACTION_CREATE_DOCUMENT).apply {
            type = "text/*"
            putExtra(Intent.EXTRA_TITLE, "test.txt")
        }
        startActivityForResult(intent, REQUEST_CODE)
    }

    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        super.onActivityResult(requestCode, resultCode, data)
        if (requestCode == REQUEST_CODE && resultCode == Activity.RESULT_OK) {
            // 書き込む
            val uri = data?.data ?: return
            contentResolver.openOutputStream(uri)?.apply {
                write(editText.text.toString().toByteArray())
                close()
            }
        }
    }

}
```
これ使えば`Downloadフォルダ`だろうと自由に保存できます。保存するときにいちいち保存先を選ぶ必要があって使いにくいけど。

これが保存画面で、保存先を選ぶ。

{% asset_img save.png SAF %}

ファイルマネージャーで見るとちゃんと作成されていることがわかる。

{% asset_img list.png List %}

**め！ん！ど！い！**

