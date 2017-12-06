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
* Life cycle (ライフサイクル)

## Fragment

## View(Widget)

## Life cycle
Androidの画面要素にはLife cycleと言う概念を含んでおり、
Activity / Fragment / View要素にはそれぞれインスタンス化前、表示前、非表示前、要素解放前など要素ごとに表示やアプリ内での状態が切り替わったタイミングで強制的に処理される
※ 今回の講座ではActivity / Fragmentのライフサイクルだけ紹介します


# Layout EditorとConstraints Layout
Android Studio 2.2から追加されたLayoutEditorとConstraintsLayoutを使用することでアプリ画面のレイアウト実装を視覚的に確認しながら実装できるようになった。

## Layout Editor
視覚的にViewやWidgetを配置することができるようになった、Constrains Layoutと合わせて使うことで画面のレイアウトを行う際にxmlによるプログラムを記述する手間が
省きながら実際に配置を確認できるようになった。

## Constraints Layout
Android 2.3(Gingerbread)以上を対象に開発を行う場合に利用することが可能。
constraints(制約)をViewやWidgetに設けることでレイアウトを決めることができる。
従来の"Linear Layout"や"Relative Layout"に比べネスト構造を減らした実装を行うことができるようになった。
