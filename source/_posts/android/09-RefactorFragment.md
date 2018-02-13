---
title: 蔵書詳細画面作成
date: 2017-11-10
tags:
---
蔵書詳細画面を作成する中でモデルクラスとFragmentの実装を学習します。

<!-- toc -->

[検索履歴一覧画面の作成](/AndroidCourse/android/08-AppDataBase)からの引き続きの学習ページです。
# 学習ポイント
* Fragment
* Fragmentによる画面遷移
* 定数
* Gson
* Glide

複数のデータをひとまとめにして管理するための方法としてモデルクラスを学習します。
また、検索結果一覧画面の各蔵書ごとの詳細情報を表示する蔵書詳細画面を作成し、遷移できるように修正していきます、その工程の中でActivity内に配置できるライフサイクルを持ったViewコンポーネント**Fragment**の利用方法と**Fragment**間の画面遷移に関して学習します。

# 検索結果一覧画面の構築移行
[検索結果一覧画面の作成](/AndroidCourse/android/06-TransitionScreen)ページでActivityを使い画面実装を行なっていましたが、これをそっくり**Fragment**を利用した実装に書き換えます。
さらに、複数表示されている検索結果一覧からクリックされた行の蔵書の詳細情報を表示する画面、"蔵書詳細画面"を**Fragment**で実装しこの画面に遷移するようアプリを作り変えていきます。

## Fragmentとは
FragmentはActivityのようにライフサイクルを持ったViewコンポーネントにあたり、ButtonやListViewなど複数のWidgetやViewを配置することが可能です、またActivity内に複数のFragmentを表示することができるので画面分割の必要な機能を作成する時には活躍が期待されます。

また、**Fragment**を利用するメリットはActivityに比べてAndroid端末が画面表示するコストが小さくなるため、アプリの強制終了のリスクを多少回避できることも上げられます。

逆に**Fragment**の利用にあたり注意しなければいけないのはActivityとは別のライフサイクルになるため、Activityがすでに存在しないタイミングで**Fragment**のライフサイクルメソッドが実行され強制終了する可能性があります。
また、**Fragment**自身はContextとしての要素は持ち合わせていないため、ActivityやApplicationクラスからContextを取得するなどしてレイアウト表示を実装する必要があります。

ではまず、検索結果一覧画面を**Fragment**を使った実装に作り変えていきます。
## 検索結果一覧Fragment作成
新しくFragmentクラスを継承した独自クラスのjavaファイルを作成していきます。

検索結果画面の時と同様にメニューもしくはプロジェクトツリーが表示されているウィンドウで右クリック(macでは２本指でクリック)から以下の項目を選択します。

> New -> Fragment -> Fragment(Blank)

{% img /android/09-RefactorFragment/addfragment01.png 500 Create New Fragment %}
新しいFragmentクラスを作成する時の設定項目は以下の通り入力したら`Finish`をクリックします。
{% img /android/09-RefactorFragment/addfragment02.png 500 Create New Fragment %}

|項目                               |設定値                                  |
|----------------------------------|----------------------------------------|
|Fragment Name                     |ResultListFragment                      |
|Create Layout XML?                |チェックを<font color="blue">つける</font>|
|Fragment Layout Name              |fragment_result_list                    |
|Include fragment factory methods? |チェックを<font color="red">つけない</font>|
|Include interface callbacks?      |チェックを<font color="red">つけない</font>|

新しく`ResultListFragment.java`と`fragment_result_list.xml`の2ファイルが作成されます。
{% img /android/09-RefactorFragment/addfragment03.png 600 Create New Fragment %}

## 検索結果一覧画面レイアウト実装
作成した`fragment_resultlist.xml`と`activity_result_list.xml`を編集します。
`fragment_result_list.xml`のファイルの場所は以下です。
> app -> res -> layou -> fragment_result_list.xml

`feagment_result_list.xml`を表示した時に**LayoutEditor**が表示されたら左下の"Text"タブをクリックしxmlでの編集画面に表示を切り替えます。
{% img /android/09-RefactorFragment/editRLfrg01.png 600 Edit Result List Fragment %}
```XML fragment_result_list.xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="kuririnz.xyz.bookdiscovery.ResultListFragment">

    <!-- ↓↓↓↓↓↓↓↓↓↓修正↓↓↓↓↓↓↓↓↓↓ -->
    <ListView
        android:id="@+id/FragmentResultListView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_margin="8dp" />
    <!-- ↑↑↑↑↑↑↑↑↑↑修正↑↑↑↑↑↑↑↑↑↑ -->

</FrameLayout>
```
一覧表示用の"ListView"を配置して画面いっぱいに表示されるように設定します。
次にFragmentを表示するactivityのレイアウトを修正していきます。
> app -> res -> layou -> activity_result_list.xml

上記レイアウトファイルを表示し、右下の"Design"タブをクリックします。
{% img /android/09-RefactorFragment/migratefrg01.png 600 Edit Result List Activity %} 
デザインビューに表示されている`ResultListView`をクリックし、<kbd>delete</kbd>キーを押下して削除します。
何も表示されていない状態になったら、`Palette -> Layouts -> FrameLayout`から"FrameLayout"をドラッグ&ドロップでデザインビューに表示します。
{% img /android/09-RefactorFragment/migratefrg02.png 600 Edit Result List Activity %} 
制約と属性の設定を行います。
{% img /android/09-RefactorFragment/migratefrg03.png 600 Edit Result List Activity %} 

**FrameLayout**に設定する制約
|FrameLayoutの辺|隣り合わせる箇所|
|:-------------|:-------------|
|上辺           |画面上端       |
|左辺           |画面左端       |
|下辺           |画面下端       |
|右辺           |画面右端       |

**FrameLayout**に設定する属性
|設定項目       |設定値                |
|--------------|---------------------|
|ID            |FragmentContainer    |
|layout_width  |match_constraint     |
|layout_height |match_constraint     |

これでレイアウトファイルの修正は完了です。

### FrameLayout コンポーネント
先ほどの実装で初めて出てきたコンポーネント"FrameLayout"です。
"FrameLayout"はAndroid SDKがリリースされた時からある古いコンポーネントです。
他の"ConstraintLayout"や"LinearLayout"、"RelativeLayout"などには子Viewを配置する上での便利な特徴がありますが、"FrameLayout"は特徴がないことが特徴となります。
"FrameLayout"での子View配置は左上の角を横/縦の基準(0,0)として正確にポイントを設定して期待する位置に配置する必要があるため、複数のコンポーネント同士を関連付けて配置するのが難しくなります。
また、Androidでポイントを指定して配置を行うと、数多く販売されているAndroid端末の全画面サイズに対応仕切れず、レイアウト崩れの原因になってしまうので表示するコンポーネントの少ない場合など細かい配置のないような場合で用いられます。

## 検索結果一覧Fragment機能実装
レイアウトの修正が終わりましたので`ResultListFragment.java`、`ResultListActivity.java`ファイルを実装します。
`ResultListFragment.java`の実装ほとんどが`ResultListActivity.java`のプログラムをコピーするだけで問題ないのですが、一部細かい違いがあるのでコピーしてエラーになった箇所など見比べて確認してみてください。

まずFragmentクラスに少し手を加えていきます。
`ResultListFragment.java`のクラス宣言より上に表示されている`import..`の一覧から
*import android.app.Fragment;*を削除します。
{% img /android/09-RefactorFragment/chngimport01.png 600 Import Change %} 
次に<font color="red">赤くなった`Fragmet`</font>をクリックし上に表示されるツールチップに従い
<kbd>Alt</kbd><kbd>+</kbd><kbd>Enter</kbd>(Macは<kbd>option</kbd><kbd>+</kbd><kbd>Enter</kbd>)を入力
{% img /android/09-RefactorFragment/chngimport02.png 600 Import Change %}
ウィンドウに選択肢が表示されたら**Fragment (android.support.v4.app)**を選択します。
{% img /android/09-RefactorFragment/chngimport03.png 600 Import Change %} 

Android SDK上にはFragmentクラスが２種類あります。
* android.app.Fragment
* android.support.v4.app.Fragment

Google者からは後者の”android.support.v4.app.Fragment”の使用を推奨されています。
大きは違いとしてAndroid API 16(OS 4.1)以下にもアプリを配信する場合は"android.support.v4.app.Fragment"をimportしてFragmentを使用する必要があります。

その他の違いとしては以下の通りです。
* android.support.v4.app.Fragmentの方がバグが少ない。
* android.app.Fragmentでは一部メソッドが使えない。

実装中今までほとんど触れてこなかったimportエリアですが、この部分にはクラス内で利用している外部のクラスを指定する必要があり、ここに対象のクラスがインポートされていないとクラス内部の実装で使用することができません、実装する上で必要なクラスはここで全てインポートします。
候補検索など小さなウィンドウから選択して実装する場合は自動的にインポートされるのであまり気にしないでプログラミングして問題ありません。

では機能の実装を進めていきます。
```java ResultListFragment.java
//↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓修正↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
public class ResultListFragment extends Fragment implements AdapterView.OnItemClickListener {
//↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑修正↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    // 定数
    // データ渡しのキー情報
    private final static String BUNDLE_KEY = "BUNDLE_TERM";

    // xmlファイルのコンポーネントと関連付ける要素
    private ListView resultListView;
    // ListViewの表示内容を管理するクラス
    private ResultListAdapter adapter;
    // OkHttp通信クライアント
    private OkHttpClient okHttpClient;
    // メインスレッドに戻ってくるためのHandler
    private Handler handler;
    // MainActivityから渡されたデータを保持する
    private String term;

    // スタティックコンストラクタ
    public static ResultListFragment getInstance(String term) {
        // ResultListFragmentインスタンスを生成
        ResultListFragment fragment = new ResultListFragment();
        // ResultListFragmentに渡すデータ格納クラスを生成
        Bundle args = new Bundle();
        // 検索文字列データを連携データにセット
        args.putString(BUNDLE_KEY, term);
        // データ格納クラスをResultListFragmentインスタンスにセット
        fragment.setArguments(args);
        // 生成したResultListFragmentを返却
        return fragment;
    }
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

    // コンストラクタ
    public ResultListFragment() {
        // Required empty public constructor
    }

    // Fragmentが表示するレイアウトを指定するライフサイクルメソッド
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.fragment_result_list, container, false);
    }

	//↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    // 親となるActivityが生成された後に実行されるライフサイクルメソッド
    @Override
    public void onActivityCreated(@Nullable Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);

        // Handlerをインスタンス化
        handler = new Handler();
        // 検索文字列変数を初期化
        term = "Android";
        // 連携データが存在するか確認
        if (getArguments() != null) {
            // 連携データ内から"term"キーのデータを代入、なければ"Android"と文字列を代入
            term = getArguments().getString(BUNDLE_KEY, "Android");
        }

        // xmlファイルのコンポーネントと関連付け
        resultListView =  getView().findViewById(R.id.FragmentResultListView);
        // OkHttp通信クライアントをインスタンス化
        okHttpClient = new OkHttpClient();
        // 通信するための情報
        // MainActivityで入力された文字列で検索する様修正
        Request request = new Request.Builder().url("https://www.googleapis.com/books/v1/volumes?q=" + term).build();
        // データの取得後の命令を実装
        Callback callBack = new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                // 失敗した時の命令
                // 通信に失敗した原因をログに出力
                Log.e("failure API Response", e.getLocalizedMessage());
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                // 成功した時の命令
                // Google Books APIから取得したデータをログに出力
                // Jsonのパースが失敗してアプリの強制終了を回避する機能
                try {
                    // JsonデータをJSONObjectに変換
                    JSONObject rootJson = new JSONObject(response.body().string());
                    // Jsonデータから蔵書リストデータ"items"を取得
                    JSONArray items = rootJson.getJSONArray("items");
                    Log.d("Success API Response", "APIから取得したデータの件数:" +
                            items.length());
                    // メインスレッドで実行する処理をインスタンス化
                    ReflectResult reflectResult = new ReflectResult(items);
                    // Handlerにてメインスレッドに処理を戻し、ReflectResultのrunメソッドを実行する
                    handler.post(reflectResult);
                } catch (JSONException e) {
                    // Jsonパースの時にエラーが発生したらログに出力する
                    e.printStackTrace();
                }
            }
        };
        // 非同期処理でAPI通信を実行
        okHttpClient.newCall(request).enqueue(callBack);

    }

    // ListViewの各行をクリックした時の命令を実装
    @Override
    public void onItemClick(AdapterView<?> adapterView, View view, int i, long l) {
        // クリックした行番号をToastで表示する
        Toast.makeText(getContext()
                , (i + 1) + "行目をクリックしました"
                , Toast.LENGTH_SHORT).show();
    }

    // 検索結果をListViewに反映するメインスレッドの処理クラス
    private class ReflectResult implements Runnable {
        // 蔵書一覧タイトルデータリスト
        private List<String> titleList;
        // 蔵書一覧概要データリスト
        private List<String> summaryList;

        // コンストラクタ
        public ReflectResult(JSONArray items) {
            // リストデータを初期化
            titleList = new ArrayList<>();
            summaryList = new ArrayList<>();
            // Jsonのパースエラーが発生した時に備えてtry~catchする
            try{
                // 蔵書リストの件数分繰り返しタイトルをログ出力する
                for (int i = 0; i < items.length(); i ++) {
                    // 蔵書リストから i番目のデータを取得
                    JSONObject item = items.getJSONObject(i);
                    // 蔵書のi番目データから蔵書情報のグループを取得
                    JSONObject volumeInfo = item.getJSONObject("volumeInfo");
                    // タイトルデータをリストに追加
                    titleList.add(volumeInfo.getString("title"));
                    // 概要データをリストに追加
                    summaryList.add(volumeInfo.getString("description"));
                }
            } catch (JSONException e) {
                e.printStackTrace();
            }
        }

        // Handlerから実行されるメソッド
        @Override
        public void run() {
            // ListViewに表示する情報をまとめるAdapterをインスタンス化
            adapter = new ResultListAdapter(getContext(), titleList, summaryList);
            // ListViewに表示情報をまとめたAdapterをセット
            resultListView.setAdapter(adapter);
            // ListViewに行をクリックした時のイベントを登録
            resultListView.setOnItemClickListener(ResultListFragment.this);
        }
    }
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
}
```
```java ResultListActivity.java
//↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓修正↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
public class ResultListActivity extends AppCompatActivity {
//↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑修正↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓削除↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    // xmlファイルのコンポーネントと関連付ける要素
    private ListView resultListView;
    // ListViewの表示内容を管理するクラス
    private ResultListAdapter adapter;
    // OkHttp通信クライアント
    private OkHttpClient okHttpClient;
    // メインスレッドに戻ってくるためのHandler
    private Handler handler;
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑削除↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    // MainActivityから渡されたデータを保持する
    private String term;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_result_list);
		
        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓削除↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // Handlerをインスタンス化
        handler = new Handler();
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑削除↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
        // 画面遷移時のデータが空でない場合
        if (getIntent().hasExtra("terms")) {
            // Key:termsにデータがあればValueを代入
            term = getIntent().getStringExtra("terms");
        } else {
            // 画面遷移時のデータがからの場合は "Android"と文字列を代入
            term = "Android";
        }
        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓削除↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // xmlファイルのコンポーネントと関連付け
        resultListView = findViewById(R.id.ResultListView);
        // OkHttp通信クライアントをインスタンス化
        okHttpClient = new OkHttpClient();
        // 通信するための情報
        // MainActivityで入力された文字列で検索する様修正
        Request request = new Request.Builder().url("https://www.googleapis.com/books/v1/volumes?q=" + term).build();
        // データの取得後の命令を実装
        Callback callBack = new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                // 失敗した時の命令
                // 通信に失敗した原因をログに出力
                Log.e("failure API Response", e.getLocalizedMessage());
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                // 成功した時の命令
                // Google Books APIから取得したデータをログに出力
                // Jsonのパースが失敗してアプリの強制終了を回避する機能
                try {
                    // JsonデータをJSONObjectに変換
                    JSONObject rootJson = new JSONObject(response.body().string());
                    // Jsonデータから蔵書リストデータ"items"を取得
                    JSONArray items = rootJson.getJSONArray("items");
                    Log.d("Success API Response", "APIから取得したデータの件数:" +
                            items.length());
                    // メインスレッドで実行する処理をインスタンス化
                    ReflectResult reflectResult = new ReflectResult(items);
                    // Handlerにてメインスレッドに処理を戻し、ReflectResultのrunメソッドを実行する
                    handler.post(reflectResult);
                } catch (JSONException e) {
                    // Jsonパースの時にエラーが発生したらログに出力する
                    e.printStackTrace();
                }
            }
        };
        // 非同期処理でAPI通信を実行
        okHttpClient.newCall(request).enqueue(callBack);
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑削除↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
		//↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // FragmentContainerにResultListFragmentを表示させる処理
        ResultListFragment resultListFragment = ResultListFragment.getInstance(term);
        // Activity内で表示するFragmentを管理するクラスをインスタンス化
        FragmentManager fm = getFragmentManager();
        // Fragmentを表示、または別のFragmentに遷移するためのクラスをインスタンス化
        FragmentTransaction ft = fm.beginTransaction();
        // FragmentManagerに新しいFragmentを追加
        // FragmentContainerにResultListFragmentを表示するよう設定
        ft.add(R.id.FragmentContainer, resultListFragment);
        // 上記の設定でFragmentManagerを更新
        ft.commit();
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    }

    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓削除↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    // ListViewの各行をクリックした時の命令を実装
    @Override
    public void onItemClick(AdapterView<?> adapterView, View view, int i, long l) {
        // クリックした行番号をToastで表示する
        Toast.makeText(ResultListActivity.this
                , (i + 1) + "行目をクリックしました"
                , Toast.LENGTH_SHORT).show();
    }

    // 検索結果をListViewに反映するメインスレッドの処理クラス
    private class ReflectResult implements Runnable {
        // 蔵書一覧タイトルデータリスト
        private List<String> titleList;
        // 蔵書一覧概要データリスト
        private List<String> summaryList;

        // コンストラクタ
        public ReflectResult(JSONArray items) {
            // リストデータを初期化
            titleList = new ArrayList<>();
            summaryList = new ArrayList<>();
            // Jsonのパースエラーが発生した時に備えてtry~catchする
            try{
                // 蔵書リストの件数分繰り返しタイトルをログ出力する
                for (int i = 0; i < items.length(); i ++) {
                    // 蔵書リストから i番目のデータを取得
                    JSONObject item = items.getJSONObject(i);
                    // 蔵書のi番目データから蔵書情報のグループを取得
                    JSONObject volumeInfo = item.getJSONObject("volumeInfo");
                    // タイトルデータをリストに追加
                    titleList.add(volumeInfo.getString("title"));
                    // 概要データをリストに追加
                    summaryList.add(volumeInfo.getString("description"));
                }
            } catch (JSONException e) {
                e.printStackTrace();
            }
        }

        // Handlerから実行されるメソッド
        @Override
        public void run() {
            // ListViewに表示する情報をまとめるAdapterをインスタンス化
            adapter = new ResultListAdapter(ResultListActivity.this, titleList, summaryList);
            // ListViewに表示情報をまとめたAdapterをセット
            resultListView.setAdapter(adapter);
            // ListViewに行をクリックした時のイベントを登録
            resultListView.setOnItemClickListener(ResultListActivity.this);
        }
    }
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑削除↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
}
```
上記のコード修正が終わったら動作確認します。
正常に検索結果画面に遷移できるか確認してみましょう。

今回Fragmentの実装では２つのライフサイクルメソッドを利用しました、[Androidの概念](/AndroidCourse/android/02-AndroidConcept)で一度ライフサイクルの図をご紹介しましたが、
Fragmentの`onCreateView()`、`onActivityCreated()`はActivityの`onCreate()`メソッドの後に実行されます。

Fragmentで画面の要素を取得するためには`getView()`メソッドを利用します。
`getView()`メソッドはFragment内のどのライフサイクルでも参照できますが、`onAttach()`〜`onCtreateView()`のライフサイクルメソッドでは、ActivityにFragmentのレイアウトが反映されていないため"null"が返却されるので注意が必要です。

またFragmentは引数のないコンストラクタを必ず用意しないといけません。
Android端末ではメモリが不足した場合にActivityやFragmentを一旦破棄し必要になったタイミングで再度生成される仕組みになっており、この時にFragmentは引数のないコンストラクタを実行しインスタンス化を行っています。
メモリからFragmentが破棄されても`Bundle`は破棄されずに残るため、`fragment.setArguments(Bundle)`を使いFragmentでデータ渡しを行う必要があります。
Fragmentのインスタンス化時に引数ありのコンストラクタを使用していた場合、万が一メモリ不足からFragmentが再生成された時にアプリが強制終了する可能性が考えられます。
画面が回転した時など画面サイズに変更が加わった場合などを考慮する場合は注意して実装する必要がある。

## 定数
文字通り宣言時にセットした値を変更することができない要素。
上記で実装した通り**final**のキーワードを変数宣言時に記述することで定数としての扱いになります。
また**static**のキーワードをつけることでクラスがインスタンス化されていなくても参照することができるようになります。
java,Androidでは定数と変数を見分けやすくするために全て大文字で定数名を宣言することが多いです。
```java
    private final static String BUNDLE_KEY = "BUNDLE_TERM";
```

## Context概要
Androidアプリ開発の中ではxmlレイアウトをjavaファイルで参照する時などには必ず必要になる"Context"は一体何か。
Activity,Service,Applicationの親クラスであり、アプリケーショングローバル情報へアクセスするためのインターフェースの役割を持つクラス。
アプリケーションのグローバル情報とは

* パーミッション(インターネット接続などの許可設定)
* アプリリソースへのアクセス(画像やレイアウトなど"res"ディレクトリ配下の要素を参照するための機能)
* アプリ情報(コンポーネント、プロセス名、テーマなど)

などが該当します。
Contextを利用することで上記の情報を参照することが可能になります。

Contextの取得方法には`getActivity()[Activity内ではthis]`,`getApplicationContext()`などいくつか方法がありますが、各Contextは利用可能な寿命が違っており、Activityを対象にしたContext(`getActivity()`で取得したContext)の場合はActivityが破棄されると同時にContextも破棄されます。
対して`getApplicationContext()`ではApplicationクラスに依存するため、アプリを破棄するまで使用できます。
現状では影響のあるプログラムはありませんが、ActivityとApplicationに設定されているテーマ(Theme)がそれぞれ違う場合に期待しない表示になるなどの不具合が発生する可能性が考えられます。

あまり使われないメソッドですが`getBaseContext()`というメソッドも存在しますが、深い話しで使用は推奨されていないようです。。。

Contextはレイアウトなどの"res"ディレクトリ内を参照したりレイアウトをjavaファイルで使うための便利かつ必要なものと認識して利用すれば問題ありません。
実際にアプリ開発で内部情報に影響されることはないと思われます。

# 蔵書詳細画面作成
ActivityからFragmentへの移行が完了したら蔵書詳細画面を作成していきます。
{% img /android/09-RefactorFragment/detailExample.png 250 Detail Example %}

検索結果画面の時と同様にメニューもしくはプロジェクトツリーが表示されているウィンドウで右クリック(macでは２本指でクリック)から以下の項目を選択します。

> New -> Fragment -> Fragment(Blank)

作成するFragment名や同時に作成されるレイアウトファイル名を入力し、以下の項目の設定を修正したら`Finish`をクリックします。
{% img /android/09-RefactorFragment/addDetail01.png 600 Add Detail Screen %} 

|項目                               |設定値                                  |
|----------------------------------|----------------------------------------|
|Fragment Name                     |DetailFragment                          |
|Create Layout XML?                |チェックを<font color="blue">つける</font>|
|Fragment Layout Name              |fragment_detail                         |
|Include fragment factory methods? |チェックを<font color="red">つけない</font>|
|Include interface callbacks?      |チェックを<font color="red">つけない</font>|

`DetailFragment.java`と`fragment_detail.xml`ファイルが生成されます。
まずは蔵書詳細画面のレイアウトを修正していきます。
`fragment_detail.xml`を開き、*Design*タブが表示されている場合は*Text*タブをクリックしてxmlファイルの実装画面に切り替えます。
{% img /android/09-RefactorFragment/addDetail02.png 600 Add Detail Screen %} 
```XML fragment_detail.xml
<!-- ↓↓↓↓↓↓↓↓↓↓削除↓↓↓↓↓↓↓↓↓↓ -->
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="kuririnz.xyz.bookdiscovery.DetailFragment">

    <!-- TODO: Update blank fragment layout -->
    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:text="@string/hello_blank_fragment" />

</FrameLayout>
<!-- ↑↑↑↑↑↑↑↑↑↑削除↑↑↑↑↑↑↑↑↑↑ -->
<!-- ↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓ -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:layout_margin="8dp"
    tools:context="kuririnz.xyz.bookdiscovery.DetailFragment">

    <TextView
        android:id="@+id/DetailTitle"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textSize="20dp"
        tools:text="蔵書タイトル"/>
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
        </LinearLayout>
    </LinearLayout>
    <TextView
        android:id="@+id/DetailDescription"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="8dp"
        android:lines="10"
        tools:text="蔵書概要を表示" />
    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="8dp">
        <TextView
            android:id="@+id/DetailPageHeader"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_alignParentLeft="true"
            android:text="ページ数："/>
        <TextView
            android:id="@+id/DetailPageText"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_toRightOf="@id/DetailPageHeader"
            tools:text="53ページ"/>
        <TextView
            android:id="@+id/DetailPublishDateHeader"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_toLeftOf="@id/DetailPublishDateText"
            android:text="発売日："/>
        <TextView
            android:id="@+id/DetailPublishDateText"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_alignParentRight="true"
            tools:text="2018/02/06"/>
    </RelativeLayout>
</LinearLayout>
<!-- ↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑ -->
```
これで蔵書詳細画面のレイアウトは完了です。

"LinearLayout"や"RelativeLayout"を使用して画面レイアウトを作成するとこのようにLayoutの中にLayoutやTextViewなどのコンポーネントを配置するネスト構造での実装が必要になります。
"ConstraintLayout"ではネスト構造にならずにレイアウトを作成することが可能です。

今回出てきた属性について簡単に解説していきます。

|対象のView|属性|説明|
|---------|----|----|
|LinearLayout|android:weightSum|縦または横方向に対して設定された数値分に均等分割する。縦横方向はorientation属性から判定される|
|weightSum属性をセットしたLinearLayoutの子View|android:layout_weight|親のLinearLayoutで設定されたweightSumのうち占有する割合値を設定する、有効にする場合には"layout_width"または"layout_height"を0dpにする必要がある。android:layout_alignParentLeft|
|RelativeLayoutの子View|android:layout_alignParentLeft|親のRelativeLayoutの左端に自Viewの左端を合わせて配置する|
|RelativeLayoutの子View|android:layout_alignParentRight|親のRelativeLayoutの右端に自Viewの右端を合わせて配置する|
|RelativeLayoutの子View|android:layout_toRightOf|設定したidを持つViewの右側に自Viewを配置する|
|RelativeLayoutの子View|android:layout_toLeftOf|設定したidを持つViewの左側に自Viewを配置する|

# 蔵書詳細画面遷移実装
蔵書検索結果一覧画面から行アイテムをクリックしたら蔵書詳細画面に遷移するように実装を修正します。
```java ResultListFragment.java

    ...一部省略

    // ListViewの各行をクリックした時の命令を実装
    @Override
    public void onItemClick(AdapterView<?> adapterView, View view, int i, long l) {
        // クリックした行番号をToastで表示する
        Toast.makeText(getContext()
                , (i + 1) + "行目をクリックしました"
                , Toast.LENGTH_SHORT).show();
        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // 蔵書詳細画面用Fragmentをインスタンス化
        DetailFragment detailFragment = new DetailFragment();
        // support.v4.app.Fragment内ではgetFragmentManager = Activity.getSupportFragmentManager
        FragmentManager fm = getFragmentManager();
        // 別のFragmentに遷移するためのクラスをインスタンス化
        FragmentTransaction ft = fm.beginTransaction();
        // Fragmentを表示させるViewのidとFragmentクラスを設定
        ft.replace(R.id.FragmentContainer, detailFragment);
        // 表示していたFragmentをバックスタックに追加
        ft.addToBackStack(null);
        // FragmentManagerに反映
        ft.commit();
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    }
    ...一部省略
```
上記コードを実装したら動作確認します。

これで自動的に表示していたFragmentがスタック（破棄せず残っているデータ）され、スタックされたFragmentが残っている場合はActivityではなく、Fragmentが一つ戻るように実装されました。
Fragmentには"ID"や"TAG"という情報を設定することができ、一つ前のFragmentではなくスタックされいているFragmentから"ID"や"TAG"を指定して画面バックを行うことも可能です。

# 蔵書詳細画面実装
蔵書の詳細情報として著作者や概要などを表示していきます。
蔵書を一つを特定するために今回は検索結果一覧画面でREST APIから取得したデータに含まれる"selfLink"キーに含まれるurl情報を蔵書詳細画面で改めてREST APIを使用して取得する形で実装していきます。
蔵書詳細画面で取得したデータのパースに関してはJSONデータをモデルクラスに一発変換してくれるライブラリを使ってJSONObjectでの実装の手間を軽くする方法を使います。
また、検索結果一覧画面で表示するデータも増えてきたため、まとめてデータを持てるようにモデルクラスを作成して管理します。

まずは、JSON文字列の簡単パースライブラリ**Gson**を導入します。
プロジェクトからapp階層の`build.gradle`を開き、"dependencies" の "{}"内に以下のコードを記述します
開くのは`build.gradle`の後ろに***(Module: app)***と表示されている方です。
{% img /android/07-AsyncProcess/includeokHttp01.png 550 IncludeokHttp %}
```gradle build.gradle(Module: app)
dependencies {
    ...一部省略
    implementation 'com.squareup.okhttp3:okhttp:3.9.1'
    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    compile 'com.google.code.gson:gson:2.2.4'
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
}
```
上記コードを追加後、<font color="blue">**Sync Now**</font>をクリックしてGSONの導入は完了です。
{% img /android/09-RefactorFragment/addDetail06.png 550 Include GSON %}
次に新しくモデルクラスを作成します。
> プロジェクトウィンドウ右クリック > New > Java Class

モデルクラスは２つ作成します。
一つは検索結果一覧でデータを表示するためのモデルクラス、もう一つは蔵書詳細画面のREST APIをパースするためのモデルクラスです。
まずは検索結果一覧に一覧表示しているデータをモデルクラスとしてまとめます。

クラス名とパッケージを確認したら`OK`をクリック
{% img /android/09-RefactorFragment/addDetail07.png 550 Include GSON %}
こちらは検索結果一覧画面で利用するモデルクラスとして変数を宣言していきます。
```java ResultListMdel.java
public class ResultListModel {

    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    // 蔵書タイトル
    public String title;
    // 蔵書概要
    public String summary;
    // 蔵書単体リンク
    public String selfLink;
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
}
```

クラス名とパッケージを確認したら`OK`をクリック
{% img /android/09-RefactorFragment/addDetail08.png 550 Include GSON %}
２つ目のクラスは蔵書詳細画面のREST APIデータをパースして使用するためのモデルクラスです。
```java DetailDataModal.java
public class DetailDataModel {

    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    // 蔵書単体リンク
    public String selfLink;
    // 蔵書概要データ
    public VolumeInfo volumeInfo;

    // 蔵書概要クラス
    public class VolumeInfo {
        // 蔵書タイトル
        public String title;
        // 蔵書サブタイトル
        public String subTitle;
        // 蔵書著者リスト
        public List<String> authors;
        // 蔵書発売日
        public String publishedDate;
        // 蔵書概要
        public String description;
        // 蔵書ページ数
        public int pageCount;
        // 蔵書サムネイル画像URL
        public ImageLinks imageLinks;
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
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
}
```
`DetailFragment.java`のプログラムを修正する前に"import"設定を修正します。
`ResultListFragment.java`と同様に`import..`の一覧から
*import android.app.Fragment;*を削除します。
{% img /android/09-RefactorFragment/addDetail03.png 600 Import Change %} 
次に<font color="red">赤くなった`Fragmet`</font>をクリックし上に表示されるツールチップに従い
<kbd>Alt</kbd><kbd>+</kbd><kbd>Enter</kbd>(Macは<kbd>option</kbd><kbd>+</kbd><kbd>Enter</kbd>)を入力
{% img /android/09-RefactorFragment/addDetail04.png 600 Import Change %}
ウィンドウに選択肢が表示されたら**Fragment (android.support.v4.app)**を選択します。
{% img /android/09-RefactorFragment/addDetail05.png 600 Import Change %} 
`DetailFragment.java`では取得したAPIデータをパースし、一旦タイトルをログに出力させてみます。
```java DetailFragment.java
public class DetailFragment extends Fragment {
	
    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    // データ渡しのキー情報
    private final static String BUNDLE_KEY = "BUNDLE_SELFLINK";

    // xmlファイルのコンポーネントと関連付ける要素
    private TextView titleText;
    private TextView subTitleText;
    private TextView authorText;
    private TextView descriptText;
    private TextView pageText;
    private TextView publishDateText;
    // APIの検索に使うISBNコード
    private String isbn;
    // APIのデータ取得後処理を行うためのHandler
    private Handler handler;
    // OkHttp通信クライアント
    private OkHttpClient okHttpClient;

    // スタティックコンストラクタ
    public static DetailFragment getInstance(String selfLink) {
        // DetailFragmentインスタンスを生成
        DetailFragment fragment = new DetailFragment();
        // DetailFragmentに渡すデータ格納クラスを生成
        Bundle args = new Bundle();
        // 検索文字列データを連携データにセット
        args.putString(BUNDLE_KEY, selfLink);
        // データ格納クラスをDetailFragmentインスタンスにセット
        fragment.setArguments(args);
        // 生成したResultListFragmentを返却
        return fragment;
    }
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

    public DetailFragment() {
        // Required empty public constructor
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.fragment_detail, container, false);
    }

    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    @Override
    public void onActivityCreated(@Nullable Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);

        // Handlerをインスタンス化
        handler = new Handler();
        // 連携データが存在するか確認
        if (getArguments() != null) {
            // 連携データ内から"BUNDLE_SELFLINK"キーのデータを代入、なければ"Android"と文字列を代入
            selfLink = getArguments().getString(BUNDLE_KEY, "");
        }

        // selfLinkが空の場合は検索結果一覧画面に強制バック
        if (TextUtils.isEmpty(selfLink)) {
            getFragmentManager().popBackStack();
        }

        // xmlファイルのコンポーネントと関連付け
        titleText = getView().findViewById(R.id.DetailTitle);
        subTitleText = getView().findViewById(R.id.DetailSubTitle);
        authorText = getView().findViewById(R.id.DetailAuthor);
        descriptText = getView().findViewById(R.id.DetailDescription);
        pageText = getView().findViewById(R.id.DetailPageText);
        publishDateText = getView().findViewById(R.id.DetailPublishDateText);

        // OkHttp通信クライアントをインスタンス化
        okHttpClient = new OkHttpClient();
        // 通信するための情報
        // ResultListFragmentから取得したselfLinkURLにREST API通信を行う
        Request request = new Request.Builder().url(selfLink).build();
        // データの取得後の命令を実装
        Callback callBack = new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                // 通信に失敗した原因をログに出力
                Log.e("failure API Response", e.getLocalizedMessage());
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                // JsonパースライブラリGsonのインスタンス化
                Gson gson = new Gson();
                // 返却されたJson文字列を一旦変数に代入
                String jsonString = response.body().string();
                // DetailDataModelクラスに代入
                DetailDataModel detailData = gson.fromJson(jsonString, DetailDataModel.class);
                // パースが正常に行えたかLogcatに出力して確認。
                Log.d("DetailFragment parse", detailData.volumeInfo.title);
            }
        };
        // 非同期処理でREST API通信を実行
        okHttpClient.newCall(request).enqueue(callBack);
    }
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
}
```
```java ResultListAdapter.java
public class ResultListAdapter extends BaseAdapter {

    // ListViewの描画に必要な変数を宣言
    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓削除↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    private List<String> titleList;
    private List<String> summaryList;
    private LayoutInflater layoutInflater;

    // コンストラクタ(インスタンス時に呼び出されるメソッドのようなもの)
    public ResultListAdapter(Context context, List<String> titleList, List<String> summaryList) {
        this.titleList = titleList;
        this.summaryList = summaryList;
        this.layoutInflater = LayoutInflater.from(context);
    }

    @Override
    public int getCount() {
        // 一覧表示する要素数を返却する
        return titleList.size();
    }
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑削除↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    private List<ResultListModel> resultList;
    private LayoutInflater layoutInflater;

    // コンストラクタ
    public ResultListAdapter(Context context, List<ResultListModel> resultList) {
        this.resultList = resultList;
        this.layoutInflater = LayoutInflater.from(context);
    }

    @Override
    public int getCount() {
        // 一覧表示する要素数を返却する
        return resultList.size();
    }
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

    ...一部省略

    @Override
    public View getView(int i, View view, ViewGroup viewGroup) {
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

        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓削除↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        titleView.setText(titleList.get(i));
        summaryView.setText(summaryList.get(i));
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑削除↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        titleView.setText(resultList.get(i).title);
        summaryView.setText(resultList.get(i).summary);
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

        // 文字情報を代入されたviewを返却
        return view;
    }
```
```java ResultListFragment.java
public class ResultListFragment extends Fragment implements AdapterView.OnItemClickListener{

    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    // 検索結果一覧データ
    private List<ResultListModel> resultList;
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

    ...一部省略
    // ListViewの各行をクリックした時の命令を実装
    @Override
    public void onItemClick(AdapterView<?> adapterView, View view, int i, long l) {
        // クリックした行番号をToastで表示する
        Toast.makeText(getContext()
                , (i + 1) + "行目をクリックしました"
                , Toast.LENGTH_SHORT).show();
        // 蔵書詳細画面用Fragmentをインスタンス化
        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓削除↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        DetailFragment detailFragment = new DetailFragment();
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑削除↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        DetailFragment detailFragment = DetailFragment.getInstance(resultList.get(i).selfLink);
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
        // support.v4.app.Fragment内ではgetFragmentManager = Activity.getSupportFragmentManager
        FragmentManager fm = getFragmentManager();
        // 別のFragmentに遷移するためのクラスをインスタンス化
        FragmentTransaction ft = fm.beginTransaction();
        // Fragmentを表示させるViewのidとFragmentクラスを設定
        ft.replace(R.id.FragmentContainer, detailFragment);
        // 表示していたFragmentをバックスタックに追加
        ft.addToBackStack(null);
        // FragmentManagerに反映
        ft.commit();
    }

    // 検索結果をListViewに反映するメインスレッドの処理クラス
    private class ReflectResult implements Runnable {
        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓削除↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // 蔵書一覧タイトルデータリスト
        private List<String> titleList;
        // 蔵書一覧概要データリスト
        private List<String> summaryList;
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑削除↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // 蔵書モデルクラスリスト
        private List<ResultListModel> resultList;
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
        
        // コンストラクタ
        public ReflectResult(JSONArray items) {
            //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓削除↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
            titleList = new ArrayList<>();
            summaryList = new ArrayList<>();
            //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑削除↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
            // Jsonのパースエラーが発生した時に備えてtry~catchする
            try{
                // 蔵書リストの件数分繰り返しタイトルをログ出力する
                for (int i = 0; i < items.length(); i ++) {
                    // 蔵書リストから i番目のデータを取得
                    JSONObject item = items.getJSONObject(i);
                    // 蔵書のi番目データから蔵書情報のグループを取得
                    JSONObject volumeInfo = item.getJSONObject("volumeInfo");
                    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓削除↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
                    // タイトルデータをリストに追加
                    titleList.add(volumeInfo.getString("title"));
                    // 概要データをリストに追加
                    summaryList.add(volumeInfo.getString("description"));
                    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑削除↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
                    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
                    // 蔵書データクラスをインスタンス化
                    ResultListModel resultData = new ResultListModel();
                    // タイトルをモデルクラスに代入
                    resultData.title = volumeInfo.getString("title");
                    // 個体蔵書データURLをモデルクラスに代入
                    resultData.selfLink = item.getString("selfLink");
                    // データに"description"キーが含まれている場合は情報を代入
                    if (volumeInfo.has("description")) {
                        // 概要をモデルクラスに代入
                        resultData.summary = volumeInfo.getString("description");
                    } else {
                        // "description"キーが含まれていない場合は空文字データを代入
                        resultData.summary = "";
                    }
                    // 蔵書情報をリストに登録
                    resultList.add(resultData);
                    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
                }
            } catch (JSONException e) {
                e.printStackTrace();
            }
        }

        // Handlerから実行されるメソッド
        @Override
        public void run() {
            // ListViewに表示する情報をまとめるAdapterをインスタンス化
            //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓削除↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
            adapter = new ResultListAdapter(getContext(), titleList, summaryList);
            //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑削除↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
            //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
            adapter = new ResultListAdapter(getContext(), resultList);
            //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
            // ListViewに表示情報をまとめたAdapterをセット
            resultListView.setAdapter(adapter);
            // ListViewに行をクリックした時のイベントを登録
            resultListView.setOnItemClickListener(ResultListFragment.this);
        }
    }
}
```
上記コード修正が終わったら動作確認してみます。
正常に処理が実行されれば蔵書詳細画面に遷移し、**Logcat**に検索結果一覧で選択した蔵書のタイトルが表示されると思います。

AndroidアプリではREST API通信を行い取得したデータをGsonというライブラリを利用することで簡単にパースすることができます。
ただGsonでパースするときには取得データと同じ階層構造を指定したクラスを作成して上げる必要があるので期待通りパースされない場合、まずJSONデータと作成したクラスの階層構造をお確認すると良いでしょう。
また取得するデータキーとクラスの変数名を合わせて実装する必要があるので入力ミスにも注意が必要です。

今回の実装で便利なUtilクラスを使いましたので紹介です、`DetailFragment.java`において"selfLink"変数にデータが格納されているか判定するために
```java
TextUtils.isEmpty(selfLink)
```
という実装をしています、`TextUtils`はString型変数の処理で便利な機能が揃っており上記のメソッドでは、引数の内容が"null"や空文字列かを判定し"true"を返却してくれます。
多く使うメソッドはこの`TextUtils.isEmpty()`が多くなると思いますが、判定処理を完結しにてくれるのでとても有効です。

続いて蔵書詳細データを画面のTextViewにセットしていきます。
```java DetailFragment.java
public class DetailFragment extends Fragment {
    ...一部省略

    @Override
    public void onActivityCreated(@Nullable Bundle savedInstanceState) {
	    ...一部省略

        // データの取得後の命令を実装
        Callback callBack = new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                // 通信に失敗した原因をログに出力
                Log.e("failure API Response", e.getLocalizedMessage());
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                // JsonパースライブラリGsonのインスタンス化
                Gson gson = new Gson();
                // 返却されたJson文字列を一旦変数に代入
                String jsonString = response.body().string();
                // DetailDataModelクラスに代入
                DetailDataModel detailData = gson.fromJson(jsonString, DetailDataModel.class);
                // パースが正常に行えたかLogcatに出力して確認。
                Log.d("DetailFragment parse", detailData.volumeInfo.title);
                //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
                // MainThreadに処理を渡し画面にデータを反映する
                handler.post(new ReflectDetail(detailData));
                //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
            }
        };
        // 非同期処理でREST API通信を実行
        okHttpClient.newCall(request).enqueue(callBack);
    }

    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    // REST APIで取得したデータを画面に反映するためのクラス
    private class ReflectDetail implements Runnable {
        // 蔵書詳細データ
        DetailDataModel detailData;

        // コンストラクタ
        public ReflectDetail(DetailDataModel detailData) {
            this.detailData = detailData;
        }

        // Handlerから実行されるメソッド
        @Override
        public void run() {
            // タイトルを反映
            titleText.setText(detailData.volumeInfo.title);
            // サブタイトルが取得できていたら反映
            if (!TextUtils.isEmpty(detailData.volumeInfo.subTitle)) {
                subTitleText.setText(detailData.volumeInfo.subTitle);
            }
            // 概要が取得できていたら反映
            if (!TextUtils.isEmpty(detailData.volumeInfo.description)) {
                descriptText.setText(detailData.volumeInfo.description);
            }
            // 著作者名が取得できていたら反映
            if (detailData.volumeInfo.authors != null && detailData.volumeInfo.authors.size() > 0) {
                String authorString = new String();
                // 著作者名が複数設定されていう場合があるので繰り返し処理で全て表示する
                for (String author : detailData.volumeInfo.authors) {
                    authorString += author + ",";
                }
                authorText.setText(authorString);
            }
            // ページ数を反映
            pageText.setText(String.valueOf(detailData.volumeInfo.pageCount));
            // 発売日が取得できていたら反映
            if (!TextUtils.isEmpty(detailData.volumeInfo.publishedDate)) {
                publishDateText.setText(detailData.volumeInfo.publishedDate);
            }
        }
    }
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
}
```
上記コードを実装したら動作確認します。
蔵書詳細画面に遷移してREST APIのデータを取得できるまでに少し間が空きますが、正常に実装されるとTextViewに文言が反映されます。

上記の実装では必要なタイミングで**論理演算子(&&)**が使われました、そしてポイントは著作者名リストの**for**文の使い方です。
## for-each
繰り返し処理を実装する方法として"for文"を学習しました、今回の**for-each**文ではコレクション型の変数に登録されている要素の回数分、繰り返し処理を実行する時の実装方法です。
コレクション型以外にもMap型(講座の中ではまだ未使用)や配列型の変数でも同様の実装が可能です。
**for-each**文の実装テンプレート
```java
for (配列要素の型 一時変数 : リスト型の変数) {
	// 繰り返し処理
}
```
一時変数にはリスト型変数の要素が0番目〜最後の要素まで代入された状態で繰り返し処理で参照することが可能です。
実際には**for-each**文の方が利用頻度は多いと思われます。

## URL画像読み込み処理の実装
このページではActivity → Fragmentの移行に始まり、新規のクラスを作る回数も多かったので非常にコーディングに時間がかかったと思います。
それも今回の実装で終わりです！

Web上にアップロードされた画像をアプリで読み込みむためにも便利なライブラリがあります。
本来はBitmapという形式で画像インスタンスを生成し使用しなくなるときにはちゃんとAndroid OSの処理の阻害にならないよう解放する必要があるのですが、
**Glide**というライブラリを使用することでそういったメモリ管理などの忘れがちな処理を補ってくれます。

では他のライブラリと同様に`build.gradle`に依存関係を追加します。
プロジェクトからappディレクトリの`build.gradle`を開き、"dependencies" の "{}"内に以下のコードを記述します
開くのは`build.gradle`の後ろに***(Module: app)***と表示されている方です。
```gradle build.gradle(Module: app)
android {
    compileSdkVersion 27
    defaultConfig {
        applicationId "kuririnz.xyz.bookdiscovery"
        minSdkVersion 23
        targetSdkVersion 27
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation 'com.android.support:appcompat-v7:27.0.2'
    implementation 'com.android.support.constraint:constraint-layout:1.0.2'
    implementation 'com.android.support:design:27.0.2'
    implementation 'com.squareup.okhttp3:okhttp:3.9.1'
    compile 'com.google.code.gson:gson:2.2.4'
    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    implementation 'com.github.bumptech.glide:glide:4.6.1'
    annotationProcessor 'com.github.bumptech.glide:compiler:4.6.1'
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
}
```
上記コードを追加後、<font color="blue">**Sync Now**</font>をクリックして**Glide**の導入は完了です。
**Glide**の最新版(バージョン4.6.1)を使用する際には`dependencies`内の`implementation 'com.android.support:appcompat-v7...'`と`implementation 'com.android.support:design...'`の指定バージョンを"27.0.2"に設定する必要があるため、合わせて"compileSdkVersion"、"targetSdkVersion"の指定を**27**に設定しなければいけません。

これは**Glide**が依存関係を持っているandroid.supportバージョンが**27**であることが原因で、**Glide**の古いバージョンの場合には開発アプリ側の"compileSdkVersion"、"targetSdkVersion"を古いものでも実装が可能だと思われます。

では**Glide**を使って蔵書詳細画面のImageViewに画像を読み込ませてみます。
```java DetailFragment.java
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
    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    private ImageView detailImage;
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    // 個体リンクのURL
    private String selfLink;
  
    ...一部省略

    @Override
    public void onActivityCreated(@Nullable Bundle savedInstanceState) {
       
       ...一部省略
        publishDateText = getView().findViewById(R.id.DetailPublishDateText);
        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        detailImage = getView().findViewById(R.id.DetailImage);
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

       ...一部省略
    }

    // REST APIで取得したデータを画面に反映するためのクラス
    private class ReflectDetail implements Runnable {
       ...一部省略

        // Handlerから実行されるメソッド
        @Override
        public void run() {
    
           ...一部省略
    
            // 発売日が取得できていたら反映
            if (!TextUtils.isEmpty(detailData.volumeInfo.publishedDate)) {
                publishDateText.setText(detailData.volumeInfo.publishedDate);
            }
            //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
            // Glideを使ってWeb上の画像をImageViewに表示させる
            if (detailData.volumeInfo.imageLinks != null) {
                Glide.with(DetailFragment.this)
                        .applyDefaultRequestOptions(RequestOptions.fitCenterTransform())
                        .load(detailData.volumeInfo.imageLinks.medium)
                        .into(detailImage);
            }
            //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
        }
    }
}
```
上記コードを実装したら動作確認します。
正常に動作した場合、蔵書詳細画面に蔵書の表紙画像が表示されます。

**Glide**はメソッドからメソッドに繋げる形の実装でも画像のURLをImageViewに表示することができます。（変数を挟まなくていいのでコード量が少なくなります）
`with()`ではContextを取得できるインスタンスを引数にセットします、`load()`の引数には画像のURLをセット、最後に`into()`の引数に画像を表示するImageViewをセットして表示します。
`applyDefaultRequestOptions()`は画像の表示設定を指定できます、今回は画像をはい出さずに表示するように"fitCenterTransform"を指定しています。

以上で、蔵書詳細画面作成の解説は完了です。
