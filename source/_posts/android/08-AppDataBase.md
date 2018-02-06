---
title: 検索履歴一覧画面の作成
lang: android
date: 2017-11-09
tags:
---
検索履歴一覧画面を作成/実装しながらアプリ内データベースの利用方法を学習する。

<!-- toc -->

[蔵書検索機能の作成](/AndroidCourse/android/07-AsyncProcess)からの引き続きの学習ページです。
# 学習ポイント
* Realmデータベースの使い方
* RecyclerView, RecyclerViewAdapterの使い方

Realmというデータベースライブラリを使いアプリが閉じたり端末電源が落ちた後でもデータが残る機能を使って検索を行った条件を保存しておき、過去の履歴が確認できる画面を作っていきます。
また、検索結果画面で使ったListViewとは別にRecyclerViewというListViewよりも自由度の高い一覧表示ができるコンポーネントを学習します。

# 新しい画面を作成する
検索履歴一覧画面の実装を進めるために、新しくActivityを作成します。