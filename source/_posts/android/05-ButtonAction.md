---
title: ボタンイベントの実装
date: 2017-11-06
---
アプリの基本イベントとなるボタンが押された時の処理を学習し、蔵書検索ボタンの機能を作り始めます。
合わせてコンソールログを表示、Toast(トースト)の機能を試します

<!-- toc -->

[検索画面レイアウト作成](/AndroidCourse/android/04-MakeLayoutDesign)からの引き続きの学習ページです。
# ページ内の学習ポイント
* レイアウトファイルとjavaファイル間のウィジェットの関連付け
* ボタンクリックイベントの実装方法
* クラス、変数
* メソッド

レイアウト作成した画面に配置したボタンがクリックされた時に実行したい命令を作りながらレイアウトファイルとjavaファイルでの関連付けやプログラミングにおける重要な変数、メソッドを学習します。

# 検索画面の機能を考える
このページからはプログラムの実装していきますが、前のページで作成したレイアウトにはどんな操作が発生するか考えるため、もう一度完成したレイアウトを見てみます。
{% img /android/04-MakeLayoutDesign/CompleteScreen.png 500 CompleteScreen %}
大まかに以下ような操作をユーザが行い、ウィジェットによっては機能として動くプログラムを実装する必要がありそうです。
1. 検索する蔵書名、またはその一部を入力する操作
1. 蔵書検索ボタンをクリック
    * 入力された蔵書名、またはその一部から蔵書の検索結果を表示する画面に遷移する機能
1. 閲覧履歴をクリック
    * 過去に検索結果から閲覧した蔵書のリストを表示する画面に遷移する機能

当ページではアプリを作る上での基礎になる"ボタンをクリックした時に画面を遷移する"の前にボタンをクリックした時にログ表示、Toastを表示するところまでを解説していきます。

# javaプログラムとxmlレイアウトの関連付け
普段何気なく使っているアプリのボタンをクリックした時、アプリではプログラムが実行され別の画面を表示したり、計算や文字の置き換えなどの命令を実行して画面の表示内容を更新するなど見えないところで色々な命令が動いています。

ボタンをクリックした時のプログラムや画面の表示内容更新などは**java**ファイルに実装していきます。
"検索画面レイアウト作成"ページでレイアウト作成したのは`activity_main.xml`というファイルでしたが、このレイアウトファイルは「MainActivity.java」のプログラムによって画面に表示されています。

では**MainActivity.java**ファイルを開きプログラムを実装していきます。
{% img /android/05-ButtonAction/setid04.png 500 rerationcode %}

前の記事[レイアウト作成](/AndroidCourse/android/04-MakeLayoutDesign)の内容で作成したxmlレイアウトですが、
実は"MainActivity.java"の中ですでに関連付けのプログラムが実装されています。
それが以下のコード内矢印に囲まれてる`setContentView()`という命令でMainActivity.javaが表示され`setContentView()`の命令を実行する際に "()" 内で指定したレイアウトxmlファイルと関連付けを行い、内部で画面に表示するという命令を実行してくれています。
```java MainActivity.java
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

`setContentView()`の様に必ず書かないといけないけど説明が難しいコードを<font color="red">***おまじない***</font>と紹介されることがありますが、実装されていないとアプリが異常終了してしまったり、今回の場合では画面が真っ白な画面が表示されるだけになってしまうので<font color="red">***おまじない***</font>と紹介されたところは*よくわからないけど書かないとよくないことが起こる*と頭の片隅に置いておいてください。

次にButtonをクリックしたときに検索などの処理を行うにはMainActivity.javaなど**javaファイルでクリックされた時のプログラムを書く**必要があります。
`setContentView()`を実行したことでMainActivity.javaから"activity_main.xml"のボタンなどウィジェットやビューを参照できる状態になっています、
あとはボタンやエディットテキストなどを各ウィジェットやビューにユニークな名称(他と被らない情報)があればjavaファイルでも判別して使えそうです。

"activity_main.xml”内のButtonやEditTextなどにはユニークな名称を設定する項目として**ID**という項目があるので、**activity_main.xml**を開きます。
{% img /android/05-ButtonAction/rerationcode01.png 500 rerationcode %}

"蔵書検索"ボタンをクリックしAttributesエリアから**"ID"**項目を以下のように修正します
{% img /android/05-ButtonAction/setid01.png 500 rerationcode %}

> button -> BookSearchBtn

合わせて他のウィジェットの**"ID"**項目も変更していきます。
※もし変更前IDが一致していなくても、Noと場所の照らし合わせを正として修正してください
{% img /android/05-ButtonAction/setid03.png 500 rerationcode %}

|No.  |変更前のID |変更後ID         |
|:---:|:---------|:--------------|
|①   |button    |BookSearchBtn  |
|②   |editText  |BookSearchEdit |
|③   |button2   |HistoryBtn     |

IDを変更するとComponent Treeの表示にも変更が反映されます
これで画面のボタンなどに"ID"項目を設定することができました、ここからがjavaプログラムの実装です、MainActivity.javaからウィジェットやビューと関連付けの方法を紹介していきます。

ButtonやEditTextなどのウィジェットをjavaプログラムで操作できる様にするにはいくつかの方法がありますが、一番よく使う方法として<font color="red">`findViewById()`</font>を使う方法で実装していきます。
"setContentView" で関連付けたレイアウトxml内のウィジェットやビューに設定した"ID"を`findViewById`の後ろの "()"内に 指定することでjavaプログラムで”ボタンをクリックした時"の命令を登録したり、"ボタンに表示する文言などを表示後に変更する"ことができる様になります。

## コード記述時の便利機能
これからコード記述する中でIDの入力中に小さいリスト表示がされると思いますので十字キーの上下を押して選択し、`Enter`キーを押せばプログラムに反映されるので試してみてください。
{% img /android/05-ButtonAction/setid05.png 650 rerationcode %}

上記の便利機能を踏まえて`findViewById`メソッドと "activity_main.xml"で設定した**ID**を使ってButtonウィジェットの関連付けを行います。
```java MainActivity.java
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
最初に実装したコードですので追加した２行にどんな意味合いがあるのかみていきましょう
{% img /android/05-ButtonAction/setid06.png 600 rerationcode %}

|No.  |各No.のコードの説明                                     |
|:---:|:----------------------------------------------------|
|①   |activity_main.xml内のウィジェットと関連づける型            |
|②   |MainActivity.javaで使う時のユニークな名前                 |
|③   |①〜②で宣言したウィジェットに情報を登録する時の記述方法       |
|④   |④のIDを使いactivity_main.xml内のウィジェット/ビューを取得するプログラム |
|⑤   |activity_main.xml内から取得するウィジェット/ビューのユニークID |

これで"蔵書検索"ボタンをxmlレイアウトとMainActivity.javaのプログラムで関連付けることができました。

xmlファイル内のウィジェットなどとjavaファイルの関連付けは非常に多く利用するコードですので利用方法を理解しておきましょう。
* ①〜⑤の工程はプログラムで表示するデータを動的に一時的に記憶したり、表示したりする上で大事なクラス（変数）の宣言から初期化）を行なっています。
* ①〜②で追加したコードを<font color="blue">クラスの宣言</font>と言い、WidgetやView、その他のクラスをjavaプログラムで使うためにわかりやすい名前をつけて準備をしています。
* 本来ボタンオブジェクトは"ボタンの色"、"ボタンに表示する文字"など情報を持っていますが、①〜②の工程だけでは名前が準備されただけで`bookSearchBtn`は全く情報を持っていません。
* `bookSearchBtn `に情報を登録するには<font color="blue">**オブジェクト化（インスタンス化とも呼ばれます）**</font>という工程が必要になります、
その工程が③〜⑤に該当しオブジェクト化を行なっており、この工程後に`bookSearchBtn`がもつ"ボタンの色"、"ボタンに表示する文字"などの情報を
参照したり変更したりすることが可能になります。

# ボタンクリックアクションの実装
"蔵書検索"ボタンのクリックした時にクリックされたことを開発者に確認させるためログを出力するプログラムを作っていきます。
プログラムが上手く動かない時などどこまでプログラムが実行されたかを確認したり、実行中のデータを確認する時にも利用できるので試してみます。
```java MainActivity.java
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

上記コードの実装が完了したら動作確認をします、`Run`アイコンをクリックしてエミュレータを起動します。
アプリ画面が表示されたら**蔵書検索**ボタンをクリックしてみてください。

Android Studioの左下<font color="red">Logcat</font>タブをクリックし、赤枠のログが出力されていることを確認してみましょう。
{% img /android/05-ButtonAction/setid07.png 650 rerationcode %}
実装したコードで表示されるログですが、出力するためには以下のような形式で実装する必要があります。
ログを表示するプログラムのとして以下のテンプレートを覚えておきましょう。
```
Log.d("tag name", "message");
```

ボタンを押した時にログを表示する命令を実行するプログラムを実装ました、ここまでの全体の工程をおさらいします。
{% img /android/05-ButtonAction/DesignCode.png 450 DesignCode %}
文字の色分けなどは以下の通りです
* <font color="blue">青文字</font>の「ボタン押下時に検索を行う」が大きな課題
* <font color="red">赤文字</font>、黒文字で書かれたところがエンジニアの気づくべき手順
* また<font color="red">赤文字</font>の「画面にボタンを表示する必要がある」「ボタンが押された時にプログラムを実行する必要がある」の２つは編集が必要なファイルごとの課題
* <font color="gray">灰色の四角</font>に囲われている内容がエンジニアがコードを記述する必要のある箇所

ここまでに聞きなれない単語がいくつか出てきました、アプリ開発に限らずプログラムを
行なっていく上で<font color="blue">**とても重要**</font>な概念ですので詳しく解説します。
## クラス、変数
プログラムで使用するデータを一時的に記憶しておいたり、計算や画面に表示するデータとして使用するために使用する概念になります。
変数を解説/理解する上で色々なたとえがあるのですが、以下Wikipediaの内容が一番理解しやすかったので一部引用させてもらいました。

{% blockquote 変数(プログラミング) https://ja.wikipedia.org/wiki/%E5%A4%89%E6%95%B0_(%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0) wikipedia %}
データを一定期間記憶し必要なときに利用できるようにするために、データに固有の名前を与えたものである。 一人一人の人間が異なる名前によって区別されるように、一つ一つの変数も名前によって区別される。
変数が表しているデータをその変数の値（あたい）という。
{% endblockquote %}

一部文中省いているところはありますが、上記の様にデータを一定期間記憶、使用することができる固有（ユニーク）な名称をつけたものになります。
変数につけた固有名称を変数名と呼びます。

変数に記憶しておけるデータの種類が決まっており、種類のことを***型***と呼びます。
型は一度決めてしまうと後から変更することはできません。
型の種類は以下の通りです、Stringは厳密には型ではありませんが、頻繁に使用するので一緒に記載しておきます。

|型名    |記憶できるデータ       |データの範囲                                   |
|:------|:-------------------:|:--------------------------------------------|
|byte   |符号つき8ビット整数型   |-128 ～ 127                                  |
|short  |符号つき16ビット整数型  |-32768 ～ 32767                              |
|int    |符号つき32ビット整数型  |-2147483648 ～ 2147483647                    |
|long   |符号つき64ビット整数型  |-9223372036854775808 ～ 9223372036854775807  |
|float  |32ビット浮動小数点数型  |1.4E-45 〜 3.4028235E38                      |
|double |64ビット浮動小数点数型  |4.9E-324 〜　1.7976931348623157E308           |
|boolean|論理型                |true(真)、false(偽)                           |
|char   |符号なし１６ビット文字型 |Unicode文字１文字 : ¥0000 〜 ¥FFFF / 0 〜 65535|
|String |文字列クラス           |複数の文字を扱うことができる                     |

クラス、変数を利用するための操作として**宣言**、**代入**、**参照**の３つがあります。
* 宣言
    * 一定期間記憶しておくデータの型、変数名を明確に記述することを宣言と呼びます、宣言を行わないと変数は参照できません
* 代入
    * 宣言した変数に一定期間記憶しておくデータを登録/更新すること
    * 宣言された変数に最初にデータを登録(格納)することを*初期化*と言います
    * 宣言されたクラスに最初にデータを*オブジェクト化(インスタンス化)*と言います
* 参照
    * 宣言され、初期化された変数のデータを利用することを指します
    * 数値であれば計算に利用したり、文字列であれば画面に表示する際に**参照**することが多いです

次に変数とクラスはプログラミング上の利用方法が似ているためここでは一緒に紹介しますが、変数とクラスでは記憶しておけるデータの量が違います。
それぞれの特徴としては
変数：**変数は１つだけデータを記憶しておくことができる**
クラス：**クラスは複数の変数を複数記憶することができ、次に紹介するメソッドも複数持つことができる**
という違いがあります。

クラスは内部に変数を持っていることも多く、今回使用している`Button`クラスは内部でボタンに表示する文字列や表示する座標位置など
様々な情報を持っており、その情報を参照するためのメソッドが用意されています。

### ウィジェットとクラス
ウィジェットもクラスに該当します、違いはというとAndroid SDKが準備した画面表示の要素の有無となります。
ウィジェット = クラスであり、Android SDKに準備されているレイアウトxmlファイルに表示できる要素を持っているものと区別しましょう。

## メソッド
複数の命令を組み合わせた処理の一まとまりをjavaでは**メソッド**と呼びます。
また**メソッド**にも名称をつけて管理しており、*メソッド名*と呼びます。

すでに上記までにメソッドは出てきており、
`onCreate()`や`setContentView()`、`findViewById()`がメソッドに該当します。

メソッドも変数に類似している箇所があり、利用するための操作として**宣言**、**実装部**、**参照**の３つがあります。
* 宣言
    * *戻り値*の型、メソッド名を明確に記述することを宣言と呼びます、宣言を行わないと参照できません
    * メソッドの実装部で命令を進めるために基準となるデータが必要な場合があり、データをメソッドに渡すことができます、この機能を*引数(引数)*と呼びます
* 実装部
    * メソッドが行う複数の命令を全て記述されている箇所
* 参照
    * メソッドを利用すること、メソッドの実装部にバグがなければ戻り値を受け取ることができる

メソッドの利用において難しいポイントは*戻り値*と*引数*になる場合が多いです。
メソッド宣言は形式が決まっているため、見合わせるとどこに何が明記されているかわかりやすくなります。
```
    戻り値の型 メソッド名 (引数の型 引数名) {
       メソッドの実装部 
    }
```
*戻り値の前に"protected","public"などのアクセス修飾子と呼ばれるコードが書かれていることが多いですが、アクセス修飾子は後の講義で解説します。*

`onCreate()`メソッドを例にすると
```
    protected void onCreate(Bundle savedInstanceState) {
        メソッドの実装部
    }
```
"void" = 戻り値の型
"onCreate" = メソッド名
"Bundle" = 引数の型
”savedInstanceState” = 引数名
ということになります。
上記で**void**という新しい型が出てきましたが、戻り値がないことを明示するコードになります。
void型の変数は存在しないのでメソッドの宣言時以外ではあまり見かけないコードです。

そして宣言箇所で"引数の型"、"引数名"が記載されていないメソッドは引数が無いメソッドということになります。

続いてメソッドを参照している箇所は`setContetView()`を例にします。
```
    setContetView(R.layout.activity_main.xml)
```

メソッド名の後の "()" 内に引数を設定します、”setContentView"メソッドは参照時に画面表示するレイアウトxmlの情報を引数に記述することで、引数に記述したレイアウトを画面表示する命令を実行しています。

# ボタンクリックアクションでトースト表示機能
先程までのコードを拡張してToast表示機能を実装していきます。
EditTextに入力された文字を画面に表示するプログラムに作り変えていきます。
```java  MainActivity.java
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
コードを修正した場合、エミュレータや実機にインストールし直さないとレイアウトやプログラムの修正は反映されないので気をつけましょう。

EditTextに文字を入力してから**蔵書検索**ボタンをクリックしてみます。
前回のログ確認と違い、以下の画面のように文字が表示されたでしょうか？
{% img /android/05-ButtonAction/setid08.png 300 rerationcode %}

この画面上に現れる機能がToastです、一方的にユーザに情報を伝える時などに利用します、
Toastは一定時間で自動的に非表示になりますので、Toast用の確認ボタンなどは存在しません。

# 色々なボタンイベントの実装
ボタンが押した時のプログラム実装解説は以上になります。
しかしボタンイベントを実装方法に絞ってもボタンが押された時のプログラムの記述方法がいくつもあります。
その例をいくつか紹介します。

ここでは色々な新しい単語も紹介しますが、単語に関しては実際に利用する時に詳しく解説します。

## パターン１
ボタンが押された時のプログラムをボタンに登録しながら実装するパターンです
ボタンが押された時のプログラムが短いものなど多く使われるパターンです

```java
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
                // ボタンをクリックした時の命令
            }
        });
    }
}
```


## パターン２
`bookSearchBtn`の宣言場所が違う実装方法
パターン２は`bookSearchBtn`インスタンスの宣言スコープが違う実装方法です

```java
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
                // ボタンをクリックした時の命令
            }
        });
    }
}
```

## パターン３
当ページで紹介したプログラムとボタンイベントのプログラムのスコープを`MainActivityクラス`全体で使えるようにした実装方法です
当ページで紹介したパターンと違い`MainActivityクラス`全体で使うことができます

```java
...
public class MainActivity extends AppCompatActivity {

    Button bookSearchBtn;

    View.OnClickListener bookSearchEvent = new View.OnClickListener() {
        @Override
        public void onClick(View view) {
            // ボタンをクリックした時の命令
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

```java
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
        // ボタンをクリックした時の命令
    }
}
```

## パターン５
新しくボタンクリックイベントのインナークラス(外部クラス)を作成する実装方法
この方法はリスト表示されている時などに使うことが多いです

```java
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
            // ボタンをクリックした時の命令
        }
    }
}
```

他にもボタンが押された時のプログラムの書き方はありますが、上記の4パターンが使われていることが多いです。
ボタンに限ったプログラムの書き方だけでもこれだけのパターンがあり、ボタンが押されたイベント以外にも同じファイルにプログラムが
記述されると最初はプログラムを見るのが嫌になってしまうこともあるかもしれません。

ですが、<font color="red">**プログラムを細かく見る（読む）**</font>ことで名前のつけ方に共通点を見つけたり単語を検索しながら考え、失敗を繰り返しながら試すことで
プログラムを深く理解することができるようになります。

以上でボタンが押された時のプログラム実装ができました、次の[検索結果画面への遷移実装](/AndroidCourse/android/06-TransitionScreen)ではリスト表示レイアウト及び、画面遷移処理に関して実装していきます。
