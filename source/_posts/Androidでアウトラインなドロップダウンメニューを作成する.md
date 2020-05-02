---
title: Androidでアウトラインなドロップダウンメニューを作成する
date: 2020-04-28 17:14:43
tags:
- Android
- MaterialDesign
- Kotlin
---

どうもこんばんわ。

# 本題
[マテリアルデザインのライブラリ](https://github.com/material-components/material-components-android)でOutlineなDropDownMenu（Spinner）ないと思ってたけどあったので紹介。

# 環境
|なまえ|あたい|
|---|---|
|Android|10|
|言語|Kotlin|

# ライブラリ入れる
appフォルダの方にあるbuild.gradleを開いて`dependencies{}`の中に以下のコードを追加です。
```gradle
// Material Design
implementation 'com.google.android.material:material:1.2.0-alpha06'
```

# style.xmlを書き換える
Material Designのコンポーネント（UIの部品）を使うにはstyle.xmlを書き換えて`Theme.MaterialComponents`系をparentに指定する必要があるみたい。
## style.xml
```xml
<resources>

    <!-- Base application theme. -->
    <style name="AppTheme" parent="Theme.MaterialComponents.Light.DarkActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>

</resources>
```

# レイアウト
[説明によると](https://material.io/develop/android/components/menu/)`TextInputLayout`の中に`AutoCompleteTextView`を入れることで実現されているらしい。

## activity_main.xml
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

    <com.google.android.material.textfield.TextInputLayout
        style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox.Dense.ExposedDropdownMenu"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Outlineなドロップダウンメニュー">

        <AutoCompleteTextView
            android:id="@+id/auto_complete_textview"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:clickable="false"
            android:cursorVisible="false"
            android:focusable="false"
            android:inputType="none" />
    </com.google.android.material.textfield.TextInputLayout>

</LinearLayout>
```

### 重要な点
`AutoCompleteTextView`はテキストを補充するEditTextなため、ただ置くだけだとテキスト変更ができるどころかIMEが表示されます。  
その対策に以下の値を設定しておく必要があります。（例ではすでに指定済みです。）

```xml
android:clickable="false"
android:cursorVisible="false"
android:focusable="false"
android:inputType="none" 
```

なんか[説明](https://material.io/develop/android/components/menu/)見る限り`android:inputType="none"`を指定すればユーザーの入力を無効にできるって書いてあるけど、そんなことなかったよ。

# Kotlin
## MainActivity.kt
```kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // Adapter作成
        val menuList = arrayListOf("Java", "Kotlin", "JavaScript", "TypeScript")
        val adapter = ArrayAdapter(this, android.R.layout.simple_list_item_1, menuList)
        // Adapter登録
        auto_complete_textview.setAdapter(adapter)
        auto_complete_textview.setText(menuList[0], false)
    }
}
```

これだけです。  

{% asset_img menu.png outline %}

# さいごに
押したときにToastを出してみる
## MainActivity.kt
```kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // Adapter作成
        val menuList = arrayListOf("Java", "Kotlin", "JavaScript", "TypeScript")
        val adapter = ArrayAdapter(this, android.R.layout.simple_list_item_1, menuList)
        // Adapter登録
        auto_complete_textview.setAdapter(adapter)
        auto_complete_textview.setText(menuList[0], false)
        // テキスト変更を検知
        auto_complete_textview.addTextChangedListener {
            Toast.makeText(this, it.toString(), Toast.LENGTH_SHORT).show()
        }
    }
}
```

{% asset_img toast.png toast %}


# 参考にしました
https://stackoverflow.com/questions/41829665/android-studioedittext-editable-is-deprecated-how-to-use-inputtype/46073055

https://material.io/develop/android/components/menu/