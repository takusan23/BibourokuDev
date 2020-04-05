---
title: 【Android / Kotlin / ARCore】ARCoreでTextViewを出す
tags: 
- Android
- ARCore
- Kotlin
author: takusan_23
slide: false
---
AR触ってみたい。

# widgets（UIの部品 例：TextView、ImageView）がARで表示できるらしい
https://developers.google.com/ar/develop/java/sceneform/create-renderables#create_from_android_widgets

![Screenshot_1577032540.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/6f5e59b3-b466-aefe-ec75-11657ee1845f.png)


ここ。sceneformってなんぞやって話だけどこれできたらくっそ面白そうだと思ったので、
今回はTextViewをAR上に表示させるところまでやろうと思います。

## エミュレーターでARCoreアプリを動かすために
いやPixelとかGalaxyとか使いますからって方は飛ばしていいぞ。
~~てかARCore対応端末無いのにARアプリ作ろうとしてる人、どこからやる気が出てるんだ。~~

### Google Play 開発者サービス（AR）を入れます
https://github.com/google-ar/arcore-android-sdk/releases
ここからAPKをDLして、エミュレーターにドラッグアンドドロップしてインストールしてください。

今回は記事作成時最新Ver（Google_Play_Services_for_AR_1.14.1_x86_for_emulator.apk）を入れます。

成功するとアプリ一覧画面に表示されると思います。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/643c568b-f56d-f61d-3b06-f995abbff688.png)

# 環境
| なまえ        | あたい                             |
|---------------|------------------------------------|
| 端末          | エミュレーター / Google Pixel 3 XL |
| Android       | 10                                 |
| minSdkVersion | 24（ARCoreのせい）                 |

# 実装
MainActivityができてる段階まで来てください。

## build.gradle（appフォルダの中）書き足す
参考：https://developers.google.com/ar/develop/java/sceneform#configure-project

こうなっていると思いますが、

```gradle:build.gradle
apply plugin: 'com.android.application'

apply plugin: 'kotlin-android'

apply plugin: 'kotlin-android-extensions'

android {
    compileSdkVersion 29
    buildToolsVersion "29.0.2"
    defaultConfig {
        applicationId "io.github.takusan23.aredittext"
        minSdkVersion 24
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
}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
    implementation 'androidx.appcompat:appcompat:1.1.0'
    implementation 'androidx.core:core-ktx:1.1.0'
    implementation 'androidx.constraintlayout:constraintlayout:1.1.3'
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'androidx.test.ext:junit:1.1.1'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.2.0'
}
```

ここから書き足していきます。

```gradle:build.gradle
apply plugin: 'com.android.application'

apply plugin: 'kotlin-android'

apply plugin: 'kotlin-android-extensions'

android {
    compileSdkVersion 29
    buildToolsVersion "29.0.2"
    defaultConfig {
        applicationId "io.github.takusan23.aredittext"
        minSdkVersion 24
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
    //Java 8が必要みたい。
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}

dependencies {

    //ArFragmentなど    
    implementation "com.google.ar.sceneform.ux:sceneform-ux:1.14.0"

    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
    implementation 'androidx.appcompat:appcompat:1.1.0'
    implementation 'androidx.core:core-ktx:1.1.0'
    implementation 'androidx.constraintlayout:constraintlayout:1.1.3'
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'androidx.test.ext:junit:1.1.1'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.2.0'
}
```

# AndroidManifest書き足す
参考：https://developers.google.com/ar/develop/java/sceneform#manifest
まず上の方にこんなかんじに、

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="なんとか～">

　　<!-- Both "AR Optional" and "AR Required" apps require CAMERA permission. -->
　　<uses-permission android:name="android.permission.CAMERA" />
    <!-- Sceneform requires OpenGL ES 3.0 or later. -->
    <uses-feature android:glEsVersion="0x00030000" android:required="true" />
    <!-- Indicates that app requires ARCore ("AR Required"). Ensures the app is
    visible only in the Google Play Store on devices that support ARCore.
     For "AR Optional" apps remove this line. -->
   <uses-feature android:name="android.hardware.camera.ar" />
```

```<application>``` の中にも書き足します

```xml
<!-- Indicates that app requires ARCore ("AR Required"). Causes the Google
     Play Store to download and install Google Play Services for AR along
     with the app. For an "AR Optional" app, specify "optional" instead of
     "required".
-->
<meta-data android:name="com.google.ar.core" android:value="required" />
```

全部つけるとこうなります

```xml:AndroidManifest.xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="io.github.takusan23.aredittext">

    <!-- Both "AR Optional" and "AR Required" apps require CAMERA permission. -->
    <uses-permission android:name="android.permission.CAMERA" />

    <!-- Sceneform requires OpenGL ES 3.0 or later. -->
    <uses-feature android:glEsVersion="0x00030000" android:required="true" />

    <!-- Indicates that app requires ARCore ("AR Required"). Ensures the app is
         visible only in the Google Play Store on devices that support ARCore.
         For "AR Optional" apps remove this line. -->
    <uses-feature android:name="android.hardware.camera.ar" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">

        <!-- Indicates that app requires ARCore ("AR Required"). Causes the Google
             Play Store to download and install Google Play Services for AR along
             with the app. For an "AR Optional" app, specify "optional" instead of
             "required".
        -->
        <meta-data android:name="com.google.ar.core" android:value="required" />


        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

# MainActivityにFragmentを置く。
参考：https://developers.google.com/ar/develop/java/sceneform#scene-view
ConstraintLayoutは使い方がよくわからないのでLinearLayoutに置き換えて、Fragmentをドラッグします。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/a152914b-b45a-b3bf-86ba-5348df99d42e.png)

できたら**ArFragment**を選択して**OK**です！

layout_heightの値を**match_parent**にして最大まで広げるようにしましょう。
それとidもfragmentだとわかりにくくなるので、ar_fragmentとかにしときましょう。

xmlだとこうなっていると思います。

```xml:activity_main.xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity" >

    <fragment
        android:id="@+id/ar_fragment"
        android:name="com.google.ar.sceneform.ux.ArFragment"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</LinearLayout>
```

# Sceneformが利用可能かチェックする
https://developers.google.com/ar/develop/java/sceneform
には書いてありませんが、[ARCoreのサンプル](https://github.com/google-ar/sceneform-android-sdk/blob/master/samples/hellosceneform/app/src/main/java/com/google/ar/sceneform/samples/hellosceneform/HelloSceneformActivity.java)には書いてあったので、Kotlinに変換して使います。

```kotlin:MainActivity.kt
fun checkIsSupportedDeviceOrFinish(activity: Activity): Boolean {
    val MIN_OPENGL_VERSION = 3.0
    if (Build.VERSION.SDK_INT < Build.VERSION_CODES.N) {
        Toast.makeText(activity, "SceneformにはAndroid N以降が必要です。", Toast.LENGTH_LONG).show()
        activity.finish()
        return false
    }
    val openGlVersionString = (activity.getSystemService(Context.ACTIVITY_SERVICE) as ActivityManager).deviceConfigurationInfo.glEsVersion
    if (java.lang.Double.parseDouble(openGlVersionString) < MIN_OPENGL_VERSION) {
        Toast.makeText(activity, "SceneformにはOpen GL 3.0以降が必要です。", Toast.LENGTH_LONG).show()
        activity.finish()
        return false
    }
    return true
}
```

これを```setContentView()```の前で使います。

```kotlin:MainActivity.kt
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    //条件満たしてなければActivity終了させる
    if(!checkIsSupportedDeviceOrFinish(this)){
        return
    }
    setContentView(R.layout.activity_main)
}
```

# ArFragment取得
MainActivityで置いたArFragmentを取得します。

```kotlin:MainActivity.kt
lateinit var arFragment: ArFragment
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    //条件満たしてなければActivity終了させる
    if(!checkIsSupportedDeviceOrFinish(this)){
        return
    }
    setContentView(R.layout.activity_main)
    //ArFragment取得
    arFragment = supportFragmentManager.findFragmentById(R.id.ar_fragment) as ArFragment
}
```

# 起動してみる
ここまでで特に問題がなければ、
カメラの権限許可がきて、許可すると映ると思います。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/e802a9d9-3d6c-623c-da86-6e2f2ed416c4.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/0b97794e-8ed5-2915-e472-9206df1710e1.png)

映れば成功です。ここまで間違えずついてこれてます。

# いよいよViewを現実に表示させる・・・！
参考：https://developers.google.com/ar/develop/java/sceneform/create-renderables#create_from_android_widgets

まずlayoutフォルダに**ar_layout.xml**という名前で作成します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/521816ac-26b0-e798-6fb2-4c5613da107d.png)

次に現実で表示させたいUI部品を並べます。
今回はTextViewを置きます。（タイトル詐欺回避）

```xml:ar_layout.xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:background="#ffffff"
    android:orientation="vertical"
    tools:context=".MainActivity">


    <TextView
        android:id="@+id/textView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="10sp"
        android:gravity="center"
        android:text="てきすとだよー" />
</LinearLayout>
```

## ARで扱えるように
参考：https://developers.google.com/ar/develop/java/sceneform/create-renderables#create_from_android_widgets

```kotlin:MainActivity.kt
    //レイアウトをARに・・・
    lateinit var viewRenderable: ViewRenderable
```

読み込みます。

```kotlin:MainActivity.kt
//レイアウトを読み込む
ViewRenderable.builder()
    .setView(this, R.layout.ar_layout)
    .build()
    .thenAccept { renderable -> viewRenderable = renderable } //読み込み成功
    .exceptionally {
        //読み込み失敗
        it.printStackTrace()
        Toast.makeText(this, "読み込みに失敗しました。", Toast.LENGTH_LONG).show()
        null
    }
```

最後に押したらレイアウトをARで表示させるところを

```kotlin:MainActivity.kt
arFragment.setOnTapArPlaneListener { hitResult, plane, motionEvent ->
    if (::viewRenderable.isInitialized) {
        //初期化済みのとき、利用可能
        // Create the Anchor.
        val anchor = hitResult.createAnchor()
        val anchorNode = AnchorNode(anchor)
        anchorNode.setParent(arFragment.arSceneView.scene)
        // Create the transformable andy and add it to the anchor.
        val node = TransformableNode(arFragment.transformationSystem)
        node.setParent(anchorNode)
        node.renderable = viewRenderable
        node.select()
    }
}
```

これで動くはずです！！！実行してみましょう！！！

![Screenshot_1577028389.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/065d6e9d-c470-8fd6-5d6e-fe5b853aa8f4.png)

てきすとだよー

## 押したら消す

```kotlin:MainActivity.kt
node.setOnTapListener { hitTestResult, motionEvent -> 
    node.isEnabled = false
}
```

isEnabledにtrueで消えますがこれでいいのかは不明。誰か頼んだ

## ARで表示したTextViewのテキストを変更したい

MainActivityにEditTextとボタンを置きます。

```xml:activity_main.xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity" >

    <fragment
        android:id="@+id/ar_fragment"
        android:name="com.google.ar.sceneform.ux.ArFragment"
        android:layout_width="match_parent"
        android:layout_weight="1"
        android:layout_height="wrap_content" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">

        <EditText
            android:id="@+id/ar_change_textview"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:ems="10"
            android:inputType="textPersonName"
            android:text="てきすとだよー" />

        <Button
            android:id="@+id/ar_change_button"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="変更" />
    </LinearLayout>

</LinearLayout>
```

そしたらボタンを押したらテキストを変更する処理を書きます。

```kotlin:MainActivity,.kt
//テキスト変更
ar_change_button.setOnClickListener {
    //テキスト取得
    val text = ar_change_textview.text.toString()
    //ARで表示するレイアウト取得
    val linearLayout = viewRenderable.view as LinearLayout
    //TextView取得
    val textView = linearLayout.findViewById<TextView>(R.id.textView)
    //変更
    textView.text = text
}
```

動くはずです。

![Screenshot_1577029034.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/52a78edd-fb8a-546b-5277-7f8201e71db4.png)

なんかEditTextの部分が黒くなるけどなんで？

## 写真にして保存する
おまけです。
疲れたのでコードだけ。レイアウトにボタンを追加してidを「ar_take_a_picture」にしてください。

```kotlin:MainActivity.kt
//撮影ボタン押したとき
ar_take_a_picture.setOnClickListener {
    //PixelCopy APIを利用する。のでOreo以降じゃないと利用できません。
    val bitmap = Bitmap.createBitmap(
        arFragment.view?.width ?: 100,
        arFragment.view?.height ?: 100,
        Bitmap.Config.ARGB_8888
    )
    val intArray = IntArray(2)
    arFragment.view?.getLocationInWindow(intArray)
    try {
        PixelCopy.request(
            arFragment.arSceneView as SurfaceView, //SurfaceViewを継承してるらしい。windowだと真っ暗なので注意！
            Rect(
                intArray[0],
                intArray[1],
                intArray[0] + (arFragment.view?.width ?: 0),
                intArray[1] + (arFragment.view?.height ?: 0)
            ),
            bitmap,
            { copyResult: Int ->
                if (copyResult == PixelCopy.SUCCESS) {
                    //成功時
                    //ここのフォルダは自由に使っていい場所（サンドボックス）
                  val mediaFolder = externalMediaDirs.first()
                  //写真ファイル作成
                  val file = File("${mediaFolder.path}/${System.currentTimeMillis()}.jpg")
                  //Bitmap保存
                  bitmap.compress(Bitmap.CompressFormat.JPEG, 100, file.outputStream())
                  Toast.makeText(this, "保存しました", Toast.LENGTH_SHORT).show()
                }
            },
            Handler()
        )
    } catch (e: IllegalArgumentException) {
        e.printStackTrace()
        Toast.makeText(this@MainActivity, "失敗しました。", Toast.LENGTH_LONG).show()
    }
}
```

?:の値がくっそ雑だけどゆるして
Android 10で動作確認済です。対象範囲別ストレージ対策。
写真データは
/sdcard/Android/media/パッケージID 
に入っています。

# おわりに
GitHubに公開しておきます。
https://github.com/takusan23/AREditText

ついでに参考にしたプロジェクトも。
https://github.com/google-ar/sceneform-android-sdk/tree/master/samples/hellosceneform

PixelCopy参考にしました。
https://friegen.xyz/getdrawingcache-deprecated/

おつです。８８８８８８
