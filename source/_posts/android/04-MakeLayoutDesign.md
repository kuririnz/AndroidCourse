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
Android Studio 2.2から追加されたLayout Editor、ConstraintLayoutを使って画面レイアウトを作成します。
蔵書、または図書館の検索を行うための画面レイアウトを作成していきます。
手順としてレイアウトを作成後に作成したレイアウトに配置されたウィジェットのプログラムを作成する手順でアプリを
作っていきます。

**ここに完成画面を表示**

## Layout Editor
視覚的にViewやWidgetを配置することができるようになった、特にConstrainsLayoutと
合わせて使うことで画面のレイアウトを行う際にxmlによるプログラムを記述する手間が
省きながら実際に配置を確認できるようになった。

**Layout Editorの画像を表示**

## ConstraintLayout
Android 2.3(Gingerbread)以上を対象に開発を行う場合に利用することが可能。
constraint(制約)をViewやWidgetに設けることでレイアウトを決めることができる。
従来の"Linear Layout"や"Relative Layout"に比べネスト構造を減らした実装を行うことができるようになった。

**ConstraintLayoutの画像を表示**

制約という単語ですがgoogle翻訳としては
> ある条件を課して、自由にはさせないこと。その物事のために必要な条件

とあります、Layoutに当てはめるとコンポーネント同士を隣接させるという制約を

# Layout Editor / ConstraintLayoutによるレイアウト作成
1. 先ずは以下パスにあるメイン画面のレイアウトファイルを非表示します
**※app -> res -> layout -> activity_main.xml**
<img src="le0.png" alt="alt" title="create layout" width="500">
2. ２つの画面らしき表示が出たら**"Hello World"**と表示されたTextViewをクリックして```delete```キーで削除します
<img src="le1.png" alt="alt" title="create layout" width="500">
3. 検索文字を入力コンポーネントを画面に追加します
```Plain Text```をクリックしたまま引っ張って画面でクリックを離すと**editText**が画面に表示されます
<img src="le2.png" alt="alt" title="create layout" width="500">
4. editTextの配置を決めていきます
```editText```の左枠に表示されている **◯**をクリックしたまま画面の左端まで引っ張って放します
<img src="le3.png" alt="alt" title="create layout" width="500">
```editText```が以下の画面のように左に近づいたと思います。
<img src="le4_1.png" alt="alt" title="create layout" width="500">
「4.」の手順により画面左端とeditTextを隣接させる"制約"が設定されたことになります。
では上下と右辺も同じように各画面端と隣接する制約を設定してみましょう。
<img src="le4_2.png" alt="alt" title="create layout" width="500">
5. 上下左右に制約を付けるとEditTextは画面の真ん中に表示されます
<img src="le7.png" alt="alt" title="create layout" width="500">
6. editTextを真ん中より少し左上に配置するために縦横の"バイアス"を以下の通り調整します
<img src="le8.png" alt="alt" title="create layout" width="500">
	* 垂直バイアス -> **30**
	* 水平バイアス -> **25**
7. 次に***図書館検索***用の入力コンポーネント```editText2```を追加し画面の上下左右と制約をつけます
<img src="le10.png" alt="alt" title="create layout" width="500">
8. editText2を真ん中より少し左に配置するために縦横の"バイアス"を以下の通り調整します
<img src="le11.png" alt="alt" title="create layout" width="500">
	* 垂直バイアス -> **50**
	* 水平バイアス -> **25**
9. 次は検索のためのボタンを配置していきます、
"Plain Text"と同じように```Button```をクリックしたまま引っ張り画面上で放します
<img src="le12.png" alt="alt" title="create layout" width="500">
10. ボタンの配置場所を設定していきます。
<img src="le14.png" alt="alt" title="create layout" width="500">
	* 上辺 -> 画面上端
	* 左辺 -> "editText"右端
	* 下辺 -> 画面下端
	* 右辺 -> 画面右端
11. 上部の"editText"と同じ高さに調整するため、バイアスを調整します
<img src="le15.png" alt="alt" title="create layout" width="500">
	* 垂直バイアス -> **30**
	* 水平バイアス -> **50**
12. 画面に```Button```を2つ追加し以下のレイアウトになるように配置と制約の設定を行ってみましょう
※：講座にて時間をおいて解説します
<img src="le21.png" alt="alt" title="create layout" width="500">
	* レイアウト完成時にはComponent Treeに```Button2```、```Button3```が追加されます

# 動作確認
Android エミュレータにて配置したレイアウトで実行されることを確認してみましょう。
今回はAndroid Studioに内臓されているエミュレータを使って動作確認を行います。


## まとめ

これで検索画面のレイアウト作成が完了です、続いて作成したレイアウトのボタンや文字入力のイベントを
実装していきます。 [非同期処理、ネットワーク通信の実装](/AndroidCourse/android/05-SearchBooks)