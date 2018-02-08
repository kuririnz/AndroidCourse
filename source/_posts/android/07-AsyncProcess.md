---
title: 蔵書検索機能の作成
date: 2017-11-08
tags:
---
非同期処理の使い方と解説、インターネット通信ライブラリを利用してインターネット上から取得したデータの利用方法を学習します。

<!-- toc -->

[検索結果一覧画面の作成](/AndroidCourse/android/06-TransitionScreen)からの引き続きの学習ページです。
# 学習ポイント
* 非同期通信の概要
* Androidアプリ開発におけるUIスレッド(メインスレッド)とサブスレッド
* REST API
* インターネット通信ライブラリ**OkHttp**の利用方法

Googleが公開している蔵書検索サービスの"Google Books API"を利用して入力された文言を元にインターネット上の蔵書情報を検索し、アプリの検索結果情報を表示します。
API通信を行う上で非同期処理の基礎知識が必要になりますので合わせて学習します。

# 非同期処理
アプリ開発における非同期処理とは画面の表示や操作に影響なく別のプログラムや命令を実行することを指します。
身近なものとしてプッシュ通知やアラーム機能も非同期処理の一部と言えます。

Androidアプリにはスレッド(Thread)という概念があり、"タスク"や"一連の仕事"と言い換えることができます。
Androidのアプリを起動した時に`activity_main.xml`のレイアウトが表示されるのも一つのスレッドでプログラムを実行した結果です。

この画面にレイアウトを表示するスレッドを**メインスレッド**と呼びます。
メインスレッドは最初に表示されるアクティビティクラスを判別すると<font color="red">自動的に作成</font>されます。

メインスレッドはウィジェットやビューの表示や更新を行える唯一のスレッドなため**UIスレッド**と呼ぶこともあります、そしてメインスレッドは表示の管理に加えてユーザの操作（タップやスワイプ）を監視する仕事も兼務しています。

プログラムの上での非同期処理は主に**メインスレッド**に影響が無いようにプログラムを実行することを指します、
そのために別のスレッドを<font color="red">自分で作成</font>し、その領域でプログラムを実行します。
自分で作成したスレッドは**サブスレッド**と呼び、複数のサブスレッドを作成することができます。

最終的な目的としてインターネット上のデータをアプリで取得することにあります。
Androidアプリは画面の表示やユーザ操作に影響がある可能性がある時間のかかる命令をメインスレッドで実行することを禁止しています。
インターネット通信処理も時間のかかる命令であるため、メインスレッドでの実装を禁止されています、
そのため、インターネット通信処理も非同期処理として実行する必要があります。

インターネット通信の実装と非同期処理を同時に進めてしまうと情報を整理できない場合もあるのでまずは非同期処理に絞って学習していきます。

## 非同期タイマー機能
非同期処理の基礎を学習するために、準備されているタイマースレッドというサブスレッドを使って画面が表示されてから3秒ごとに、入力されたテキストをログ出力するプログラムを作っていきます。
```java MainActivity.java
    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓修正↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    // レイアウトxmlと関連付けるWidget
    private Button bookSearchBtn;
    private EditText bookSearchEditor;
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑修正↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    // Timerスレッドクラス
    private Timer timer;
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

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
                // 画面遷移するためのIntentをインスタンス化
                Intent intent = new Intent(MainActivity.this, ResultListActivity.class);
                // 画面遷移アクションを実行
                startActivity(intent);
            }
        };
        // 蔵書検索ボタンが押された時に実行するプログラムをボタンに登録
        bookSearchBtn.setOnClickListener(bookSearchEvent);

        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // 準備されているTimerスレッドをインスタンス化
        timer = new Timer();
        // ３秒ごとに実行するタスク(TimerTask)をインスタンス化
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
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    }
```
コードを実装したらエミュレータを起動して確認します。
下図の通りAndroid Studioの**Logcat**で確認すると3秒毎にログが出力を確認できましたか？
{% img /android/07-AsyncProcess/timerexample.png 650 example timer log %}

次は画面が表示されてから3秒後から3秒毎にトーストが表示される様に修正してみます。
```java MainActivity.java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
       ...一部省略

        // 蔵書検索ボタンが押された時に実行するプログラムをボタンに登録
        bookSearchBtn.setOnClickListener(bookSearchEvent);

        // 準備されているTimerスレッドをインスタンス化
        timer = new Timer();
        // ３秒ごとに実行するタスク(TimerTask)をインスタンス化
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
                //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
            }
        };

        // Timerスレッドの実行スケジュールを設定
        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓修正↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // 画面が表示されてから3秒後から3秒毎にtimerTaskのプログラムを実行
        timer.schedule(timerTask, 3000, 3000);
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑修正↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    }
```
コードの実装が終わったら`Run`ボタンからエミュレータで動作を確認します。
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
紹介した通りメインスレッド以外のスレッドでは表示や更新は実行できません、
それに対して上記はTimerスレッドの領域でToastを表示するコードを実装してしまい、アプリ実行時に強制終了してしまいました。
強制終了した原因などログは`Logcat`領域に表示されるので確認して見ましょう。
{% img /android/07-AsyncProcess/MainThreadError.png ThreadSequence %}
プログラム実行時に発生したエラーメッセージは赤文字で表示されます。
先ほどのエラー原因を説明している箇所は以下
> <font color="red">java.lang.RuntimeException: Can't create handler inside thread that has not called Looper.prepare()</font>

以降の行の"at ..."は強制終了する前に実行されたメソッドなど命令が順に表示されます。
各行後半の "()" 内はプログラムのファイル名とプログラムを実行した行数が表示され、クリックするとプログラムファイルが表示されます。
以下の行のエラーメッセージ上で "()" 内が青くなっています、これは自分で管理するファイルであることを表しており、
多くの場合にエラーの原因になったプログラムの行を示している箇所となります。
> <font color="red">at kuririnz.xyz.bookdiscovery.MainActivity$2.run(</font><font color=
"blue">MainActivity.java:53</font><font color="red">)</font>

エラーが発生した時には原因や問題を解析するためには`Logcat`を確認する癖を付けましょう！
コードを修正する前に新しく使う**Handlerクラス**と**Runnableインターフェース**を紹介します。

## Handler クラス
Handlerクラスはスレッドから別のスレッドへ通信を行う機能を持ったクラスです。
今回のプログラムではToastを画面に表示する為TimerThreadからMainThreadへ通信を行います。
通信と記述していますが、今回の通信ではMainThreadでToastを表示することが目的になるため、
データを渡すことが目的の通信では無いことを意識してみてみてください。

## Runnable インターフェース
RunnableはThreadで処理する領域を宣言し、Thread内のプログラムを実装、実行する機能を持っています。
**Thread = Runnable**と認識して問題ありません。
今回のプログラムではTimerThreadで３秒ごとにlogを出力する時、MainThreadでToastを表示する処理を実行する時にRunnableインターフェースを使用しています。

では改めてタイマースレッドからメインスレッドに作業を渡して画面にトースト表示するよう修正します。
```java MainActivity.java
public class MainActivity extends AppCompatActivity {

    // レイアウトxmlと関連付けるWidget
    private Button bookSearchBtn;
    private EditText bookSearchEditor;
    // Timerクラス
    private Timer timer;
    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    // メインスレッドに帰って来るためのハンドラー
    private Handler handler;
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // ハンドラーオブジェクトをMainThreadでインスタンス化
        handler = new Handler();
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
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
                //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
                // Timerスレッドを止める
                timer.cancel();
                //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
                // 画面遷移するためのIntentをインスタンス化
                Intent intent = new Intent(MainActivity.this, ResultListActivity.class);
                // 画面遷移アクションを実行
                startActivity(intent);
            }
        };
        // 蔵書検索ボタンが押された時に実行するプログラムをボタンに登録
        bookSearchBtn.setOnClickListener(bookSearchEvent);

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
                //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑修正↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
            }
        };
    }
}
```
上記コードを実装できたら、エミュレータで確認します。
以下の様に3秒ごとに入力中の文字がToastに表示されたでしょうか？
{% img /android/07-AsyncProcess/AsyncTimerToast.png 300 TimerThreadTest %}
Handlerクラスを使いタイマースレッドからメインスレッドに戻ることができました。
実感は湧きにくいかもしれませんが、画面にToastが表示されたことが何よりの結果です。

サブスレッドからメインスレッドに戻るためにはHandlerクラスをインスタンス化するタイミング、または方法を意識しないといけません。
”onCreate()”メソッドはメインスレッドで実行されています、そのため "onCreate()"メソッド内でインスタンス化した`handler`はメインスレッドの属性を持ったことになり、サブスレッドから`handler.post(...)`実行したRunnableのプログラムはメインスレッドでのプログラムとして扱われたことになります。

以上で、非同期処理の基礎の解説は終了です、難しい箇所も多いので自分なりにプログラムを修正して試してみてください。

# 蔵書検索機能
非同期処理の基礎を学んだところで、本命のインターネット上にあるデータを取得するプログラムを実装していきます。
今回のアプリは蔵書検索する機能を実装するためにインターネット上にあるデータが必要になりますが、その問題を解決するために "Google Books API"サービスを使いインターネット上からデータを取得する機能を実装していきます。

API通信とは「REST API」通信の略称で
REST APIとは**Representational State Transfer Application Programming Interface**の略称で
さらにそれぞれ"REST"と"API"で使われることもあります。

## REST
Webサービスの設計モデルを表しており、REST APIのURLにHTTPメソッドでアクセスすることでデータの送受信が行えます。

RESTの設計条件として以下が該当します。
{% blockquote Qiita https://qiita.com/TakahiRoyte/items/949f4e88caecb02119aa REST入門 基礎知識 %}
* アドレス指定可能なURLで公開されていること
* インターフェースが統一されていること(HTTPメソッドに準じていること)
* ステートレスであること
* 処理結果がHTTPステータスコードで通知されること
{% endblockquote %}

## API
ウィキペディアには以下の様にあります。
{% blockquote wikipedia https://ja.wikipedia.org/wiki/%E3%82%A2%E3%83%97%E3%83%AA%E3%82%B1%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0%E3%82%A4%E3%83%B3%E3%82%BF%E3%83%95%E3%82%A7%E3%83%BC%E3%82%B9 API %}
**ソフトウェアコンポーネントが互いにやり取りするのに使用するインターフェース仕様**
{% endblockquote %}
...文章からイメージするのは難しそうです。。。
ですので、まずは以下の様な認識で使っていきましょう。
<font color="blue">**インターネット上に公開された、URLを知っていれば誰でも使えるデータ**</font>

例として実際にAPIのデータをgoogle ChromeなどWebブラウザで確認できるので確認してみましょう。
以下で紹介しているURLはgoogleが一般公開している"google books API"というAPIで、蔵書の情報を取得することができるAPIです。
また、Google Books APIを使うためには "?q=\*\*\*"という検索条件を必ずつけないといけないのでサンプルで参考書タイトルで検索する指定しています。
{% blockquote Google Books APIs https://developers.google.com/books/docs/v1/using Reference %}
https://www.googleapis.com/books/v1/volumes?q=ほんきで学ぶAndroidアプリ開発入門
{% endblockquote %}
この画面に表示されているデータは`json`という形式でまとめられたデータで、上記URLにアプリからアクセスすると
画面に表示されているデータをアプリで受け取ることができます。
キュレーションアプリやニュースアプリなどはこの様にデータを取得して画面に表示させてなどしています。

Google Books APIでは"?q=\*\*\*"の"\*"の文字を変更することで表示される結果が変わります、
これはホームページなどと同じくサーバがURLを受け取り、サーバ内のプログラムが実行された結果`json`データが返されているからです。

上記で紹介したAPIの検索条件等は[Google Books APIs](https://developers.google.com/books/docs/v1/using#WorkingVolumes)に記載されています。（全て英語です）

下図はAPI通信を行う流れからデータを取得し、画面に反映するまでの簡単な流れを示しています。
{% img /android/07-AsyncProcess/APISequence.svg 550 APISequence %}

Andoridアプリでは画面表示や更新はMainThreadのみ可能と解説をしましたが、更に<font color="blue">**MainThreadは時間のかかる処理を行なってはいけない**</font>という制約もあるため、MainThreadでネットワーク通信を行うとアプリが強制終了してしまいます。

上記の図の通りですが<font color="red">**API通信を実行し、画面に反映するためには大まかに以下の手順が必要になります**</font>
1. メインスレッドでハンドラーをインスタンス化
1. メインスレッドからサブスレッドを起動
1. サブスレッドでAPI通信処理を実行し検索結果データを取得
1. ハンドラーを使いサブスレッドで取得したAPIで取得した検索結果データをメインスレッドへ連携
1. メインスレッドで検索結果データを使い画面を更新

## OkHttpライブラリの導入
API通信、ネットワーク通信処理を行うために、ここでは**[OkHttp]**という*ライブラリ*を使っていきます、**OkHttp**はAndroid に元々備わっているネットワーク通信機能をさらに簡単に実装することができる機能群です。
このようにプログラムを一部簡単にしてくれる機能群を*ライブラリ*と呼びます。

**OkHttp**を導入するために修正するファイルは`build.gradle`というファイルです。
このファイルは２つありますが、開くのは`build.gradle`の後ろに***(Module: app)***と表示されている方です。
{% img /android/07-AsyncProcess/includeokHttp01.png 550 IncludeokHttp %}
`build.gradle`を開き "dependencies" の "{}"内に以下のコードを記述します
```gradle build.gradle
dependencies {
    ...一部省略
    implementation 'com.squareup.okhttp3:okhttp:3.9.1'
}
```
コードの記述が終わったら右上に表示された<font color="blue">Sync Now</font>リンクをクリックします。
{% img /android/07-AsyncProcess/includeokHttp02.png 600 Include okHttp %}
これで**OkHttp**ライブラリの導入は完了です。

## API通信機能
**OkHttp**の導入が完了しましたので、実際にアプリで使っていきます！
実装の完成形として`MainActivity`の蔵書検索ボタンがクリックされたらEditTextの文字データを`ResultListActivity`に渡します。
そして`ResultListActivity`では受け取った文字データをGoogle Books APIから検索する蔵書情報として利用して期待する検索結果をインターネット上から取得します。
画面への反映は次のページで解説します。

javaファイルの実装の前にAndroidアプリでインターネット通信を行うためには、パーミッション(permission)と呼ばれる許可設定が必要なので`AndroidManifest.xml`ファイルを修正します。
```XML AndroidManifest.xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="PackageName">

    <!-- Androidアプリでインターネット通信を許可する設定 -->
    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        ...一部省略>
    </application>
</manifest>
```
ではjavaファイルの実装です、まずはアプリでGoogle Books APIを使ってデータを取得するプログラムを実装します。
```java ResultListActivity.java
    ...一部省略
    // ListViewの表示内容を管理するクラス
    private ResultListAdapter adapter;
    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    // OkHttp通信クライアント
    private OkHttpClient okHttpClient;
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        ...一部省略
        // ListViewに行をクリックした時のイベントを登録
        resultListView.setOnItemClickListener(ResultListActivity.this);

        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // OkHttp通信クライアントをインスタンス化
        okHttpClient = new OkHttpClient();
        // 通信するための情報
        Request request = new Request.Builder().url("https://www.googleapis.com/books/v1/volumes?q=ほんきで学ぶAndroidアプリ開発入門").build();
        // データの取得後の命令を実装
        Callback callBack = new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                // 失敗した時の命令
                // 通信に失敗した原因をログに出力
                Log.e("failure API Response", e.getLocalizedMessage());
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                // 成功した時の命令
                // Google Books APIから取得したデータをログに出力
                Log.d("Success API Response", response.body().string());
            }
        };
        // 非同期処理でAPI通信を実行
        okHttpClient.newCall(request).enqueue(callBack);
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    }
```
上記コードを実装したら動作確認してみましょう。
`Logcat`にwebブラウザで確認したデータが表示されたでしょうか？
表示されていれば正常にインターネット上からデータを取得できたことになります。
{% img /android/07-AsyncProcess/apiresponse.png 700 API Response Log %}
上記コードで出てきましたが、以下の様にメソッドからメソッドを呼び出す様な方法を**メソッドチェーン**と呼びます、プログラムの設計次第ではの様な実装方法も可能ですが、自作するクラスでは使う機会は少ないですが、今回の様にライブラリではよく使われる実装パターンなので覚えておきましょう。
```java
Request request = new Request.Builder().url("").build();
や
okHttpClient.newCall(request).enqueue(...)
```

取得した蔵書データですがWebブラウザで確認した形式と同じく`Json`形式で取得しました。
Jsonはシンプルなデータ形式で **:(コロン)**を挟む左にデータの名称、右にデータが記述されており、複数データがある場合は**,(カンマ)**で区切って複数のデータを持つことができます。
Jsonデータは必ず**{}(波括弧)**で閉じられています。
この様に名称とデータを合わせて持つデータを**KeyValuePair**や**KeyValue形式**データと呼びます。
以下の例では"AndroidCourse"というKey(名称)に"Androidアプリの開発講座"というValue(文字列データ)が関連づいた**KeyValue**データです。
```json
{
    "AndroidCourse":"Androidアプリの開発講座"
}
```
Jsonのデータには文字列や数値、配列や真偽値などを設定することが可能です。
APIを使う場合の多くはこのJson形式でデータを取得することが多いので解析できる様に覚えておくと便利です。

次に取得したデータをアプリ内でパース(解析)して取得した蔵書データの件数と件数分のタイトルをログに出力してみます。
```java ResultListActivity.java
        ...一部省略
        // OkHttp通信クライアントをインスタンス化
        okHttpClient = new OkHttpClient();
        // 通信するための情報
        Request request = new Request.Builder().url("https://www.googleapis.com/books/v1/volumes?q=ほんきで学ぶAndroidアプリ開発入門").build();
        // データの取得後の命令を実装
        Callback callBack = new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                // 失敗した時の命令
                // 通信に失敗した原因をログに出力
                Log.e("failure API Response", e.getLocalizedMessage());
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                // 成功した時の命令
                // Google Books APIから取得したデータをログに出力
                //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓修正↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
                // Jsonのパースが失敗してアプリの強制終了を回避する機能
                try {
                    // JsonデータをJSONObjectに変換
                    JSONObject rootJson = new JSONObject(response.body().string());
                    // Jsonデータから蔵書リストデータ"items"を取得
                    JSONArray items = rootJson.getJSONArray("items");
                    Log.d("Success API Response", "APIから取得したデータの件数:" +
                            items.length());
                    // 蔵書リストの件数分繰り返しタイトルをログ出力する
                    for (int i = 0; i < items.length(); i ++) {
                        // 蔵書リストから i番目のデータを取得
                        JSONObject item = items.getJSONObject(i);
                        // 蔵書のi番目データから蔵書情報のグループを取得
                        JSONObject volumeInfo = item.getJSONObject("volumeInfo");
                        // 繰り返しの番号と蔵書のタイトルをログに出力
                        Log.d("Response Item Title", (i + 1) + "番目のデータタイトル：" + volumeInfo.getString("title"));
                    }
                } catch (JSONException e) {
                    // Jsonパースの時にエラーが発生したらログに出力する
                    e.printStackTrace();
                }
                //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑修正↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
            }
        };
        // 非同期処理でAPI通信を実行
        okHttpClient.newCall(request).enqueue(callBack);
```
上記コードを実装したら動作確認します。

Json形式のデータはプログラムにおいて文字列データです。
文字列の形式を元に**KeyValue**データとしてパース(解析)するために`JSONObject`クラス、配列の場合は`JSONArray`クラスを使って実装します。
まずはJson形式文字列を`JSONObject`のインスタンス化する時の引数にセットすることで、
`JSONObject`は引数のJson形式文字列データを**KeyValue**形式の`JSONObject`に変換します、今回のコードでは下記の１行が該当します。
```java
    JSONObject rootJson = new JSONObject(response.body().string());
```
次に**KeyValue**データのKey情報を使って *"title"*のデータを取得していきます。
そのための階層をWebブラウザで確認すると
{% img /android/07-AsyncProcess/jsonparse.png 500 Json Parse %}

> {}(rootObject) -> items(JSONArray) -> volumeInfo(JSONObject) -> title(String)

の位置に表示されているのがわかりますので順次１階層毎に参照していきます。
１階層下の子要素を参照するメソッドは各データ型毎に用意されており、間違ったメソッドで参照すると正常に解析できず強制終了してしまいます。
ですがjava言語も優しさは残っており、予期せぬ不具合を検知する機能として`try〜catch`構文という強制終了を防ぐプログラムの記述方法があり、上記のコードはそれを適用しています。
`try {}`の波括弧内に不具合が発生しそうなコードを実装し、後ろの`catch() {}`で不具合が発生した時に実行するコードを記述することができます。
`catch()`の引数には発生しそうな不具合の種類を指定しなければならず、今回は"JSONException"とJSONのパースに失敗した時に不具合をキャッチしてくれます。

## 検索結果画面データ反映
データの解析方法まで試せましたが、「インターネット通信はMainThreadではできなかったんじゃないの？」と感じた方もいるかもしれません。
通信処理を実行した以下の`enqueue()`メソッドでサブスレッドを起動しているのです、開発側すら非同期を気にせず通信が実装できてしまいました。
```
    okHttpClient.newCall(request).enqueue(callBack);
```
サブスレッド通信であるので通信後のCallBackもサブスレッドで実行します、そのため`CallBack > onResponse()`メソッド内でListViewを更新することはできません。
ここで "Handler"クラスが必要になります。
ここからの実装で"onResponse()"メソッドからメインスレッドにデータを連携しListViewに反映していきます。
```java ResultListAdapter.java
public class ResultListAdapter extends BaseAdapter {

    // ListViewの描画に必要な変数を宣言
    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓修正↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    private List<String> titleList;
    private List<String> summaryList;
    private LayoutInflater layoutInflater;
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑修正↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

    // コンストラクタ(インスタンス時に呼び出されるメソッドのようなもの)
    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓修正↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    public ResultListAdapter(Context context, List<String> titleList, List<String> summaryList) {
        this.titleList = titleList;
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑修正↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
        this.summaryList = summaryList;
        this.layoutInflater = LayoutInflater.from(context);
    }

    @Override
    public int getCount() {
        // 一覧表示する要素数を返却する
        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓修正↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        return titleList.size();
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑修正↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    }

    ...一部省略
    @Override
    public View getView(int i, View view, ViewGroup viewGroup) {
        // 各行の表示レイアウト読み込みや、描画情報の設定を実装する
        // getViewで返却されたViewがListViewに表示される

        // viewの中身が空かチェック
        if (view == null) {
            // viewがレイアウトを読み込んでいない場合は"row_result_list"を読み込む
            view = layoutInflater.inflate(R.layout.row_result_list, viewGroup, false);
        }

        // row_result_listのTitleとSummaryに文言を代入
        TextView titleView = view.findViewById(R.id.RowListTitle);
        TextView summaryView = view.findViewById(R.id.RowListSummary);

        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓修正↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        titleView.setText(titleList.get(i));
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑修正↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
        summaryView.setText(summaryList.get(i));

        // 文字情報を代入されたviewを返却
        return view;
    }
}
```

```java ResultListActivity.java
public class ResultListActivity extends AppCompatActivity implements AdapterView.OnItemClickListener{

    ...一部省略
    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓削除↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    // 検証用コレクションデータ
    List<String> listData = Arrays.asList("Android アプリ開発の環境構築"
            , "Android OS とは"
            , "Androidの概念"
            , "Androidアプリ開発を始める"
            , "検索画面レイアウト作成"
            , "ボタンイベントの実装"
            , "検索結果画面への遷移実装"
            , "非同期処理、REST API通信の実装"
            , "検索履歴機能"
            , "Firebase導入");
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑削除↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    // ListViewの表示内容を管理するクラス
    private ResultListAdapter adapter;
    // OkHttp通信クライアント
    private OkHttpClient okHttpClient;
    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    // メインスレッドに戻ってくるためのHandler
    private Handler handler;
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_result_list);

        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // Handlerをインスタンス化
        handler = new Handler();
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
        // xmlファイルのコンポーネントと関連付け
        resultListView = findViewById(R.id.ResultListView);

        ...一部省略
            @Override
            public void onResponse(Call call, Response response) throws IOException {
                // 成功した時の命令
                // Google Books APIから取得したデータをログに出力
                // Jsonのパースが失敗してアプリの強制終了を回避する機能
                try {
                    // JsonデータをJSONObjectに変換
                    JSONObject rootJson = new JSONObject(response.body().string());
                    // Jsonデータから蔵書リストデータ"items"を取得
                    JSONArray items = rootJson.getJSONArray("items");
                    Log.d("Success API Response", "APIから取得したデータの件数:" +
                            items.length());
                    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓修正↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
                    // メインスレッドで実行する処理をインスタンス化
                    ReflectResult reflectResult = new ReflectResult(items);
                    // Handlerにてメインスレッドに処理を戻し、ReflectResultのrunメソッドを実行する
                    handler.post(reflectResult);
                    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑修正↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
                } catch (JSONException e) {
                    // Jsonパースの時にエラーが発生したらログに出力する
                    e.printStackTrace();
                }
            }
        };
        // 非同期処理でAPI通信を実行
        okHttpClient.newCall(request).enqueue(callBack);
    }

    ...一部省略

    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    // 検索結果をListViewに反映するメインスレッドの処理クラス
    private class ReflectResult implements Runnable {
        // 蔵書一覧タイトルデータリスト
        private List<String> titleList;
        // 蔵書一覧概要データリスト
        private List<String> summaryList;

        // コンストラクタ
        public ReflectResult(JSONArray items) {
            // リストデータを初期化
            titleList = new ArrayList<>();
            summaryList = new ArrayList<>();
            // Jsonのパースエラーが発生した時に備えてtry~catchする
            try{
                // 蔵書リストの件数分繰り返しタイトルをログ出力する
                for (int i = 0; i < items.length(); i ++) {
                    // 蔵書リストから i番目のデータを取得
                    JSONObject item = items.getJSONObject(i);
                    // 蔵書のi番目データから蔵書情報のグループを取得
                    JSONObject volumeInfo = item.getJSONObject("volumeInfo");
                    // タイトルデータをリストに追加
                    titleList.add(volumeInfo.getString("title"));
                    // 概要データをリストに追加
                    summaryList.add(volumeInfo.getString("description"));
                }
            } catch (JSONException e) {
                e.printStackTrace();
            }
        }

        // Handlerから実行されるメソッド
        @Override
        public void run() {
            // ListViewに表示する情報をまとめるAdapterをインスタンス化
            adapter = new ResultListAdapter(ResultListActivity.this, titleList, summaryList);
            // ListViewに表示情報をまとめたAdapterをセット
            resultListView.setAdapter(adapter);
            // ListViewに行をクリックした時のイベントを登録
            resultListView.setOnItemClickListener(ResultListActivity.this);
        }
    }
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
}
```
上記コードが実装できたら動作確認します。
以下の様にGoogle Books APIにて取得した検索結果が表示されましたか？
{% img /android/07-AsyncProcess/reflectresult.png 300 Reflect Result %}

今回の修正で`ResultListAdapter`クラスの変数がタイトルリストと概要リストの２つに増えたことで初期化時のコンストラクタで必要になる引数もタイトルと概要データの２つに増えました。
さらに`ResultListActivity`ではタイマースレッドでメインスレッドに戻った時の処理とは実装方法が違い、内部に新しく`Runnable`インターフェースを持ったクラスを作成し、そのクラスでメインスレッドに戻った時の処理を実装しています。
新しく作成した"ReflectResult"クラスはコンストラクタでJSONデータのパースの続きを行い、全タイトルデータと全概要データを新しいリスト変数(titleList, summaryList)に代入しています。
そして全タイトルデータと全概要データを使ってListViewに表示する行データを作成する様に修正しました。

アプリは非同期通信を行い検索結果を取得してからリストを表示しているため、一時的に真っ白な画面が表示される様になりました、他のアプリでは真っ白になる時間をプログレスバーやローディングアニメーションにてユーザに読み込み中であることを伝えます。
ローディングアニメーションなどの実装方法は先のページで解説していきます。

## 入力文字列で蔵書検索する
先ほどまでは決まった文字列("ほんきで学ぶAndroidアプリ開発入門")でしか検索できませんが、
動的に検索が行える様に、`MainActivity`の"EditText"で入力された文字列を`ResultListActivity`に渡しGoogle Books APIの検索条件になる様にプログラムを修正していきます。
```java MainActivity.java
    ...一部省略
            @Override
            public void onClick(View view) {
                // コンソールログにボタンが押されたことを出力(表示)
                Log.d("BookSearchBtn", "onClick: BookSearch Button");
                // 入力された文字をToast(トースト)に表示
                Toast.makeText(getBaseContext()
                        , "入力された文字は [" + bookSearchEditor.getText().toString() + "]です。"
                        , Toast.LENGTH_LONG).show();
                // Timerスレッドを止める
                timer.cancel();
                // 画面遷移するためのIntentをインスタンス化
                Intent intent = new Intent(MainActivity.this, ResultListActivity.class);
                //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
                // EditTextに入力された文字列を"KeyValuePair"でResultListActivityに渡す
                intent.putExtra("terms", bookSearchEditor.getText().toString());
                //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
                // 画面遷移アクションを実行
                startActivity(intent);
            }
    ...一部省略
```
```java ResultListActivity.java
public class ResultListActivity extends AppCompatActivity implements AdapterView.OnItemClickListener{

    // xmlファイルのコンポーネントと関連付ける要素
    private ListView resultListView;
    // ListViewの表示内容を管理するクラス
    private ResultListAdapter adapter;
    // OkHttp通信クライアント
    private OkHttpClient okHttpClient;
    // メインスレッドに戻ってくるためのHandler
    private Handler handler;
    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    // MainActivityから渡されたデータを保持する
    private String term;
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

    @Override
    protected void onCreate(Bundle savedInstanceState) {

        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_result_list);

        // Handlerをインスタンス化
        handler = new Handler();
        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // 画面遷移時のデータが空でない場合
        if (getIntent().hasExtra("terms")) {
            // Key:termsにデータがあればValueを代入
            term = getIntent().getStringExtra("terms");
        } else {
            // 画面遷移時のデータがからの場合は "Android"と文字列を代入
            term = "Android";
        }
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

        // xmlファイルのコンポーネントと関連付け
        resultListView = findViewById(R.id.ResultListView);
        // OkHttp通信クライアントをインスタンス化
        okHttpClient = new OkHttpClient();
        // 通信するための情報
        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓修正↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // MainActivityで入力された文字列で検索する様修正
        Request request = new Request.Builder().url("https://www.googleapis.com/books/v1/volumes?q=" + term).build();
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑修正↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

        ...一部省略
    }
    ...一部省略
```
上記コードを実装したら動作確認します。
`MainActivity`で入力された文字列で検索した結果が表示される様になったでしょうか？
アプリ開発をしていると画面を分割したいけど、データを渡したい！！なんてことも出てくるので"画面遷移時にデータを渡せる"ということだけ覚えておけばインターネット上にも情報は沢山あるので見直して実装することができれば問題ありません。

多少インターネット通信中の真っ白な画面が不恰好ではありますが、検索結果を画面に一覧表示することができました。

# プログラム課題
今回作成したプログラムでは検索処理を実行した際に漫画などの一部検索を行うとアプリが強制終了してしまうケースがあります。
原因は取得したデータの中に"description"というキーが存在せず、解析に失敗しているためです、これを解決するために、"description"データがあるかチェックする処理を追加し、"description"が存在しない場合は空文字データをリストに追加するようにプログラムを修正してください。

<font color="red">**ヒント**</font>
* 空文字をString変数に代入する方法は`変数名 = "";`です
* JSONObject内にキーが存在するかチェックするメソッドがあります
    * `JSONObject.has("key name");`

解説は時間をおいて行います。

以上で**蔵書検索機能の作成**は完了です。
次の[検索履歴一覧画面](/AndroidCourse/android/08-AppDataBase)の作成ではアプリ内データベースを使って検索した履歴一覧を表示する画面を作成します。

[※1]: https://qiita.com/TakahiRoyte/items/949f4e88caecb02119aa
[OkHttp]: http://square.github.io/okhttp/