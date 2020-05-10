---
title: ExoPlayerとMediaSessionを何となく使う
date: 2020-05-10 22:33:50
tags:
- Android
- MediaSession
- ExoPlayer
- Kotlin
---
どうもこんばんわ。  
なんかいきなり暑くなってあつい。  
夏は虫（こわい）と頭痛が痛くなる（水不足か何か）のと弱冷房車に当たる（マジでいらんだろ。スマホの発熱のほうが熱い）のがあるので嫌いです。

# 本題
MediaSessionってのがあるんですよ。JavaScriptの方じゃないです。（JavaScriptの方はこれ↓）

{% asset_img js.png js %}

AndroidのMediaSessionもJSのMediaSessionと多分同じ感じで、
- **GoogleAssistantに今再生してる曲なに？**って聞くと答えが帰ってきたり
- Always On Display に曲名を表示したり
- などなど  

GoogleAssistantで操作できるのもこれ。別にBroadcastReceiverとかで受け取ってるとかではない。

ちなみに音楽プレーヤーの通知にあるあの操作パネルは別にMediaSession無しでも作れる。

## ここまでExoPlayer要素なし
MediaSessionってなんかネット上にも情報がなくてよくわからないんですが、  
ExoPlayerとうまく連携してくれるライブラリがあるので今回はそのライブラリに頼ってMediaSessionを作っていこうと思います。

# 作る
## 環境
| なまえ    | あたい  |
|-----------|---------|
| Android   | 11 DP 4 |
| 言語      | Kotlin  |
| ExoPlayer | 2.11.3  |

## 再生する曲
今回は適当にフリー音源を使います。今回は甘茶の音楽工房様のファミポップⅢを使わせてもらいます。  
別にExoPlayerで再生できれば何でもいいです。

## ライブラリ入れるなど

```gradle
dependencies {
    // MediaSession
    implementation "androidx.media:media:1.1.0"
    // ExoPlayer
    implementation 'com.google.android.exoplayer:exoplayer-core:2.11.3'
    implementation 'com.google.android.exoplayer:extension-mediasession:2.10.4'
    // 省略
}
```

## 音楽入れる
本当は端末内の音楽を再生するのがいいんでしょうけど、Androidのファイル読み書きがややこしいので今回はres/rawに入れて再生することにします。  
ExoPlayerが対応してる再生方法なら何でもいいと思います。

# 仕様など
めんどいのでServiceではなくActivityで作ります。（普通の音楽アプリならServiceで作る。）  
音楽は前述通りres/rawから読み込む形で。

# MainActivity.kt
## ExoPlayer再生するまで
ExoPlayerってバージョン上がると一気に非推奨になったりして追いかけるの大変。
```kotlin
class MainActivity : AppCompatActivity() {

    // ファイル名
    val FILE_NAME = "famipop3"

    lateinit var exoPlayer: SimpleExoPlayer

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // ExoPlayerで再生するまで
        exoPlayer = SimpleExoPlayer.Builder(this).build()
        val dataSourceFactory = DefaultDataSourceFactory(this, "@takusan_23")
        val uri = RawResourceDataSource.buildRawResourceUri(R.raw.famipop3)
        val source = ProgressiveMediaSource.Factory(dataSourceFactory).createMediaSource(uri)
        exoPlayer.prepare(source)

        // 再生
        play_button.setOnClickListener {
            exoPlayer.playWhenReady = !exoPlayer.playWhenReady
        }
    }

    override fun onDestroy() {
        super.onDestroy()
        exoPlayer.release()
    }

}
```

ファイル名のところは各自書き換えてね。

## MediaSession連携
めんどいので全部張ります
```kotlin
class MainActivity : AppCompatActivity() {
    
    lateinit var exoPlayer: SimpleExoPlayer

    // MediaSession
    lateinit var mediaSession: MediaSessionCompat
    lateinit var mediaSessionConnector: MediaSessionConnector

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // ExoPlayerで再生するまで
        exoPlayer = SimpleExoPlayer.Builder(this).build()
        val dataSourceFactory = DefaultDataSourceFactory(this, "@takusan_23")
        val uri = RawResourceDataSource.buildRawResourceUri(R.raw.famipop3)
        val source = ProgressiveMediaSource.Factory(dataSourceFactory).createMediaSource(uri)
        exoPlayer.prepare(source)

        // mp3から再生時間だけ取る。
        val mediaMetadataRetriever = MediaMetadataRetriever()
        val afd = resources.openRawResourceFd(R.raw.famipop3)
        mediaMetadataRetriever.setDataSource(afd.fileDescriptor, afd.startOffset, afd.length)
        val duration = mediaMetadataRetriever.extractMetadata(MediaMetadataRetriever.METADATA_KEY_DURATION).toLong()

        // MediaSession連携
        mediaSession = MediaSessionCompat(this, "sample").apply {
            isActive = true
        }
        mediaSessionConnector = MediaSessionConnector(mediaSession)
        mediaSessionConnector.setPlayer(exoPlayer)
        // メタデータ
        mediaSessionConnector.setMediaMetadataProvider {
            // メタデータ
            val mediaMetadataCompat = MediaMetadataCompat.Builder().apply {
                putString(MediaMetadataCompat.METADATA_KEY_TITLE, "ファミポップⅢ")
                putString(MediaMetadataCompat.METADATA_KEY_MEDIA_ID, "famipop")
                putString(MediaMetadataCompat.METADATA_KEY_DISPLAY_TITLE, "ファミポップⅢ")
                putString(MediaMetadataCompat.METADATA_KEY_DISPLAY_SUBTITLE, "甘茶の音楽工房")
                putString(MediaMetadataCompat.METADATA_KEY_ARTIST, "甘茶の音楽工房")
                putLong(MediaMetadataCompat.METADATA_KEY_DURATION, duration) // これあるとAndroid 10でシーク使えます
            }.build()
            mediaMetadataCompat
        }

        // 再生
        play_button.setOnClickListener {
            exoPlayer.playWhenReady = !exoPlayer.playWhenReady
            showNotification()
        }

    }

    @RequiresApi(Build.VERSION_CODES.O)
    private fun showNotification() {
        val channelId = "play_notification"
        val notificationManager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
        if (notificationManager.getNotificationChannel(channelId) == null) {
            val notificationChannel = NotificationChannel(channelId, "音楽通知", NotificationManager.IMPORTANCE_LOW)
            notificationManager.createNotificationChannel(notificationChannel)
        }
        val notification = NotificationCompat.Builder(this, channelId).apply {
            // MediaStyle。コントローラー
            setContentTitle("ファミポップⅢ")
            setContentText("甘茶の音楽工房")
            setSmallIcon(R.drawable.ic_audiotrack_black_24dp)
            setStyle(androidx.media.app.NotificationCompat.MediaStyle().setMediaSession(mediaSession.sessionToken))
            addAction(R.drawable.ic_audiotrack_black_24dp, "", PendingIntent.getBroadcast(this@MainActivity, 1, Intent(), PendingIntent.FLAG_UPDATE_CURRENT))
        }.build()
        notificationManager.notify(1, notification)
    }

    override fun onDestroy() {
        super.onDestroy()
        // 終了処理
        exoPlayer.release()
        mediaSession.release()
    }

}
```

## 注意など
- 上記のコードは`addAction()`でアイコン出してるけどIntentが空なので押してもなにもなりません。
- 通知の`setStyle()`に入れる`NotificationCompat`は`androidx.media.app.NotificationCompat`です。`androidx.core.app.NotificationCompat`ではないです（名前同じなのややこC）
- Android 10から通知の音楽コントローラーにシークバーが追加できるんですが（`MediaMetadataCompat.METADATA_KEY_DURATION`に負の値を指定しなければいい）これ一つ以上`addAction()`を追加しないとシークバーがいつまで経っても表示されません。  
- `mediaSessionConnector.setMediaMetadataProvider{}`でメタデータを別に作成してますが、多分mp3の中にメタデータがあれば勝手に作ってくれると思います（要検証）
- メタデータハードコートしてるけど`MediaMetadataRetriever`からタイトルやら作者を取るほうが良いです。（かくのめんどい）

# おわりに
シークバー出すのに`addAction()`で一個以上アイコンを出さないとだめってことに気付かずに時間が溶けたのでもう疲れた。

{% asset_img assistant.png assistant %}

写真取れなかったけどAlways On Displayでも(Android端末差はあるだろうけど)表示されてます。

# 参考にしました。
https://stackoverflow.com/questions/24030756/mediaextractor-mediametadataretriever-with-raw-asset-file
https://stackoverflow.com/questions/30852975/exoplayer-reading-mp3-file-from-raw-folder