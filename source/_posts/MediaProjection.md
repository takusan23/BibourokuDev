---
title: 【Android / MediaProjection / Kotlin】Androidで画面録画をする
tags:
- Android
- MediaProjection
- 画面録画
- AndroidQ
- Kotlin
author: takusan_23
slide: false
---
どうもこんばんは。テスト期間つらい。
今回は画面を録画できるアプリを作ってみようと思います。

## 画面録画の歴史？
ここではRoot化で利用できる画面録画はなしの方向で。Root取れば内部音声が取れるって聞いたことある。
あと外部出力端子を利用して録画する話もなしの方向で。
私が知ってる限りでは

Xperia Z3 / Z3 Compact でスクリーンレコーダーが搭載された（4.4）。
（ですがいつの間にかスクリーンレコーダー機能はなくなったそうです。（Xperia 1で復活したらしい））
（あとスモールアプリとかどこ行ったの？）
↓
5.0で「MediaProjection API」が追加されてサードパーティアプリでも録画ができるようになった。
Google Play ゲームで画面録画機能が追加されたりした。
しかしこの手の画面録画アプリは音声をマイクから撮っているので端末の音以外が入ってしまう。
↓
（Samsungとかが内部音声が取れるアプリを独自で追加しているらしい？）
↓
10.0で端末の内部音声を取れる「AudioPlaybackCapture API」が追加された。
今回は触れません。というか実装方法がわからなかった。残念。😥

## 端末紹介
|                   |            |
|-------------------|------------|
| 端末              | Pixel 3 XL |
| Androidバージョン | 10         |
| compileSdkVersion | 29         |

## 作るもの
画面録画してmp4で保存する。
音声はマイクから取ることにする。


# 実装
本題です。ここまで長かったね。
## Android 10でも動くように
保存先を対象範囲別ストレージにする。
ServiceにAndroid 10から追加された属性「foregroundServiceType」を書き足す。
## 流れ

権限があるか確認する（マイクの録音）
↓
画面録画していいか聞く。
↓
許可が得られたらServiceを起動する。
↓
Serviceの中で画面録画を行う。

Serviceが終了すると同時に録画も終了させる。

## レイアウト
ActivityはMainActivityだけです。

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
        android:id="@+id/rec_button"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="録画" />

    <Button
        android:id="@+id/stop_button"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="停止" />
</LinearLayout>
```

## Manifest
必要な権限は

| 権限               | 説明                 |
|--------------------|----------------------|
| FOREGROUND_SERVICE | サービスの実行に必要 |
| RECORD_AUDIO       | マイクを使うのに必要 |

この段階では「ScreenRecordService」は作成してないので赤くなるけど後で作るのでおｋ。
**android:foregroundServiceType="mediaProjection"**を忘れずに

```xml:AndroidManifest.xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="io.github.takusan23.screenrecordsample">

    <uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
    <uses-permission android:name="android.permission.RECORD_AUDIO"/>

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <service android:name=".ScreenRecordService" android:foregroundServiceType="mediaProjection"/>
    </application>

</manifest>
```

## 権限をもらう
録画のボタンを押したらまずマイクへのアクセス権があるか確認します。
ない場合は権限をリクエストするダイアログを表示させます。

権限がある場合は録画を開始するダイアログを表示させます。

```kotlin:MainActivity.kt
    //リクエストの結果
    val code = 512
    val permissionCode = 810
    lateinit var projectionManager: MediaProjectionManager

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        projectionManager =
            getSystemService(Context.MEDIA_PROJECTION_SERVICE) as MediaProjectionManager

        //画面録画をしてもいいか聞く
        rec_button.setOnClickListener {
            //その前にマイクへアクセスしていいか尋ねる
            if (ContextCompat.checkSelfPermission(this, Manifest.permission.RECORD_AUDIO)
                != PackageManager.PERMISSION_GRANTED
            ) {
                requestPermissions(arrayOf(Manifest.permission.RECORD_AUDIO), permissionCode)
            } else {
                //マイクの権限があるので画面録画リクエスト
                //ダイアログを出す
                startActivityForResult(projectionManager.createScreenCaptureIntent(), code)
            }
        }
        //停止ボタンで止められるように
        stop_button.setOnClickListener {
            val intent = Intent(this, ScreenRecordService::class.java)
            stopService(intent)
        }
    }

    //権限の結果受け取る
    override fun onRequestPermissionsResult(
        requestCode: Int,
        permissions: Array<out String>,
        grantResults: IntArray
    ) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults)
        if (requestCode == permissionCode) {
            //マイクへアクセス権げっと
            Toast.makeText(this, "権限が付与されました。", Toast.LENGTH_SHORT).show()
        }
    }
```

## 画面録画をするServiceを起動させる
ところでAndroidの画面録画をするコードとかをいろいろ参考にしたわけですが、Android 10で追加された属性「foregroundServiceType」のせいかActivityでは画面録画が出来ないっぽいんですよね。
なので仕方なくServiceで。まあServiceで実装したほうが広く使えそうだもんね。

MainActivity.ktに「onActivityResult」を追加します。

```kotlin:MainActivity.kt
    //画面録画の合否を受け取る
    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        super.onActivityResult(requestCode, resultCode, data)
        //成功＋結果が画面録画の物か
        if (resultCode == Activity.RESULT_OK && requestCode == code) {
            //Service起動
            //Manifestに「android:foregroundServiceType="mediaProjection"」を付け足しておく
            val intent = Intent(this, ScreenRecordService::class.java)
            intent.putExtra("code", resultCode) //必要なのは結果。startActivityForResultのrequestCodeではない。
            intent.putExtra("data", data)
            //画面の大きさも一緒に入れる
            val metrics = resources.displayMetrics;
            intent.putExtra("height", metrics.heightPixels)
            intent.putExtra("width", metrics.widthPixels)
            intent.putExtra("dpi", metrics.densityDpi)

            startForegroundService(intent)
        }
    }
```
## Service作成
それから「ScreenRecordService.kt」を作成して下さい。
作成したら以下のように。
setSmallIconが赤くなると思います。これはベクターアセットから追加したためです。
なので画像のようにして見つけて下さい。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/f504e4f1-57ae-e459-8f51-eb96af80bfea.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/ebe8ef29-bd69-3afc-6aec-100a92f9322f.png)

```kotlin:ScreenRecordService.kt
class ScreenRecordService : Service() {
    //Intentに詰めたデータを受け取る
    var data: Intent? = null
    var code = Activity.RESULT_OK

    //画面録画で使う
    lateinit var mediaRecorder: MediaRecorder
    lateinit var projectionManager: MediaProjectionManager
    lateinit var projection: MediaProjection
    lateinit var virtualDisplay: VirtualDisplay

    //画面の大きさ
    //Pixel 3 XLだとなんかおかしくなる
    var height = 2800
    var width = 1400
    var dpi = 1000


    override fun onBind(p0: Intent?): IBinder? {
        return null
    }

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {

        //データ受け取る
        data = intent?.getParcelableExtra("data")
        code = intent?.getIntExtra("code", Activity.RESULT_OK) ?: Activity.RESULT_OK

        //画面の大きさ
        //   height = intent?.getIntExtra("height", 1000) ?: 1000
        //   width = intent?.getIntExtra("width", 1000) ?: 1000
        dpi = intent?.getIntExtra("dpi", 1000) ?: 1000

        //通知を出す。
        val notificationManager =
            getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
        //通知チャンネル
        val channelID = "rec_notify"
        //通知チャンネルが存在しないときは登録する
        if (notificationManager.getNotificationChannel(channelID) == null) {
            val channel =
                NotificationChannel(channelID, "録画サービス起動中通知", NotificationManager.IMPORTANCE_HIGH)
            notificationManager.createNotificationChannel(channel)
        }
        //通知作成
        val notification = Notification.Builder(applicationContext, channelID)
            .setContentText("録画中です。")
            .setContentTitle("画面録画")
            .setSmallIcon(R.drawable.ic_cast_black_24dp)    //アイコンはベクターアセットから
            .build()

        startForeground(1, notification)

        //録画開始
        startRec()

        return START_NOT_STICKY
    }

    //Service終了と同時に録画終了
    override fun onDestroy() {
        super.onDestroy()
        stopRec()
    }

}
```

## 画面録画
ScreenRecordService.ktに付け足してください。
ここはよくわからんのであまり言えませんが、「setVideoEncodingBitRate」を高くすれば画質アップ＆ファイルサイズアップです。
ファイル名は数字.mp4になります。```${System.currentTimeMillis()}.mp4```の部分がファイル名になっているので変えたい場合はどうぞ。
保存先は**Android/data/io.github.takusan23.screenrecordsample/file**の中です。


```kotlin:ScreenRecordService.kt
    //録画開始
    fun startRec() {
        if (data != null) {
            projectionManager =
                getSystemService(Context.MEDIA_PROJECTION_SERVICE) as MediaProjectionManager
            //codeはActivity.RESULT_OKとかが入る。
            projection =
                projectionManager.getMediaProjection(code, data!!)

            mediaRecorder = MediaRecorder()
            mediaRecorder.setAudioSource(MediaRecorder.AudioSource.MIC)
            mediaRecorder.setVideoSource(MediaRecorder.VideoSource.SURFACE)
            mediaRecorder.setOutputFormat(MediaRecorder.OutputFormat.MPEG_4)
            mediaRecorder.setVideoEncoder(MediaRecorder.VideoEncoder.H264)
            mediaRecorder.setAudioEncoder(MediaRecorder.AudioEncoder.AMR_NB)
            mediaRecorder.setVideoEncodingBitRate(1080 * 10000) //1080は512ぐらいにしといたほうが小さくできる
            mediaRecorder.setVideoFrameRate(30)
            mediaRecorder.setVideoSize(width, height)
            mediaRecorder.setAudioSamplingRate(44100)
            mediaRecorder.setOutputFile(getFilePath())
            mediaRecorder.prepare()

            virtualDisplay = projection.createVirtualDisplay(
                "recode",
                width,
                height,
                dpi,
                DisplayManager.VIRTUAL_DISPLAY_FLAG_AUTO_MIRROR,
                mediaRecorder.surface,
                null,
                null
            )

            //開始
            mediaRecorder.start()
        }
    }

    //録画止める
    fun stopRec() {
        mediaRecorder.stop()
        mediaRecorder.release()
        virtualDisplay.release()
        projection.stop()
    }

    //保存先取得。今回は対象範囲別ストレージに保存する
    fun getFilePath(): File {
        //ScopedStorageで作られるサンドボックスへのぱす
        val scopedStoragePath = getExternalFilesDir(null)
        //写真ファイル作成
        val file = File("${scopedStoragePath?.path}/${System.currentTimeMillis()}.mp4")
        return file
    }
```

## 完成！
開始ボタン押した。
![Screenshot_20191010-200940.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/8942a6c6-0950-3c74-128a-a0fed4934bab.png)

「今すぐ開始」を押した。
ステータスバーのバイブアイコンの隣にキャスト中のアイコンが出てますね。
![Screenshot_20191010-200946.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/174fed18-0694-8111-6287-94f26c3bfd7b.png)

停止ボタンを押して作成した動画ファイルを見た
![Screenshot_20191010-201017.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/409918/62653173-7209-4a74-9ff6-63f8eb695d83.png)

## Pixel 3 XL で遭遇した問題
### 動画の大きさがおかしい問題
よくわからないので以下のように書き換えておきました。
本来はこの値を固定するべきでは無いと思いますがPixel 3 XLでしか動かさないので別にいいや。

```kotlin:ScreenRecordService.kt
    //画面の大きさ
    //Pixel 3 XLだとなんかおかしくなる
    var height = 2800
    var width = 1400
    var dpi = 1000

   //以下省略

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {

        //データ受け取る
        data = intent?.getParcelableExtra("data")
        code = intent?.getIntExtra("code", Activity.RESULT_OK) ?: Activity.RESULT_OK

        //画面の大きさ
        //   height = intent?.getIntExtra("height", 1000) ?: 1000
        //   width = intent?.getIntExtra("width", 1000) ?: 1000
        dpi = intent?.getIntExtra("dpi", 1000) ?: 1000

       //以下省略
```

## 内部音声は取れないの？
Android 10から「AudioPlaybackCapture API」が追加されましたが
内部音声はMediaRecorderではなくAudioRecordを使わないと録音できない。残念。
AudioRecordの保存とかどうやるんだろう。
きっとそのうち出てくると思うのでしばらく待つことにする。

## 終わりに
Rootなしで内部音声が取れるのは期待ですね。
この記事を書いてるときに思い出しましたが、Android Q Beta 3ぐらいのときAndroidに標準で画面録画が搭載されそうになった（開発者オプションの機能フラグで有効にしないといけない）。
Beta版の段階では電源ボタン長押しで表示されるスクリーンショットを取るボタンを長押しすれば利用できたそう。（残念ながら私の環境では落ちて使えなかった。）
けど正式版では実装されなかった。ぜひ標準で欲しい機能なんだけどなあ。

それと
Pixel 4　と　docomo の発表会が楽しみですね！

書きたいことは多分全部書いたので、
以上です。おつ。８８８８８８８８８８８８８８８８８８

## GitHub
https://github.com/takusan23/ScreenRecordSample
試してみたい方どうぞ～
