---
title: ボタンイベントの実装
date: 2017-11-06
---
アプリの基本イベントとなるボタンが押された時の処理を学習し、蔵書検索ボタンの機能を作り始めます。
合わせてコンソールログの出力、Toast(トースト)の機能を試します

<!-- toc -->

当ページではアプリのボタンが押下された時のプログラム実装を学習します。
またボタンや画面の裏で値を保持しておく機能として変数という概念を学習します。
[前のページで検索画面のレイアウト実装を解説しています](/AndroidCourse/android/04-MakeLayoutDesign)
次のページでは非同期処理やAPI通信機能を使い実際に蔵書検索の機能の作り込みを解説します。

# ボタン押下処理
アプリを作る上で一つは必ず存在するボタン。
普段何気なく使っているボタンが押された時、プログラムが実行され別の画面を表示したり、何か計算を行なってユーザに向けて画面に表示するなど
見えないところで色々なプログラムが動いています。
当ページでは画面の操作としてボタンを押した時にログを表示したり、Toast(トースト)と呼ばれるユーザに向けて情報を表示するプログラムの実装を解説していきます。

今回プログラムは「MainActivity.java」というファイルにボタンが押された時のプログラムを記述していきます。
ですが、前のページで画面レイアウトを作成したのが`activity_main.xml`というファイルでした。
別のファイルです...なので「MainActivity.java」に`activity_main.xml`ファイルを取り込む必要があります。
「MainActivity.java」で`activity_main.xml`内のボタンを使う準備を行います。

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
必ず書かないといけないコードを紹介する時に<font color="red">「おまじない」</font>という言葉で紹介されることもありますが、このコードが書かれていないと画面には何も表示されず、真っ白な画面で止まってしまうので必ず記述しましょう。

簡単に図にしてみます。
<img src="LoadLayout.png" alt="alt" title="loadlayout" width="550">

上記のコードが記述されることで画面にレイアウトが表示されていることがわかりました、次にButtonをクリックしたときに検索などの処理を行うにはMainActivity.javaなど**javaファイルでクリックされた時のプログラムを書く**必要があります。
ただ、レイアウトを作成したのは"activity_main.xml"ファイルですので別のファイルです、ですがjavaファイルのプログラムからxmlファイルのボタンなどを関連付けることができます。

そのためには"activity_main.xml"に配置したButtonをMainActivity.javaで関連付け、操作するための名前をつける必要があります。

ButtonやEditTextなどのウィジェットをjavaプログラムから関連付けるにはいくつかの方法があります、一番よく使う方法として<font color="red">**findViewById()**</font>を使う方法です、"setContentView"で読み込んだレイアウトの中からウィジェットやビューを関連付けることができます。

ウィジェットやビューを関連づけるために`activity_main.xml`内のButtonやEditTextなどにユニークID(名称)を設定する必要があります。
ユニークIDの設定はレイアウトファイルで行いますので**activity_main.xml**を開いてください。
<img src="rerationcode01.png" alt="alt" title="reration code" width="500">

はじめに"蔵書検索"ボタンをjavaプログラムから関連づけるためのユニークID(名称)を設定します、
"蔵書検索"ボタンをクリックしAttributesエリアから<font color="red">**"ID"**</font>項目を以下のように修正します
<img src="setid01.png" alt="alt" title="reration code" width="500">

> button -> BookSearchBtn

このAttributesの<font color="red">**"ID"**</font>という項目がjavaプログラムから関連づける時に必要な情報になります。

では他の項目も同じように<font color="red">**"ID"**</font>項目を変更していきます。
※もし変更前IDが一致していなくても、Noと場所の照らし合わせを正として修正してください
<img src="setid03.png" alt="alt" title="reration code" width="500">

|No.  |変更前のID |変更後ID         |
|:---:|:---------|:--------------|
|①   |button    |BookSearchBtn  |
|②   |editText  |BookSearchEdit |
|③   |button2   |HistoryBtn     |

IDを変更するとComponent Treeの内容も変更が反映されます
各ウィジェットにIDが設定できたら"MainActivity.java"にプログラムを追加してウィジェットを`activity_main.xml`内のWidgetなどと関連付け、MainActivity.javaで使う時の名前をつけていきます。

MainActivity.javaを編集します。
<img src="setid04.png" alt="alt" title="reration code" width="500">
では**findViewById**メソッドと**"activity_main.xml"に設定したID**を使ってButtonウィジェットを探します

コード記述する中でIDの入力中に小さいリスト表示がされると思いますので十字キーの上下を押して選択し、`Enter`キーを押せばプログラムに反映されます。

<img src="setid05.png" alt="alt" title="reration code" width="650">
<font color="RoyalBlue">***MainActivity.java***</font>
```
...
public class MainActivity extends AppCompatActivity {

    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    // レイアウトxmlと関連付けるWidget
    Button bookSearchBtn;
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // 蔵書検索ボタンをjavaプログラムで操作できるように名前をつける
        bookSearchBtn = findViewById(R.id.BookSearchBtn);
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    }
}
```

上記追加した２行で<font color="red">activity_main.xmlの"BookSearchBtn"（"蔵書検索"ボタン）を、MainActivityで使う時の名前として`buttonSearch`という名前を設定する</font>というプログラムを記述することができました。
２行の内訳としては以下です
<img src="setid06.png" alt="alt" title="reration code" width="600">

|No.  |各No.のコードの説明                                     |
|:---:|:----------------------------------------------------|
|①   |activity_main.xml内のウィジェットと関連づける型            |
|②   |MainActivity.javaで使う時のユニークな名前                 |
|③   |①〜②で宣言したウィジェットに情報を登録する時の記述方法       |
|④   |④のIDを使いactivity_main.xml内のウィジェット/ビューを取得するプログラム |
|⑤   |activity_main.xml内から取得するウィジェット/ビューのユニークID |

これで"蔵書検索"ボタンをxmlレイアウトとMainActivity.javaのプログラムで関連付けることができました。

今後プログラムを開発していく上で重要な箇所ですので更に深掘りした解説を以下にまとめます
> この①〜⑤の工程はプログラムで表示するデータを動的に保持したり、表示したりする上で大事なクラス（変数）の宣言から初期化を行なっています。
 
> ①〜②で追加したコードを<font color="blue">クラスの宣言</font>と言い、WidgetやView、その他のクラスをjavaプログラムで使うために
わかりやすい名前をつけて準備をしています。

> 本来ボタンオブジェクトは"ボタンの色"、"ボタンに表示する文字"など情報を持っていますが、
①〜②の工程だけでは名前が準備されただけで`bookSearchBtn`は全く情報を持っていません。

> `bookSearchBtn `に情報を登録するには<font color="blue">**オブジェクト化（インスタンス化とも呼ばれます）**</font>という工程が必要になります、
その工程が③〜⑤に該当しオブジェクト化を行なっており、この工程後に`bookSearchBtn`がもつ"ボタンの色"、"ボタンに表示する文字"などの情報を
参照したり変更したりすることが可能になります。

## ボタンクリックアクションの実装
次に"蔵書検索"ボタンのクリックした時にクリックされたことを開発者に確認させるためログを出力するプログラムを作っていきます。
プログラムが上手く動かない時などどこまでプログラムが実行されたかを確認したり、データを確認するためにも使われるので使い方を覚えていきましょう。

<font color="RoyalBlue">***MainActivity.java***</font>
```
...
public class MainActivity extends AppCompatActivity {

    // レイアウトxmlと関連付けるWidget
    Button bookSearchBtn;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // 蔵書検索ボタンをjavaプログラムで操作できるように名前をつける
        bookSearchBtn = findViewById(R.id.BookSearchBtn);
        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // 蔵書検索ボタンが押された時の処理を実装
        View.OnClickListener bookSearchEvent = new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                // コンソールログにボタンが押されたことを出力(表示)
                Log.d("BookSearchBtn", "onClick: BookSearch Button");
            }
        };
        // 蔵書検索ボタンが押された時に実行するプログラムをボタンに登録
        bookSearchBtn.setOnClickListener(bookSearchEvent);
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    }
}
```

上記まで実装が完了したら動作確認をしてみましょう、`Run`アイコンをクリックしてエミュレータを起動します。
アプリ画面が表示されたら**蔵書検索**ボタンをクリックしてみてください。
以下のようにAndroid Studioの左下<font color="red">Logcat</font>タブをクリックし、赤枠のログが出力されていることを確認してみましょう。
<img src="setid07.png" alt="alt" title="reration code" width="650">
上記で表示させたログですが、出力するためには以下のようにコードを記述する必要があります。
ログはプログラム不具合の原因を調査したり、期待するデータが取得できていることなどに活用しますのでログを出力させるための
テンプレートとして以下の書き方を覚えておきましょう。
```
Log.d("tag name", "message");
```

ボタンを押した時に何か処理を行うためのプログラムを実装できたので工程などをおさらいしてみましょう。
<img src="DesignCode.png" alt="alt" title="DesignCode" width="450">

文字の色分けなどは以下の通りです
* <font color="blue">青文字</font>の「ボタン押下時に検索を行う」が大きな課題
* <font color="red">赤文字</font>、黒文字で書かれたところがエンジニアの気づくべき手順
* また<font color="red">赤文字</font>の「画面にボタンを表示する必要がある」「ボタンが押された時にプログラムを実行する必要がある」の２つは編集が必要なファイルごとの課題
* <font color="gray">灰色の四角</font>に囲われている内容がエンジニアがコードを記述する必要のある箇所


次はToast機能を使って**蔵書検索**ボタンの横に配置したEditTextに入力された文字を画面に表示するプログラムを作っていきます。
MainActivity.javaのコードを修正していきます。
<font color="RoyalBlue">***MainActivity.java***</font>
```
...
public class MainActivity extends AppCompatActivity {

    // レイアウトxmlと関連付けるWidget
    Button bookSearchBtn;
    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    EditText bookSearchEditor;
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // 蔵書検索ボタンをjavaプログラムで操作できるように名前をつける
        bookSearchBtn = findViewById(R.id.BookSearchBtn);
        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // 蔵書検索する文字を入力するEditTextをjavaプログラムで操作できるように名前をつける
        bookSearchEditor = findViewById(R.id.BookSearchEdit);
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
        // 蔵書検索ボタンが押された時の処理を実装
        View.OnClickListener bookSearchEvent = new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                // コンソールログにボタンが押されたことを出力(表示)
                Log.d("BookSearchBtn", "onClick: BookSearch Button");
                //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
                // 入力された文字をToast(トースト)に表示
                Toast.makeText(getBaseContext()
                        , "入力された文字は [" + bookSearchEditor.getText().toString() + "]です。"
                        , Toast.LENGTH_LONG).show();
                //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
            }
        };
        // 蔵書検索ボタンが押された時に実行するプログラムをボタンに登録
        bookSearchBtn.setOnClickListener(bookSearchEvent);
    }
}
```
コードの修正が終わったら`Run`アイコンをクリックしてエミュレータにアプリをインストールし直します。
アプリ画面が表示されたら文字を入力してから**蔵書検索**ボタンをクリックしてみましょう。
前回のログ確認と違い、以下の画面のように文字が表示されたでしょうか？
<img src="setid08.png" alt="alt" title="reration code" width="300">

この画面上に現れる機能がToastです、ボタンが押された時にユーザに何か情報を伝える時などに利用します。
またToastは一定時間で自動的に非表示になりますのでユーザが確認ボタンを押す必要もなく簡易なプログラムで利用することができます。

# 色々なボタンイベントの実装

ボタンが押された時のプログラムの実装解説としては以上になります。
しかしボタンイベントを実装方法に絞ってもボタンが押された時のプログラムの記述方法がいくつもあります。
その例をいくつか紹介します。

ここでは色々な新しい単語も紹介しますが、単語に関しては実際に利用する時に詳しく解説します。

## パターン１
ボタンが押された時のプログラムをボタンに登録しながら実装するパターンです
ボタンが押された時のプログラムが短いものなど多く使われるパターンです

```
...
public class MainActivity extends AppCompatActivity {

    Button bookSearchBtn;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        bookSearchBtn = findViewById(R.id.BookSearchBtn);
        bookSearchBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // ボタンが押された時のプログラム
            }
        });
    }
}
```


## パターン２
`bookSearchBtn`の宣言場所が違う実装方法
パターン２は`bookSearchBtn`インスタンスの宣言スコープが違う実装方法です

```
...
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Button bookSearchBtn = findViewById(R.id.BookSearchBtn);
        bookSearchBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // ボタンが押された時のプログラム
            }
        });
    }
}
```

## パターン３
当ページで紹介したプログラムとボタンイベントのプログラムのスコープを`MainActivityクラス`全体で使えるようにした実装方法です
当ページで紹介したパターンと違い`MainActivityクラス`全体で使うことができます

```
...
public class MainActivity extends AppCompatActivity {

    Button bookSearchBtn;

    View.OnClickListener bookSearchEvent = new View.OnClickListener() {
        @Override
        public void onClick(View view) {
            // ボタンが押された時のプログラム      
        }
    };
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        bookSearchBtn = findViewById(R.id.BookSearchBtn);
        bookSearchBtn.setOnClickListener(bookSearchEvent);
    }
}
```

## パターン４
`View.OnClickLister`インターフェースをMainActivityクラスに付与した実装方法
パターン４はボタンが押された時の実装をMainActivityクラスに委譲した実装方法です

```
...
    public class MainActivity extends AppCompatActivity implements View.OnClickListener {

    Button bookSearchBtn;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        bookSearchBtn = findViewById(R.id.BookSearchBtn);
        bookSearchBtn.setOnClickListener(this);
    }

    @Override
    public void onClick(View view) {
        // ボタンが押された時のプログラム
    }
}
```

## パターン５
新しくボタンクリックイベントのインナークラス(外部クラス)を作成する実装方法
この方法はリスト表示されている時などに使うことが多いです

```
...
public class MainActivity extends AppCompatActivity {

    Button bookSearchBtn;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        bookSearchBtn = findViewById(R.id.BookSearchBtn);
        bookSearchBtn.setOnClickListener(new ButtonClick());
    }
    
    class ButtonClick implements View.OnClickListener {
        @Override
        public void onClick(View view) {
            // ボタンが押された時のプログラム
        }
    }
}
```

他にもボタンが押された時のプログラムの書き方はありますが、上記の4パターンが使われていることが多いです。
ボタンに限ったプログラムの書き方だけでもこれだけのパターンがあり、ボタンが押されたイベント以外にも同じファイルにプログラムが
記述されると最初はプログラムを見るのが嫌になってしまうこともあるかもしれません。

ですが、<font color="red">**プログラムを細かく見る（読む）**</font>ことで名前のつけ方に共通点を見つけたり単語を検索しながら考え、失敗を繰り返しながら試すことで
プログラムを深く理解することができるようになります。

以上でボタンが押された時のプログラム実装ができました、次のページでいよいよ本を検索するプログラムを記述していきます。
[非同期処理、REST API通信の実装](/AndroidCourse/android/05-ButtonAction)
