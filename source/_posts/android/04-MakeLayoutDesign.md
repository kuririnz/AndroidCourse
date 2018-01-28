---
title: 検索画面レイアウト作成
date: 2017-11-05
---
Layout Editor / ConstraintLayoutを使用したレイアウト実装と動作確認

<!-- toc -->

[Androidアプリ開発を始める](/AndroidCourse/android/03-StartAndroidDevelopment)から引き続きの学習ページです。
# ページ内の学習ポイント
* Layout Editor
* Widget / View などのコンポーネント
* エミュレータの起動、確認方法

蔵書検索アプリ起動時に表示される検索画面のレイアウト構成を作りながら、
Layout Editorや画面を構成する要素について学習します。
また、作成したレイアウトをエミュレータという仮想デバイスを使い表示確認を通して使い方を学習します。

# Layout Editor / ConstraintLayout
Androidアプリに表示する画面のレイアウト構成を作るためにはLayout EditorというAndroid Studioの機能を使います。

Androidアプリのプロジェクト開始時に生成されたレイアウトxmlファイルにウィジェット(Widget)やビュー(View)を配置し、ウィジェットごとに制約を設定して以下画像のように表示位置を決めていきます。
{% img /android/04-MakeLayoutDesign/CompleteScreen.png 500 CompleteScreen %}

## Layout Editor
Android Studio 2.2で追加されたレイアウト構成の機能、以前までも類似する機能はあったが使いにくく、xmlファイルを直接修正することが多かったのですが、改善されたとこで視覚的にWidgetやViewを配置しやすくなった。
特にConstrainsLayoutと合わせて使うことで画面のレイアウト作成でxmlファイルを直接修正する必要が無くなってきたことで初学者の学習コストやハードルも下がってきました。

Layout Editor内、各領域の名称は以下の通りです。
<img src="https://developer.android.com/studio/images/write/layout-editor-callouts_2-2_2x.png?hl=ja" alt="alt" title="Layout Editor" width="550">

|No.  |領域名                 |説明                                             |
|:---:|:---------------------|:-----------------------------------------------|
|①   |Palette               |レイアウトに配置できるウィジェットとレイアウトリストを表示されています |
|②   |Component Tree        |レイアウトのビュー階層を表示/移動することができます、Viewの重なりもここで調整します |
|③   |ツールバー              |レイアウトの外観設定やプロパティを編集するためのボタンが表示されています |
|④   |デザイン エディタ        |実際の見た目が確認できる<font color="red">デザインビュー</font>と制約を確認できる<font color="red">ブループリントビュー</font>が表示されまる |
|⑤   |Properties             |現在選択されているビューの詳細な情報が表示されます |

## ConstraintLayout
Android 2.3(Gingerbread)以上を対象に開発を行う場合に利用することが可能なコンポーネントです。
constraint(制約)をWidgetやViewに設定することで画面内での配置を決めることができます。

また、従来からあるの"Linear Layout"や"Relative Layout"コンポーネントに比べネスト構造を減らした実装を行うことができるようになりxmlファイルを確認する場合でも簡単になりました。

## 検索画面のレイアウト作成
アプリ起動時に表示される検索画面のレイアウトを作成を始めます、
Androidアプリで画面レイアウトを構成するのは、`xml`という形式のファイルになります。
`activity_main.xml`というファイルを開き修正していきます。
> app -> res -> layout -> activity_main.xml

<img src="le0.png" alt="alt" title="create layout" width="500">
以下の画像のようにLayout Editorが表示されたら、サンプルの為で始めから表示されている**"Hello World"**をクリックして```delete```キーを押して削除します
<img src="le1.png" alt="alt" title="create layout" width="500">
これで画面上に何もなくなりスッキリしました。

では新しくウィジェットなどを配置する前に今回開発する蔵書検索アプリの起動時画面を見直してみましょう。
{% img /android/04-MakeLayoutDesign/CompleteScreen.png 500 CompleteScreen %}
完成した画面を見ると以下のウィジェットが配置されています。
* 文字入力できるウィジェットが一つ
* ボタンとしてクリックができるウィジェットが２つ

全部で３つのウィジェットを使いレイアウトが構成されていることがわかりました。

まずはアプリで文字入力を行うために`EditText`というウィジェットからデザインビューに配置していきます。
Android開発で用意されているウィジェットやビューはLayout Editor内の<font color="red">Paletteエリア</font>に全て表示されています。

では`EditText`を配置するために<font color="red">Paletteエリア</font>から**Plain Text**をクリックしたまま引っ張ってデザインビューでクリックを離します。
{% img /android/04-MakeLayoutDesign/le2.png 500 create layout %}
続いてウィジェットやビューをデザインビューに乗せたら表示させたい位置に配置するために、"editText"の各変に制約を設定していきます、
"editText"は画面中央より少し左上に表示したいので、まずは画面左隅と隣合うように制約をつけます。

"editText"左枠に表示されている **◯**をクリックしたまま画面の左端まで引っ張って放します。
{% img /android/04-MakeLayoutDesign/le3.png 500 create layout %}
"editText"が以下の画面のように画面左隅に近づきます。
{% img /android/04-MakeLayoutDesign/le4.png 500 create layout %}
上記の手順で画面左端と"editText"の左辺を隣接する"制約"が設定されました。
続けて上下と右辺も同じように各画面端と隣接する制約を設定してみてください。
{% img /android/04-MakeLayoutDesign/le5.png 500 create layout %}
上下左右の全てに制約を付けると"editText"は画面の真ん中に表示さされます。
{% img /android/04-MakeLayoutDesign/le6.png 500 create layout %}
"editText"は中央より少し左上に配置したいので制約の縦横"バイアス"細かい位置調整します
バイアス設定のツマミを動かすと対象の方向で位置を動かすことができます
{% img /android/04-MakeLayoutDesign/le7.png 500 create layout %}

|バイアス方向|設定値|
|:---------|:----|
|垂直バイアス|40   |
|水平バイアス|25   |

ボタンを２つ配置していきます、<font color="red">Paletteエリア</font>から`Button`をクリックしたまま引っ張りデザインビュー上で放します
{% img /android/04-MakeLayoutDesign/le8.png 500 create layout %}
"蔵書検索"ボタンを配置のため制約を設定していきます
{% img /android/04-MakeLayoutDesign/le9.png 500 create layout %}

|Buttonの辺|隣り合わせる箇所  |
|:---------|:-------------|
|上辺       |画面上端       |
|左辺       |"editText"右端 |
|下辺       |画面下端       |
|右辺       |画面右端       |

先ほどの"editText"と同じ高さに調整するため、バイアスを調整します
{% img /android/04-MakeLayoutDesign/le10.png 500 create layout %}

|バイアス方向|設定値|
|:---------|:----|
|垂直バイアス|40   |
|水平バイアス|50   |

**Button**に表示される文言を変更するために、右の<font color="red">Attributesエリア</font>から"text"項目を探し、**"蔵書検索"**と入力します
{% img /android/04-MakeLayoutDesign/le11.png 500 create layout %}

残りの"閲覧履歴"ボタンを画面の通り制約を使って配置してみましょう
<font color="red">ヒント１：ウィジェットの辺同士に制約をつけることもできます</font>
{% img /android/04-MakeLayoutDesign/le12.png 500 create layout %}

これで検索画面のレイアウトが完成しましたのでエミュレータにて表示確認をしてみましょう。

## 表示確認
レイアウトやプログラムを修正したら動かして確認してみたくなりますね、
今回はAndroid Studioに内臓されているエミュレータ（仮想デバイス）を使って表示確認をしてみます。

エミュレータや実機でアプリを実行するためには下記画面の`Run`ボタンをクリックします
{% img /android/04-MakeLayoutDesign/runemurate01.png 500 Run Project %}

"Select Deployment Target"と書かれたダイアログが表示されますが、ここには接続されている実機やエミュレータなど、Androidアプリを実行するデバイスを選ぶことができます。

USBで接続されたAndroidデバイスや既にエミュレータをインストールされているPCでない場合は以下のように何も表示されないと思います。
{% img /android/04-MakeLayoutDesign/runemurate02.png 500 Run Project %}
今回はエミュレータを新規作成して使用しますので、`Create New Virtual Device`をクリックして新しいエミュレータを作成します。
作成するエミュレータを選びますので<font color="red">**Pixel**</font>を選択して`Next`をクリックします
{% img /android/04-MakeLayoutDesign/runemurate03.png 500 Run Project %}
エミュレータにインストールするAndorid OSのバージョンを選択します。
<font color="red">**Marshmallow**</font>を選択して`Next`をクリックします
{% img /android/04-MakeLayoutDesign/runemurate04.png 500 Run Project %}
エミュレータの名前や設定変更画面では変更は行わずに`Finish`をクリックします
{% img /android/04-MakeLayoutDesign/runemurate05.png 500 Run Project %}
エミュレータの作成が終わると、実行するデバイスの選択画面に戻ります
今回は作成したエミュレータの名前が表示されるので、<font color="red">**Pixel API 23**</font>を選択して`OK`をクリックします
{% img /android/04-MakeLayoutDesign/runemurate06.png 500 Run Project %}
Androidのエミュレータが起動し
{% img /android/04-MakeLayoutDesign/runemurate07.png 300 Run Project %}
エミュレータの画面に作成したレイアウトが表示されれば成功です！
{% img /android/04-MakeLayoutDesign/runemurate08.png 250 Run Project %}

これでAndroidアプリのレイアウト作成とエミュレータでの実行方法と表示確認は完了です。
次のページでは作成したレイアウトのボタンやEditTextがクリックや文字入力された時のプログラムを実装していきます。
[ボタンイベントの実装](/AndroidCourse/android/05-ButtonAction)