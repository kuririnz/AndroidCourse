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
    Button bookSearchBtn;
    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    Button historyBtn;
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    EditText bookSearchEditor;
    // Timerクラス
    Timer timer;
    // メインスレッドに帰って来るためのハンドラー
    Handler handler;

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
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

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
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
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
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    }
```
続いて**Module**階層の`build.gradle`も修正します。
{% img /android/08-AppDataBase/includeRealm02.png 450 Include Realm %}
```gradle build.gradle(Module: app)
apply plugin: 'com.android.application'
//↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
apply plugin: 'realm-android'
//↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
android {
    compileSdkVersion 26
    defaultConfig {...
    }
    ...
}
```
２つの`build.gradle`コードを修正したら右上に表示された<font color="blue">Sync Now</font>リンクをクリックします。
{% img /android/08-AppDataBase/includeRealm03.png 600 Include Realm %}
これで**Realm**ライブラリの導入は完了です。

# 検索履歴機能

[Realm]: https://realm.io/docs/java/latest