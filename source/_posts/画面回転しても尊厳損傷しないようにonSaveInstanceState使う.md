---
title: 画面回転しても尊厳損傷しないようにonSaveInstanceState使う
date: 2020-05-17 00:32:19
tags:
- Android
- Kotlin
---
あなたのインナーチャイルド、今日も元気？  
カツドンチャンネルで好きな動画置いときますね  
<iframe width="312" height="176" src="https://ext.nicovideo.jp/thumb/sm27998318" scrolling="no" style="border:solid 1px #ccc;" frameborder="0"><a href="https://www.nicovideo.jp/watch/sm27998318">キレて食器を破壊したのでみなさん見て下さい</a></iframe>
<iframe width="312" height="176" src="https://ext.nicovideo.jp/thumb/sm30369283" scrolling="no" style="border:solid 1px #ccc;" frameborder="0"><a href="https://www.nicovideo.jp/watch/sm30369283">パワー系ガイジのムーブを披露しながら歌うサイコマルマイン</a></iframe>

# 本題
画面回転する→onSaveInstanceState()→onDestroy()→onCreate()という感じ（超省略）でふりだしに戻ります。

# つくる
## ScreenRotationFragment.kt / fragment_screen_roration.xml
今回はFragmentでの値の引き継ぎですのでまずFragment作ります

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center"
    android:orientation="vertical">

    <EditText
        android:id="@+id/editText"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:ems="10"
        android:inputType="textPersonName"
        android:text="Name" />
</LinearLayout>
```

```kotlin
class ScreenRotationFragment : Fragment() {

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View? {
        return inflater.inflate(R.layout.fragment_screen_roration, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        // 画面回転のときは savedInstanceState が null 以外になる
        if (savedInstanceState != null) {
            editText.setText(savedInstanceState.getString("text"))
        }
    }

    override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)
        outState.apply {
            putString("text", editText.text.toString())
        }
    }

}
```

## MainActivity.kt / activity_main.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_main_fragment"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal"
    tools:context=".MainActivity">

</LinearLayout>
```
```kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // 画面回転してもFragmentが残るらしい
        val fragment = if (supportFragmentManager.findFragmentByTag("rotation_fragment") != null) {
            supportFragmentManager.findFragmentByTag("rotation_fragment") as ScreenRotationFragment
        } else {
            ScreenRotationFragment()
        }
        supportFragmentManager
            .beginTransaction()
            .replace(R.id.activity_main_fragment, fragment, "rotation_fragment") // 第三引数にタグ入れる
            .commit()

    }
}
```

これで実行してEditTextにテキスト入れて画面回転してもなんと！値が復元されてるんですね～

{%asset_img port.png port%}

{%asset_img land.png lanf%}

# データクラスのときは？
複数の値をまとめたデータクラスのときはどうやって引き継ぐのかって話ですが`Serializable`ってのを使うそうですよ？

## データクラスつくる
### TestData.kt
```kotlin
data class TestData(
    val text: String
) : Serializable
```
`Serializable`ってのを使います

## Fragment書き換え
```kotlin
class ScreenRotationFragment : Fragment() {

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View? {
        return inflater.inflate(R.layout.fragment_screen_roration, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        // 画面回転のときは savedInstanceState が null 以外になる
        if (savedInstanceState != null) {
            // データクラスもらう
            val testData = savedInstanceState.getSerializable("data") as TestData
            editText.setText(testData.text)
        }
    }

    override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)
        // 画面回転時に保存するときはここに書く
        outState.apply {
            val testData = TestData(editText.text.toString())
            putSerializable("data", testData)
        }
    }

}
```

これでも値が復元されるはずです。

{%asset_img data_port.png port%}

{%asset_img data_land.png lanf%}

# 終わりに
`Serializable`ともう一個、`Parcelable`ってのが有るんだけどなんか実装がだるい（IDEがやってくれるけど）ので今回は見送った。  
別に今回のような使い方なら`Serializable`で十分だと思う。