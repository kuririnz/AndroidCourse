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
* Gson

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
        args.putString("term", term);
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
            term = getArguments().getString("term", "Android");
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

<font color="red">**蔵書詳細画面のサンプルをここに表示**</font>

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
蔵書を単一で特定するためには"ISBN"と呼ばれる蔵書ごとに振られたユニークなコードがあり、このコードを元に再度APIからデータを取得するを詳細画面では実装していきます。
蔵書詳細画面で取得したデータのパースに関してはJSONデータをモデルクラスに一発変換してくれるライブラリを使ってJSONObjectでの実装の手間を軽くする方法を使います。

"ISBN"ですがデータ自体は検索結果一覧画面のAPIから戻ってきたデータに含まれており、検索結果一覧画面のパース処理を追加して使用できる状態に実装を修正します。
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

クラス名とパッケージを確認したら`OK`をクリック
{% img /android/09-RefactorFragment/addDetail07.png 550 Include GSON %}
新しいファイルができたら各項目用の変数を作っていきます。
```java DetailDataModal.java
public class DetailDataModel {

    public List<item> items;

    public class item {
        public VolumeInfo volumeInfo;
    }

    public class VolumeInfo {
        public String title;
        public String subTitle;
        public List<String> authors;
        public List<Identifiers> industryIdentifiers;
        public String publishedDate;
        public String description;
        public int pageCount;
        public List<ImageLinks> imageLinks;
    }

    public class Identifiers {
        public String type;
        public String identifier;
    }

    public class ImageLinks {
        public String smallThumbnail;
        public String thumbnail;
    }
}
```

続いてFragment等を修正します。
`ResultListFragment.java`と同様に`import..`の一覧から
*import android.app.Fragment;*を削除します。
{% img /android/09-RefactorFragment/addDetail03.png 600 Import Change %} 
次に<font color="red">赤くなった`Fragmet`</font>をクリックし上に表示されるツールチップに従い
<kbd>Alt</kbd><kbd>+</kbd><kbd>Enter</kbd>(Macは<kbd>option</kbd><kbd>+</kbd><kbd>Enter</kbd>)を入力
{% img /android/09-RefactorFragment/addDetail04.png 600 Import Change %}
ウィンドウに選択肢が表示されたら**Fragment (android.support.v4.app)**を選択します。
{% img /android/09-RefactorFragment/addDetail05.png 600 Import Change %} 

```java DetailFragment.java

```
