# Rust 入門教材
## 〜他言語経験者のためのRust〜

---

## はじめに

この教材は「他の言語は書けるけど、Rustははじめて」という人向けです。  
PythonやJavaScript、Javaなどの経験がある前提で、**Rustが他の言語と何が違うのか**を軸に説明します。

Rustの最大の特徴は一言でいうと：

> **「コンパイル時に、メモリ安全性をプログラマーに代わって保証する言語」**

GCなし・ランタイムなし・でもメモリ安全。これがRustの売りです。

---

## 目次

1. [環境構築](#1-環境構築)
2. [Hello, World!](#2-hello-world)
3. [変数と型](#3-変数と型)
4. [所有権（Ownership）— Rust最大の概念](#4-所有権ownership)
5. [借用と参照（Borrowing & References）](#5-借用と参照)
6. [スライス](#6-スライス)
7. [構造体（Struct）](#7-構造体struct)
8. [列挙型（Enum）とパターンマッチ](#8-列挙型enumとパターンマッチ)
9. [エラー処理](#9-エラー処理)
10. [トレイト（Trait）](#10-トレイトtrait)
11. [ジェネリクス](#11-ジェネリクス)
12. [クロージャとイテレータ](#12-クロージャとイテレータ)
13. [モジュールとクレート](#13-モジュールとクレート)
14. [練習課題](#14-練習課題)

---

## 1. 環境構築

### インストール

```bash
# rustup（Rustのバージョン管理ツール）をインストール
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# インストール確認
rustc --version
cargo --version
```

### Cargoとは？

`cargo` はRustのビルドツール兼パッケージマネージャーです。  
npmやpipに相当するものだと思えばOKです。

```bash
# 新しいプロジェクトを作成
cargo new hello_rust
cd hello_rust

# ビルドして実行
cargo run

# テストを実行
cargo test
```

---

## 2. Hello, World!

```rust
fn main() {
    println!("Hello, World!");
}
```

### ポイント
- `fn` で関数を定義（functionの略）
- `main` がエントリーポイント
- `println!` は**マクロ**（関数ではない。末尾の `!` がマクロの印）
- 文末はセミコロン `;`

---

## 3. 変数と型

### 変数の宣言

```rust
fn main() {
    // デフォルトで immutable（変更不可）
    let x = 5;
    // x = 6; // ← コンパイルエラー！

    // mutableにしたい場合は mut をつける
    let mut y = 5;
    y = 6; // OK

    println!("x = {}, y = {}", x, y);
}
```

> 💡 **他言語との違い**  
> JavaScriptの `const` / `let` に似ていますが、Rustは**デフォルトが immutable**です。  
> 「変えるつもりがない変数は変えられない」が基本姿勢。これがバグを防ぎます。

### シャドーイング

```rust
fn main() {
    let x = 5;
    let x = x + 1; // 同名で再宣言できる（シャドーイング）
    let x = x * 2;
    println!("{}", x); // 12
}
```

### 基本的な型

```rust
fn main() {
    let i: i32 = -10;       // 32ビット整数（符号あり）
    let u: u64 = 100;       // 64ビット整数（符号なし）
    let f: f64 = 3.14;      // 64ビット浮動小数点
    let b: bool = true;     // ブール
    let c: char = 'あ';     // Unicodeスカラー値（シングルクォート）
    let s: &str = "hello";  // 文字列スライス
    let string = String::from("hello"); // ヒープ上の文字列
}
```

### タプルと配列

```rust
fn main() {
    // タプル（異なる型OK、固定長）
    let tup: (i32, f64, bool) = (500, 6.4, true);
    let (x, y, z) = tup; // 分割代入
    println!("{}", tup.0); // インデックスアクセス

    // 配列（同じ型、固定長）
    let arr = [1, 2, 3, 4, 5];
    println!("{}", arr[0]);
}
```

---

## 4. 所有権（Ownership）

> ⚡ **ここがRustの核心です。じっくり読んでください。**

### なぜ所有権が必要か？

他の言語のメモリ管理方法：
- **GC言語（Java, Python, Go）**: ランタイムが使われなくなったメモリを自動解放 → 便利だがパフォーマンスコスト
- **手動管理（C, C++）**: プログラマーが `malloc/free` → 速いがバグの温床（二重解放、解放後アクセス）

Rustの方法：
- **コンパイラが所有権ルールをチェックして自動解放** → GCなしでメモリ安全

### 所有権の3つのルール

1. Rustの各値は**オーナー（owner）**と呼ばれる変数を持つ
2. 一度に存在できるオーナーは**1つだけ**
3. オーナーがスコープを外れると、値は**自動的に解放（drop）**される

```rust
fn main() {
    {
        let s = String::from("hello"); // s がオーナー
        // s を使った処理
    } // ← ここでスコープ終了 → s が自動的に解放される
    // println!("{}", s); // ← ここでは使えない！
}
```

### ムーブ（Move）

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1; // 所有権が s1 から s2 へ「移動（ムーブ）」

    // println!("{}", s1); // ← コンパイルエラー！s1 はもう無効
    println!("{}", s2); // OK
}
```

> 💡 **なぜこうなるの？**  
> `String` はヒープにデータを持ちます。`s2 = s1` でポインタをコピーすると、2つの変数が同じヒープデータを指してしまいます。スコープ終了時に2回解放されてしまう（二重解放バグ）のを防ぐため、Rustは所有権を「移動」させます。

### コピー（Copy）

整数などスタック上にある小さな型は、コピーが安いのでムーブではなくコピーになります。

```rust
fn main() {
    let x = 5;
    let y = x; // コピー（x も y も有効）
    println!("x={}, y={}", x, y); // 両方使える
}
```

`Copy` トレイトを実装している型（整数、浮動小数点、bool、char、固定長タプルなど）はこの挙動になります。

### 関数と所有権

```rust
fn main() {
    let s = String::from("hello");
    takes_ownership(s); // 所有権が関数に移動
    // println!("{}", s); // ← もう使えない

    let x = 5;
    makes_copy(x); // i32 はコピーなので
    println!("{}", x); // まだ使える
}

fn takes_ownership(s: String) {
    println!("{}", s);
} // ここで s がdrop

fn makes_copy(x: i32) {
    println!("{}", x);
}
```

---

## 5. 借用と参照

所有権を移さずに値を使いたいとき → **参照（reference）** を使います。

```rust
fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(&s1); // & で参照を渡す
    println!("'{}' の長さは {} です", s1, len); // s1 はまだ使える！
}

fn calculate_length(s: &String) -> usize { // & で参照を受け取る
    s.len()
} // s はただの参照なので、ここで drop されない
```

> 参照を作ることを「**借用（borrow）**」と言います。借りているだけなので、関数が終わっても元の値は消えません。

### ミュータブルな参照

```rust
fn main() {
    let mut s = String::from("hello");
    change(&mut s); // ミュータブル参照を渡す
    println!("{}", s);
}

fn change(s: &mut String) {
    s.push_str(", world");
}
```

### 参照のルール（重要）

同時に存在できる参照には制限があります：

```
✅ イミュータブル参照（&T）は何個でも同時に持てる
✅ ミュータブル参照（&mut T）は1つだけ持てる
❌ イミュータブル参照とミュータブル参照を同時に持つことはできない
```

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &s;
    let r2 = &s;
    println!("{} {}", r1, r2); // OK（イミュータブルは複数OK）

    let r3 = &mut s;
    // println!("{} {}", r1, r3); // ← エラー！イミュータブルとミュータブルの共存NG
}
```

> 💡 **なぜこのルール？**  
> データ競合（data race）をコンパイル時に防ぐためです。複数スレッドからの同時書き込みによるバグを、実行前に排除できます。

---

## 6. スライス

配列や文字列の一部を参照する型です。

```rust
fn main() {
    let s = String::from("hello world");

    let hello = &s[0..5];  // "hello"
    let world = &s[6..11]; // "world"

    println!("{} {}", hello, world);

    // 配列のスライス
    let a = [1, 2, 3, 4, 5];
    let slice = &a[1..3]; // [2, 3]
}
```

---

## 7. 構造体（Struct）

```rust
// 定義
struct User {
    username: String,
    email: String,
    age: u32,
    active: bool,
}

fn main() {
    // インスタンス生成
    let mut user = User {
        email: String::from("test@example.com"),
        username: String::from("yamada"),
        age: 25,
        active: true,
    };

    // フィールドアクセス（インスタンスが mut なら変更可能）
    user.age = 26;

    println!("{} ({}歳)", user.username, user.age);
}
```

### メソッド

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // メソッド（selfを受け取る）
    fn area(&self) -> u32 {
        self.width * self.height
    }

    // 関連関数（selfを受け取らない。他言語の static メソッド相当）
    fn square(size: u32) -> Rectangle {
        Rectangle { width: size, height: size }
    }
}

fn main() {
    let rect = Rectangle { width: 30, height: 50 };
    println!("面積: {}", rect.area());

    let sq = Rectangle::square(25); // :: で関連関数を呼ぶ
    println!("正方形の面積: {}", sq.area());
}
```

---

## 8. 列挙型（Enum）とパターンマッチ

RustのEnumは非常に強力で、**値を持てる**のが特徴です。

```rust
enum Message {
    Quit,                       // データなし
    Move { x: i32, y: i32 },   // 名前付きフィールド
    Write(String),              // String を持つ
    ChangeColor(i32, i32, i32), // 3つのi32を持つ
}
```

### `match` で分岐

```rust
fn process(msg: Message) {
    match msg {
        Message::Quit => println!("終了"),
        Message::Move { x, y } => println!("({}, {}) に移動", x, y),
        Message::Write(text) => println!("テキスト: {}", text),
        Message::ChangeColor(r, g, b) => println!("色: ({}, {}, {})", r, g, b),
    }
}
```

> `match` は**全パターンを網羅しないとコンパイルエラー**になります。漏れを防いでくれる強力な機能です。

### Option<T> — nullの代わり

Rustには `null` がありません。代わりに `Option<T>` を使います。

```rust
fn divide(a: f64, b: f64) -> Option<f64> {
    if b == 0.0 {
        None
    } else {
        Some(a / b)
    }
}

fn main() {
    match divide(10.0, 2.0) {
        Some(result) => println!("結果: {}", result),
        None => println!("0除算エラー"),
    }

    // if let を使った簡潔な書き方
    if let Some(result) = divide(10.0, 3.0) {
        println!("結果: {}", result);
    }
}
```

---

## 9. エラー処理

### Result<T, E>

Rustはエラーを戻り値で表現します。例外（Exception）はありません。

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let file = File::open("hello.txt");

    match file {
        Ok(f) => println!("ファイルを開いた"),
        Err(e) => match e.kind() {
            ErrorKind::NotFound => println!("ファイルが見つからない"),
            _ => println!("その他のエラー: {:?}", e),
        },
    }
}
```

### `?` 演算子（エラーの伝播）

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_file_contents(path: &str) -> Result<String, io::Error> {
    let mut f = File::open(path)?; // エラーなら即座に return Err(...)
    let mut contents = String::new();
    f.read_to_string(&mut contents)?;
    Ok(contents)
}

fn main() {
    match read_file_contents("hello.txt") {
        Ok(text) => println!("{}", text),
        Err(e) => println!("エラー: {}", e),
    }
}
```

> `?` は「エラーなら早期リターン、成功なら値を取り出す」の省略記法です。

---

## 10. トレイト（Trait）

他言語の**インターフェース**に相当します。「この型は○○ができる」という契約を定義します。

```rust
trait Summary {
    fn summarize(&self) -> String;

    // デフォルト実装も持てる
    fn preview(&self) -> String {
        format!("続きを読む: {}", self.summarize())
    }
}

struct Article {
    title: String,
    content: String,
}

struct Tweet {
    username: String,
    body: String,
}

impl Summary for Article {
    fn summarize(&self) -> String {
        format!("{}: {}...", self.title, &self.content[..50.min(self.content.len())])
    }
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.body)
    }
}

// トレイト境界：Summaryを実装した型なら何でも受け取る
fn notify(item: &impl Summary) {
    println!("速報！ {}", item.summarize());
}
```

---

## 11. ジェネリクス

型を抽象化して、複数の型に対応するコードを書けます。

```rust
// ジェネリックな関数
fn largest<T: PartialOrd>(list: &[T]) -> &T {
    let mut largest = &list[0];
    for item in list {
        if item > largest {
            largest = item;
        }
    }
    largest
}

fn main() {
    let numbers = vec![34, 50, 25, 100, 65];
    println!("最大値: {}", largest(&numbers));

    let chars = vec!['y', 'm', 'a', 'q'];
    println!("最大値: {}", largest(&chars));
}
```

---

## 12. クロージャとイテレータ

### クロージャ

環境をキャプチャできる無名関数です。

```rust
fn main() {
    let x = 10;
    let add_x = |n| n + x; // x を環境からキャプチャ
    println!("{}", add_x(5)); // 15

    // よく使うパターン
    let numbers = vec![1, 2, 3, 4, 5];

    let doubled: Vec<i32> = numbers.iter()
        .map(|n| n * 2)
        .collect();

    let evens: Vec<&i32> = numbers.iter()
        .filter(|n| *n % 2 == 0)
        .collect();

    let sum: i32 = numbers.iter().sum();

    println!("{:?}", doubled); // [2, 4, 6, 8, 10]
    println!("{:?}", evens);   // [2, 4]
    println!("{}", sum);       // 15
}
```

---

## 13. モジュールとクレート

### モジュール

```rust
// src/main.rs
mod math {
    pub fn add(a: i32, b: i32) -> i32 {
        a + b
    }

    pub fn subtract(a: i32, b: i32) -> i32 {
        a - b
    }
}

fn main() {
    println!("{}", math::add(5, 3));      // 8
    println!("{}", math::subtract(5, 3)); // 2

    // use でパスを短くできる
    use math::add;
    println!("{}", add(10, 20));
}
```

### 外部クレートの追加

`Cargo.toml` に追記します（npmの `package.json` 相当）：

```toml
[dependencies]
serde = { version = "1.0", features = ["derive"] }
rand = "0.8"
```

```bash
cargo build  # 自動でダウンロード＆ビルド
```

---

## 14. 練習課題

以下の課題に取り組んでみましょう。

### 課題1: 温度変換（基礎）

摂氏を華氏に変換する関数を作ってください。

```
C → F: F = C × 9/5 + 32
```

```rust
fn celsius_to_fahrenheit(c: f64) -> f64 {
    // ここを実装
}

fn main() {
    println!("0°C = {}°F", celsius_to_fahrenheit(0.0));   // 32
    println!("100°C = {}°F", celsius_to_fahrenheit(100.0)); // 212
}
```

### 課題2: フィボナッチ数列（再帰とループ）

n番目のフィボナッチ数を返す関数を、ループを使って実装してください。

```rust
fn fibonacci(n: u64) -> u64 {
    // ここを実装
}

fn main() {
    for i in 0..10 {
        print!("{} ", fibonacci(i));
    }
    // 期待出力: 0 1 1 2 3 5 8 13 21 34
}
```

### 課題3: 構造体とメソッド（中級）

`Stack<T>` 構造体を実装してください。

- `push(value: T)`: 値を積む
- `pop() -> Option<T>`: 値を取り出す（空なら None）
- `peek() -> Option<&T>`: 先頭を覗く（取り出さない）
- `is_empty() -> bool`: 空かどうか

```rust
struct Stack<T> {
    // ここを実装
}

impl<T> Stack<T> {
    fn new() -> Self {
        // ここを実装
    }
    // メソッドを実装
}

fn main() {
    let mut stack: Stack<i32> = Stack::new();
    stack.push(1);
    stack.push(2);
    stack.push(3);
    println!("{:?}", stack.peek()); // Some(3)
    println!("{:?}", stack.pop());  // Some(3)
    println!("{:?}", stack.pop());  // Some(2)
}
```

### 課題4: エラー処理（発展）

CSVのような文字列（`"1,2,3,4,5"`）を受け取り、数値のVecを返す関数を作ってください。  
パースに失敗した場合は `Err` を返してください。

```rust
fn parse_numbers(input: &str) -> Result<Vec<i32>, String> {
    // ここを実装
}

fn main() {
    println!("{:?}", parse_numbers("1,2,3,4,5")); // Ok([1, 2, 3, 4, 5])
    println!("{:?}", parse_numbers("1,2,abc,4"));  // Err(...)
}
```

---

## まとめと次のステップ

| 学んだこと | 要点 |
|---|---|
| 所有権 | メモリ安全性の核心。一度に1オーナー |
| 借用と参照 | 所有権を移さずに値を使う手段 |
| Option/Result | null や例外の代わり。コンパイラが漏れを検出 |
| トレイト | インターフェース相当。型の能力を定義 |
| ジェネリクス | 型を抽象化。コードの再利用 |

### おすすめの次のステップ

1. **[The Rust Book](https://doc.rust-lang.org/book/)** — 公式ドキュメント（英語だが最高品質）
2. **[Rustlings](https://github.com/rust-lang/rustlings)** — 小さな演習問題集（`cargo install rustlings` で使える）
3. **[Rust by Example](https://doc.rust-lang.org/rust-by-example/)** — コードで学ぶ公式サイト
4. 練習として：CLIツール、簡単なWebサーバー（[Actix-web](https://actix.rs/)）を作ってみる

---

*この教材は Rust 1.70+ を対象としています。*
