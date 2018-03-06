---
title: 蔵書詳細の元ウェブページの表示
date: 2018-03-06
tags:
---
蔵書詳細の元となるウェブページを表示する中でWebViewを使ったWebページの実装を学習します。

<!-- toc -->

[操作性・ユーザ体験の改善](/AndroidCourse/android/10-OrganiseExperience)からの引き続きの学習ページです。
# 学習ポイント
* WebView実装の基礎
* 既存のブラウザアプリで指定のURLを表示
* Chrome Custom Tabs

Androidアプリにて自身(または法人)で管理している既存のホームページなどを表示したいなどの場合があります。
その場合にAndroidではアプリ内で表示する方法、またはAndroidデバイスにインストールされたブラウザアプリ(Chromeやmobile safari)にURLを教えて表示する２つの方法があります。
本ページでは上記２つの機能を実装しながら機能を学習します。

# WebView と Chrome Custom Tabs
最近はAndroidに置いてChrome Custom TabsというWebViewやChromeアプリより高速な表示を行えるコンポーネントも出てきました、これには一部デメリットもあるのですが、非常に高速でセキュリティ面を考慮したコンポーネントになっています。

## メリット & デメリット

|コンポーネント       |メリット|デメリット|
|:------------------|-------|---------|
|WebView            |・URL毎のハンドリング可 <br> ・javascriptの実行可|・ソーシャルログイン不可|
|Chrome Custom Tabs |・表示が早い <br> ・ソーシャルログイン可|・URL毎のハンドリング不可 <br> ・javascriptの実行不可|

ソーシャルログインに関しては[Google Developers](https://developers.googleblog.com/2016/08/modernizing-oauth-interactions-in-native-apps.html)でも勧告され2017/4/20よりWebViewでのソーシャルログインは廃止されGoogleのSDKかChrome Custom Tabsでの実装に切り替えるよう支持されています。

# WebView表示の実装
新しくFragmentを作成しレイアウトにWebViewを配置し、幅や高さを設定していきます。
今回はProjectウィンドウから`Fragment`ディレクトリを選択した状態で新規Fragment作成の手順を行います。
> New -> Fragment -> Fragment(Blank)

{% img /android/11-BrowseWebPage/CrWebViewFragment.png 550 CreateWebViewFragment %}

|項目                               |設定値                                  |
|----------------------------------|----------------------------------------|
|Fragment Name                     |BTWebViewFragment                       |
|Create Layout XML?                |チェックを<font color="blue">つける</font>|
|Fragment Layout Name              |fragment_btweb_view                     |
|Include fragment factory methods? |チェックを<font color="red">つけない</font>|
|Include interface callbacks?      |チェックを<font color="red">つけない</font>|
|Source Language                   |Java                                    |


**WebViewでの表示を行うために現状からは以下３つの修正が必要になります。**
1. Modelクラス(DetailDataModel.java)にWebページリンクを取得するパラメータ(変数)を追加
2. WebView表示Fragmentの実装
3. DetailFragmentクラス → WebView表示Fragmentへの遷移ボタンを配置

## 1.DetailDataModelクラスを編集
WebViewへのURLを調べるために[Google Androidアプリ開発ガイド](https://www.googleapis.com/books/v1/volumes/zgkQ8p4zUPsC)の蔵書単体情報を取得するAPIの検索結果を確認します。
APIの結果には**previewLink**、**infoLink**という項目があり、それぞれのURLをさらにブラウザで確認するとGoogle BooksのURLとPlayStoreの書籍画面のURLであることがわかります。
{% img /android/11-BrowseWebPage/ConfirmParams.png 500 ConfirmResponse %}
また２つの情報の階層としては`VolumeInfo`の中であることがわかりましたので、一旦この項目２つを`DetailDataModel`クラスのパラメータに追加します。
```java DetailDataModel.java
    // 蔵書概要クラス
    public class VolumeInfo {
    	...一部省略
        // 蔵書サムネイル画像URL
        public ImageLinks imageLinks;
        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // Google BooksへのリンクURL
        public String previewLink;
        // Play StoreへのリンクURL
        public String infoLink;
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    }

    // 蔵書サムネイルクラス
    public class ImageLinks {
        // 蔵書小サイズサムネイル
        public String smallThumbnail;
        // 蔵書サムネイル
        public String thumbnail;
        // 中サイズ表示画像
        public String medium;
    }
```

取得パラメータの追加は完了しました、取得したパラメータのURLは`BTWebViewFragment`へ遷移する時の連携データとして使用します。

## 2.WebView表示Fragmentの実装
続いて実際に表示する`BTWebViewFragment`を実装していきます、レイアウトには`WebView`コンポーネントを配置し、BTWebViewFragmentクラスにはWebViewでの表示プログラムを実装します。

### レイアウト作成
以下ファイルを開き、テキストエディタモードでレイアウトを編集します。

> app -> res -> layout -> fragment_btweb_view.xml

```XML fragment_btweb_view.xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="kuririnz.xyz.bookdiscovery.Fragment.BTWebViewFragment">

    <!-- ↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓削除↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓ -->
    <!-- TODO: Update blank fragment layout -->
    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:text="@string/hello_blank_fragment" />
    <!-- ↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑削除↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑ -->
    <!-- ↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓ -->
        <WebView
        android:id="@+id/FragmentWebView"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
    <!-- ↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑ -->
</FrameLayout>
```

`BTWebViewFragment`に置けるレイアウトとしてはWebViewで対象URLのウェブページを表示するだけなので`WebView`コンポーネントをは配置し画面全体に表示して完了です。
`FrameLayout`は他のLayout(LinearLayout,RelativeLayout,ConstraintLayout)と違い子Viewを配置するための便利な機能を持ち合わせていないことが特徴で、
今回のように単純なレイアウトであったり、表示座標を固定して表示したい場合にのみ`FrameLayout`コンポーネントを使用することをお勧めします。

### BTWebViewFragment実装
`BTWebViewFragment`クラスは蔵書詳細画面から表示するURLデータを連携してもらい、
アプリ内のWebViewで表示するための設定をしてからURLの読み込みを開始するよう実装します。
```java BTWebViewFragment.java
public class BTWebViewFragment extends Fragment {

    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    // データ連携用定数
    public static final String BUNDLE_URL = "BUNDLE_URL";

    // バインドコンポーネント
    private WebView webview;
    // メンバ変数
    private String defaultUrl;

    // スタティックコンストラクタ
    public static BTWebViewFragment getInstance(String previewLink) {
        // BTWebViewFragmentインスタンスを生成
        BTWebViewFragment fragment = new BTWebViewFragment();
        // BTWebViewFragmentへデータを渡すためのBundleを初期化
        Bundle args = new Bundle();
        // Google Booksのウェブページリンクをデータ渡し
        args.putString(BUNDLE_URL, previewLink);
        // データ格納クラスをBTWebViewFragmentインスタンスにセット
        fragment.setArguments(args);
        // 生成したFragmentを返却
        return fragment;
    }
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

    ...一部省略
	
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.fragment_web_view, container, false);
    }
    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    @Override
    public void onActivityCreated(@Nullable Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);

        // 遷移時の連携データを取得
        if (getArguments() != null) {
            // 遷移時に登録したKeyValueデータがない場合はGoogleページを表示
            defaultUrl = getArguments().getString(BUNDLE_URL, "https://www.google.co.jp/");
        }

        // レイアウトのコンポーネントをバインド
        webview = getView().findViewById(R.id.FragmentWebView);
        // 自身のWebViewで表示するためにWebViewClientをWebViewに設定
        webview.setWebViewClient(new WebViewClient());
        // URLの読み込み
        webview.loadUrl(defaultUrl);
    }
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
}
```
これで`BTWebViewFragment`クラスの実装も完了です。

## 3.DetailFragmentクラス → WebView表示Fragmentへの遷移ボタンを配置
最後に**DetailFragment**から`BTWebViewFragment`へ遷移するためのボタンを配置し、遷移するためのコードを実装します。
以下のようなレイアウトになるように実装します。
{% img /android/11-BrowseWebPage/UpdateDetailLayout.png 250 UpdateDetailFragment %}
```XML fragment_detail.xml
...一部省略
    <LinearLayout
        android:orientation="horizontal"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:weightSum="2">
        <ImageView
            android:id="@+id/DetailImage"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:paddingLeft="8dp"
            android:scaleType="fitCenter"
            android:adjustViewBounds="true"/>
        <LinearLayout
            android:orientation="vertical"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1">
            <TextView
                android:id="@+id/DetailSubTitle"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                tools:text="蔵書サブタイトル"/>
            <TextView
                android:id="@+id/DetailAuthor"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="8dp"
                tools:text="蔵書作者名"/>
            <Button
                android:id="@+id/TransitionWebView"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="WebViewで確認"/>
        </LinearLayout>
    </LinearLayout>
...一部省略
```
```java Detailfragment.java
public class DetailFragment extends Fragment {

    // 定数
    // データ渡しのキー情報
    private final static String BUNDLE_KEY = "BUNDLE_SELFLINK";

    // xmlファイルのコンポーネントと関連付ける要素
    private TextView titleText;
    private TextView subTitleText;
    private TextView authorText;
    private TextView descriptText;
    private TextView pageText;
    private TextView publishDateText;
    private ImageView detailImage;
    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    private Button transWebviewBtn;
    // Play Store リンクURL
    private String infoLink;
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    // 個体リンクのURL
    private String selfLink;

    ...一部省略

    @Override
    public void onActivityCreated(@Nullable Bundle savedInstanceState) {
        ...一部省略
        detailImage = getView().findViewById(R.id.DetailImage);
        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        transWebviewBtn = getView().findViewById(R.id.TransitionWebView);

        // BTWebViewFragmentへの遷移処理を実装
        transWebviewBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                // BTWebViewFragmentをインスタンス化
                BTWebViewFragment fragment = BTWebViewFragment.getInstance(infoLink);
                // 別のFragmentに遷移するためのクラスをインスタンス化
                FragmentTransaction ft = getFragmentManager().beginTransaction();
                // 現在、DetailFragmentを表示しているR.id.FragmentContainerをBTWebViewFragmentに置き換え
                ft.replace(R.id.FragmentContainer, fragment);
                // 表示していたFragmentをバックスタックに追加
                ft.addToBackStack(null);
                // 変更を反映
                ft.commit();
            }
        });
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

        ...一部省略
            @Override
            public void onResponse(Call call, Response response) throws IOException {
                // JsonパースライブラリGsonのインスタンス化
                Gson gson = new Gson();
                // 返却されたJson文字列を一旦変数に代入
                String jsonString = response.body().string();
                // DetailDataModelクラスに代入
                DetailDataModel detailData = gson.fromJson(jsonString, DetailDataModel.class);
                //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
                // Play Store へのリンクを代入
                infoLink = detailData.volumeInfo.infoLink;
                //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
                // パースが正常に行えたかLogcatに出力して確認。
                Log.d("DetailFragment parse", detailData.volumeInfo.title);
                // MainThreadに処理を渡し画面にデータを反映する
                handler.post(new ReflectDetail(detailData));
            }
        ...一部省略
    }
    ...一部省略
}
```
上記の通り、実装を進めると以下の箇所でエラーが出てくると思われます。
> ft.replace(R.id.FragmentContainer, fragment);

原因は`BTWebViewFragment`クラスで継承している**Fragment**クラスの問題になります。
以前に[蔵書詳細画面作成](/AndroidCourse/android/09-RefactorFragment)でFragmentクラスは以下の２種類あると解説をしました。
> Android SDK上にはFragmentクラスが２種類あります。
> * android.app.Fragment
> * android.support.v4.app.Fragment

DetailFragmentが継承しているFragmentは**android.support.v4.app.Fragment**になっているため、`getFragmentManager()`メソッドで取得できるFragmentManagerクラスは**support.v4のFragment**クラスだけということになります。
AppCompatAcvitiyクラスでは`getFragmentManager()`と`getSupportFragmentManager()`の２つメソッドがありFragmentとの関連は以下の通りになります。

* getFragmentManager() -> android.app.Fragment管理用
* getSupportFragmentManager() -> android.support.v4.app.Fragment管理用

ということになります、気になる場合は`ResultListActivity`の実装を確認してみてください。
ではDetailFragmentのエラーを解消するために`BTWebViewFragment`を修正します。
```java BTWebViewFragment.java
package kuririnz.xyz.bookdiscovery.Fragment;

//↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓削除↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
import android.app.Fragment;
//↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑削除↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
//↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
import android.support.v4.app.Fragment;
//↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
import android.os.Bundle;
import android.support.annotation.Nullable;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.webkit.WebView;
import android.webkit.WebViewClient;

import kuririnz.xyz.bookdiscovery.R;

/**
 * 蔵書詳細情報のウェブページを表示するFragment
 */
public class BTWebViewFragment extends Fragment {
    ...一部省略
}
```
上記実装ができたら動作確認します。
蔵書詳細画面から追加した**WEBVIEWで確認**ボタンをクリックしてアプリ内webViewで表示されることを確認します。

