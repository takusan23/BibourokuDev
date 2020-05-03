---
title: Android11のMANAGE_EXTERNAL_STORAGEを試す
date: 2020-05-04 00:44:24
tags:
- Android
- Android11
---
どうもこんばんわ。  
ニコ生で「とある科学の超電磁砲T」の1~12話一挙放送見ました。初見だったけど面白かったです。OPかっこいい。  
1期2期もいつか見たいですね。

# 本題
Android 11 からまーたファイルアクセスの方法が変わるらしい。はー？  
どうやらファイルマネージャーアプリとかバックアップアプリ向けのファイルアクセス権限**MANAGE_EXTERNAL_STORAGE**が追加されるそうな。

# `MANAGE_EXTERNAL_STORAGE` #とは
Android 9以前のように生パス（おそらく`/storage/emulated/0/Download`みたいなパスのこと）でのアクセスができるようになる。  
`ACTION_OPEN_DOCUMENT_TREE`でSDカードへアクセスできないしSDカードアクセスするにはこれしかない？。もうSDカード使わせる気ないなこれ。  

そんな有能そうな権限ですが  

> デベロッパープレビューの今後のバージョンでは、この権限が必要なアプリ向けのガイドラインをGoogle Playに提供する予定です。

[引用元](https://developer.android.com/preview/privacy/storage#manage-device-storage)

がなんとか書いてあるのでなんか追加で作業が必要なのかな。  
むやみにぽんぽん打てる訳ではないっぽい？

ちなみに`MANAGE_EXTERNAL_STORAGE`でも外部のアプリ固有領域(Android/data)にはアクセスできません。この領域ってもうファイルマネージャーとか端末単体じゃ見れないのかな（できないのそこそこ不便）。

# 環境
|name|value|
|---|---|
|Android|11 DP3|
|Android Studio|4.1 Canary 8|
|言語|Kotlin|

# やってみる
## Android 11のAPI使えるように
Android 11のSDKは各自入れてきてね。  
appフォルダの方のbuild.gradle開いてandroid{}のところを書き換えます。
```gradle
android {
    compileSdkVersion 'android-R'
    buildToolsVersion "29.0.3"

    defaultConfig {
        applicationId "com.example.manageexternalstoragesample"
        minSdkVersion "R"
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
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    kotlinOptions {
        jvmTarget = '1.8'
    }
}
```

`compileSdkVersion 'android-R'`でいいらしい？

## 権限追加
何故か補充が効かないのでコピペ
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.manageexternalstoragesample">

    <uses-permission android:name="android.permission.MANAGE_EXTERNAL_STORAGE" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.ManageExternalStorageSample">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

## レイアウト
そろそろConstraintLayout使えるようになりたい（難しそう）

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
    
    <Button
        android:id="@+id/button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="権限付与" />

    <Button
        android:id="@+id/show"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="取得" />

    <Button
        android:id="@+id/read"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="書き込み" />
</LinearLayout>
```

## 権限もらう
これも補充が効かないのでIntentの中身はコピペで
```kotlin
button.setOnClickListener {
    val intent = Intent("android.settings.MANAGE_ALL_FILES_ACCESS_PERMISSION")
    startActivity(intent)
}
```
Android 11から他のアプリに重ねるの権限取得で直接アプリの設定に飛べなくなりましたが、これも直接飛べずに一覧画面から選ぶ形式になってます。

権限取得ボタン押すと一覧画面でます

{% asset_img list.png list %}

{% asset_img screen.png screen %}


## 生パスで読み込んでみる
Android 10から使えなくなった以下のコードも ~~`MANAGE_EXTERNAL_STORAGE`権限を持っていれば・・・！~~  
Android 11では**読み込みなら読み込み権限(READ_EXTERNAL_STORAGE)すらいらない**です。  
ただこの仕様はAndroid 11で追加されたため、Android 10でこれ使うにはManifestに`requestLegacyExternalStorage="true"`(Scoped Storage無効化)を指定する必要があります。
```kotlin
show.setOnClickListener {
    File("/storage/emulated/0/").listFiles().forEach {
        println(it.name)
    }
}
```
出力結果
```console
Android
Music
Podcasts
Ringtones
Alarms
Notifications
Pictures
Movies
Download
DCIM
```
以下略

## 生パスで書き込んで見る
Android 10から使えなくなった以下のコードも`MANAGE_EXTERNAL_STORAGE`権限の前では・・・！  
あとボタンのIDが`read`になってるけど`write`が命名的には正解ですね。プログラム的には間違ってないけど。
```kotlin
read.setOnClickListener {
    File("/storage/emulated/0/書き込みテスト.txt").apply {
        createNewFile()
        writeText("書き込めてる～？")
    }
}
```

Kotlinの拡張関数`writeText()`使ってるから一行で書き込めてるけどJavaだと長くなりそう。

多分これもAndroid 10を対象にするにはScoped Storageを無効にする必要があります多分。

{% asset_img write.png write %}

# おわりに
くっそめんどくさくね？(SAFだのScopedStorageだのMediaStoreだの)  
あとSDカード入る実機ほしい。Xperiaの21:9使ってみたい。  
（S7 EdgeあるけどなんかSDM 820の割にNexus 7 2013のほうが動くからハズレSoC引いた？）