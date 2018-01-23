---
title: レイアウト作成
date: 2017-11-05
---
Layout Editor / ConstraintLayoutを使用したレイアウト実装と動作確認

<!-- toc -->

当ページではAnrdoidアプリのプロジェクト作成完了から検索画面のレイアウト完了まで解説しています。
[前のメージでAndroidアプリのプロジェクト作成に関して解説しています](/AndroidCourse/android/03-StartAndroidDevelopment)
次のページで検索画面のプログラム実装、REST API通信に関して解説します。

# Layout Editor / ConstraintLayout
Layout Editor、ConstraintLayoutの使い方を学びながら画面のレイアウトを作成していきます。
このページで作成する画面には最終的に以下の機能を実装していきます。
* 文字入力機能
* 画面遷移
* 蔵書/図書館検索結果取得

Androidアプリのプロジェクト開始時に生成されたxmlレイアウトファイルにウィジェットなどを配置し、ウィジェットごとに制約を設定して以下画像のように表示位置を決めていきます。

**ここに完成画面を表示**

## Layout Editor
視覚的にViewやWidgetを配置することができるようになった、特にConstrainsLayoutと
合わせて使うことで画面のレイアウトを行う際にxmlによるプログラムを記述する手間が
省きながら実際に配置を確認できるようになった。

<img src="https://developer.android.com/studio/images/write/layout-editor-callouts_2-2_2x.png?hl=ja" alt="alt" title="Layout Editor" width="550">
各領域の名称は以下の通りです

|No.  |領域名                 |説明                                             |
|:---:|:---------------------|:-----------------------------------------------|
|①   |Palette               |レイアウトに配置できるウィジェットとレイアウトリストを表示されています |
|②   |Component Tree        |レイアウトのビュー階層を表示/移動することができます、Viewの重なりもここで調整します |
|③   |ツールバー              |レイアウトの外観設定やプロパティを編集するためのボタンが表示されています |
|④   |デザイン エディタ        |実際の見た目が確認できる<font color="red">デザインビュー</font>と制約を確認できる<font color="red">ブループリントビュー</font>が表示されまる |
|⑤   |Properties             |現在選択されているビューの詳細な情報が表示されます |


## ConstraintLayout
Android 2.3(Gingerbread)以上を対象に開発を行う場合に利用することが可能。
constraint(制約)をViewやWidgetに設けることでレイアウトを決めることができる。
従来の"Linear Layout"や"Relative Layout"に比べネスト構造を減らした実装を行うことができるようになった。

**ConstraintLayoutの画像を表示**

制約という単語ですがgoogle翻訳としては
> ある条件を課して、自由にはさせないこと。その物事のために必要な条件

とあります、Layoutに当てはめるとコンポーネント同士を隣接させるという制約を

## レイアウト作成
では検索画面のレイアウト作成にはいっていきます
まずは以下のxmlレイアウトファイルを開きましょう
> app -> res -> layout -> activity_main.xml

<img src="le0.png" alt="alt" title="create layout" width="500">
以下の画像のようにLayout Editorが表示されたら、サンプルの為で始めから表示されている**"Hello World"**をクリックして```delete```キーで削除します
<img src="le1.png" alt="alt" title="create layout" width="500">
これで画面上に何もなくなりスッキリしました。
まず検索を行うためには何かしらの情報が必要になります今回は、**蔵書検索**を行いたいので、そのために蔵書名の一部が知りたいです、検索する蔵書は利用者によって違いますので、検索したい本の一部を入力してもらえると検索の情報になりそうです。

文字入力をアプリで受け付けるためには文字入力を受け付けるためのウィジェットを使う必要があり、
その場合は```EditText```というウィジェットを使います、ではこれをデザインビューに配置していきます。

Layout Editor内にある<font color="red">Palette</font>から```Plain Text```をクリックしたまま引っ張ってデザインビューでクリックを離すと**editText**が表示されます
<img src="le2.png" alt="alt" title="create layout" width="500">
続いてウィジェットやビューをデザインビューに乗せたら表示させたい位置に配置します、そのために"editText"に制約をつけていきます。
まずは"editText"は画面左隅と隣合うように配置させるため、"editText"左枠に表示されている **◯**をクリックしたまま画面の左端まで引っ張って放します
<img src="le3.png" alt="alt" title="create layout" width="500">
"editText"が以下の画面のように画面左隅に近づいたと思います。
<img src="le4.png" alt="alt" title="create layout" width="500">
先ほどの手順により画面左端とeditTextを隣り合わせる"制約"が設定されたことになります。
では上下と右辺も同じように各画面端と隣接する制約を設定してみましょう。
<img src="le5.png" alt="alt" title="create layout" width="500">
上下左右に制約を付けるとeditTextは画面の真ん中に表示されると思います
<img src="le6.png" alt="alt" title="create layout" width="500">
続いて一つ目のeditTextを中央より少し左上に配置するために縦横の"バイアス"を以下の通り調整します
バイアスを動かすと対象の方向に制約の範囲で動かすことができます
<img src="le7.png" alt="alt" title="create layout" width="500">

|バイアス方向|設定値|
|:---------|:----|
|垂直バイアス|40   |
|水平バイアス|25   |

続いて検索を実行するためのボタンを配置していきます
"Plain Text"の時と同じように今度は<font color="red">Palette</font>から```Button```をクリックしたまま引っ張りデザインビュー上で放します
<img src="le8.png" alt="alt" title="create layout" width="500">
"蔵書検索"用のボタンを配置のため制約を設定していきます
<img src="le9.png" alt="alt" title="create layout" width="500">

|Buttonの辺|隣り合わせる箇所  |
|:---------|:-------------|
|上辺       |画面上端       |
|左辺       |"editText"右端 |
|下辺       |画面下端       |
|右辺       |画面右端       |

上の"editText"と同じ高さに調整するため、バイアスを調整します
<img src="le10.png" alt="alt" title="create layout" width="500">

|バイアス方向|設定値|
|:---------|:----|
|垂直バイアス|40   |
|水平バイアス|50   |

残りの"検索履歴"ボタンを画面の通り制約を使って配置してみましょう
<img src="le11.png" alt="alt" title="create layout" width="500">
> レイアウト完成時にはComponent Treeに**Button2**が追加されます




最後に各ボタンに表示される文字を修正します
右上の**Button**をクリックし、右に表示される<font color="red">Attributes</font>エリアから"text"項目を探し、**蔵書検索**と入力します
<img src="le12.png" alt="alt" title="create layout" width="500">

同じ様に"Button2"ボタンの表示文言を修正してみましょう、
<img src="le13.png" alt="alt" title="create layout" width="500">
修正後にボタンに表示される文字列は**検索履歴**です

これで検索画面のレイアウトは完成です、変更された画面を確認してみましょう。

---

# 動作確認
配置したレイアウトで実行されることを確認してみましょう、
今回はAndroid Studioに内臓されているエミュレータ（仮想デバイス）を使って動作確認を行います。

アプリを実行するためには下記画面の`Run`ボタンをクリックします
<img src="runemurate01.png" alt="alt" title="Run Project" width="500">
実行するデバイスかエミュレータの選択画面が表示されますが、以下のように何も表示されないか、
AndroidデバイスをPCに繋いでいる場合はデバイスが表示されているかもしれません。

今回はエミュレータを使用しますので、`Create New Virtual Device`をクリックしてエミュレータを作成します。
<img src="runemurate02.png" alt="alt" title="Run Project" width="500">
作成するエミュレータを選びますので<font color="red">**Pixel**</font>を選択して`Next`をクリックします
<img src="runemurate03.png" alt="alt" title="Run Project" width="500">
エミュレータにインストールするAndorid OSのバージョンを選択します。
<font color="red">**Marshmallow**</font>を選択して`Next`をクリックします
<img src="runemurate04.png" alt="alt" title="Run Project" width="500">
エミュレータの名前や設定変更画面では変更は行わずに`Finish`をクリックします
<img src="runemurate05.png" alt="alt" title="Run Project" width="500">
エミュレータの作成が終わり実行するデバイスの選択画面に戻ります
今回は作成したエミュレータの名前が表示されるので、<font color="red">**Pixel API 23**</font>を選択して`OK`をクリックします
<img src="runemurate06.png" alt="alt" title="Run Project" width="500">
Androidのエミュレータが起動し
<img src="runemurate07.png" alt="alt" title="Run Project" width="300">
"layout_main.xml"で作成したレイアウトが表示されれば成功です！
<img src="runemurate08.png" alt="alt" title="Run Project" width="250">

これで検索画面のレイアウト作成が完了です、次のページで今回作成したレイアウトのボタンやEditTextにプログラムでイベントを
実装していきます。 [ボタンイベントの実装](/AndroidCourse/android/05-ButtonAction)