---
title: レイアウト作成
date: 2017-11-05
---
Layout Editor / ConstraintLayoutを使用したレイアウト実装と動作確認

<!-- toc -->

# Layout Editor / ConstraintLayout
Android Studio 2.2から追加されたLayout Editor、ConstraintLayoutを使って画面レイアウトを作成します。

**ここに完成画面を表示**

## Layout Editor
視覚的にViewやWidgetを配置することができるようになった、特にConstrainsLayoutと
合わせて使うことで画面のレイアウトを行う際にxmlによるプログラムを記述する手間が
省きながら実際に配置を確認できるようになった。

## ConstraintLayout
Android 2.3(Gingerbread)以上を対象に開発を行う場合に利用することが可能。
constraint(制約)をViewやWidgetに設けることでレイアウトを決めることができる。
従来の"Linear Layout"や"Relative Layout"に比べネスト構造を減らした実装を行うことができるようになった。

制約という単語ですがgoogle翻訳としては
> ある条件を課して、自由にはさせないこと。その物事のために必要な条件

とあります、Layoutに当てはめるとコンポーネント同士を隣接させるという制約を

## ConstraintLayoutによるレイアウトの基本
1. 先ずは以下パスにあるメイン画面のレイアウトファイルを非表示します
**※app -> res -> layout -> activity_main.xml**
<img src="le0.png" alt="alt" title="create layout" width="500">
2. ２つの画面らしき表示が出たら**"Hello World"**と表示されたTextViewをクリックして```delete```キーで削除します
<img src="le1.png" alt="alt" title="create layout" width="500">
3. 検索文字を入力コンポーネントを画面に追加します
```PlainText```をクリックしたまま引っ張って画面でクリックを離すと**editText**が画面に表示されます
<img src="le2.png" alt="alt" title="create layout" width="500">
4. editTextの配置を決めていきます
```editText```の左枠に表示されている **◯**をクリックしたまま画面の左端まで引っ張って離します
<img src="le3.png" alt="alt" title="create layout" width="500">
```editText```が以下の画面のように左に近づいたと思います。
<img src="le4_1.png" alt="alt" title="create layout" width="500">
「4.」の手順により画面左端とeditTextを隣接させる"制約"が設定されたことになります。
では上下と右辺も同じように各画面端と隣接する制約を設定してみましょう。
<img src="le4_2.png" alt="alt" title="create layout" width="500">
5. 上下左右に制約を付けるとEditTextは画面の真ん中に表示されます
<img src="le7.png" alt="alt" title="create layout" width="500">
6. editTextを真ん中より少し左上に配置するために縦横の"バイアス"を以下の通り調整します
	* 垂直方向バイアス -> **30**
	* 水平方向バイアス -> **25**
<img src="le8.png" alt="alt" title="create layout" width="500">
7. 次に***図書館検索***用の入力コンポーネント```editText2```を追加し画面の上下左右と制約をつけます
<img src="le10.png" alt="alt" title="create layout" width="500">
8. editText2を真ん中より少し左に配置するために縦横の"バイアス"を以下の通り調整します
	* 垂直方向バイアス -> **50**
	* 水平方向バイアス -> **25**
<img src="le11.png" alt="alt" title="create layout" width="500">

<img src="le12.png" alt="alt" title="create layout" width="500">

<img src="le13.png" alt="alt" title="create layout" width="500">

<img src="le14.png" alt="alt" title="create layout" width="500">

<img src="le15.png" alt="alt" title="create layout" width="500">

<img src="le16.png" alt="alt" title="create layout" width="500">



入力画面,機能を実装してエミュレータ起動、デバッグ方法を記載