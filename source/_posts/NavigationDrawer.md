---
title: 【Android / Kotlin】 ナビゲーションドロワーのつくりかた
tags:
- Android
- Kotlin
- MaterialDesign
- NavigationDrawer
author: takusan_23
slide: false
---
どうも。こんばんわ

# なにがあったのか
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/05545e12-a4c7-83ae-31b8-7c63a448b994.png)

この画面からナビゲーションドロワーを作成すると前と違ってめっちゃフォルダ・ファイルが作成されるようになった。どゆこと

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/1d14c929-e9e5-6c41-90a6-47a5044c4241.png)


しかもbuild.gradleに書き足さないとエラーが出るってなんだよ。

```
Cannot inline bytecode built with JVM target 1.8 into bytecode that is being built with JVM target 1.6. Please specify proper '-jvm-target' option
```

このエラーはappフォルダにあるほうの```build.gradle```を開いて、android{}の波かっこの中に以下の文を書き足すことで動きます。

```gradle
compileOptions {
    sourceCompatibility = 1.8
    targetCompatibility = 1.8
}
kotlinOptions {
    jvmTarget = "1.8"
}
```

ということで今回は空の状態からナビゲーションドロワーを作っていこうと思います。
ViewModelなんて触ったことないので...

# 環境
| なまえ                   | なかみ                                   |
|--------------------------|------------------------------------------|
| Androidバージョン        | 10                                       |
| Android Studioバージョン | 3.5                                      |
| 端末                     | Pixel 3 XL                               |
| 言語                     | Kotlin(なのでfindViewByIdを使いません。) |

# 本題
つくれたらappフォルダに入っているbuild.gradleを開いてマテリアルデザインのライブラリを入れます。

```gradle
implementation 'com.google.android.material:material:1.2.0-alpha01'
```

# メニュー用意
ナビゲーションドロワーに入れるメニューですね。

resディレクトリを右クリックしてフォルダを作成してください。
名前を**menu**にしてね。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/b9133c44-8194-94f1-5589-42a9af9c82b2.png)

作れたら**menu**フォルダの中に**drawer_menu**って名前で作ってね。（別にわかりやすい名前があるならそれでもいいよ。）
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/9808ba04-4b40-3977-f347-2825b549a126.png)

そしたらその**drawer_menu.xml**を開いて適当にメニューを作成します。
が、その前にアイコンを持ってきましょう。

## メニューに表示させるアイコンを用意する

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/3b1bba2d-037c-fce6-5c14-076de0882370.png)

ベクターアセットからお好みの画像を持ってきてください。
今回は適当にドロイド君で
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/663e99a7-dd18-ebac-6278-d89d331e5035.png)

## メニュー項目追加

drawer_menu.xmlに書いてください。
itemを追加するときはtitle(必須)とid(識別に使う)は絶対書いてね。アイコンはいらないかもだけどあったほうがよさげ。

```xml:drawer_menu.xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:android="http://schemas.android.com/apk/res/android">
    <group android:checkableBehavior="single">
        <item android:icon="@drawable/ic_android_black_24dp" android:title="Java" android:id="@+id/drawer_menu_java" />
        <item android:icon="@drawable/ic_android_black_24dp" android:title="Kotlin" android:id="@+id/drawer_menu_kotlin"/>
        <item android:icon="@drawable/ic_android_black_24dp" android:title="JS" android:id="@+id/drawer_menu_js"/>
    </group>
</menu>
```

# レイアウト作成
いよいよレイアウトですよ。レイアウト作るの楽しい。

```xml:activity_main.xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

何もしていなければこのままですね。こっからドロワーを追加していきます。

こう書き換えます。

```xml:activity_main.xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.drawerlayout.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <com.google.android.material.navigation.NavigationView
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:id="@+id/navigation_drawer"
        app:menu="@menu/drawer_menu"
        android:layout_gravity="start"/>

</androidx.drawerlayout.widget.DrawerLayout>
```

できたら実行してみてください。この段階ではまだ何もMainActivity.ktに手を付けてません。
起動出来たら端から左へスワイプしてみてください。メニューが出ると思います。

![Screenshot_20191107-220921.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/6de175c5-27c9-72da-219c-54a99f8edf25.png)

## Android 10でジェスチャーナビゲーション使ってるんですけど開けず戻るになる方へ
左の端っこを長押しすると少しだけナビゲーションドロワーが出てくるのでそのままスワイプすればいいです。

![Screenshot_20191107-220915.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/a6832bae-4805-f378-cf41-f956f2d839d7.png)

ジェスチャーナビゲーションと相性悪い。ケースとかつけてると開くの大変。

# ハンバーガーメニューをつくる
この左上にある三本線のアイコンのことです。
![Screenshot_20191107-225021.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/373a27b9-695b-babc-472a-ee0743d12acc.png)


ハンバーガーメニューはナビゲーションドロワーを開く三本の横線のことです。海を渡るとハンバーガーメニューって呼ばれてるそうです。

これは少し大変なので頑張っていきましょう。

## styles.xmlを書き足す
こんな感じに

```xml:style.xml
<resources>

    <!-- ドロワー用レイアウト -->
    <style name="DrawerTheme" parent="Theme.AppCompat.Light.DarkActionBar">

        <item name="windowActionBar">false</item>
        <item name="windowNoTitle">true</item>

        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>


</resources>
```

元からあるAppThemeは消しません。ナビゲーションドロワーを置いているActivityでのみ使うためです。
Activityを追加したときのためにAppThemeを残してあります。

## Manifest書き換える

AndroidManifest.xmlを開いてね。
開けたら以下の一行を探してください。

```xml:AndroidManifest.xml
android:theme="@style/AppTheme"
```

これをさっき作った**DrawerTheme**へ書き換えます。

```xml:AndroidManifest.xml
android:theme="@style/DrawerTheme"
```

## ツールバーを設置
ツールバーっていうのはアプリ名が書いてある上のバーのことです。
ツールバーはstyles.xmlで消す設定にしたのでレイアウトにツールバーを置く必要があります。

ということで書き換えます。

```xml:activity_main.xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.drawerlayout.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <androidx.appcompat.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:background="?attr/colorPrimary"
            android:minHeight="?attr/actionBarSize"
            android:theme="?attr/actionBarTheme" />
    </LinearLayout>

    <com.google.android.material.navigation.NavigationView
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:id="@+id/navigation_drawer"
        app:menu="@menu/drawer_menu"
        android:layout_gravity="start"/>

</androidx.drawerlayout.widget.DrawerLayout>
```

LinearLayoutの中にToolbarを入れました。

## Toolbarにアイコンを付ける
ようやくKotlinでコーディングです。
ということでMainActivity.ktを開いてください。

```kotlin:MainActivity.kt
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        //Toolbarを登録
        setSupportActionBar(toolbar)

        //ハンバーガーメニューのアイコンを実装
        //第４、５引数はアクセシビリティで使うみたい。
        val actionBarDrawerToggle = ActionBarDrawerToggle(
            this, drawer_layout, toolbar, R.string.app_name, R.string.app_name
        )
        drawer_layout.addDrawerListener(actionBarDrawerToggle)
        actionBarDrawerToggle.syncState()

    }
}
```

ActionBarDrawerToggle()の第４、５引数はアクセシビリティで使うみたいです（ドキュメントによると）。
今回は適当に入れました。

出来たら実行してみてください。
ハンバーガーメニュー押しても開くし、端っこから左へスワイプでも開く＋だんだんアイコンが変化してくると思います。

![Screenshot_20191107-224549.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/59b1491b-7923-1ec6-ba3b-a7b4b0318dfd.png)

# ヘッダーを付ける
ヘッダーを付けるとメニューの上に何かViewを置くことができます。
今回は適当に画像でも置いてみましょう。
![Screenshot_20191107-225843.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/7d06b19b-f2e3-9951-3250-1eaa62883e09.png)


## ヘッダーのレイアウト作成
layoutフォルダに**drawer_header_layout.xml**でレイアウトファイルを作成してください。
画像はプロジェクトに入ってたものを使うことに。

```xml:drawer_header_layout.xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_margin="10sp"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <ImageView
        android:id="@+id/imageView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:srcCompat="@mipmap/ic_launcher_round" />

    <TextView
        android:id="@+id/textView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_vertical"
        android:layout_weight="1"
        android:text="ナビゲーションドロワーの練習です" />
</LinearLayout>
```

## ナビゲーションドロワーにヘッダーを入れる
activity_main.xmlを開いて、NavigationViewにapp:headerLayoutの属性？を付けます。


```xml:activity_main.xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.drawerlayout.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <androidx.appcompat.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:background="?attr/colorPrimary"
            android:minHeight="?attr/actionBarSize"
            android:theme="?attr/actionBarTheme" />
    </LinearLayout>

    <com.google.android.material.navigation.NavigationView
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:id="@+id/navigation_drawer"
        app:menu="@menu/drawer_menu"
        app:headerLayout="@layout/drawer_header_layout"
        android:layout_gravity="start"/>

</androidx.drawerlayout.widget.DrawerLayout>
```

出来たら実行してみてください。
メニューの上に画像が出てくると思います。

![Screenshot_20191107-225843.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/7d06b19b-f2e3-9951-3250-1eaa62883e09.png)

## ヘッダーをステータスバーの下まで潜れないの？
できますよ～
![Screenshot_20191107-230521.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/208826b5-ecd7-75b1-a41e-8f56ef413b4d.png)

activity_main.xmlを開いてください。
変更点はDrawerLayoutにfitsSystemWindows属性がついた点ですね。    android:fitsSystemWindows="true"を書き足すことでステータスバーの下へ侵略できます。

```xml:activity_main.xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.drawerlayout.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/drawer_layout"
    android:fitsSystemWindows="true"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <androidx.appcompat.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:background="?attr/colorPrimary"
            android:minHeight="?attr/actionBarSize"
            android:theme="?attr/actionBarTheme" />
    </LinearLayout>

    <com.google.android.material.navigation.NavigationView
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:id="@+id/navigation_drawer"
        app:menu="@menu/drawer_menu"
        app:headerLayout="@layout/drawer_header_layout"
        android:layout_gravity="start"/>

</androidx.drawerlayout.widget.DrawerLayout>
```

しかしこのままだとステータスバーの色のせいで見えなくなるのでステータスバーの色を透明にする必要があります。
styles.xmlを開いてドロワー用レイアウトへ属性を追加します。
何を追加するのかって話ですがandroid:statusBarColorを追加します。値は```@android:color/transparent```（透明）です。


```xml:styles.xml
    <!-- ドロワー用レイアウト -->
    <style name="DrawerTheme" parent="Theme.AppCompat.Light.DarkActionBar">

        <item name="windowActionBar">false</item>
        <item name="windowNoTitle">true</item>
        <item name="android:statusBarColor">@android:color/transparent</item>

        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>
```

これで起動すればステータスバーの色が透明になって見れるようになります。
![Screenshot_20191107-230521.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/208826b5-ecd7-75b1-a41e-8f56ef413b4d.png)

# メニューを押したときに何かしたい
こんな感じに

```kotlin:MainActivity.kt
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        //Toolbarを登録
        setSupportActionBar(toolbar)

        //ハンバーガーメニューのアイコンを実装
        //第４、５引数はアクセシビリティで使うみたい。
        val actionBarDrawerToggle = ActionBarDrawerToggle(
            this, drawer_layout, toolbar, R.string.app_name, R.string.app_name
        )
        drawer_layout.addDrawerListener(actionBarDrawerToggle)
        actionBarDrawerToggle.syncState()

        //メニューを押したとき
        navigation_drawer.setNavigationItemSelectedListener {
            when(it.itemId){
                R.id.drawer_menu_java->{
                    //java押したとき
                    Snackbar.make(navigation_drawer,"30億のデバイスで走るJava",Snackbar.LENGTH_SHORT).show()
                }
                R.id.drawer_menu_kotlin->{
                    //kotlin押したとき
                    Snackbar.make(navigation_drawer,"apply{}便利すぎる",Snackbar.LENGTH_SHORT).show()
                }
                R.id.drawer_menu_js->{
                    //JS押したとき
                    Snackbar.make(navigation_drawer,"undefined",Snackbar.LENGTH_SHORT).show()
                }
            }
            //押したらナビゲーションドロワーを閉じる
            drawer_layout.closeDrawer(navigation_drawer)
            true
        }
    }
}
```

Javaを押したときSnackbarがでて「30億のデバイスで走るJava」と表示されれば成功です。おめでとう！

# 終わりに
ナビゲーションドロワーのレイアウトを少し変えればばボタンとかおけます。
複雑になりそうだけど。

```xml:activity_main.xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.drawerlayout.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/drawer_layout"
    android:fitsSystemWindows="true"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <androidx.appcompat.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:background="?attr/colorPrimary"
            android:minHeight="?attr/actionBarSize"
            android:theme="?attr/actionBarTheme" />
    </LinearLayout>

    <com.google.android.material.navigation.NavigationView
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:id="@+id/navigation_drawer"
        android:layout_gravity="start">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical">

            <ImageView
                android:id="@+id/imageView2"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                app:srcCompat="@mipmap/ic_launcher_round" />

            <Button
                android:layout_gravity="center_vertical"
                android:id="@+id/button"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="ボタンとかおけます。" />
        </LinearLayout>

    </com.google.android.material.navigation.NavigationView>

</androidx.drawerlayout.widget.DrawerLayout>
```

![Screenshot_20191107-233906.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/421e765a-99cf-59ba-053c-374c35a3716c.png)

それからstyles.xmlのparentをTheme.MaterialComponents.Light.DarkActionBarにするとメニュー選択時が少し変わります。

![Screenshot_20191107-234614.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/a1e21809-93b5-06bb-a8db-d7d9864a58e0.png)


おつ！８８８８８８８８８８８８８８８。
