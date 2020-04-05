---
title: 【Android】 台風に備えて気圧を測れるアプリを作る
tags: 
- Android
- 台風
- Kotlin
author: takusan_23
slide: false
---
こんにちは。中間テスト終わりました。
今回はやばい台風が来るそうなので気圧が測れるアプリを作りたいと思います。

## スマホにはいろんなセンサーがついている
せっかくなので使いましょう。
というのは建前で本当は [この記事](https://qiita.com/bellx2/items/fc1de7197f583001ca59) でやってることAndroidでも作れるのではと思ったのが本当の理由。

## 紹介
|                    |            |
|--------------------|------------|
| 端末               | Pixel 3 XL |
| Android バージョン | Android 10 |
| 最低バージョン     | ろりぽっぷ |
| 言語               | Kotlin     |

## 実装
### レイアウト
真ん中にTextViewを置いただけのものとする。
面倒なのでConstraintLayoutをそのまま使うことにする。

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
        android:textSize="50dp"
        android:id="@+id/barometer_tv"
        android:text="Hello World!"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

### コード

Sensor.TYPE_PRESSUREが気圧計になります。
onDestroy()で登録を解除しています。
受け取るセンサーがほかにもあるときは```event?.sensor?.type```を利用して分けましょう。


```kotlin:MainActivity.kt
    lateinit var sensorManager: SensorManager
    lateinit var sensorEventListener: SensorEventListener
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        sensorManager = getSystemService(Context.SENSOR_SERVICE) as SensorManager
        //今回は気圧計
        val sensorList = sensorManager.getSensorList(Sensor.TYPE_PRESSURE)
        //受け取る
        sensorEventListener = object : SensorEventListener {
            override fun onAccuracyChanged(sensor: Sensor?, accuracy: Int) {
                //つかわん
            }

            override fun onSensorChanged(event: SensorEvent?) {
                //値はここで受けとる
                //今回は気圧計のみだからいいけどほかにも登録するときは分岐してね
                if(event?.sensor?.type == Sensor.TYPE_PRESSURE){
                    //気圧計の値
                    val barometer = event.values[0]
                    //TextViewに設定
                    barometer_tv.text = "$barometer hPa"
                }
            }
        }
        //登録
        sensorManager.registerListener(
            sensorEventListener,
            sensorList[0],  //配列のいっこめ。気圧計
            SensorManager.SENSOR_DELAY_NORMAL  //更新頻度
         )
    }

    override fun onDestroy() {
        super.onDestroy()
        sensorManager.unregisterListener(sensorEventListener)
    }
```

これでTextViewに気圧が表示されてると思います。

![Screenshot_20191011-112836.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/0afb2d9f-c99b-dafd-1086-a8ae4c51dee9.png)

### 小数点を消す
```roundToInt()``` できれいになります。


```kotlin
                //値はここで受けとる
                //今回は気圧計のみだからいいけどほかにも登録するときは分岐してね
                if(event?.sensor?.type == Sensor.TYPE_PRESSURE){
                    //気圧計の値
                    val barometer = event.values[0]
                    //整数に
                    val barometerInt = barometer.roundToInt()
                    //TextViewに設定
                    barometer_tv.text = "$barometerInt hPa\n($barometer hPa)"
                }

```

![Screenshot_20191011-113656.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/de5824b5-5099-f8ba-f406-691f77d36027.png)

以上です。
お疲れ様です。８８８８８
