---
title: 操作性、ユーザ体験の改善
date: 2017-11-11
tags:
---
操作性、ユーザ体感の改善の中でダイアログやデータベース以外の保存機能を学習します。

<!-- toc -->

[蔵書詳細画面作成](/AndroidCourse/android/09-RefactorFragment)からの引き続きの学習ページです。
# 学習ポイント
* DialogFragment
* SharedPreference
* プロジェクトのパッケージ整理

Androidアプリでデータ通信中にユーザ操作を受けずにユーザにアプリの状況を伝えるためにダイアログ表示の方法を学習します。
また、アプリ内でデータベース以外にデータを保持する方法としてSharedPreferenceの機能を学習します。

# ユーザ体験の改善
前ページまでに実装してきたアプリの中で、データ読み込み中に画面が真っ白になってしまい読み込み時間が長くなるとアプリが止まってしまったと感じてアプリを終了するユーザがいるかもしれません。
そこで、アプリが動いていることを通知するためにダイアログ表示を行うように処理を実装していきます。

# データ読み込み中のダイアログ表示機能
Androidアプリでダイアログを表示するためには`DialogFragment`クラスを継承したカスタムクラスを作成し、作成したカスタムクラスにどういった内容でダイアログ表示を行うか実装して行く。

## カスタムDialogFragment作成
以下の手順から新規javaクラスを作成します。

> New -> Fragment -> Fragment(Blank)

> ***ここに作成するjavaクラス生成時の設定画像を表示***

|項目                               |設定値                                  |
|----------------------------------|----------------------------------------|
|Fragment Name                     |ProgressDialogFragment                  |
|Create Layout XML?                |チェックを<font color="blue">つける</font>|
|Fragment Layout Name              |fragment_progress_dialog                |
|Include fragment factory methods? |チェックを<font color="red">つけない</font>|
|Include interface callbacks?      |チェックを<font color="red">つけない</font>|
|Source Language                   |Java                                    |

ダイアログ表示用のFragmentを作成したらレイアウトXMLを修正していきます。
LayoutEditor左下から**Text**タブをクリックしテキストエディタを表示し、以下の修正通り修正します。
以下レイアウトファイルを表示します。
> app -> res -> layout -> fragment_progress_dialog.xml

ここでテキストエディタモードでレイアウトを編集するとき、画面右端に表示されている***Preview***をクリックすると、リアルタイムにプレビューを表示されるので利用してみてください。


```XML fragment_progress_dialog.xml
<!-- ↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓削除↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓ -->
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="kuririnz.xyz.bookdiscovery.ProgressDialogFragment">

    <!-- TODO: Update blank fragment layout -->
    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:text="@string/hello_blank_fragment" />

</FrameLayout>
<!-- ↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑削除↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑ -->
<!-- ↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓ -->
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <RelativeLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true">
        <ProgressBar
            android:id="@+id/ProgressCircle"
            style="@android:style/Widget.DeviceDefault.Light.ProgressBar.Large"
            android:layout_margin="10dp"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_toRightOf="@+id/ProgressCircle"
            android:layout_centerVertical="true"
            android:layout_margin="10dp"
            android:text="書籍情報を検索中..."
            android:textSize="15dp"
            android:textColor="@color/colorAccent"
            android:textAlignment="center"/>

    </RelativeLayout>
</RelativeLayout>
<!-- ↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑ -->
```

`ProgressDialogFragment`が作成できたらダイアログ表示用のFragmentにするため、親クラスを**DialogFramgnet**に変更します。
この**DialogFragment**への変更で気をつけて欲しいのがインポートされるパッケージです。
以下のパッケージが追加されていることを確認してください。
> import android.support.v4.app.DialogFragment;

```java ProgressDialogFragment.java
//↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
public class ProgressDialogFragment extends DialogFragment {
//↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    ...一部省略

        @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {

        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // ソフトバックボタンを無効化
        getDialog().setCancelable(false);
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.fragment_progress_dialog, container, false);
    }
}
```
```java ResultListFragment.java
public class ResultListFragment extends Fragment implements AdapterView.OnItemClickListener{

	...一部省略
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
        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // プログレスFragmentをインスタンス化
        ProgressDialogFragment progressDialog = new ProgressDialogFragment();
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

        ...一部省略
    }

    ...一部省略

    // 検索結果をListViewに反映するメインスレッドの処理クラス
    private class ReflectResult implements Runnable {
    	...一部省略
        // Handlerから実行されるメソッド
        @Override
        public void run() {
	        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
            // プログレスFragmentを終了させるためにマネージャークラスを取得
            FragmentTransaction ft = getChildFragmentManager().beginTransaction();
            // FragmentManagerに登録されたFragmentからダイアログフラグメントを抽出
            ProgressDialogFragment progressDialog = (ProgressDialogFragment) getChildFragmentManager().findFragmentByTag("Dialog");
            // DialogFragmentを取得できた場合
            if (progressDialog != null) {
                // ダイアログを非表示にする
                progressDialog.dismiss();
                // FragmentManagerの管理から除外
                ft.remove(progressDialog);
            }
            // FragmentManagerへの変更を反映(確定)
            ft.commit();
            //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

            // ListViewに表示する情報をまとめるAdapterをインスタンス化
            adapter = new ResultListAdapter(getContext(), resultList);
            // ListViewに表示情報をまとめたAdapterをセット
            resultListView.setAdapter(adapter);
            // ListViewに行をクリックした時のイベントを登録
            resultListView.setOnItemClickListener(ResultListFragment.this);
        }
	}
```
上記実装が終わったら動作確認してください。
検索結果画面へ画面遷移した際に半透過に一部白いウィンドウのダイアログが表示されると思います。

### DialogとAlertDialog
`DialogFragment`クラスはDialogクラスのオブジェクトを保持しており、Dialogオブジェクトに生成したレイアウトを表示させます。
またDialogには利用方法によって利用するクラスが２種類ありますそれが**Dialog**と**AlertDialog**です。
Dialogは開発者が指定したレイアウトを表示するだけで、ユーザへの情報を表示するために利用することが多いと思われます。

AlertDialogは開発者が指定したレイアウトの他に"OK","Cancel"などのボタンがメインレイアウトとは別に準備されており、ユーザに応答を求めるときや確認のダイアログとして利用することが多いです。

## 読み込みキャンセル機能
先ほどの実装では検索結果が表示されるまでアプリユーザは何もすることができませんでしたが、読み込み中でも検索画面に戻れるように`AlertDialog`を使用してキャンセルボタンをダイアログに追加します。
```java ProgressDialogFragment.java
public class ProgressDialogFragment extends DialogFragment {
    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {

        getDialog().setCancelable(false);
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.fragment_progress_dialog, container, false);
    }
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

    //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    @NonNull
    @Override
    public Dialog onCreateDialog(Bundle savedInstanceState) {
        // ダイアログ表示するレイアウトを生成
        View view = getActivity().getLayoutInflater().inflate(R.layout.fragment_progress_dialog, null, false);

        // アラートダイアログビルダーを使ってボタン付きのダイアログを生成
        AlertDialog.Builder builder = new AlertDialog.Builder(getActivity())
                .setView(view)
                .setPositiveButton("キャンセル", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialogInterface, int i) {
                        getActivity().finish();
                    }
                })
                .setCancelable(false);
        // 表示するダイアログを生成して返却
        return builder.create();
    }
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
}
```
```java ResultListFragment.java
    // 検索結果をListViewに反映するメインスレッドの処理クラス
    private class ReflectResult implements Runnable {

		...一部省略
        // Handlerから実行されるメソッド
        @Override
        public void run() {
        	//↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
            // Activityが終了していたら処理をしない
            if (getActivity() == null || getActivity().isFinishing() || !getActivity().hasWindowFocus()) {
                return;
            }
            //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
            // プログレスFragmentを終了させるためにマネージャークラスを取得
            FragmentTransaction ft = getChildFragmentManager().beginTransaction();
            // FragmentManagerに登録されたFragmentからダイアログフラグメントを抽出
            ProgressDialogFragment progressDialog = (ProgressDialogFragment) getChildFragmentManager().findFragmentByTag("Dialog");
            // DialogFragmentを取得できた場合
            if (progressDialog != null) {
                // ダイアログを非表示にする
                progressDialog.dismiss();
                // FragmentManagerの管理から除外
                ft.remove(progressDialog);
            }
            // FragmentManagerへの変更を反映(確定)
            ft.commit();
			...一部省略
        }
    }
```
上記コードが実装できたら動作確認してください。
読み込み中のダイアログにキャンセルボタンが追加され、キャンセルボタンを押下すると検索画面に戻れるようになっているはずです。

### DialogFragmentのライフサイクル
`DialogFragment`を使用してダイアログ表示を行う場合に、ライフサイクルが通常のFragmentクラスと少し違うため、注意が必要です。

getLayoutInflater -> onCreateDialog -> onCreateView -> onActivityCreated

# 最後に検索した文字列を保持する機能
以前にアプリが終了されてもデータが残る機能としてデータベース(Realm)を紹介しましたが、
データの量が少ない場合などに手軽に使える機能をAndroidは備えており、容易に使えます。
今回は`SharedPreference`という機能を使い検索した文言を保持する機能を実装していきます。
```java MainActivity.java
public class MainActivity extends AppCompatActivity {

	//↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    // 最後の検索文言を保持/取得するキー定数
    private final static String PREF_KEY = "LAST_TERM";
    //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

    // レイアウトxmlと関連付けるWidget
    private Button bookSearchBtn;
    private Button historyBtn;
	...一部省略

	       View.OnClickListener bookSearchEvent = new View.OnClickListener() {
            @Override
            public void onClick(View view) {
            	...一部省略
                } finally {
                    // Realmインスタンスがちゃんとクローズされること
                    realm.close();
                }

                //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
                // SharedPreferenceに保存
                SharedPreferences.Editor editor = MainActivity.this.getPreferences(Context.MODE_PRIVATE).edit();
                editor.putString(PREF_KEY, termString).apply();
                //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

                // 検索結果画面へ遷移するためのIntentをインスタンス化
                Intent intent = new Intent(MainActivity.this, ResultListActivity.class);
                // EditTextに入力された文字列を"KeyValuePair"でResultListActivityに渡す
                intent.putExtra("terms", termString);
                // 画面遷移アクションを実行
                startActivity(intent);
            }
        };
                // 蔵書検索ボタンが押された時に実行するプログラムをボタンに登録
        bookSearchBtn.setOnClickListener(bookSearchEvent);

        // 検索履歴ボタンをクリックした時の処理を実装
        historyBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                // 検索履歴画面へ遷移するためのIntentをインスタンス化
                Intent intent = new Intent(MainActivity.this, HistoryActivity.class);
                // 画面遷移アクションを実行
                startActivity(intent);
            }
        });

        //↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓追加↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        // SharedPreferenceに最後の検索条件が残っていたら登録する
        // SharedPreferenceにデータがない場合は空文字を設定する
        String lastTerm = this.getPreferences(Context.MODE_PRIVATE).getString(PREF_KEY, "");
        bookSearchEditor.setText(lastTerm);
        //↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑追加↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑

		...一部省略
}
```
上記コードを実装したら動作確認してください。
初回起動時のEditTextは空文字(何も表示されてない)となっており、何かしらの文言で検索を行いアプリを終了させます。
改めてアプリを起動したときにEditTextに検索した文言が表示されていることが確認できます。

## SharedPreference
Android端末のアプリ専用領域に環境設定としてKey-Value形式でデータを保存/参照できる機能。
SharedPreferenceのインスタンス化の引数ではインスタンス化するSharedPreferenceのアクセス許容範囲を指定できます。
今回は`Context.MODE_PRIVATE`ということで自アプリからしかアクセスできないSharedPreferenceを作成し、その中に"LAST_TERM"という鍵[Key]情報に[最後に検索した文字列]というデータ(Value)を保存しました。
JSONの概念が理解できている方は`{"LAST_TERM":$term}`と保存されている認識で良いと思います。

またSharedPreferenceは名前をつけて複数作成することができ、**JSONファイルを作成してJSON形式でデータを保持している**と思っていただいても良いと思います。

簡単に利用できるSharedPreferenceですが、起動時にメモリ上に読み込む仕様になっているため、大量のSharedPreferenceやデータの保持はアプリの挙動に影響を与える可能性があるので少量で使うようにすることをお勧めします。

# Androidプロジェクトの整理
ここに至るまでに複数のActivityやFragmentを作成し、類似するネーミングのものも存在するため、ツリー表示が見にくく感じます。
そこで作成したファイルごとにパッケージを分けて保存することで探しやすさと視認性をよくしていきます。