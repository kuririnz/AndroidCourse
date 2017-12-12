---
title: Androidの概念
date: 2017-11-18 17:36:11
tags:
---
Androidアプリ開発に必要になる概念を学んでいきます

<!-- toc -->

# Android開発における４大要素
Android アプリの開発には重要に扱われている4大要素があります。
各要素を一言で表すと以下４つになります。
１.画面
２.アプリ/機能の呼び出し
３.バックグラウンド実行
４.外部からの呼び出し受付

また上記の要素をAndroidアプリの開発に当てはめると以下名称にて使われます。
* Activity(1.画面)
* Intent(2.アプリ/機能の呼び出し)
* Service(3.バックグラウンド実行)
* Broad Cast Receiver (4.外部からの呼び出し受付)

## Activity
アプリ開発における画面を指し、ユーザーからの操作の受付、ユーザーへの情報表示のために使用します。
Activity内にはwidgetまたはviewと呼ばれる要素（ボタンや画像）を配置して機能を追加する。
また画面単位の要素としてFragmentが存在し、Activity内に表示させて使うことが推奨されている。

## Intent
Activity間の呼び出しを行うための仕組み。
画面遷移やActivity間でのデータのやりとり、他にもメールアプリやブラウザアプリなど、
外部アプリの呼び出しを行う時に利用します。

## Service
バックグラウンドで長時間の作業を行うためのコンポーネントです。
例として、Content Providerや非同期通信によるデータのやりとり、画面を表示しないまま音楽再生をする場合などに利用します。

## Content Provider
構造化されたデータへのアクセスを管理する仕組み。
アプリ内のデータベースとのやりとりや端末内の別アプリが管理しているデータへのアクセスを行う場合などに利用します。

# その他の要素

* Fragment
* View
	* ViewGroup
	* Widget
* Life cycle (ライフサイクル)

## Fragment
コンテンツやWidget,ライフサイクルを持ったView
子要素を持つことが可能なのでActivityと同じようにViewGroupやWidgetを配置することができる。
また、Activityと独立したライフサイクルを持っており、ライフサイクルはActivityに近い形で実装されている。

## View
ウィジェットやViewGroupの総称、こちらにもライフサイクルに近い機能は存在しているが、
ActivityやFragmentとは違ったサイクルが存在する。
**LifeCycle**項目で紹介します。

### ViewGroup
子要素を持つことができるView
LinearLayout,RelativeLayout、最近追加されたものとしてはConstraintsLayoutが該当します。
他にListViewやGridView、RecyclerViewもViewGroupに該当します。
どのコンポーネントも内部にView要素を持つことができる要素になります。

LinearLayoutやRelativeLayoutは画面内の配置を容易にしてくれる機能を持っていたり、
ListViewなどは一覧データを並べて表示する機能があります。

### Widget
子要素を持つことができないView、これをViewと呼ぶことが多い気がします
ButtonやTextView、CheckBoxなどがWidgetと呼ばれる要素になります。
コンポーネント自体がユーザインターフェースの役割をこなすことができるViewになります

## LifeCycle
Androidの画面要素にはLife cycleと言う概念を含んでおり、
Activity / Fragment View要素にはそれぞれインスタンス化前、表示前、非表示前、破棄前など要素ごとに表示やアプリ内での状態が切り替わったタイミングで強制的に処理される

各クラスのライフサイクルに関して
Activityのライフサイクル[※1]
<img src="https://developer.android.com/images/activity_lifecycle.png?hl=ja" alt="alt" title="activity life cycle" width="350">

Fragmentのライフサイクル[※2]
<img src="https://developer.android.com/images/fragment_lifecycle.png?hl=ja" alt="alt" title="fragment life cycle" width="300">

Viewのライフサイクル[※3]
<img src="viewinlifecycle.png" alt="alt" title="view life cycle" width="450">

# Layout EditorとConstraints Layout
Android Studio 2.2から追加されたLayoutEditorとConstraintsLayoutを使用することでアプリ画面のレイアウト実装を視覚的に確認しながら実装できるようになった。

## Layout Editor
視覚的にViewやWidgetを配置することができるようになった、Constrains Layoutと合わせて使うことで画面のレイアウトを行う際にxmlによるプログラムを記述する手間が
省きながら実際に配置を確認できるようになった。

## Constraints Layout
Android 2.3(Gingerbread)以上を対象に開発を行う場合に利用することが可能。
constraints(制約)をViewやWidgetに設けることでレイアウトを決めることができる。
従来の"Linear Layout"や"Relative Layout"に比べネスト構造を減らした実装を行うことができるようになった。

[※1]: https://developer.android.com/guide/components/activities.html?hl=ja
[※2]: https://developer.android.com/guide/components/fragments.html?hl=ja
[※3]: https://developer.android.com/reference/android/view/View.html#pubmethods