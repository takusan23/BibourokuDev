---
title: ExoPlayerにオフライン再生機能あるらしいので使ってみる
date: 2020-05-09 20:21:48
tags:
- Android
- ExoPlayer
- HLS
- Kotlin
---
どうもこんばんわ。  

# 本題
ExoPlayerにオフライン再生用のメディアをダウンロードする機能があるらしい。  
調べても全然出てこなかったのでサンプル置いておきますね。

# オフライン再生用のファイルは
よくわからんファイルだった。mp4とかではなかった。  
ファイルダウンローダーでは無いってことだね。

# 参考にします
https://exoplayer.dev/downloading-media.html

# 環境
|なまえ|あたい|
|---|---|
|Android|11 DP 4|
|言語|Kotlin|

# つくる

## 必要なもの
mp4かhlsのアドレス。動画再生するので仕方ないね。

## ExoPlayer導入
appフォルダの方の`build.gradle`を開いて追加
```gradle
dependencies {
    // ExoPlayer
    implementation 'com.google.android.exoplayer:exoplayer-core:2.11.3'
    implementation 'com.google.android.exoplayer:exoplayer-hls:2.11.3'
    // 省略
}
```

## AndroidManifest.xml
サービス実行権限とインターネットアクセス権限が必要です。
```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
```

# サービス作成

## AndroidManifest.xml
```xml
<service
    android:name=".DownloadServiceTest"
    android:exported="false">
    <!-- This is needed for Scheduler -->
    <intent-filter>
        <action android:name="com.google.android.exoplayer.downloadService.action.RESTART" />
        <category android:name="android.intent.category.DEFAULT" />
    </intent-filter>
</service>
```

## DownloadServiceTest.kt
詳しくないのでなにも言えない。  
ついでに最低Androidバージョン間違えちゃったので、`@RequiresApi(Build.VERSION_CODES.O)`が入っちゃった。ごめんね。  
あとAndroid 8以降で動くように勝手に`startForeground`呼んでるけどあってるのかな？

```kotlin
class DownloadServiceTest : DownloadService(FOREGROUND_NOTIFICATION_ID_NONE) {

    // 通知出すらしい
    val NOTIFICATION_CHANNEL = "cache_notification"
    val SERVICE_NOTIFICATION_CHANNEL = "service_notification"

    // ?
    val JOB_ID = 4545

    /**
     * Android 8からService使うには通知出さないといけなくなった。
     * */
    @RequiresApi(Build.VERSION_CODES.O)
    override fun onCreate() {
        super.onCreate()
        val notificationManager =
            this.getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
        if (notificationManager.getNotificationChannel(SERVICE_NOTIFICATION_CHANNEL) == null) {
            val notificationChannel =
                NotificationChannel(SERVICE_NOTIFICATION_CHANNEL, "ExoPlayerでダウンロードテスト通知", NotificationManager.IMPORTANCE_LOW)
            notificationManager.createNotificationChannel(notificationChannel)
        }
        val notification = Notification.Builder(this, SERVICE_NOTIFICATION_CHANNEL).apply {
            setContentTitle("ExoPlayerでダウンロードテスト")
            setContentText("サービスが実行中です")
            setSmallIcon(R.drawable.ic_file_download_black_24dp)
        }
        startForeground(1, notification.build())
    }

    override fun getDownloadManager(): DownloadManager {
        // Note: This should be a singleton in your app.
        val databaseProvider = ExoDatabaseProvider(this)
        // A download cache should not evict media, so should use a NoopCacheEvictor.
        val downloadCache = SimpleCache(
            File("${this.getExternalFilesDir(null)?.path}/cache"),
            NoOpCacheEvictor(),
            databaseProvider
        )
        // Create a factory for reading the data from the network.
        val dataSourceFactory =
            DefaultHttpDataSourceFactory("@takusan_23")
        // Create the download manager.
        val downloadManager = DownloadManager(
            this,
            databaseProvider,
            downloadCache,
            dataSourceFactory
        )
        downloadManager.addListener(object : DownloadManager.Listener {
            override fun onIdle(downloadManager: DownloadManager) {
                super.onIdle(downloadManager)
                println("終了？")
            }
        })
        return downloadManager
    }

    @RequiresApi(Build.VERSION_CODES.O)
    override fun getForegroundNotification(downloads: MutableList<Download>): Notification {
        val notificationManager =
            this.getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
        if (notificationManager.getNotificationChannel(NOTIFICATION_CHANNEL) == null) {
            val notificationChannel =
                NotificationChannel(NOTIFICATION_CHANNEL, "オフライン再生準備", NotificationManager.IMPORTANCE_LOW)
            notificationManager.createNotificationChannel(notificationChannel)
        }
        val notification = Notification.Builder(this, NOTIFICATION_CHANNEL).apply {
            setContentTitle("オフライン再生準備")
            setContentText(downloads.size.toString())
            setSmallIcon(R.drawable.ic_file_download_black_24dp)
        }
        return notification.build()
    }

    override fun getScheduler(): Scheduler? {
        return PlatformScheduler(this, JOB_ID)
    }
}
```

コピペで使えると思います。

# Activity

## レイアウト
皆さんはちゃんとIDに名前つけてあげてね
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

    <SurfaceView
        android:id="@+id/surfaceView"
        android:layout_width="160dp"
        android:layout_height="90dp" />

    <Button
        android:id="@+id/button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="サービス起動" />

    <Button
        android:id="@+id/button2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="取得" />

    <Button
        android:id="@+id/button3"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="再生" />

    <Button
        android:id="@+id/button4"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="一時停止・再生" />
</LinearLayout>
```

## MainActivity.kt
`CONTENT_URL`の中は各自インターネット上にあるDLしてもいい動画を指定しておいてください。
```kotlin
class MainActivity : AppCompatActivity() {

    lateinit var exoPlayer: SimpleExoPlayer

    // ネットにある動画URL指定しといて
    val CONTENT_URL = ""

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        exoPlayer = SimpleExoPlayer.Builder(this).build()

        button.setOnClickListener {
            // サービス起動
            val intent = Intent(this, DownloadServiceTest::class.java)
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
                startForegroundService(intent)
            } else {
                startService(intent)
            }
        }

        button2.setOnClickListener {
            cache()
        }

        button3.setOnClickListener {
            playCache()
        }

        button4.setOnClickListener {
            if (::exoPlayer.isInitialized) {
                exoPlayer.playWhenReady = !exoPlayer.playWhenReady
            }
        }

    }

    private fun cache() {
        val downloadRequest = DownloadRequest(
            "cache",
            DownloadRequest.TYPE_PROGRESSIVE,
            CONTENT_URL.toUri(),  /* streamKeys= */
            Collections.emptyList(),  /* customCacheKey= */
            null,
            ByteArray(1024)
        )
        DownloadService.sendAddDownload(this, DownloadServiceTest::class.java, downloadRequest, false)
    }

    private fun playCache() {
        val cache =
            SimpleCache(File("${this.getExternalFilesDir(null)?.path}/cache"), LeastRecentlyUsedCacheEvictor(1024), ExoDatabaseProvider(this))
        val upstreamDataSourceFactory = DefaultDataSourceFactory(this, "@takusan_23")
        val dataSourceFactory = CacheDataSourceFactory(
            cache, upstreamDataSourceFactory
        )
        val mediaSource =
            ProgressiveMediaSource.Factory(dataSourceFactory)
                .createMediaSource(CONTENT_URL.toUri())
        exoPlayer.prepare(mediaSource)
        exoPlayer.setVideoSurfaceView(surfaceView)
        exoPlayer.playWhenReady = true
    }
    
    override fun onDestroy() {
        super.onDestroy()
        if (::exoPlayer.isInitialized) {
            exoPlayer.release()
        }
    }

}
```

これで  
**サービス起動**押して  
**取得**を押すと取得を始めます。`cache()`関数  
取得後**アプリを再起動**すると**再生**を押して再生ができます。`playCache()`関数  
再起動する前に再生するとなんか落ちます。謎

# HLSに対応する
HLS版`cache()`、`playCache()`を作りました。  
参考：https://exoplayer.dev/downloading-media.html

```kotlin
private fun cacheHLS() {
    val sourceFactory = DefaultDataSourceFactory(
        this,
        "@takusan_23",
        object : TransferListener {
            override fun onTransferInitializing(source: DataSource?, dataSpec: DataSpec?, isNetwork: Boolean) {
            }
            override fun onTransferStart(source: DataSource?, dataSpec: DataSpec?, isNetwork: Boolean) {
            }
            override fun onTransferEnd(source: DataSource?, dataSpec: DataSpec?, isNetwork: Boolean) {
            }
            override fun onBytesTransferred(source: DataSource?, dataSpec: DataSpec?, isNetwork: Boolean, bytesTransferred: Int) {
            }
        })
    val downloadHelper = DownloadHelper.forHls(
        this,
        CONTENT_URL.toUri(),
        sourceFactory,
        DefaultRenderersFactory(this)
    )
    downloadHelper.prepare(object : DownloadHelper.Callback {
        override fun onPrepared(helper: DownloadHelper) {
            DownloadService.sendAddDownload(this@MainActivity, DownloadServiceTest::class.java, downloadHelper.getDownloadRequest(ByteArray(1024)), false)
            downloadHelper.release()
        }
        override fun onPrepareError(helper: DownloadHelper, e: IOException) {
        }
    })
}
private fun playHLSCache() {
    val cache =
        SimpleCache(File("${this.getExternalFilesDir(null)?.path}/cache"), LeastRecentlyUsedCacheEvictor(1024), ExoDatabaseProvider(this))
    val upstreamDataSourceFactory = DefaultDataSourceFactory(this, "@takusan_23")
    val dataSourceFactory = CacheDataSourceFactory(
        cache, upstreamDataSourceFactory
    )
    val sourceFactory = DefaultDataSourceFactory(
        this,
        "@takusan_23",
        object : TransferListener {
            override fun onTransferInitializing(source: DataSource?, dataSpec: DataSpec?, isNetwork: Boolean) {
            }
            override fun onTransferStart(source: DataSource?, dataSpec: DataSpec?, isNetwork: Boolean) {
            }
            override fun onTransferEnd(source: DataSource?, dataSpec: DataSpec?, isNetwork: Boolean) {
            }
            override fun onBytesTransferred(source: DataSource?, dataSpec: DataSpec?, isNetwork: Boolean, bytesTransferred: Int) {
            }
        })
    val downloadHelper = DownloadHelper.forHls(
        this,
        CONTENT_URL.toUri(),
        sourceFactory,
        DefaultRenderersFactory(this)
    )
    downloadHelper.prepare(object : DownloadHelper.Callback {
        override fun onPrepared(helper: DownloadHelper) {
            val mediaSource =
                DownloadHelper.createMediaSource(helper.getDownloadRequest(ByteArray(1024)), dataSourceFactory)
            exoPlayer.prepare(mediaSource)
            exoPlayer.setVideoSurfaceView(surfaceView)
            exoPlayer.playWhenReady = true
        }
        override fun onPrepareError(helper: DownloadHelper, e: IOException) {
        }
    })
}
```

よくわからんな。  

# 終わりに
もう良くわかんないからソースだけ置いておくわ。詳しい人たのんだ
https://github.com/takusan23/ExoPlayerMediaDownloadSample