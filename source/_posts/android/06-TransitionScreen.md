---
title: 検索結果一覧画面の作成
date: 2017-11-07 
tags:
---
検索結果一覧を表示する検索結果画面の作成とリスト表示レイアウトの実装方法を学習します。
また検索画面から検索結果画面を表示するための画面遷移処理の実装方法を学習します。

<!-- toc -->

[ボタンイベントの実装](/AndroidCourse/android/05-ButtonAction)からの引き続きの学習ページです。
# 学習ポイント
* ListViewを利用したリストレイアウトの実装方法
* 新規画面の作り方
* 画面遷移処理の実装方法
* xmlファイルでのレイアウト作成のおさらい
* コレクション / 配列
* 継承
* コンストラクタ

蔵書検索を行なった結果複数の蔵書情報を取得できる場合があります、複数取得した蔵書情報を一覧表示します。
その前段として、一覧表示を行うためのウィジェットListViewの使い方を学習します。
また、検索画面で入力された文字の検索を行うために画面遷移処理及び、画面遷移時に次の画面へ情報を渡す処理の実装方法を学習します。

# 新しい画面を作成する
Androidアプリに置いて**画面 = Activity**であることは[Androidの概念](/AndroidCourse/android/02-AndroicConcept)で解説しました。
今回は蔵書検索アプリのプロジェクトに新しいActivityを追加していきます。

新しいActivityの追加はメニューかもしくは左のプロジェクトツリーが表示されているウィンドウで右クリック(macでは２本指でクリック)から新しいActivityの追加を行えます。

新しく作成するActivityは**Empty Activity**を選びます。
1. メニュー > File > New > Activity > Empty Activity
{% img /android/06-TransitionScreen/createactivity01_1.png 500 CreateNewActivity %}
2. ウィンドウ右クリック > New > Activity > Empty Activity
{% img /android/06-TransitionScreen/createactivity01_2.png 500 CreateNewActivity %}

新しく作成するActivityのファイル名と同時に作成するxmlファイルの名称の入力を求められます。
以下の名称で新しいActivityを作成します。

|項目|設定値|
|-------------|------------------|
|Activity Name|ResultListActivity|
|Generate Layout File|チェックを<font color="blue">つける</font>|
|Layout Name|activity_result_list|
|Luncher Activity|チェックを<font color="red">つけない</font>|
|Backwords Compatibility(AppCompat)|チェックを<font color="blue">つける</font>|
{% img /android/06-TransitionScreen/createactivity02.png 500 CreateNewActivity %}
項目の入力が終わったら`Finish`ボタンをクリックします。

新しいActivityの作成が終わるとウィンドウは閉じAndroid Studioのエディタに戻ります。
プロジェクトツリーには<font color="green">ResultListActivity.java</font>と<font color="green">activity_result_list.xml</font>のファイルが追加されます。
{% img /android/06-TransitionScreen/createactivity03.png 350 CreateNewActivity %}

## AndroidManifestファイルの確認
Activityがプロジェクトに追加された場合でも、アプリで表示するためにはアプリの設計書となる`AndroidManifest.xml`に利用することを記述する必要があります。
`AndroidManifest.xml`の場所は以下にあります。
> app -> manifests -> AndroidManifest.xml

{% img /android/06-TransitionScreen/createactivity04.png 400 CreateNewActivity %}
作成した**ResultListActivity**が記載されているか確認すると
{% img /android/06-TransitionScreen/createactivity05.png 550 CreateNewActivity %}
無事に記述されていることが確認できました、他の行を確認すると検索画面として表示している**MainActivity**も記載されています。

`AndroidManifest.xml`ではアプリ名やアプリアイコン、アプリ起動時に表示するActivityなどの設定が記述された概要設計書のようなファイルになります、Push通知などの機能を追加する時にもファイルの修正が必要になるので覚えて置いてください。

# 画面遷移処理機能
新しいActivityが作成できたので、MainActivityの*蔵書検索ボタン*をクリックした時、ResultListActivityに表示が切り替わる様にMainActivity.javaを修正していきます。
```java MainActivity.java
...一部省略
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
            // 画面遷移するためのIntentをインスタンス化
            Intent intent = new Intent(MainActivity.this, ResultListActivity.class);
            // 画面遷移アクションを実行
            startActivity(intent);
            //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
        }
    };
```
コードの修正が終わったらエミュレータで動作確認してください。
以下のように画面遷移できましたか？
{% img /android/06-TransitionScreen/TransitionScreen.gif 300 TransitionScreen %}
またエミュレータ左下の三角ボタンをクリックすると前の画面に戻ります。
{% img /android/06-TransitionScreen/BackScreen.png 400 BackScreen %}

## Intent
Android開発における４大要素の一つとして紹介されている`Intent`が出てきました。
Intentは別Activityの呼び出し（画面遷移）や別のアプリを起動する時などに利用する要素になります、画面遷移時の利用方法としては
1. Intentクラスを宣言、インスタンス化
2. startActivityメソッドの引数に"1."で宣言したIntentをセット

この２つの手順でActivity間の画面遷移が実装できてしまいます。
今回画面遷移時に生成したIntentですが、
一つ目の引数にはContextと呼ばれるクラスをセット<font color="red">(Contextは少々ややこしいクラスなので今後のページで詳しく解説します）</font>Contextは現在開いているActivityが該当するので自身を表す”this”と言う属性を記述してセットする。
２つ目の引数に遷移したいActivityのクラス情報をセットする。

最初のうちは画面遷移を行う場合、上記のルールで画面遷移のコードを実装するよう癖をつけておきましょう。

# 検索結果画面レイアウト作成
検索画面同様にLayoutEditorとConstraintsLayoutを使って検索結果画面のレイアウト構成を作成していきます。
{% img /android/06-TransitionScreen/re_le00.png 200 CreateResultListLayout %}

ResultListActivityの新規作成時、同時に作成された`activity_result_list.xml`を修正します。
> app -> res -> layout -> activity_result_list.xml

検索結果画面は検索した結果を取得し、その一覧を表示します、一覧表示を行うためのコンポーネントとして`ListView`を利用します。
デザインビューに`ListView`を乗せるには検索画面の時の”Button”や"EditText"と同じように”コンポーネントをクリックしたままデザインビューまでマウスカーソルをマウスカーソルを移動させ、離す”と言う手順です。
**Palette**から`ListView`を探しデザインビューにビューに乗せます。
{% img /android/06-TransitionScreen/re_le01.png 500 CreateResultListLayout %}
画面の上下左右と制約を設定します。
{% img /android/06-TransitionScreen/re_le02.png 500 CreateResultListLayout %}
`ListView`に表示するデータの設定はjavaファイルで実装しますのでjavaファイルと関連付けをするために**Attributes**から*ID*項目を設定します。
{% img /android/06-TransitionScreen/re_le03.png 400 CreateResultListLayout %}

|項目   |設定値     |
|------|----------|
|ID    |ResultList|

# 一覧表示機能
検索結果画面の一覧表示機能を実装していきます、まずはListViewの使い方を理解するために理解するために10行のリスト表示を行い、各行に行数番号の文字を表示してみます。
```java ResultListActivity.java
public class ResultListActivity extends AppCompatActivity {

    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    // xmlファイルのコンポーネントと関連付ける要素
    ListView resultListView;
    // 検証用コレクションデータ
    List<String> listData = Arrays.asList("1", "2", "3", "4", "5", "6", "7", "8", "9", "10");
    // ListViewの表示内容を管理するクラス
    ArrayAdapter<String> adapter
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_result_list);

        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // xmlファイルのコンポーネントと関連付け
        resultListView = findViewById(R.id.ResultListView);
        // ListViewに表示する情報をまとめるAdapterをインスタンス化
        adapter = new ArrayAdapter<>(ResultListActivity.this
                , android.R.layout.simple_list_item_1
                , listData);
        // ListViewに表示情報をまとめたAdapterをセット
        resultListView.setAdapter(adapter);
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    }
}
```
合わせてListViewの行をクリックされた時のイベントをセットしていきます。
行毎のクリックイベントはMainActivity.javaのボタンクリックとは別の実装方法を試していきます、クラス名の後ろに`implements AdapterView.OnItemClickListener`と入力
```java ResultListActivity.java
//↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓修正↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
public class ResultListActivity extends AppCompatActivity implements AdapterView.OnItemClickListener {
//↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
	...
}
```
するとエラーの様な<font color="red">赤い下線</font>が表示されます。
クラス名 "ResultListActivity"をクリックし少し経過すると、赤い電球のようなアイコンが表示されるのでクリックします。
{% img /android/06-TransitionScreen/additemclick01.png 500 add Item ClickEvent %}
`implement methods`をクリックします。
{% img /android/06-TransitionScreen/additemclick02.png 500 add Item ClickEvent %}
`onItemClick...`が選択されている(青くなっている)状態で`OK`をクリックします。
{% img /android/06-TransitionScreen/additemclick03.png 300 add Item ClickEvent %}
`onItemClick()`メソッドが自動的に追加されます。
{% img /android/06-TransitionScreen/additemclick04.png 500 add Item ClickEvent %}
あとは行をクリックした時の命令を実装します。
```java ResultListActivity.java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_result_list);

        // xmlファイルのコンポーネントと関連付け
        resultListView = findViewById(R.id.ResultListView);
        // ListViewに表示する情報をまとめるAdapterをインスタンス化
        adapter = new ArrayAdapter<>(ResultListActivity.this
                , android.R.layout.simple_list_item_1
                , listData);
        // ListViewに表示情報をまとめたAdapterをセット
        resultListView.setAdapter(adapter);

        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // ListViewに行をクリックした時のイベントを登録
        resultListView.setOnItemClickListener(ResultListActivity.this);
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    }

    // ListViewの各行をクリックした時の命令を実装
    @Override
    public void onItemClick(AdapterView<?> adapterView, View view, int i, long l) {
        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // クリックした行番号をToastで表示する
        Toast.makeText(ResultListActivity.this
                , i + "行目をクリックしました"
                , Toast.LENGTH_SHORT).show();
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    }

```
コードの実装が終わったらエミュレータで動作確認してみましょう。

今回のリストでは各行にテキストを１つ表示するリストとして実装しました。
Android SDKに用意されているリストと行のレイアウトを使用して表示させています。
更に各行をクリックすると"＊行目をクリックしました"とToastが表示されます。

多くの現場ではこのような使い方はすくなく**Adapter**クラスと行のレイアウトをカスタマイズして利用することがほとんどです。
次の工程では**Adapter**クラスのカスタマイズと行に表示するレイアウトをカスタマイズしていきます。

## コレクション、配列
Androidアプリの開発に用いられるjava言語では複数のデータを一つのまとまりとして利用することができます。
データの持ち方としては２つ方法がありそれぞれ**コレクション**、**配列**という概念があります、
どちらの形式においても一つのまとまりのデータは同じ形である必要があり、<font color="blue">**型の違うデータを複数記憶することはできません**</font>

主な違いとして
* 配列は宣言時に要素の上限を決める必要がある。
* コレクションは上限設定はなく動的に要素の数を増減できる。

その他の差異の確認のため、例として３つの手順をそれぞれの概念を使って実装してみます。
* 整数型のコレクション/配列を空で宣言
* 2つのデータを追加/代入
* 1つ目のデータを参照してログを表示

```java コレクション
    // 空のコレクションを宣言
    List<Integer> integerList = new ArrayList<>();
    // コレクションの1番目に10を追加
    integerList.add(10);
    // コレクションの2番目に20を追加
    integerList.add(20);
    // コレクションの1番目のデータを表示
    Log.d("collection_arrarys", "Collection data check: " + integerList.get(0));
    /* "Collection data check: 10" と表示されます */
```

```java 配列
    // 空の配列を宣言
    int integerArray[] = new int[2];
    // 配列の1番目に10を代入
    integerArray[0] = 10;
    // 配列の2番目に20を代入
    integerArray[1] = 20;
    // 配列の1番目のデータを表示
    Log.d("collection_arrarys", "Array data check: " + integerArray[0]);
    /* "Array data check: 10" と表示されます */
```

配列で値を代入する時や参照する時に変数名の後の"[]"は要素の場所を表すための**順番(添字)**を指定する箇所です、プログラムの配列で気をつけるポイントとして、1番目のデータを参照するためには '0' と記述する必要があることです。
よって２番目のデータを参照するためには"[]"の添え字に '1' と記述する必要があります。
３番目以降も同様に一つ小さい値を記述する必要があります。

データの数は動的な場合が多くほどんどの状況下でコレクションを使用して一覧表示などを作っています。

# カスタム一覧表示機能
一覧表示のサンプルプログラムができたところで、最終的にはインターネット上の蔵書情報を取得し画面に表示します。
その前にリストの表示内容をカスタマイズ方法を覚えていきましょう。
ListViewの表示をカスタマイズするためには行のレイアウトファイルを作成し、"Adapter"クラスを継承して新しいAdapterを作成していきます。
下図の様に各行が縦２行のデータになり、１行目は大きい文字、２行目は小さめの文字で表示する様なリストを作ります。
{% img /android/06-TransitionScreen/custom_list00.png 200 example custom list %}
**継承**に関しては実装時に解説します。
## ListViewの行レイアウト作成
表示をカスタムしたListViewを作るには行毎のレイアウトを作成する必要があります。
先ほど１〜１０の数字が表示されていたところ縦に２つ並ぶテキストに作り変えて表示していきます。

新規レイアウトファイルを作成するため、以下の通り"Layout XML File"をクリックします。
> ウィンドウ右クリック > New > XML > Layout XML File

{% img /android/06-TransitionScreen/createrow01.png 400 Create List Row %}
レイアウトファイル名、一番上位の要素を設定して`Finish`ボタンをクリックします。
{% img /android/06-TransitionScreen/createrow02.png 500 Create List Row %}

|項目              |設定値          |
|:---------------:|---------------|
|Layout File Name |row_result_list|
|Root Tag         |android.support.constraint.ConstraintLayout|

レイアウト作成されたら、`row_result_list.xml`ファイルを開き、 "TextView"をデザインビューに配置します。
{% img /android/06-TransitionScreen/createrow03.png 500 Create List Row %}
"TextView"の上、左右を画面の端と制約を設定します。
{% img /android/06-TransitionScreen/createrow04.png 500 Create List Row %}
"TextView"の要素範囲を最大に広げるため、*layout_width*属性を<font color="red">match_constraint</font>に変更します。
{% img /android/06-TransitionScreen/createrow05.png 500 Create List Row %}
"TextView"の表示文字サイズを大きくするため*textAppearance*属性を<font color="red">AppCompat.Headline</font>に変更します。
{% img /android/06-TransitionScreen/createrow06.png 500 Create List Row %}
*ID*属性を<font color="red">RowListTitle</font>に変更します。
{% img /android/06-TransitionScreen/createrow07.png 500 Create List Row %}
タイトルではない文言を表示するため"TextView"をもう一つデザインビューに配置します。
{% img /android/06-TransitionScreen/createrow08.png 500 Create List Row %}
２つ目の "TextView"は上辺を<font color="red">RowListTitle</font>の下辺に制約をつけ、左右・下を画面の端と制約を設定します。
{% img /android/06-TransitionScreen/createrow09.png 500 Create List Row %}
２つ目の "TextView"の*ID*属性を<font color="red">RowListSummary</font>に変更したら行のデザインは完成です。
{% img /android/06-TransitionScreen/createrow10.png 500 Create List Row %}

## 継承
これから作成するカスタムAdapterクラスを作成するためには"BaseAdapter"クラスを継承したクラスを作成します。
そのために、**継承**とは何かを解説します。

継承とは継承元となるクラスのメソッドや変数などの機能を受け継いだクラスを指します。
継承元になるクラスを**親クラス(スーパークラス)**と呼び、継承先となるクラスを**子クラス**と呼びます。

継承を行うメリットは同じプログラムを記述する回数が減るのでコード量が減ること、また元の処理を拡張することができることです。
継承クラスの作り方は以下のような記述を行うだけで継承が成立します。
```java
 class 子クラス名 extends 親クラス名 { ... }
```
当記事の中でも実は継承クラスを作っており、
`MainActivity.java`、`ResultListActivity.java`などはActivityクラスを継承したクラスになります。
２つのクラスを見てみると
```java MainActivity.java
public class MainActivity extends AppCompatActivity { ... }
```
```java ResultListActivity.java
public class ResultListActivity extends AppCompatActivity implements AdapterView.OnItemClickListener { ... }
```
と２つのActivityは "AppCompatActivity"クラスを継承しています。

継承を行うとメソッドなどの機能を受け継ぐと説明しました。
たとえば`MainActivity.java`の "onCreate"メソッドの宣言をみると以下の様に記述されています。
```java MainActivity.java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
		...一部省略
	}
```
メソッド名の上の行に`@Override`と記述されている、アノテーションと呼ばれる<font color="red">おまじない</font>によって、"AppCompatActivity"が持つ "onCreate"メソッドの命令を一旦無視(**オーバーライド(override)**)して `MainActivity`の "onCreate"メソッドの命令を実行すると実装しているのです。
<font color="blue">***親クラスのメソッドを子クラスで拡張する場合はこのアノテーションのおまじないを記述する必要があるので気をつけましょう。***</font>

続いての行の命令`super.onCreate(savedInstanceState);`では改めて親クラスである "AppCompatActivity"の "onCreate"メソッドを実行しています。
更に次の行からが継承のメリットになります。
当たり前に感じるかもしれませんが、`MainActivity`ではボタンクリックイベントや画面遷移の実装を行なっています。
それに対して`ResultListActivity`ではListViewの一覧表示処理を実装しています。
同じonCreateメソッドに対して別の命令を実行することができています、 "AppCompatActivity"を継承することで画面表示を行う "onCreate"メソッドが呼び出され実際に使用している`MainActivity`や`ResultListActivity`の "onCreate"メソッドから実行され現在の様に画面にレイアウトが表示されています。

## カスタムAdapterクラスの作成
まずは新しいクラスを作成します、新規クラス作成時に親クラスを設定することができます。

新規Javaクラスファイルを作成するため、以下の通り"Java Class"をクリックします。
> ウィンドウ右クリック > New > Java Class

{% img /android/06-TransitionScreen/createadpt01.png 500 Create List Adapter %}
Javaクラス名、親クラスの設定して`OK`ボタンをクリックします。
{% img /android/06-TransitionScreen/createadpt02.png 500 Create List Adapter %}

|項目          |設定値                     |
|:-----------:|--------------------------|
|Name         |ResultListAdapter         |
|Kind         |Class                     |
|Superclass   |android.widget.BaseAdapter|
|Interface(s) |-                         |
|Package      |元の設定のまま              |
|Visibility   |Public                    |
|Modifiers    |None                      |

新しい`ResultListApater`クラスが作成されエディタに表示されますが、BaseAdapterクラスを継承したクラスを作成した場合には必ず*オーバーライド*しないといけないメソッドがいくつかあるので、まずはそのメソッド群を`ResultListAdapter`に宣言していきます。

クラス名 "ResultListAdapter"をクリックし少し経過すると、赤い電球のようなアイコンが表示されるのでクリックします。
{% img /android/06-TransitionScreen/createadpt03.png 500 Create List Adapter %}
小さいウィンドウが表示されるので <font color="blue">Implement methods</font>をクリックします。
{% img /android/06-TransitionScreen/createadpt04.png 500 Create List Adapter %}
新しいダイアログが表示されるのでそのまま`OK`をクリックします。
{% img /android/06-TransitionScreen/createadpt05.png 300 Create List Adapter %}
ダイアログで選択していたメソッドが`ResultListAdapter`に追加され、エラーも消えます。
{% img /android/06-TransitionScreen/createadpt06.png 500 Create List Adapter %}

## カスタム一覧機能の実装
ListViewの表示をカスタマイズするために作成した`ResultListAdapter.java`を実装し`ResultListActivity.java`で利用する実装にコードを修正します。

実装の前に**コンストラクタ**という機能を紹介します。
### コンストラクタ
コンストラクタはクラスをインスタンス化した時に実行される命令(メソッドと認識しても概ね大丈夫です)でクラスの内部的な初期化処理を行う機能です。
今回`ResultListAdapter`は内部で持つデータだけでは機能を果たせず、少なくとも`ResultListActivity`から　"Context"データを引数として受け取ることで、
 行毎のレイアウトを読み込むために"LayoutInflater"クラスをインスタンス化する必要があります。
そのため、オリジナルのコンストラクタを実装することになります。
他の場合でもクラス内の変数を初期化する場合などにコンストラクタを利用するとプログラムが見やすくなります。

---

ではカスタムレイアウトでの検索結果一覧画面を実装します。
```java ResultListAdapter.java
public class ResultListAdapter extends BaseAdapter {

    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    // ListViewの描画に必要な変数を宣言
    List<String> summaryList;
    LayoutInflater layoutInflater;

    // コンストラクタ(インスタンス時に呼び出されるメソッドのようなもの)
    public ResultListAdapter(Context context, List<String> summaryList) {
        this.summaryList = summaryList;
        this.layoutInflater = LayoutInflater.from(context);
    }
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

    @Override
    public int getCount() {
        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓修正↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // 一覧表示する要素数を返却する
        return summaryList.size();
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    }

    @Override
    public Object getItem(int i) {
        // indexやオブジェクト情報などを返却する
        // 一旦nullのまま
        return null;
    }

    @Override
    public long getItemId(int i) {
        // 行で表示しているLayoutIdやindex、特別なIDを返却する
        // 一旦nullのまま
        return 0;
    }

    @Override
    public View getView(int i, View view, ViewGroup viewGroup) {
        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
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

        titleView.setText((i + 1) + "ページ目");
        summaryView.setText(summaryList.get(i));
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓修正↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // 文字情報を代入されたviewを返却
        return view;
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    }
}
```

```java ResultListActivity.java
public class ResultListActivity extends AppCompatActivity implements AdapterView.OnItemClickListener {

    // xmlファイルのコンポーネントと関連付ける要素
    ListView resultListView;
    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓修正↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
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
    // ListViewの表示内容を管理するクラス
    ResultListAdapter adapter;
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_result_list);

        // xmlファイルのコンポーネントと関連付け
        resultListView = findViewById(R.id.ResultListView);
        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓修正↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // ListViewに表示する情報をまとめるAdapterをインスタンス化
        adapter = new ResultListAdapter(ResultListActivity.this, listData);
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
        // ListViewに表示情報をまとめたAdapterをセット
        resultListView.setAdapter(adapter);

        // ListViewに行をクリックした時のイベントを登録
        resultListView.setOnItemClickListener(ResultListActivity.this);
    }

    // ListViewの各行をクリックした時の命令を実装
    @Override
    public void onItemClick(AdapterView<?> adapterView, View view, int i, long l) {
        // クリックした行番号をToastで表示する
        Toast.makeText(ResultListActivity.this
                , i + "行目をクリックしました"
                , Toast.LENGTH_SHORT).show();
    }

}```
コードの実装が終わったらエミュレータで確認して見ましょう。

今回のコード修正のポイントは`ResultListAdapter`クラスで利用した`LayoutInflater`クラスです。
各Activityでは画面に表示するレイアウトファイルの読み込みを`setContentView()`メソッドが担当していましたが、ListViewに表示するレイアウトの読み込みは`LayoutInflater`クラスを使う必要があるので注意しましょう。

以上で**検索結果一覧画面の作成**は一旦完了です。
次の[蔵書検索機能の作成](/AndroidCourse/android/07-AsyncProcess)ではAPI通信を通して蔵書検索機能を作成します、
データが検索機能が完成したら、検索データを "ResultListActivity"で表示出来るよう修正していきます。