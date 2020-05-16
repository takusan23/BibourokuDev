---
title: MastodonとMisskey同時に見れるクライアント作った？
date: 2020-05-16 20:46:04
tags:
- Android
- Kotlin
- Mastodon
- Misskey
---
どうもこんばんわ。  
少し前だけどニコ生で世話やきキツネの仙狐さん一挙放送見ました。かわいいいいいいいいいい  


# 本題
PlayStoreに出すかは考え中。  
{%asset_img ss.png ss %}

ダークモードがちょっとかっこいい  
{%asset_img ss_dark.png ss_dark %}

2列になってるけど設定で減らせたり増やせたりできます。  
時系列順になってるけど2列に出してるせいでなんか見ずらい気がする。(でも一列のクライアントいっぱいあるし・・・)

# なんで作ったの？
ViewModel+LiveData（画面回転しても値を保持し続けてくれるやつ）でなんか一個作ろうとした。使わなかったけど。

## なんで使わないの？
ArrayListがViewModelで使えない？`add()`しても増えない（そもそもArrayListをViewModelで使うのが間違い説がある）  
あとLiveDataの通知は`add()`じゃこない。  
しゃーないので（すでにデータクラスとか作っちゃった）ViewModel+LiveData抜きで作った。  
**代わりに`onSaveInstanceState`で値を引き継ぐようにしたので倒しても大丈夫（再度APIを叩くことが無い）**

# ダウンロード
https://github.com/takusan23/KaisendonMk2/releases/tag/1.0  
Playストアは考え中。

# ソースコード
https://github.com/takusan23/KaisendonMk2

# できること
逆に言うとここに書いてなければできません。
- MastodonとMisskeyを同時に見れる
    - Mastodon
        - ホーム/通知(ストリーミングのみ)/ローカル
    - Misskey
        - ホーム/通知(ストリーミングのみ)/ローカル
    - 複数アカウント
        - 私はMastodonとMisskeyそれぞれ一個ずつしか持ってないけどね。
- ストリーミングAPI
- 投稿
    - 公開範囲
    - カスタム絵文字一覧
    - 端末の情報（キャリア名とか）
- モバイルデータ通信のときは画像非表示
    - 4GBしか使えないんや。このクライアントの目玉（のつもり）
        - 今はずっとWi-Fi環境下だけど
- ダークモード
- カスタム絵文字、GIFカスタム絵文字
- 投稿操作
    - ふぁぼ、ブースト
    - Misskeyはリアクション
- タイムラインの背景変える機能
- タイムラインのフォント変える機能
- 突然のクラッシュ
    - 複数のTLをまとめて時系列順に並べる処理で出てる。

# できないこと
- 画像投稿
- 添付メディア表示もない
- 時間指定投稿
- アンケ
- アカウント情報見る
    - フォローも
    - プロフィール編集もない
- 通知はストリーミングのみ
- できることに書いてない事はできない

# やりたいこと
- 添付メディア表示
- プロフィール表示


- あとは・・・他クライアントに任せた。
    - モバイルデータ回線のとき画像を非表示にする機能が欲しかっただけなので

# 使い方
初回起動時は強制的にログイン画面が出ます。  
ログインできたら左下のレンチマーク押して**読み込むタイムラインの設定**を選ぶと
- ホーム
- 通知
- ローカル

から選べるので見たいタイムラインにチェックしてください。  

複数ログインのときはレンチマーク押して**ログイン**を押して、ログインを済ませて、**読み込むタイムラインの設定**を選ぶと追加されてるので後は楽しんで。

# 作るのに大変だったところとか
## Twitterと違う点
色んな所[(こことかわかりやすい)](https://blog.yukiya.me/2020/05/15/joinmastodon2020/)でまとめられてるのであんまり書かないけど

|できること|Twitter|Mastodon|
|---|---|---|
|ブックマーク|あるけどAPIがない|使ったこと無い(インスタンスが対応してない)|
|アンケート|あるけどAPIない（アンケ機能ついてから数年経ってる）|APIもあるしAPI限定の複数投票許可がある|
|本人認証|公式サイトって一般で取れんの？|自分のホームページに指定したHTML貼り付けて、プロフィールの補足情報にリンク貼れば認証済みになる|
|リアルタイム更新|FilterStream（いつまで使えるか不明）かTweetDeck|WebSocketで利用可能|
|APIの利用|審査を通る必要がある（垢がいつ消えるか怖い&電話番号必須）|Via芸できるよ（クライアントから作れる`/api/v1/apps`。鯖がいっぱいあるのでその都度アプリ(クライアント)を作る）|
|お気に入り|他の人のお気に入り見れる|自分しか見れないのでブックマーク代わりにでも（**ただしふぁぼ通知は相手に行くのと投稿のお気に入り登録したユーザー一覧には表示される**）|

## MastodonとMisskeyはAPIぜんぜん違う

まあそれはそうだろって話（PleromaはMastodonと互換性が有るらしい？要検証）なんだけどMastodonにあってMisskeyに無いとかって結構有るからMastodonのがそのまま使えるわけじゃない。世の中のクライアント作ってる人すごいね

ついでにMisskeyのAPIは(多分)全てPOSTリクエストです。タイムラインの取得もPOSTです。投稿もPOSTです。何故かは知りませんが。

OkHttp+Coroutineを使ったサンプル
```kotlin
// ユーザーエージェント
private val USER_AGENT = "KaisendonMk2;@takusan_23"

// application/json
private val APPLICATON_JSON = "application/json".toMediaType()

private fun baseTimeLineAPI(url: String, limit: Int = 100): Deferred<Response> =
    GlobalScope.async {
        val postData = JSONObject().apply {
            put("limit", limit)
            put("i", instanceToken.token)
        }.toString().toRequestBody(APPLICATON_JSON)
        val request = Request.Builder().apply {
            url("https://misskey.m544.net/api/$url")
            header("User-Agent", USER_AGENT)
            post(postData)
        }.build()
        val okHttpClient = OkHttpClient()
        val response = okHttpClient.newCall(request).execute()
        return@async response
    }
```
MisskeyのTL取得APIは💯件まで取れます。  

## MisskeyAPIのドキュメントが
`Misskey API`って調べて一番上に出てくるサイトを開いてAPIリファレンスを見ると、**アクセストークンの取得になんかアプリを登録しろ**って書いてあるんですが、この方法で大丈夫でした→ https://misskey.m544.net/docs/ja-JP/api  
そもそもアプリを登録しろってやり方なんですが、自由に鯖が作れる（私は作ったこと無いけど）Mastodon/Misskeyの仕組みではまず無理ですね。

## カスタム絵文字
最近は~~ニコ動~~（**例のアレ**）でもPartyParrotブームが来たわけですが、MastodonやMisskeyにも動く絵文字（カスタム絵文字）があります。   
文字の中に画像を入れるのってHTMLとかだと簡単そうでGIFにも対応できるんですが、今回はAndroidです。TextViewじゃ無理やろ。    
世の中のクライアントがどの様にカスタム絵文字を描画してるのかわかりませんが、私はMarkdown表示ライブラリに画像を表示する機能があったのでそれを使っています。  
Markdown表示ライブラリ→ https://github.com/noties/Markwon

```kotlin
/**
 * カスタム絵文字にTextViewを対応させる
 * @param content 文字列
 * @param textView setText代わり
 * */
fun setCustomEmoji(textView: TextView, content: String) {
    // Markdownのライブラリ入れた
    val markwon = Markwon.builder(textView.context)
        .usePlugin(HtmlPlugin.create())
        .usePlugin(ImagesPlugin.create(textView.context))
        .usePlugin(GifPlugin.create())
        .usePlugin(object : AbstractMarkwonPlugin() {
            // 読み込み中は別のDrawableを表示する
            override fun configureImages(builder: AsyncDrawableLoader.Builder) {
                builder.placeholderDrawableProvider {
                    // your custom placeholder drawable
                    textView.context.getDrawable(R.drawable.ic_refresh_black_24dp)
                }
            }
        }).build()
    markwon.setMarkdown(textView, customEmojiReplaceText)
}
```

## 自由にファイルアクセスできないこんな世の中でTypeface#createFromFile()
~~ポイズン~~  
フォント変更機能はほしいです。TextViewにもフォントファイルを適用する（ユーザーが自由に適用できる）機能あります。`Typeface#createFromFile()`って言うんですけどね。  
これ引数にUriは指定できません（多分）。`file://`から始まるパスである必要があります。  
このアプリではまずSAFでttfファイルを選んでもらって

```kotlin
val intent = Intent(Intent.ACTION_OPEN_DOCUMENT).apply {
    type = "font/*"
}
startActivityForResult(intent, FONT_REQUEST_CODE)
```

アプリ固有ディレクトリにコピーして使います

```kotlin
// アプリ固有ディレクトリにコピー
val uri = data?.data ?: return
val file = File("${context?.getExternalFilesDir(null)}/font.ttf")
file.createNewFile()
// こぴー
val byteArray = context?.contentResolver?.openInputStream(uri)?.readBytes()
if (byteArray != null) {
    file.writeBytes(byteArray)
}
```

これで`Typeface.createFromFile()`が使えます。

## RecyclerViewで表示
MastodonとMisskeyは別のデータクラスなのでRecyclerViewのAdapterにわたすときに考えないといけないんですよね。  
今回はAdapterに渡すためのデータクラスを別に用意しました。

```kotlin
/**
 * RecyclerViewに渡すデータクラス
 * @param allTimeLineData 色つけたりするから；；
 * どっちか一個だけ入れろ↓。
 * @param statusData
 * @param notificationData
 * */
data class TimeLineItemData(
    val allTimeLineData: AllTimeLineData,
    val statusData: StatusData? = null, // null以外ならTL表示
    val notificationData: NotificationData? = null,// null以外なら通知表示
    val misskeyNoteData: MisskeyNoteData? = null, // null以外ならNote表示
    val misskeyNotificationData: MisskeyNotificationData? = null // null以外なら通知表示
) : Serializable
```
Mastodonの投稿か通知かMisskeyの投稿か通知のどれかが入ることになってます。`AllTimeLineDataはCardViewの色情報が入ってます。`

それからRecyclerViewでレイアウトを複数用意する方法ですが`getItemViewType()`を使います。

ちょっと長いけど

```kotlin
// レイアウトの定数（onCreateViewHolder()で使う）
companion object {
    /** Mastodon トゥート */
    val TOOT_LAYOUT = 0
    /** Mastodon 通知 */
    val NOTIFICATION_LAYOUT = 1
    /** Mastodon ブースト */
    val TOOT_BOOST_LAYOUT = 2
    /** Misskey 投稿 */
    val MISSKEY_NOTE_LAYOUT = 3
    /** Misskey 通知 */
    val MISSKEY_NOTIFICATION_LAYOUT = 4
    /** Misskey Renote */
    val MISSKEY_RENOTE_LAYOUT = 5
}
override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder {
    // レイアウト分岐
    val view = when (viewType) {
        TOOT_BOOST_LAYOUT -> BoostViewHolder(LayoutInflater.from(parent.context).inflate(R.layout.adapter_boost, parent, false))
        TOOT_LAYOUT -> TootViewHolder(LayoutInflater.from(parent.context).inflate(R.layout.adapter_timeline, parent, false))
        NOTIFICATION_LAYOUT -> NotificationViewHolder(LayoutInflater.from(parent.context).inflate(R.layout.adapter_notification, parent, false))
        MISSKEY_NOTE_LAYOUT -> MisskeyNoteViewHolder(LayoutInflater.from(parent.context).inflate(R.layout.adapter_misskey_note, parent, false))
        MISSKEY_NOTIFICATION_LAYOUT -> MisskeyNotificationViewHolder(LayoutInflater.from(parent.context).inflate(R.layout.adapter_misskey_notification, parent, false))
        MISSKEY_RENOTE_LAYOUT -> MisskeyRenoteViewHolder(LayoutInflater.from(parent.context).inflate(R.layout.adapter_misskey_renote, parent, false))
        else -> TootViewHolder(LayoutInflater.from(parent.context).inflate(R.layout.adapter_timeline, parent, false))
    }
    return view
}
// 通知と投稿で分岐させる
override fun getItemViewType(position: Int): Int {
    return when {
        // Mastodon
        timeLineItemDataList[position].statusData != null && timeLineItemDataList[position].statusData!!.reblogStatusData != null -> TOOT_BOOST_LAYOUT
        timeLineItemDataList[position].statusData != null -> TOOT_LAYOUT
        timeLineItemDataList[position].notificationData != null -> NOTIFICATION_LAYOUT
        // Misskey
        timeLineItemDataList[position].misskeyNoteData != null && timeLineItemDataList[position].misskeyNoteData!!.renote != null -> MISSKEY_RENOTE_LAYOUT
        timeLineItemDataList[position].misskeyNoteData != null -> MISSKEY_NOTE_LAYOUT
        timeLineItemDataList[position].misskeyNotificationData != null -> MISSKEY_NOTIFICATION_LAYOUT
        else -> TOOT_LAYOUT
    }
}
```

## データクラスで画面回転耐久
~~これは別の記事で書きたい。~~書きました→ https://takusan23.github.io/Bibouroku/2020/05/17/画面回転しても尊厳損傷しないようにonSaveInstanceState使う/

# 終わりに
コルーチンいっぱいつかった。  
`withContext(Dispatchers.IO)`って便利だね。括弧が減りそう。
```kotlin
// コルーチン
GlobalScope.launch(Dispatchers.Main) {
    // UIスレッドのコルーチン -> UIスレッドではないスレッドへ切り替え
    val response = withContext(Dispatchers.IO) {
        // インターネットから持ってくるなど
    }
    // 帰ってきたらUIスレッドに戻る
}
```