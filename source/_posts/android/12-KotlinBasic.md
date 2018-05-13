---
title: Kotlin プログラミング基礎
date: 2018-03-30 17:06:33
tags:
---
Kotlinを使ったプログラミングとして基礎を学習します。

<!-- toc -->

# 学習ポイント
* Kotlin言語の特徴
* Kotlin - Javaの差分

Google I/O 2017にて正式にAndroid開発言語として公式サポートとなった言語Kotlinについて学習します。

# Kotlin言語の特徴
Javaをベースに開発された言語であるため、Javaと同じくJVM上で動作する言語、そのため様々なプラットフォームでの実行が可能。
開発元のJetBrains社もJavaと完全な相互運用可能を提言しています。

完全互換性言語ということでAndroidアプリの開発においてJavaで作成したプロジェクトに対して１ファイルだけKotlinという導入も可能である。
またJavaを使い慣れていれば抵抗感が少なくKotlinのプログラムが行えるような設計がなされています。
Kotlinはjava言語ではないのでないのでファイル拡張子も変わるのですが、***[ファイル名].kt***の形式でファイルが作成されます。
当ページでは以下の項目に関して解説をしていきます。

* 変数
* 関数
* 制御構文の変更点、let
* クラスの定義
* Unit, Nothing, Any
* object, companion object
* ラムダ式
* try-catch

Kotlinは公式サイトより導入せずとも試すことができます。
[Kotlin Playground](https://try.kotlinlang.org/)

## 変数
Kotlinは最近の言語で増えてきた推論型変数、オプショナル（Null-safety）などの概念が追加されています。

先ずは変数ですが宣言の方法が以下のキーワードを使います。

* var -> 変更可能な変数
* val -> １度だけ代入できる変数(ローカル変数としての利用を想定)

**推論型変数**
変数の宣言のみを記述する場合には明示的に型を指定する必要がありますが、
宣言と同時に代入を行う場合、代入されたデータの型として推論されそれ以降では代入したデータの方として扱われる変数となりす。
そのため他の型やクラス以外のデータは代入できなくなります。

また、Kotlinでは原則宣言時に初期化が必須なのですが、宣言時に値を入れない方法もあり、varキーワードの前に`lateinit`と記述することで宣言のみを記述できます。
```java Java
	//変数宣言のみ
	String hoge;
	// 変数宣言と同時に初期化
	String fuga = "Android";
```
```kotlin Kotlin
	//変数宣言のみ
	lateinit var hoge: String
	// 変数宣言と同時に初期化
	// String型の変数として扱われる
	var fuga = "Android"
```
そして変数の宣言時には合わせてそれ以降のプログラムで宣言した変数にNullの代入可否を設定する必要があり、その設定を***Null-Safety(オプショナル)***と呼びます
```kotlin Kotlin
	// Nullを許容しない変数
	var hoge = "hogehoge"
	hoge = null			// コンパイルエラー
	// Nullを許容する変数
	var fuga: String? = "fugafuga"
	fuga = null			// 代入可
```
オプショナルを設定することで実行時エラー（NullPointerException）などの発生をエラーを予測して実装と動作確認することができます。
さらにNull許容型の変数を作成した場合でもNullのチェックは簡潔に行えるよう機能が追加されており、その機能が`let`キーワードになります。
```java Java
	String hoge = fuga != null ? fuga : "hogehoge";
```
```kotlin Kotlin
	val hoge = fuga?.let { fuga } else { "hogehoge" }
```
上記Kotlinの例では`fuga`がnullではない場合はfugaを代入、nullの場合は"hogehoge"をhogeに代入する式担っています。
上記では`let-else`式ですが、`let`単体の式の場合で変数の結果がnullの場合にはnullが代入される仕様になっています。

また、Kotlinではキャストの方法も変更されて変更されており、キャストには`as`キーワードを使用します。
```java Java
	long numLong = 10;
	// Long型をInt型にキャスト
	int num = (int) numLong;
```
```kotlin Kotlin
	val numLong: Long = 10
	// Long型をInt型にキャスト
	var num: Int = num as Int
```

## 関数
関数の定義方法に変更が加わりましたが、参照方法に変更はありません。

以下はInt型引数が１つで戻り値型がInt型のメソッドの定義方法になります。
```java Java
// メソッドの定義
// [アクセス修飾子] [戻り値型] [メソッド名]([引数型] [引数名], [引数型] [引数名]...) {
//     メソッドの処理
// }
	public int calc(int x) {
	   return x + 10;
	}
	// メソッドの参照
	calc(20);
}
```
```kotlin Kotlin
// メソッドの定義
// [アクセス修飾子] fun [メソッド名]([引数名]: [引数型], [引数名]: [引数型]): [戻り値型]...) {
//     メソッドの処理
// }
	public fun calc(x: Int): Int {
		return x + 10
	}
	// メソッドの参照
	calc(20)
}
```
## 制御構文
Kotlinでもif文、for文、while、do-whileなどはある程度javaと同じように利用できると思います。
Kotlinではswitchの代わりになるもので**when**という制御構文が追加されています。
また、Kotlinには三項演算子(条件演算子)はなくなり、if文を短縮して記述することですることで三項演算子の様に利用することができます。

### if文
java言語と基本的な利用方法は同じです、Kotlinでの特殊な利用方法として代入式としてif文を利用することができます。

代入式としてif文を利用する場合は最後に記述した変数や式が右辺の変数に代入されます。
Kotlinではif文が代入式として使用できるので三項演算子(条件演算子)は使えなくなっています。
```java Java
	// 通常のif文の記述方法
	if (a > 1 && a < 31) {
	    // 実行したい処理を記述
	}
	// if文を利用した変数への代入
	int num  = a < i ? 0 : 10 ;
```
```kotlin Kotlin
	// 通常のif文の記述方法
	if (a > 1 && a < 31) {
	    // 実行したい処理を記述
	}
	// if文を利用した変数への代入
	var num: Int if(a < i) { 0 } else { 10 }
```

### when文
Javaや他の言語であるswitchの置き換え構文になります。
case文の代わりには`->（アロー演算子）`を使用します、各条件の時に１行で済むコードであればアロー演算子の後に半角スペースを挟んで記述できます。
複数行であればアロー演算子の後に`{}`をつけてブロック内にコードを記述します。
さらに複数の条件を指定したい場合は、条件を`,（カンマ）`で区切り条件を設定します。
```java Java
	switch(num) {
		case 0:
		    // numが0の場合の処理
		break;
		case 1:
		    // numが1の場合の処理
		break;
		case 2:
		case 3:
		    // numが2,3の場合の処理		
		default:
		    // 全てのcase条件に該当しなかった場合の処理
	}
```
```kotlin Kotlin
	when(num) {
		0 ->    // numが0の場合の処理
		1 -> {
		    // numが1の場合の処理
		    // 処理が複数行の場合は{} ブロックをブロックを記述
		}
		2,3 ->  // numが2,3の場合の処理
		else -> // 上記全ての条件に該当しなかった場合
	}
```
when構文では分岐条件の設定に`in`で条件範囲を設けたり`is`で型のチェックを条件に設定することも可能です。
`!in`や`!is`と使うことで範囲外、型ではないなどの判定条件としても設定できます。
```kotlin Kotlin
	when(num) {
		in 1..10 ->    // numが1~10の範囲内だった時の処理
		is Int ->      // numがInt型だった場合の処理
	}
```
when構文もif文と同様に代入式として使用することも可能です、さらにwhen構文に引数を与えなかった場合、if-else文代替えとして認識され分岐条件の設定は論理型の判定になります。
```kotlin Kotlin
	// hogeにwhen文で判定した結果が代入される
	val hoge = when {
		// 条件設定はtrue/falseの結果になるよう記述が必要
		fuga == 0 -> 10
		fuga == 10 -> 20
		else -> 30
	}
```

### for文
Kotlinでのfor文はJavaに置ける拡張for(foreach)文の形式でしか使用することができません。
拡張for文形式で指定回数の繰り返しを行う例も合わせ紹介します。
```java Java
	// 0~10を2ずつ加算するfor文
	for(int i=0; i<10; i+=2) {
	    // 繰り返し時の処理
	}
	// 拡張for文
	for(int num: numbers) {
	    // 繰り返し時の処理
	}
```
```kotlin Kotlin
	// 0~10を2ずつ加算するfor文
	for (i in 1..10 step 2) {
	    // 繰り返し時の処理		
	}
	// イテレータによるfor文
	for(num : numbers) {
	    // 繰り返し時の処理		
	}
```
また`downTo`キーワードを使うことで降順的にfor文を処理することも可能です。
```kotlin Kotlin
	// 20 ~ 0までの範囲でfor文を実行
	for(i in 20 downTo 0) {
		// 繰り返し時の処理
	}

	// 20 ~ 0までの範囲で2ずつ下げるfor文を実行
	for(i in 20 downTo 0 step 2) {
		// 繰り返し時の処理
	}
```
もしインデックス付きのfor文を処理したい場合は`indices`メンバや`withIndex()`メソッドを使用するとインデックスを使いながらfor文を実行することもできます。
```kotlin Kotlin
	// インデックスを使ったfor文
	for(i in array.indices) {
	    println(array[i])
	}
	// インデックスと値を使ったfor文
	for((index, value) in array.withIndex()) {
	    println("インデックス：${index}, \t値：${value}")
	}
```

### while文
while, do-while文に関してはJavaや他の言語と同様の記述ルールになります。

## クラス
クラスに関してもJavaから一部変更が入っていますが、概ね同じルールで記述していきます。
class定義の際にpublic修飾は不要となり、コンストラクタの記述方法が変更されました。

### クラス定義
以下で紹介するのは引数が２つあり、コンストラクタが一つ（プライマリコンストラクタ）の実装例になります。
コンストラクタに引数がない場合はJava同様記述する必要はなく、デフォルトコンストラクタが有効になります。
``` java Java
// 引数が２つありメンバを初期化しているクラス
public class hoge {
	int num;
	String str;
	hoge(int num, String str) {
		this.num = num;
		this.str = str;
	}
}
// デフォルトコンストラクタを有効にしているクラス
public class fuga {
	int num;
}
```
```kotlin Kotlin
// 引数が２つありメンバを初期化しているクラス
class hoge(Int num, String str) {
	var num = num
	var str = str
}
// デフォルトコンストラクタを有効にしているクラス
class fuga {
	int num
}
```
プライマリコンストラクタに初期化時の処理をつける場合は以下の通り実装する
```kotlin Kotlin
class hoge(Int num, String str) {
	var num = num
	var str = str
	init {
		// プライマリコンストラクタの
		// 初期化処理
	}
}
```
セカンダリコンストラクタを定義する場合は`constructor`キーワードを使用して定義します。
```kotlin Kotlin
class hoge(num: Int, str: String) {
	var num = num
	var str = str
	// セカンダリコンストラクタの定義
	// プライマリコンストラクタへ移譲する定義が必要
	constructor(num: Int): this(num, "hoge") {
		// セカンダリコンストラクタの初期化処理
	}
}
```
インスタンスの作成はJavaと違い*new*キーワードなしでコンストラクタを呼び出します。
```java Java
	hoge inc = new hoge(0, "str");
```
```kotlin Kotlin
	val inc = hoge(0, "str");
```
### 継承、インターフェース
継承や、インターフェースの実装に使用していた、**extends**や**implements**キーワードもなくなり,
プライマリコンストラクタの後ろに`: [親Class名]([親Classプライマリコンストラクタの実引数])`の形式で継承関係を記述し、
`,（カンマ）`区切りで実装するインターフェースを記述します。
Kotlinでは継承関係を記述しなかった場合、暗黙的に`Any`クラスの子クラスとなります。
`Any`クラスと`java.util.Object`とは別のクラスになりますので注意してください。
またKotlinでは継承元となる親クラスの定義に`open`キーワードの修飾が必要になるので注意してください。
メソッドを`orverride`させるにはメソッドの定義にも先頭に`open`を記述してください。
```java Java
// SuperHogeクラスを継承し、Fugaインターフェースを実装するHogeクラスの定義
// SuperHogeクラスのプライマリコンストラクタは引数なしです。
public class Hoge extends SuperHoge inmplements Fuga {
	// SuperHogeクラスのfooメソッドをオーバーライド
	@orverride public void foo() {}
}
// 継承元の親クラスを定義
public classSuperHoge{
	// orvrerideできるメソッドの定義
	public void foo() {}
}
```
```kotlin Kotlin
// SuperHogeクラスを継承し、Fugaインターフェースを実装するHogeクラスの定義
// SuperHogeクラスのプライマリコンストラクタは引数なしです。
class Hoge: SupreHoge(), Fuga {
	// SuperHogeクラスのfooメソッドをオーバーライド
	orverride fun foo() {}
}
// 継承元の親クラスを定義
open class SuperHoge() {
	// orvrerideできるメソッドの定義
	open fun foo() {}
}
```
**抽象クラス**に関してはJava同様`abstract`キーワードを使い抽象クラスとして定義することができます、また抽象クラスは`open`キーワードを付与しなくてもオーバーライドすることは可能です。

### 静的メンバへのアクセス
KotlinはJavaと違い**static**な変数や定数、メソッドを定義することができません。
その場合の方法としてKotlinではクラスブロック外に変数、定数、メソッドを定義することができるので**static**メソッドの代わりに使用することができます。
また、クラスブロック内に`companion object`（コンパニオンオブジェクト）キーワードを使ったブロックを作りブロック内に変数、定数、メソッドを定義する方法もあります。
前者の場合はクラス名の修飾なしに各定義を参照することができます、後者はクラス名の修飾を使って各定義を参照できます。

クラス外に定義された内容はプロジェクト単位でユニークな扱いを受けるので、他のクラスで同名の定義を行うとコンパイルエラーが発生します。
そのため、Fragmentなどのように静的コンストラクタを定義したい場合などは`companion object`に定義をし、クラス外には画面遷移時のパラメータキーやプロジェクト内でユニークな定数に分けるなどすると使い易くなります。
```kotlin Kotlin
package com.sample
// クラス外の定数定義
const val FOO = 10
// クラス外のメソッド定義
fun bar (a: Int): Int {
	return a * 2
}

class Hoge {
	// コンパニオンオブジェクトを使ったメソッドの定義
	companion object {
		fun fuga (b: Int): Int  {
			return b *3
		}
	}
}

// Hogeクラス(Hoge.kt)の要素を参照するクラス
class Hogehoge {

	fun runMethod() {
		// Hoge.ktのbarメソッドを参照
		val res1 = bar(FOO)
		// Hoge.ktのfugaメソッドを参照
		val res2 = Hoge.fuga(FOO)
	}
}
```
`const`キーワードを付けて記述した場合はJavaにおいて**static**な要素として扱われるのに対し、`const`なしで記述した物はJavaに変換された時にアクセサメソッドが定義されてしまうので、できるだけ`const`を付けて記述した方が良いです。

## object
`object`にはいつかの用途があり、一つが**クラス**内で紹介した`companion object`です。
その他では以下の使い方で使用されることがあります、それぞれの使用方法について紹介していきます。

* 無名オブジェクトの定義
* シングルトンの定義

### 無名オブジェクトの定義
Android Javaにおいてはボタンなどのクリックイベントなど、引数にインスタンス化したオブジェクトを返す箇所などが無名クラスでの置き換えになります。

以下例は画面のボタンコンポーネントに無名オブジェクトを使いクリックイベントをセットしています。
```java Java
public class HogeActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.hogehoge);
        Button btn = findViewById(R.id.button);
        // インターフェースをオブジェクト化してクリックイベントをセット
        btn.setOnClickListener(new View.OnClickListener() { ... });
    }
}
```
```kotlin Kotlin
class HogeActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val btn = findViewById(R.id.button)
        // 無名オブジェクトを使ってクリックイベントをセット
        btn.setOnClickListener(object : View.OnClickListener { ... }) 
    }
```

### シングルトンの定義
APIやデータベースなど都度インスタンス化して使うのが手間になるクラスも存在します、その場合にプロジェクト内でクラスのインスタンスが一つしか生成されない様に設計されたクラスをシングルトンと呼びます。
Javaでのシングルトンの使い方はメソッドからメソッドへ繋ぐメソッドチェーンで使用するのが一般的でしたが
Kotlinでは`objectの宣言`によってJavaのクラスメソッドのようにシングルトンを使用できるようになりました。

以下の例はシングルトンクラス'Hoge'を作成し、Fugaクラスにて利用している例になります。
```java Java
// シングルトンクラスHogeの定義
public class Hoge {
    private static Hoge instance = new Hoge();
	private Hoge() {}
    public static Hoge getInstance() {
    	if (instance == null) {
    		instance = new Hoge();
    	} 
        return instance;
    }
    public double calcPlus10(double dec) {
    	return dec + 10;
    }
}
// Hogeクラスを使用するFugaクラス
public class Fuga {
	int a;
	public Fuga() {
	    // シングルトンHogeクラスを参照
	    a = Hoge.getInstance().calcPlus10(10);
	}
}
```
```kotlin Kotlin
// シングルトンクラスHogeの定義
object Hoge {
	fun calcPlus10(dec: Double): Double {
		return dec + 10
	}
}
// Hogeクラスを使用するFugaクラス
class Fuga {
	var a: int = 0;
	init {
	    // シングルトンHogeクラスを参照
	    a = Hoge.calcPlus10(10)
	}
}
```
## ラムダ式
ラムダ式は**メソッドの定義を式として扱い、即時実行された結果を機能**を指します。
関数型リテラルとも呼ばれることがあり、関数が宣言されたのではなく式として宣言した変数に代入された形となります。
実際に使用してみると、*'変数名(引数)'*の形式で参照するため、見栄えとして後から関数を作成したように感じます。

以下の例は二つの値の和を求めるラムダ式になります。
```kotlin Kotlin
	// 無名関数の定義
	val sum = {x: Int, y: Int -> x + y}
	// 無名関数の使用例
	val total = sum(3, 4)
```
またラムダ式を使うことでリストなどのコレクション型のデータから特定の条件に該当するデータだけを抽出する時によく利用します。
ラムダ式を使う場合に引数の値が１つの場合、暗黙的に一時変数名`it`として設定されていることに注意してください。

以下ではコレクションとの組み合わせによる例を紹介します。
```kotlin Kotlin
	val tmpList: listOf(1,2,3)
	// コレクションのメソッドに対してラムダを適用した例
	tmpList.count {it % 2 ==1} // List型で奇数の1,3が含まれたオブジェクトを取得できる
	tmpList.filter {it == 2} // List型で2が含まれたオブジェクトを取得できる
```
## Any, Unit, Nothing
Javaには存在しなかったキーワードが出てきたのでいくつか紹介します。
### Any
JavaではルートクラスがObjectクラスでしたが、KotlinではルートクラスがObjectではなく`Any`になります、KotlinにおいてもJavaのObjectクラスは存在しますが、Anyのサブ(子)クラスとして存在します。
Kotlinにはプリミティブ型は存在せず、全てがクラスとして扱われるようになります。
そのため、Javaでのプリミティブ型をBoxing(ボクシング)する必要がなく、コンパイラが自動的にプリミティブ型に最適化されるよう設計されています。
### Unit
またJavaでメソッドなど戻り値がない場合に使うキーワードとして**void**がありますが`Unit`という**void**と同じく意味のない値を返すことを示すキーワードが追加されています、
Kotlinでのメソッドの戻り値がない場合は何も型の指定を行いませんが、暗黙的に`Unit`が戻り値として設定される仕様になっています。
### Nothing
`Nothing`は値が返されないことを示すキーワードになっており、メソッドの引数として設定すると、必ずExceptionをthrowするメソッドになります。
場面としてはあまり使われない記号になります。
## 検査例外
KotlinにはJavaと違い検査例外がないため、Javaの場合にコンパイルエラーが発生する箇所でもKotlinではコンパイルエラーが発生しません。
そのため、Kotlinではtry-catch文を一度も書かずとも実装を完結することができます。
ただコンパイル的な問題がないだけでプロジェクトとしての問題は別になるので、必要な箇所にはtry-catch文を記述するようにした方が良いでしょう。

以上で、Kotlin言語の使い方であったり、Java言語の比較の内容を紹介してきました。
このページの情報があればJava→Kotlinへの移行を行うときの不具合などの対応はできると思われます。