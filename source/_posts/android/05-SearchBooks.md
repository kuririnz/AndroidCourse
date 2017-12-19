---
title: 非同期処理、REST API通信の実装
date: 2017-11-06
---
非同期処理とAPIを利用した通信処理ライブラリの実装と確認を行います


<!--toc-->

当ページでは検索画面のレイアウト完了から蔵書/図書館検索の結果を取得しログに出力するまでを解説します。
[前のページで検索画面のレイアウト実装を解説しています](/AndroidCourse/android/04-MakeLayoutDesign)
次のページで検索結果の表示、アプリ内データベースの利用に関して解説します。


# 非同期処理
Androidアプリを開発するのに気を付けなければいけない概念として**Thread(スレッド)**という概念があります。
スレッドとは
> 脈絡、筋道。電子メールなどでは、ある一つの話題に関連した投稿の集まり

などの意味合いがありますが、LINEなどでは何かイベントを行う時に作成するグループなどがスレッドに近いと思います。
プログラミングで使われた場合、ニュアンスとしては似ているのですが、
以下のような捉え方をします
> 結果を得るための一連のプログラムを処理する領域

Androidアプリの画面が表示されるのもスレッドが画面を表示するという一連のプログラムを処理した結果Android端末にアプリの画面を表示しています。
このスレッドを**メインスレッド**や**UIスレッド**と表します、またメインスレッドでの通信処理を禁止されており、REST API通信を使う場合にはメインスレッドに対して別のスレッド=**サブスレッド**を使ってAPIからデータを取得したりする必要があります。

細かいポイントは実装を進めながら解説していきますので、
まずはxmlレイアウトとjavaプログラムの関連付けからみていきましょう

## javaプログラムとxmlレイアウトの関連付け
一つ前の[レイアウト作成](/AndroidCourse/android/04-MakeLayoutDesign)の内容で作成したxmlレイアウトですが、
今回プログラムを記述する"MainActivity.java"が処理された時に表示されるプログラムがすでに記述されています。
それが以下のコード内矢印に囲まれてるプログラムでMainActivity.javaが処理された時にレイアウトの読み込みを行なっています。

<font color="RoyalBlue">***MainActivity.java***</font>
```
...
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        //↓↓↓↓↓↓↓↓↓↓↓↓レイアウト読み込み↓↓↓↓↓↓↓↓↓↓↓↓
        setContentView(R.layout.activity_main);
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    }
}
```

次に画面のButtonやEditTextをjavaプログラムに関連付けを行う方法です。
ButtonやEditTextをjavaプログラムと関連づけるにはいくつかの決まった記述方法があります、中でも一番よく使う方法を紹介します。
まずは関連付けたいウィジェットにユニークなID(名称)をつける必要があるため、**activity_main.xml**を開きます。
<img src="rerationcode01.png" alt="alt" title="reration code" width="500">

"蔵書検索"ボタンから見ていきましょう
"蔵書検索"ボタンをクリックしAttributesエリアから<font color="red">**"ID"**</font>項目を以下のように修正します


<img src="setid01.png" alt="alt" title="reration code" width="500">

> button -> BookSearchBtn

他の項目も同じように<font color="red">**"ID"**</font>項目を変更していきます。
※もし変更前IDが一致していなくても問題ないのでNoと照らし合わせて修正してください
<img src="setid03.png" alt="alt" title="reration code" width="500">

|No.  |変更前のID |変更後ID         |
|:---:|:---------|:--------------|
|①   |button    |BookSearchBtn  |
|②   |button2   |LibSearchBtn   |
|③   |button3   |HistoryBtn     |
|④   |editText  |BookSearchEdit |
|⑤   |editText2 |LibSearchEdit  |

IDを変更するとComponent Treeの内容を変更が反映されます
各ウィジェットにユニークIDを設定したら設定したIDを使って"MainActivity.java"のプログラムからウィジェットを呼び出して関連付けを行います
MainActivity.javaを開きます。
<img src="setid04.png" alt="alt" title="reration code" width="500">

<font color="RoyalBlue">***MainActivity.java***</font>
```
...
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //↓↓↓↓↓↓↓↓↓↓↓↓ボタンの関連付け↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        Button button = findViewById(R.id.BookSearchBtn);
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    }
}
```

idの入力を行なっている最中に小さいリスト表示がされると思いますので十字キーの上下を押して選択し、```return```キーを押せばプログラムに反映されます。

<img src="setid05.png" alt="alt" title="reration code" width="650">


## ボタンイベントの実装


## 文字入力イベントの実装

# REST API通信処理

REST APIとは**Representational State Transfer Application Programming Interface**の略称で
それぞれ"REST"と"API"で使われることもあります。

## REST
Webサービスの設計モデルを表しており、REST APIのURLにHTTPメソッドでアクセスすることでデータの送受信が行えます。

RESTの設計条件として以下が該当します。
* アドレス指定可能なURLで公開されていること
* インターフェースが統一されていること(HTTPメソッドに準じていること)
* ステートレスであること
* 処理結果がHTTPステータスコードで通知されること

## API

> ソフトウェアコンポーネントが互いにやり取りするのに使用するインターフェース仕様

とあります、