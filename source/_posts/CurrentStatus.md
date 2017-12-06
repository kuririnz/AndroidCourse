---
title: Android OS とは
date: 2017-11-18
---
Android OSの現在のシェアや動向を紹介します。

<!-- toc -->

# Android OSとは

* google社が作ったスマートフォン向け OS(オペレーションシステム)
* 無料で世界中の人が使えるオープンソースOS
* 世界で一番普及しているプログラミング言語 「Java」を使って開発する
* Google I/O 2017にてモダン言語 「Kotlin」を正式採用

## スマートフォン OSのシェア

現在世界で普及しているスマートフォン/タブレット端末に搭載されているOSは
iOSやAndroid OS,Windows Phoneの3つのOSが市場で多くのシェアを持っています。
2~3年前は Tizen, FireFox OS, ubuntu mobileなどいくつかのモバイルOSの開発が
ありましたが現在では開発を終了してしまっています。

|OS               |開発者       |最新のバージョン|
|:----------------|:----------:|:------------:|
|Android          |Google      |8.0           |
|iOS              |Apple       |11.1          |
|Windows 10 Mobile|Microsoft   |1607          |


### 世界のスマートフォン OS シェア


### 日本のスマートフォン OS シェア


## Android アプリの開発環境

Android アプリの開発には **Android Studio**というソフトウェアを使います。
> Android StudioはJET BRAINS社が開発したフリーのAndroid開発ソフトウェアです。
> 2015年頃までEclipseというソフトウェアでの開発もサポートしていましたが、現在はサポートを終了しています。

Androidアプリの開発は主に**java**や**Kotlin**と言うプログラミング言語で開発を行います。
それぞれのプログラミング言語の特徴を紹介します。

* java
	* Oracle(オラクル)社が開発したプログラミング言語
	* どんなOS環境での実行も可能なプログラミング言語
	* JVM(Java Virtual Machine)と言う仮想マシン上で動く
* Kotlin
	* JET BRAINS社が "java" をベースに開発したプログラミング言語
	* javaよりコード量を少なくAndroidアプリを開発できる
	* javaの動作環境があれば実行可能

### Android OSプラットフォーム

現在(2017/11)で最も多く利用されている Android OSバージョンは6.0(Marshmallow)になります。
販売されているAndroidデバイスは各販売元の企業によってカスタマイズされた物となっていることが多く、デバイスのスペックを含めた環境から販売元の企業によってアップデートできるOSを管理されており、最新のOSにアップデートできないことが多いです。
そのため、多くのユーザにアプリを配信したい場合は最も多く利用されているプラットフォームのコードネームより2つ前のコードネームまでを配信の対象にすると良いと思われます。

現在であれば**6.0(Marshmallow)**から2つ前のコードネームなので***4.4(KitKat)***までをサポート対象に含めることで、全世界のAndoroid OS ユーザの内、90%にアプリを配信することができることになります。

## Android アーキテクチャ

Android OSのアーキテクチャはLinux Kernelを基盤に作られています。
実際の開発に必ず必要になる知識ではありませんので紹介までにとどめさせていただきます。

*参考サイト*
> https://developer.android.com/about/dashboards/index.html#Platform
> https://developer.android.com/guide/platform/index.html?hl=ja#system-apps