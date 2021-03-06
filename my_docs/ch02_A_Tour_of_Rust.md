# 2章 Rustツアー
## 2.4 コマンドライン引数の処理
```
use std::str::FromStr;
```
- 標準ライブラリのトレイトFromStrをスコープに取り込む
- **トレイト**とは、型が実装できるメソッドの集合
- FromStrトレイトを実装した型は、from_strメソッドを持つ
- from_strメソッド…文字列を解析してその型の値に変換する
- 型u64はFromStrを実装しているのでu64::from_strメソッドを呼び文字列をu64型に変換することができる
  - コマンドライン引数を変換するのに使用する
- FromStrというトレイトは、コード中にでてこないが、トレイとのメソッドを使用するにはスコープに入れなければならない

```
use std::env;
```
- std::envモジュール：実行環境の情報を取得するための関数や型が定義されている
- コマンドライン引数を取得するためにargs関数を利用する

```
let mut numbers = Vec::new();
```
- 可変なローカル変数numbersを宣言
- 空のベクタで初期化
- Vecはサイズ可変のベクタ型
  - C++のstd::vector、Pythonのリスト、あるいはJavaScriptの配列に相当する
  - ベクタは動的に大きくなったり小さくなったりするように設計されている
  - ベクタの最後に追加できるようにするには、束縛する変数に**mut**を付けなければならない
  - numbersはVec<u64>であるが宣言する際にu64は指定する必要はない => u64をpushしているので、コンパイラはベクタの型を推論できる

```
for arg in env::args().skip(1){
```
- forループを用いてコマンドライン引数を処理している
- 変数argに順番に1つづつ引数をセットしてボディ部を評価する
- std::envモジュールのargs関数は**イテレータ(iterator)**を返す
- このイテレータは、必要に応じて引数を１つずつ生成し、引数がなくなったらそれを教えてくれる
- イテレータ
  - ベクタの要素を生成するイテレータ、ファイルの各行を生成するイテレータ、通信チャネルからのメッセージを生成するイテレータなど、ループの対象となり得るすべてのものがイテレータとなっている
  - イテレータはforループ以外でも使うことができる
- skip(1)：args関数で得られるイテレータが最初に生成する値は、そのプログラムの名前なのでそれをスキップする

```
numbers.push(u64::from_str(&arg)
  .expect("error parsing argument"));
```
- u64::form_strを呼び出してargに格納されたコマンドライン引数を符号なし64ビット整数にパースする。
- u64::form_strはu64の方に関連付けられた関数（型関連関数）
  - C++あるいはJavaのstaticメソッドのようなもの
- u64::form_strの戻り値はResult型
  - Result型は列挙型で次の2値を持つ
    - Ok(v)：パースが成功したことを示し、Vは生成した値
    - Err(e)：パースが失敗したことを示し、eはその理由を説明するエラー値
  - 失敗する可能性があることを行う関数はすべてResult型を返す。
- Rustは例外機構を持たない
  - すべてのエラーはRusultかパニックで処理される
- パースが成功したかをチェックするには、Resultのexpectメソッドを用いる
  - 結果がErr(e)であれば、expectはeの説明をするメッセージを出力し、プログラムの実行を中断する
  - 結果がOk(v)であれば、expectは値vを返す。
    - ここではnumbers.pushでベクタの最後に追加している

```
if numbers.len() == 0 {
    eprintln!("Usage: gcd NUMBER ...");
    std::process::exit(1);
}
```
- コマンドライン引数が入力されなかった場合は、標準エラー出力にエラーメッセージを出力し、プログラムを中断している


```
let mut d = numbers[0];
for m in &numbers[1..]{
    d = gcd(d, *m);
}
```
- &numbers[1..]:ベクタの2番目以降の要素への**参照(reference)**を**借用**している
- *mの*は、mを参照解決（dereference）する演算子。参照されている値を返す
- ベクタはnumbersに所有されているので、main関数の最後でnumbersがスクープから外れると、ベクタは自動的に開放される

- Rustではmainが何も返さなければプログラムが成功したことになる
- expectやstd::process::exitなどの関数を明示的に呼び出した場合のみ、プログラムはエラーを表す0以外のステータスコードを返す

## 2.5 Webページを公開する
### Cargo.tomlに追加
```
[dependencies]
actix-web = " 1 . 0 . 8 "
serde = { version = " 1 . 0 ", features = ["derive"] }
```
- Webフレームワークであるactix-webとシリアライズを行うserdeを導入する
- actix-webとは
  - actixとは、Rust製のActorフレームワーク（Java/Scalaでいうと、Akkaがメジャー）
  - actixをベースとしてWeb開発用機能を追加したのが、軽量・高速なWeb開発フレームワークである**actix-web**
  - actix-webで開発されたアプリは、実行ファイルにHTTPサーバーを含んでいるため、そのまま使うこともできるし、apacheやnginxの後ろに置くこともできる

### main.rs

```
use actix_web::{web, App, HttpResponse, HttpServer};
```
- actix_wweb::{...}のように書くと波括弧の中に書いた名前を、コードの中で直接使えるようになる
次のと同様。
```
use actix_web::web;
use actix_web::App;
use actix_web::HttpResponse;
use actix_web::HttpServer;
```

- main関数のやっていること
  - HttpServer::newを呼び出して、パス"/"へのリクエストに対応するサーバーを作り、そこに接続するように促すメッセージを出力し、ローカルマシンのTCPポート3000番をリッスンする

- HttpServer::newの引数として渡しているものは、Rustのクロージャ式
  - || {App::new()...}という形をしている
  - クロージャが引数を取る場合は||の間に記述する
  -{...}の部分がクロージャのボディ部
  
以降下記のGoogleスライド
https://docs.google.com/presentation/d/14G7SZe_OYps0XXF68NumTnJ8tPF_v_29at1PJxesxU8/edit#slide=id.p