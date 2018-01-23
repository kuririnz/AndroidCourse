---
title: 非同期処理、REST API通信の実装
date: 2017-12-12 16:55:01
tags:
---
非同期処理の使い方と解説、APIを利用したインターネット通信処理ライブラリの実装と確認を行います

<!-- toc -->

当ページでは検索画面でボタンを押した後、蔵書/図書館検索の結果をAPIを利用して取得し、ログに出力するまでを解説します。
[前のページでは蔵書検索などボタンが押下された時の実装に関して解説しています](/AndroidCourse/android/05-ButtonAction)
次のページで検索結果の表示、アプリ内データベースの利用に関して解説します。

# 非同期処理
Androidアプリを開発するのに気を付けなければいけない概念として**スレッド(Thread)**という概念があります。
スレッドとは
> 脈絡、筋道。電子メールなどでは、ある一つの話題に関連した投稿の集まり

などの意味合いがあります。
身近なものでは飲み会や待ち合わせを行う時にLINEなどで作成したグループなどはスレッドと似ていると思います。

Androidのアプリが起動された時に画面に`activity_main.xml`のレイアウトが表示されるのも一つのスレッドでプログラムを実行された結果になります。

この画面にレイアウトを表示するスレッドを**メインスレッド**と呼びます。
メインスレッドは最初に表示されるアクティビティクラスを判別すると<font color="red">自動的に作成</font>されます。

メインスレッドはウィジェットやビューを更新、操作できる唯一のスレッドとなっているため**UIスレッド**と呼ばれることもあります。

非同期処理とは主に**メインスレッド**のプログラムには影響が無いようにプログラムを実行することを指します、
そのためにはメインスレッドに対して**サブスレッド**と呼ばれる別のスレッドを<font color="red">自分で作成</font>し、その中でプログラムを実行します。

今回は非同期処理を学習するために、もともと準備されているタイマースレッドというサブスレッドを使って画面が表示されてから3秒ごとに、入力されたテキストをログ出力するプログラムを作っていきます。
プログラムは前回までのコードを元に修正していきます。
<font color="RoyalBlue">***MainActivity.java***</font>
```
...
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // 蔵書検索ボタンをjavaプログラムで操作できるように名前をつける
        bookSearchBtn = findViewById(R.id.BookSearchBtn);
        // 蔵書検索する文字を入力するEditTextをjavaプログラムで操作できるように名前をつける
        bookSearchEditor = findViewById(R.id.BookSearchEdit);
        // 蔵書検索ボタンが押された時の処理を実装
        View.OnClickListener bookSearchEvent = new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                // コンソールログにボタンが押されたことを出力(表示)
                Log.d("BookSearchBtn", "onClick: BookSearch Button");
                // 入力された文字をToast(トースト)に表示
                Toast.makeText(getBaseContext()
                        , "入力された文字は [" + bookSearchEditor.getText().toString() + "]です。"
                        , Toast.LENGTH_LONG).show();
            }
        };
        // 蔵書検索ボタンが押された時に実行するプログラムをボタンに登録
        bookSearchBtn.setOnClickListener(bookSearchEvent);

        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // 準備されているTimerスレッドをインスタンス化
        Timer timer = new Timer();
        // 定期実行するタスク(TimerTask)をインスタンス化
        TimerTask timerTask = new TimerTask() {
            @Override
            public void run() {
                // コンソールログに更新された内容を出力
                Log.d("SubThread Process"
                        , "「" +  bookSearchEditor.getText().toString() + "」に更新されました。");
            }
        };

        // Timerスレッドの実行スケジュールを設定
        // 3秒毎にtimerTaskのプログラムを実行
        timer.schedule(timerTask, 0, 3000);
	    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    }
...
```

上記の追加プログラムが記述できたらエミュレータで確認してみてください。
Android Studioの**Logcat**で確認すると3秒毎にログが出力されていることがわかります。

次に画面が表示されてから3秒後から3秒毎にトーストを表示させてみましょう。
<font color="RoyalBlue">***MainActivity.java***</font>
```
		...

        // 蔵書検索ボタンが押された時に実行するプログラムをボタンに登録
        bookSearchBtn.setOnClickListener(bookSearchEvent);

        // 準備されているTimerスレッドをインスタンス化
        Timer timer = new Timer();
        // 定期実行するタスク(TimerTask)をインスタンス化
        TimerTask timerTask = new TimerTask() {
            @Override
            public void run() {
                // コンソールログに更新された内容を出力
                Log.d("SubThread Process"
                        , "「" +  bookSearchEditor.getText().toString() + "」に更新されました。");
			    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
                // 更新された内容をトーストに表示
                Toast.makeText(getBaseContext()
                        , "「" +  bookSearchEditor.getText().toString() + "」に更新されました。"
                        , Toast.LENGTH_SHORT).show();
			    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
            }
        };

        // Timerスレッドの実行スケジュールを設定
	    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓修正↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // 画面が表示されてから3秒後から3秒毎にtimerTaskのプログラムを実行
        timer.schedule(timerTask, 3000, 3000);
		//↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    }
...
```

上記のコードを入力したら`Run`ボタンを押して実行してみましょう。

・
・
・
・
・
・
・
・
・
・

アプリが強制終了してしまったのではないでしょうか？
最初に紹介した通り**メインスレッド**以外のスレッドでは画面を操作することができません。
上記で追加したコードはタイマースレッド内でToastを画面に表示する操作を実行しようとして強制終了してしまいました。
異常終了した際のエラーログはログ出力を行なっていた`Logcat`内に表示されているので確認してみましょう。
<img src="MainThreadError.png" alt="alt" title="Thread Sequence" width="750"/>

上記の通りエラーのメッセージ箇所は赤文字で表示されます。

今回のエラーの原因を説明している箇所は以下の一文
「 <font color="red">java.lang.RuntimeException: Can't create handler inside thread that has not called Looper.prepare()</font> 」

以降の行で"at ..."と説明されているのは実行されたプログラムが順に表示されます。。
各行後半の`()`内はプログラムのファイル名とプログラムを実行した行数が表示され、クリックするとプログラムファイルが表示されます。
以下の行のエラーメッセージ上で`()`内が青くなっていますが、これは自分で書いたプログラムファイルであることを表しており、
多くの場合にエラーの原因になったプログラムの行を示している箇所となります。
「 <font color="red">at kuririnz.xyz.bookdiscovery.MainActivity$2.run(</font><font color=
	"blue">MainActivity.java:53</font><font color="red">)</font> 」

今後エラーが発生した時に原因や問題を解析する時にはまずここを見るよう癖付けましょう！

タイマースレッド(サブスレッド)で画面操作できないことを確認できました、今度は正常にトーストが画面に表示されるように修正していきましょう。
コードを修正する前に新しく使う`Handler`クラスと`Runnable`クラスを紹介します。

### Handler クラス
Handlerクラスはスレッドから別のスレッドへ通信を行う機能を持ったクラスです。
今回のプログラムではToastを画面に表示する為TimerThreadからMainThreadへ通信を行います。
通信とは記述していますが、今回の通信では<font color="red">MainThreadでToastを表示する</font>ことが目的になるため、
データを渡すことが目的の通信では無いことを意識してみてみてください。

### Runnable クラス
Runnableクラスは実際にThreadで処理するプログラムを実行するためのクラスです。
<font color="red">**Thread = Runnable**</font>と認識して問題ありません。
今回のプログラムではTimerThreadで３秒ごとにlogを出力する時、MainThreadでToastを表示する処理を実行する時にRunnableクラスを使用しています。

<font color="RoyalBlue">***MainActivity.java***</font>
```
...
public class MainActivity extends AppCompatActivity {

    // レイアウトxmlと関連付けるWidget
    Button bookSearchBtn;
    EditText bookSearchEditor;
    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    // メインスレッドに帰って来るためのハンドラー
    Handler handler;
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // ハンドラーオブジェクトをMainThreadでインスタンス化
        handler = new Handler();
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
        // 蔵書検索ボタンをjavaプログラムで操作できるように名前をつける
        bookSearchBtn = findViewById(R.id.BookSearchBtn);

        ...

        // 定期実行するタスク(TimerTask)をインスタンス化
        TimerTask timerTask = new TimerTask() {
            @Override
            public void run() {
                // コンソールログに更新された内容を出力
                Log.d("SubThread Process"
                        , "「" +  bookSearchEditor.getText().toString() + "」に更新されました。");
                //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓修正↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
                handler.post(new Runnable() {
                    @Override
                    public void run() {
                        // 更新された内容をトーストに表示
                        Toast.makeText(getBaseContext()
                                , "「" +  bookSearchEditor.getText().toString() + "」に更新されました。"
                                , Toast.LENGTH_SHORT).show();
                    }
                });
                //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
            }
        };
...
```

上記のコードを実装できたら、`Run`ボタンからエミュレータで動作確認してみましょう。
以下の様に3秒ごとに入力中の文字がToastに表示されたでしょうか？
<img src="AsyncTimerToast.png" alt="alt" title="TimerThreadTest" width="300"/>

実際に使ってみた`Handler`クラスですが、正しくMainThreadで実行するにはコツが必要です、
コツというのは`Handler`クラスをインスタンス化するタイミングです。
深く理解できればこのタイミングの調整は不要になりますが、全てを一度に覚えきるのは難しいです、
ですので他の非同期処理後にMainThreadで処理を行うのにHandlerを利用する場合はまず、**onCreateメソッドのsetContetView(...)**の次行でインスタンス化するよう意識付けてください。
> ※サブスレッド間の通信に関してはこの一連の記事では紹介していないので別途調査が必要です。

以上で、非同期処理の基礎の解説は終了です、難しい箇所も多いので自分なりにプログラムを修正して試してみましょう。
続いてはインターネット通信処理を通しでデータを取得するAPIの使い方を実装していきます。

---

# REST API通信処理

非同期処理が作れる様になったらインターネット上にあるデータを取得するプログラムを作っていきます、
そのためにはインターネット上にあるデータを開発するアプリから取得して使う方法があります。
これを多くの場合にAPI通信などと呼びます。

今回のアプリでは蔵書を検索する機能を実装するためにこのAPI通信が必要になります。

REST APIとは**Representational State Transfer Application Programming Interface**の略称で
それぞれ"REST"と"API"で使われることもあります。

## REST
Webサービスの設計モデルを表しており、REST APIのURLにHTTPメソッドでアクセスすることでデータの送受信が行えます。

RESTの設計条件として以下が該当します。
* アドレス指定可能なURLで公開されていること
* インターフェースが統一されていること(HTTPメソッドに準じていること)
* ステートレスであること
* 処理結果がHTTPステータスコードで通知されること

> *キュレーションサイトQiitaより引用[※1]*

## API
ウィキペディアには以下の様にあります。
<font color="red">**ソフトウェアコンポーネントが互いにやり取りするのに使用するインターフェース仕様**</font>

...文章からイメージするのは難しそうです。。。
ですので、まずは以下の様な認識で使っていきましょう。
<font color="blue">**インターネット上に公開された、URLを知っていれば誰でも使えるデータ**</font>

例として実際にAPIのデータをgoogle ChromeなどWebブラウザの画面で確認できるもので確認してみましょう。
以下のurlはgoogleが一般公開している"google books API"というAPIで、蔵書の情報を取得するAPIです。
> https://www.googleapis.com/books/v1/volumes

この画面に表示されているデータは`json`という形式でまとめられたデータになっており、上記urlにアプリからアクセスすると
画面に表示されているデータを取得することができます。
多くのアプリではこの様にデータを取得して画面に表示させることで様々なデータを表示しています。

次に指定の本を検索してみましょう、Webブラウザのurlに少し付け加えてみます。
先ほどと結果の違いがわかる様に新しくタブを開いて以下のurlを表示してみましょう。
> https://www.googleapis.com/books/v1/volumes?q=ほんきで学ぶAndroidアプリ開発入門

Webブラウザを確認すると、表示される内容が変わっているのがわかります。
追加した、"?q=ほんきで学ぶAndroidアプリ開発入門"と文字の部分一致で検索した結果が表示されました。

Webブラウザで結果を確認していただいた通り、APIというのはURLや設定されているパラメータの値によって検索結果が変わります、
これはホームページと同じくサーバがURLを受け取りプログラムによって表示する`json`情報を返しているからです。

上記で紹介したAPIの検索条件等は[Google Books APIs](https://developers.google.com/books/docs/v1/using#WorkingVolumes)に記載されています。（全て英語です）

下図はAPII通信を行うプログラムの流れとして、
API通信を行う流れからデータを取得し、画面に反映するまでの簡単な流れを図にしています。
<img src="APISequence.svg" alt="alt" title="API Sequence" width="550"/>

上記で解説していた非同期処理ですが、こちらでも利用します。
むしろAPI通信を行いデータを画面に反映するために使用すると言っても過言ではありません。

というのもAndoridアプリ開発の上で画面更新はMainThreadのみ可能と解説をしましたが、更に<font color="blue">**MainThreadは時間のかかる処理を行なってはいけない**</font>という制約も持ち合わせているため、MainThreadでネットワーク通信を行うとアプリが強制終了してしまいます。
（サブスレッドで画面を操作しようとした時と同じ様に終了します）

<font color="red">**そのためAPI通信を実行し、画面に反映するためには大まかに以下の手順が必要になります**</font>
1. メインスレッドでハンドラーをインスタンス化
1. メインスレッドからサブスレッドを起動
1. サブスレッドでAPI通信処理を実行
1. ハンドラーを使いサブスレッドで取得したAPIデータをメインスレッドへ連携
1. メインスレッドでAPIデータを使い画面を操作



# まとめ
非同期通信やスレッドに関する概念は最初イメージするのが難しく整理しにくい内容だと思いますが、以下のポイントだけ
抑えておきましょう！

* Androidアプリ開発においては画面の更新、操作を行う場合はメインスレッドを使う必要がある
* Androidアプリ開発ではネットワーク通信処理を行う場合はサブスレッド（メインスレッド以外）で行う必要がある
* Threadクラスはインスタンス化したスレッドと同じスレッドとして処理を行うことができる
* HandlerクラスはThreadを起動させる時に利用する

[※1]: https://qiita.com/TakahiRoyte/items/949f4e88caecb02119aa

