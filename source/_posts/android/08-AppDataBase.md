---
title: 検索履歴一覧画面の作成
lang: android
date: 2017-11-09
tags:
---
検索履歴一覧画面を作成/実装しながらアプリ内データベースの利用方法を学習する。

<!-- toc -->

[蔵書検索機能の作成](/AndroidCourse/android/07-AsyncProcess)からの引き続きの学習ページです。
# 学習ポイント
* Realmデータベースの使い方
* RecyclerView, RecyclerViewAdapterの使い方

Realmというデータベースライブラリを使いアプリが閉じたり端末電源が落ちた後でもデータが残る機能を使って検索を行った条件を保存しておき、過去の履歴が確認できる画面を作っていきます。
また、検索結果画面で使ったListViewとは別にRecyclerViewというListViewよりも自由度の高い一覧表示ができるコンポーネントを学習します。

# 新しい画面を作成する
検索履歴一覧画面の実装を進めるために、新しくActivityを作成します。

検索結果画面の時と同様にメニューもしくはプロジェクトツリーが表示されているウィンドウで右クリック(macでは２本指でクリック)から新しいActivityの追加を行います。

新しく作成するActivityは**Empty Activity**を選びます。
* メニュー > File > New > Activity > Empty Activity
{% img /android/06-TransitionScreen/createactivity01_1.png 500 CreateNewActivity %}
* ウィンドウ右クリック > New > Activity > Empty Activity
{% img /android/06-TransitionScreen/createactivity01_2.png 500 CreateNewActivity %}

|項目|設定値|
|-------------|------------------|
|Activity Name|HistoryActivity|
|Generate Layout File|チェックを<font color="blue">つける</font>|
|Layout Name|activity_history|
|Luncher Activity|チェックを<font color="red">つけない</font>|
|Backwords Compatibility(AppCompat)|チェックを<font color="blue">つける</font>|

{% img /android/08-AppDataBase/crehistactivity.png 500 CreateNewActivity %}
項目の入力が終わったら`Finish`ボタンをクリックします。

AndroidStudioでは作成したActivityのエディタ画面が表示されます。

# 画面遷移処理機能
検索履歴一覧画面、`HistoryActivity`ができたので`MainActivity`からの画面遷移処理を実装します。
"検索履歴"ボタンはまだ画面との関連付けを実装していないので、関連付けから実装します。
```java MainActivity.java
public class MainActivity extends AppCompatActivity {

    // レイアウトxmlと関連付けるWidget
    private Button bookSearchBtn;
    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    private Button historyBtn;
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    private EditText bookSearchEditor;
    // Timerクラス
    private Timer timer;
    // メインスレッドに帰って来るためのハンドラー
    private Handler handler;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // ハンドラーオブジェクトをMainThreadでインスタンス化
        handler = new Handler();
        // 蔵書検索ボタンをjavaプログラムで操作できるように名前をつける
        bookSearchBtn = findViewById(R.id.BookSearchBtn);
        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // 検索履歴ボタンを関連付ける
        historyBtn = findViewById(R.id.HistoryBtn);
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

		...一部省略

        // 蔵書検索ボタンが押された時に実行するプログラムをボタンに登録
        bookSearchBtn.setOnClickListener(bookSearchEvent);

        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // 検索履歴ボタンをクリックした時の処理を実装
        historyBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                // 検索履歴画面へ遷移するためのIntentをインスタンス化
                Intent intent = new Intent(MainActivity.this, HistoryActivity.class);
                // 画面遷移アクションを実行
                startActivity(intent);
            }
        });
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
		...一部省略
	}
}
```

# Realmデータベースの導入
**[Realm]**データベースは次世代次世代モバイル向けデータベースと言われており、データ抽出速度など基本的な機能が既存のモバイル向けデータベースより優れている点、またデータの登録、更新などの際に必要だったSQLと呼ばれるデータベースを操作する言語の学習も不要になっている点が大きい。

では実際に導入していきます。
ライブラリの導入には`build.gradle`を修正します、今回はプロジェクト階層の`build.gradle`ファイルです。

ファイルの"dependencies"にRealmの設定を追記します。
{% img /android/08-AppDataBase/includeRealm01.png 450 Include Realm %}
```gradle build.gradle(Project: ***)
    dependencies {
        classpath 'com.android.tools.build:gradle:3.0.1'
        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        classpath "io.realm:realm-gradle-plugin:4.3.3"
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    }
```
続いて**Module**階層の`build.gradle`も修正します。
{% img /android/08-AppDataBase/includeRealm02.png 450 Include Realm %}
```gradle build.gradle(Module: app)
apply plugin: 'com.android.application'
//↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
apply plugin: 'realm-android'
//↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
android {
    compileSdkVersion 26
    defaultConfig {...
    }
    ...
}
```
２つの`build.gradle`コードを修正したら右上に表示された<font color="blue">Sync Now</font>リンクをクリックします。
{% img /android/08-AppDataBase/includeRealm03.png 650 Include Realm %}
Gradleの同期処理が実行されるので同期処理の終了にて**Realm**ライブラリの導入は完了です。

# 検索履歴機能
Realmデータベースを使うにはアプリの起動した時（MainActivity）など起動時画面が表示されるより前にRealmのインスタンスを初期化する必要があります。
起動時画面が表示される前にアプリ全体を管理する`Application`というクラスのライフサイクルが実行され、Activityが表示されています。
デフォルトのプロジェクトには`Application`クラスは存在せずAndroid SDK側で自動的に処理されています、そのため必要に応じて`Application`クラスを継承したカスタムクラスを作って実装していきます。
## Applicationクラス作成
新規Javaファイルを作成します
> プロジェクトウィンドウ右クリック > New > Java Class

{% img /android/06-TransitionScreen/createadpt01.png 500 Create Custom Application %}
新しいクラスの作成に関する設定を行ったら`OK`ボタンをクリックします。
{% img /android/08-AppDataBase/initialrealm01.png 500 Create Custom Application %}

|項目          |設定値                            |
|:-----------:|---------------------------------|
|Name         |BDApplication                    |
|Kind         |Class                            |
|Superclass   |android.app.Application          |
|Interface(s) |-                                |
|Package      |\*\*\*\.\*\*\*\.bookbookdiscovery|
|Visibility   |Public                           |
|Modifiers    |None                             |

`Application`クラスが作成されたら、Realmのインスタンス化を実装します。
エラーはないですが、メソッドが実装されていないのでデベロッパが実装する必要があります。
Realmのインスタンス化までを実装します。
**開発時の便利ショートカットの紹介です。**
コンストラクタや継承メソッドやインターフェースメソッド(インターフェースは後で解説します)を自動的に実装してくれる*"Generate"*ショートカットがありますOSごとにコマンドは違いますが、`BDApplication.java`を表示して以下のコマンドを入力します。
* Windowsの場合：<kbd>Alt</kbd><kbd>+</kbd><kbd>Insert</kbd>
* Macの場合：<kbd>command</kbd><kbd>+</kbd><kbd>n</kbd>

以下のウィンドウが表示されるので、今回は***Override Methoeds...***を選択します。
{% img /android/08-AppDataBase/initialrealm02.png 400 Create Custom Application %}
次のウィンドウでは***onCreate():void***を選択し`OK`をクリックします。
{% img /android/08-AppDataBase/initialrealm03.png 300 Create Custom Application %}
以下の様に選択したメソッドがクラスに実装されます。
{% img /android/08-AppDataBase/initialrealm04.png 500 Create Custom Application %}

ではRealmインスタンスを初期化していきます。
```java BDApplication.java
public class BDApplication extends Application {

    @Override
    public void onCreate() {
        super.onCreate();

        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // Realmをインスタンス化
        Realm.init(this);
        // Realm データベースを設定
        RealmConfiguration conf = new RealmConfiguration.Builder().name("BookDiscovery.realm").build();
        Realm.setDefaultConfiguration(conf);
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    }
}
```

作成したカスタムApplicationクラスはこのままでは実行されません、Activityよりも階層にしてみれば上に位置するものなのでActivityからも設定することはできません。
ここで出てくるのが`AndroidManifest.xml`です、API通信の実装の時にはインターネット通信を許可する設定を追記しました、今回は実行するApplicationクラスを設定を変更します。
```XML AndroidManifest.xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="kuririnz.xyz.bookdiscovery">

    <!-- Androidアプリでインターネット通信を許可する設定 -->
    <uses-permission android:name="android.permission.INTERNET" />

    <application
        <!-- ↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓ -->
        android:name=".BDApplication"
        <!-- ↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑ -->
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        <!-- 一部省略  -->
    </application>
     <!-- 一部省略  -->
</manifest>
```
上記で初期化、および`カスタムApplication`クラスの反映が完了しました。

## RealmObject作成
通常データベースにはテーブルという概念があります、テーブルとは複数の項目とその項目に沿ったデータを複数もった表のようなもので、表で再現すると以下のようなデータの持ち方をしています。

|検索日付        |検索文字列          |
|:--------------|------------------|
|2018/2/6 13:02 |東京都             |
|2018/2/6 13:25 |Android入門        |
|2018/2/6 14:03 |ごはんのお供        |
|2018/2/13 16:46|ミステリー小説      |
|2018/2/13 17:11|プログラマー三代美徳 |

例えば上記の表を*SearchHistory*というテーブルとした場合
{% blockquote %}
*SearchHistory*テーブル[表]は"検索日付"、"検索文字列"というカラム[列、項目]を持ち、現状は５ロウ(Row)[行]分のデータを持っている。
{% endblockquote %}
と言い換えることができます。

Realmでは上記のテーブルを`RealmObject`クラスを継承したカスタムクラスを作成するだけでテーブル構成が完成します。
今回作成する検索履歴機能は上記の表と同じ構成で作成します、新規javaファイルの作成から実装していきましょう。
> プロジェクトウィンドウ右クリック > New > Java Class

{% img /android/06-TransitionScreen/createadpt01.png 500 Create Custom Application %}
新しいクラスの作成に関する設定を行ったら`OK`ボタンをクリックします。
{% img /android/08-AppDataBase/initialrealm05.png 500 Create Custom Application %}

|項目          |設定値                            |
|:-----------:|---------------------------------|
|Name         |SearchHistoryModel               |
|Kind         |Class                            |
|Superclass   |io.realm.RealmObject             |
|Interface(s) |-                                |
|Package      |\*\*\*\.\*\*\*\.bookbookdiscovery|
|Visibility   |Public                           |
|Modifiers    |None                             |

上記で作成された`SearchHistoryModel`クラスがテーブル（表）になります。
次にカラム(列)を作っていきます。
```java SearchHistoryModel.java
public class SearchHistoryModel extends RealmObject {

    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    // 検索日時カラム
    private String searchDate;
    // 検索文字列カラム
    private String searchTerm;
	//↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
}
```
次に各カラムのデータを参照、代入するメソッドを実装します。
それぞれ参照を**ゲッター(getter)**、代入を**セッター(setter)**と呼びます。
`SearchHistoryModel`クラスを開いた状態で、*"Generate"*ショートカットを入力して**Getter and Setter**を選択します。
{% img /android/08-AppDataBase/realmtable01.png 400 Create Custom Table %}
表示された２項目を選択し`OK`をクリックします。
{% img /android/08-AppDataBase/realmtable02.png 300 Create Custom Table %}
以下の通り４つのメソッドが追加されれば完了です。
{% img /android/08-AppDataBase/realmtable03.png 550 Create Custom Table %}

このように複数のデータを管理するためのクラスを**モデルクラス**と呼びます。
リスト構造の表示を行いたいときなどモデルクラスのように表示したい項目をまとめて保存しておくことでデータの管理が簡潔になり、プログラムが未訳なります。

## 履歴登録機能
登録先となるRealmObjectが完成したので、実際にRealmDBへの登録処理を実装します。
今回データの登録タイミングは検索結果画面へ遷移する前とします。
```java MainActivity.java
    ...一部省略
    @Override
    protected void onCreate(Bundle savedInstanceState) {

        ...一部省略
        // 蔵書検索ボタンをクリックした時の処理を実装
        View.OnClickListener bookSearchEvent = new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                // コンソールログにボタンが押されたことを出力(表示)
                Log.d("BookSearchBtn", "onClick: BookSearch Button");
                // EditTextの文字列を取得
                String termString = bookSearchEditor.getText().toString();
                // Timerスレッドを止める
                timer.cancel();
                //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
                // Realmインスタンスを生成
                Realm realm = Realm.getDefaultInstance();
                try {
                    // 検索履歴テーブルへのアクセスを開始
                    realm.beginTransaction();
                    // 新規検索履歴データを作成
                    SearchHistoryModel history = realm.createObject(SearchHistoryModel.class);
                    // 検索文字列カラムにデータを登録
                    history.setSearchTerm(termString);
                    // 現在時刻を文字列で取得する
                    Date now = new Date();
                    // 現在時刻を定まった形式で文字列に変換
                    String dateStr = new SimpleDateFormat("yyyy/MM/dd HH:mm").format(now);
                    // 現在日時の文字列をカラムデータに登録
                    history.setSearchDate(dateStr);
                    // 検索履歴テーブルへのアクセスを終了
                    realm.commitTransaction();
                } finally {
                    // Realmインスタンスがちゃんとクローズされること
                    realm.close();
                }
                //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
                
                // 検索結果画面へ遷移するためのIntentをインスタンス化
                Intent intent = new Intent(MainActivity.this, ResultListActivity.class);
                // EditTextに入力された文字列を"KeyValuePair"でResultListActivityに渡す
                intent.putExtra("terms", termString);
                // 画面遷移アクションを実行
                startActivity(intent);
            }
        };
        // 蔵書検索ボタンが押された時に実行するプログラムをボタンに登録
        bookSearchBtn.setOnClickListener(bookSearchEvent);
        ...一部省略
    }
```
上記の実装でRealm DBへのデータ登録処理は完了です。
モバイル向けデータベースのデメリットとして中身を直接確認しにくいという点があるのですが、これはRealmDBも同様です。
RealmにはGUIでデータを確認するアプリがありますが、データを保存した`BookDiscovery.realm`を取り出すのに手間がかかるため、実際に読み込んで参照した方がデータの確認は早いのです。

データ確認のため、検索履歴一覧画面にTextViewを配置しRealmから`SearchHistoryModel`の内容を全て抽出し、表示してみます。
検索履歴一覧画面のレイアウトファイルを開きます。
> app -> res -> layout -> activity_history.xml

`activity_history.xml`を表示したら、"TextView"を画面に配置します。
{% img /android/08-AppDataBase/confirmrealm01.png 550 realmconfirm %}
画面の上下左右と制約を設定します。
{% img /android/08-AppDataBase/confirmrealm02.png 550 realmconfirm %}
最後に**Attributes**エリアから属性を設定します。
{% img /android/08-AppDataBase/confirmrealm03.png 550 realmconfirm %}

|設定項目       |設定値            |
|--------------|-----------------|
|ID            |HistoryText      |
|layout_width  |match_sonstraint |
|text          |テキストなし       |

上記の表の通り、入力が完了したらレイアウトは一旦OKです！
最後に`HistoryActivity.java`でRealm Realm DBからデータを取得し、配置したTextViewに表示させていきます。
```java HistoryActivity.java
public class HistoryActivity extends AppCompatActivity {

    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    // Realmインスタンスを宣言
    private Realm realm;
    // 画面紐付けコンポーネントを宣言
    private TextView historyTextView;
    // 検索履歴テーブルのデータを全て取得
    private RealmResults<SearchHistoryModel> resultData;
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_history);

        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // 画面コンポーネント関連付け
        historyTextView = findViewById(R.id.HistoryText);
        // Realmクラスをインスタンス化
        realm = Realm.getDefaultInstance();
        // 検索履歴テーブルのデータを全て取得
        resultData = realm.where(SearchHistoryModel.class).findAll();
        // 検索履歴の件数が１件以上なら繰り返し処理でTextViewに表示する
        if (!resultData.isEmpty() && resultData.size() > 0) {
            // 検索履歴テーブルの行数分、繰り返し処理を実行する
            for (int i = 0; i < resultData.size(); i++) {
                // 検索履歴画面のtextViewに検索文字列を随時結合して表示する
                historyTextView.setText(historyTextView.getText() + resultData.get(i).getSearchTerm());
            }
        }
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    }
    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    @Override
    protected void onDestroy() {
        super.onDestroy();
        // Realmインスタンスをちゃんとクローズすること
        realm.close();
    }
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
}
```
上記のコードを実装したら動作確認します。
動作確認の手順として以下の手順で操作を行い、[1.]のタイミングに入力した文字列が検索履歴一覧画面に表示されれば成功です。
1. EditTextに何かしらの文字列を入力
1. "蔵書検索"ボタンをクリック
1. Androidエミュレータ(または実機)のバックボタンをクリック
1. "検索履歴"ボタンをクリック

Realmデータベースの基礎的な使い方は以上です。

上記のコードでは新しく`onDestroy`というライフサイクルメソッドが登場しました、この`onDestroy`はActivityを表示するために保持しておいた領域を解放する前に実行されるメソッドです。
RealmインスタンスはActivityごとにインスタンス化して使用しているため、Activityが解放される前にRealmの領域も解放しないといけません、**Realmの解放を忘れるとアプリの強制終了の原因にもなり得る**のでRealmを利用する場合は注意して実装を行います。

# 検索履歴一覧画面リスト表示対応
Realmにて検索した文字列が正常に保存されていることが確認できましたが、現状とても見難いレイアウトです、これをリスト表示にしていきます。

検索履歴一覧画面での一覧表示には"ListView"ではなく、**RecyclerView**というコンポーネントを利用します。

**RecyclerView**は"Android API 22"の頃に追加されたコンポーネントでListViewや格子状にViewを表示するGridViewよりさらに自由な配置を行えるようになった一覧表示コンポーネントです。
またListViewやGridViewに比べ軽い動作が行えることから最近ではこちらのコンポーネントで一覧画面を作ることが増えています。

## 検索履歴一覧画面レイアウト修正
**RecyclerView**コンポーネントは以下の場所にあります。
> Palette -> AppCmpat -> RecyclerView

{% img /android/08-AppDataBase/addrecycler01.png 450 Add RecyclerView %}
続いて制約の設定です。
コンポーネントが端に寄っていて設定しにくい場合、少し上下させて右端/左端にドラッグ&ドロップするとやりやすいです。
{% img /android/08-AppDataBase/addrecycler02.png 300 Add RecyclerView %}
制約設定が終わったら**RecyclerView**の属性を設定していきます。
{% img /android/08-AppDataBase/addrecycler03.png 450 Add RecyclerView %}

|設定項目       |設定値                |
|--------------|---------------------|
|ID            |HistoryRecycler      |
|layout_width  |match_constraint     |
|layout_height |match_constraint     |

先ほど表示していたTextView(ID:HistoryText)は履歴が０件だった場合のメッセージを表示するように修正します、TextViewの用途が変わるのでIDも変更します。
{% img /android/08-AppDataBase/addrecycler04.png 450 Add RecyclerView %}

|設定項目       |設定値                |
|--------------|---------------------|
|ID            |EmptyRecyclerText    |
|text          |検索履歴はありません    |
|visibility    |invisible            |

テキストの配置が不恰好なので位置を調整します、TextViewの文字位置の調整は"textAlignment"属性を修正します。
"textAlignment"属性は基本属性の画面には表示されておらず、全属性を表示する詳細ウィンドウに切り替える必要があります、詳細ウィンドウへの切り替えはAttributesウィンドウの左右に矢印のアイコンをクリックします。
{% img /android/08-AppDataBase/addrecycler04.png 450 Add RecyclerView %}
"textAlignment"属性を探し、"center"を選択して完了です。

## 検索歴一覧画面一覧表示実装
検索履歴一覧画面の行レイアウトを作成していきます。
新規でxmlレイアウトファイルを作成し、検索日時と検索時の文字列を表示する行レイアウトを作成します。
> プロジェクトウィンドウ右クリック > New > XML > Layout XML File

{% img /android/08-AppDataBase/addrowrecycler01.png 450 Add RecyclerView Row %}
xmlレイアウトファイル名と一番上の階層になるLayoutを設定して`Finish`をクリックします
{% img /android/08-AppDataBase/addrowrecycler02.png 450 Add RecyclerView Row %}
デザインビューが表示されたらXMLの編集画面に切り替えてレイアウトを実装していきます。
デザインエリアの左したにある**Text**と書かれてるタブをクリックします。
{% img /android/08-AppDataBase/addrowrecycler03.png 450 Add RecyclerView Row %}
検索履歴一覧画面の行レイアウトはLinearLayoutという初期からあるレイアウトコンポーネントを使います、またXMLを使った実装も試していきます。
```XML row_history_recycle.xml
<?xml version="1.0" encoding="utf-8"?>
<!-- ↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓修正↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓ -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical">
<!-- ↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑修正↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑ -->

    <!-- ↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓ -->
    <TextView
        android:id="@+id/RowHistoryDate"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_margin="8dp"
        android:textSize="14dp"
        tools:text="蔵書検索した日付を表示"/>

    <TextView
        android:id="@+id/RowHistoryTerm"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_margin="8dp"
        android:textSize="20dp"
        tools:text="蔵書検索した文字列を表示"/>
    <!-- ↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑ -->
</LinearLayout>
```
toolsを設定する際にもjavaクラスでのimportと同じように自動的に追加が可能ですので、`tools:text..`と入力した後に`tools`にマウスカーソルでクリックし、以下のようにツールチップが表示されたら
Windowsの場合は[Alt + insert],Macの場合は[option + Enter]でxml内での参照が追加されます。
{% img /android/08-AppDataBase/addrowrecycler04.png 500 Add RecyclerView Row %}

この`tools`という属性ですが、プレビューの際にだけ反映される項目を設定できる属性になります。そのため**アプリを実行した際には追加した２つのTextViewはプログラム上、表示する文字列を持っていない状態**となります。。

そして上記xmlファイルで初めて利用した`LinearLayout`というコンポーネントは縦もしくは横どちらか一方方向に子Viewを並べて表示するという特性を持っています。
縦横の設定は`orientation`属性を使用します。

|設定値     |並べて表示する方向              |
|:---------|:----------------------------|
|horizontal|横(左から右)へ子Viewを並べて表示 |
|vertical  |縦(上から下)へ子Viewを並べて表示 |

`orientation`属性が設定されていない場合は**horizontal**が設定されたと認識され、不具合通知は表示されないので注意しましょう。

XMLでレイアウトの実装を行う時に気をつけるポイントとして`layout_width`属性、`layout_height`属性があります。
設定できる値とその意味合いをそれぞれ記述します。

|設定値                |設定値の意味                   |
|:--------------------|:----------------------------|
|wrap_content         |要素の表示に必要な領域を自動的に確保 |
|match_parent         |親要素と同じ幅(縦/横)に設定        |
|match_constraint(0dp)|親要素と同じ幅(縦/横)に設定ConstraintLayoutの場合で"Design"タブを開いている場合に設定できる。xml上の値は"0dp" |
|fill_parent          |親要素と同じ幅(縦/横)に設定Android OS 2.2(API Level 8)以降は非推奨 |

アプリ開発の上で端末の種類が多いAndroidでは固定値を設定することはほぼレイアウト崩れに直結するので基本的には"wrap_content"や"match_parent"を使い隣接するViewとのスペース設定で調整することになると思います。

行レイアウトができたので行ごとの表示プロセスを担当するAdapterクラスを実装していきます。
一覧表示を行うViewを使う場合、Adapterクラスを継承して独自のレイアウト表示する方法はListViewと一緒です。
RecyclerViewには`RecyclerView.Adapter`というRecyclerView用のAdapterクラスがあるのでこれを継承した独自クラスを作成します。
{% img /android/08-AppDataBase/addadapter01.png 400 Add RecyclerView Adapter %}
クラス作成時の設定項目を入力したら`OK`をクリックします
{% img /android/08-AppDataBase/addadapter02.png 400 Add RecyclerView Adapter %}
`HistoryRecyclerAdapter.java`が表示されたら<font color="red">赤い電球</font>アイコンをクリックします。
{% img /android/08-AppDataBase/addadapter03.png 500 Add RecyclerView Adapter %}
表示された小さいウィンドウから*Implement methods*をクリックします。
{% img /android/08-AppDataBase/addadapter04.png 500 Add RecyclerView Adapter %}
３つのメソッドが選択状態（背景が青色）になっていることを確認し`OK`をクリックします。
{% img /android/08-AppDataBase/addadapter05.png 300 Add RecyclerView Adapter %}
選択した３つのメソッドが`HistoryRecyclerAdapter.java`に表示されたら新しいクラスの作成は完了です。
{% img /android/08-AppDataBase/addadapter06.png 500 Add RecyclerView Adapter %}
検索結果画面では利用していませんでしたが、検索履歴一覧画面では**ViewHolderパターン**での実装を行います。
> **ViewHolderパターン**とはスクロールするたびにViewを生成/抽出(findViewById)などの処理コストが高く描画に時間がかかってしまうため、一度生成したViewHolderを再利用することでコストを軽減するパターンを指します。

`HistoryRecyclerAdapter`の主なメソッドを実装する前にViewHolderの実装を行なっていきます。
```java HistoryRecyclerAdapter.java
//↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓修正↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
public class HistoryRecyclerAdapter extends RecyclerView.Adapter<HistoryRecyclerAdapter.HistoryHolder> {
//↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑修正↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

    @Override
    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓修正↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        return null;
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑修正↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    }

    @Override
    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓修正↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    public void onBindViewHolder(RecyclerView.ViewHolder holder, int position) {
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑修正↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    }

    @Override
    public int getItemCount() {
        return 0;
    }

    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    // ViewHolderパターンクラス
    class HistoryHolder extends RecyclerView.ViewHolder {
        // Rowレイアウトと関連づけるコンポーネントを宣言
        public TextView historyDate;
        public TextView historyTerm;

        // コンストラクタ
        public HistoryHolder(View itemView) {
            super(itemView);
            // xmlファイルと関連付け
            historyDate = itemView.findViewById(R.id.RowHistoryDate);
            historyTerm = itemView.findViewById(R.id.RowHistoryTerm);
        }
    }
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
}
```
上記の実装で**ViewHolderパターン**での実装は完了です。
続いて履歴データを表示するための実装を進めます。
修正箇所等かなり多いので追加、修正箇所に注意しながら実装を進めてください。
```java HistoryRecyclerAdapter.java
public class HistoryRecyclerAdapter extends RecyclerView.Adapter<HistoryRecyclerAdapter.HistoryHolder> {

    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    // 表示に必要なクラスを宣言
    private Context context;
    private LayoutInflater inflater;
    private RealmResults<SearchHistoryModel> historyData;

    // コンストラクタ
    public HistoryRecyclerAdapter(Context context, RealmResults<SearchHistoryModel> historyData) {
        // レイアウトのインスタンス化に必要なクラス
        this.context = context;
        this.inflater = LayoutInflater.from(context);
        // 表示するデータリスト
        this.historyData = historyData;
    }
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓修正↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    // 表示するレイアウトを指定するメソッド
    @Override
    public HistoryHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        // row_history_recycleのレイアウトをインスタンス化
        View view = inflater.inflate(R.layout.row_history_recycle, parent, false);
        // HistoryHolderクラスをインスタンス化して返却する
        return new HistoryHolder(view);
    }

    // 表示するレイアウトにデータを設定するメソッド
    @Override
    public void onBindViewHolder(HistoryHolder holder, int position) {
        // 検索履歴一覧から一つの履歴データを抽出
        SearchHistoryModel history = historyData.get(position);
        // ViewHolderにデータをセット
        holder.historyDate.setText(history.getSearchDate());
        holder.historyTerm.setText(history.getSearchTerm());
    }

    // 表示するリストの件数を指定するメソッド
    @Override
    public int getItemCount() {
        // RealmDBから取得した検索履歴全件分、行の表示処理を繰り返す
        return historyData.size();
    }
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑修正↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

    ...一部省略
}
```

```java HistoryActivity.java
public class HistoryActivity extends AppCompatActivity {

    // Realmインスタンスを宣言
    private Realm realm;
    // 履歴0件用TextView
    private TextView historyTextView;
    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    // 検索履歴RecyclerView
    private RecyclerView historyRecycler;
    // 検索履歴RecyclerAdapter
    private HistoryRecyclerAdapter historyAdapter;
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    // 検索履歴抽出データ
    RealmResults<SearchHistoryModel> resultData;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_history);

        // 画面コンポーネント関連付け
        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓修正↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        historyTextView = findViewById(R.id.EmptyRecyclerText);
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑修正↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
        historyRecycler = findViewById(R.id.HistoryRecycler);
        // Realmクラスをインスタンス化
        realm = Realm.getDefaultInstance();
        // 検索履歴テーブルのデータを全て取得
        resultData = realm.where(SearchHistoryModel.class).findAll();
        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓修正↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // 検索履歴の件数が１件以上なら繰り返し処理でTextViewに表示する
        if (!resultData.isEmpty() && resultData.size() > 0) {
            // adapterクラスをインスタンス化
            historyAdapter = new HistoryRecyclerAdapter(this, resultData);
            // RecyclerViewの表示形式を決める
            RecyclerView.LayoutManager layoutManager = new LinearLayoutManager(this);
            // RecyclerViewの初期設定
            historyRecycler.setAdapter(historyAdapter);
            historyRecycler.setLayoutManager(layoutManager);
        } else {
            // 検索履歴の件数が１件もない場合、履歴0件のメッセージを表示する
            // 検索履歴一覧を非表示に設定
            historyRecycler.setVisibility(View.GONE);
            historyTextView.setVisibility(View.VISIBLE);
        }
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑修正↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    }
    ...一部省略
}
```
最後に`historyTextView`ですが、変数としての意味合いが変わっているので変数名を変更します。
ただ、一箇所づつの変更は手間なので一気に全箇所が変更される方法をで修正します。
変数名にマウスカーソルを当て右クリックから
> Refactor -> Rename...

をクリックします
{% img /android/08-AppDataBase/refactvalid01.png 500 Refactor Valid Name %}
変更する変数名が赤枠で囲われるので変数名を修正します。
{% img /android/08-AppDataBase/refactvalid02.png 500 Refactor Valid Name %}
変数名の修正に合わせて他に使用されている同変数も修正されているのがわかります。
変数名の修正が終わったわ`Enter`キーを押下し完了です。
{% img /android/08-AppDataBase/refactvalid03.png 500 Refactor Valid Name %}

上記までコードを実装したら動作確認します。
閲覧履歴ボタンをクリックし、画面遷移した検索履歴一覧画面に検索した文字列のリストが表示されてるでしょうか？

[Realm]: https://realm.io/docs/java/latest