# Go 入門教材

**対象者:** Python・JavaScript・Javaなど他言語の経験者でGoを学びたい人
**目標:** Goの基本概念を理解し、シンプルなプログラムを書けるようになる

---

## 目次

1. [環境構築](#1-環境構築)
2. [Hello, World!](#2-hello-world)
3. [変数と型](#3-変数と型)
4. [関数](#4-関数)
5. [制御構文](#5-制御構文)
6. [配列・スライス・マップ](#6-配列スライスマップ)
7. [構造体とメソッド](#7-構造体とメソッド)
8. [インターフェース](#8-インターフェース)
9. [エラー処理](#9-エラー処理)
10. [ゴルーチンとチャネル](#10-ゴルーチンとチャネル)
11. [パッケージとモジュール](#11-パッケージとモジュール)
12. [練習課題](#12-練習課題)

---

## 1. 環境構築

### Goのインストール

公式サイト [https://go.dev/dl/](https://go.dev/dl/) からインストーラーをダウンロードする。

インストール確認:

```bash
go version
# go version go1.22.0 linux/amd64
```

### プロジェクトの作成

Goでは **モジュール** 単位でプロジェクトを管理する。

```bash
mkdir hello-go
cd hello-go
go mod init hello-go   # go.mod ファイルが生成される
```

### よく使う `go` コマンド

| コマンド | 説明 |
|---|---|
| `go run main.go` | ファイルをコンパイルして即実行 |
| `go build` | バイナリをビルド |
| `go test ./...` | テストを実行 |
| `go fmt ./...` | コードを自動フォーマット |
| `go get <pkg>` | 外部パッケージを追加 |
| `go mod tidy` | 不要な依存関係を整理 |

💡 **他言語との比較:** RustのCargoに相当するのがGoの `go` コマンド。ただしGoはツールチェーンが言語に内蔵されており、別途インストール不要。

---

## 2. Hello, World!

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

```bash
go run main.go
# Hello, World!
```

### コードの構成要素

- `package main` — 実行可能プログラムのエントリポイント。すべてのGoファイルはパッケージに属する
- `import "fmt"` — 標準ライブラリのフォーマットパッケージをインポート
- `func main()` — プログラムの開始点。`main` パッケージの `main` 関数が最初に呼ばれる
- `fmt.Println` — 改行付きで標準出力に表示

⚡ **Goの特徴:** 未使用のインポートや未使用の変数があると **コンパイルエラー** になる。コードの清潔さをコンパイラが強制する。

---

## 3. 変数と型

### 変数宣言

Goには変数宣言の方法が複数ある。

```go
package main

import "fmt"

func main() {
    // var キーワードによる宣言（関数の外でも使える）
    var name string = "Alice"
    var age int = 30

    // 型推論（型を省略できる）
    var score float64 = 98.5

    // := による短縮宣言（関数内のみ）
    city := "Tokyo"
    isActive := true

    fmt.Println(name, age, score, city, isActive)
}
```

💡 `:=` は宣言と代入を同時に行うGoらしい書き方。関数内でよく使われる。

### 基本型一覧

```go
// 整数
var i int = 42         // プラットフォーム依存（32 or 64 bit）
var i8 int8 = 127
var i64 int64 = 1000000
var u uint = 10        // 符号なし整数

// 浮動小数点
var f32 float32 = 3.14
var f64 float64 = 3.14159265358979

// 真偽値
var b bool = true

// 文字列（UTF-8エンコード）
var s string = "こんにちは"

// バイト（uint8の別名）とルーン（int32の別名、Unicodeコードポイント）
var by byte = 'A'
var r rune = 'あ'
```

### ゼロ値

Goでは宣言時に初期値を指定しなければ **ゼロ値** が自動で設定される。

```go
var i int       // 0
var f float64   // 0.0
var b bool      // false
var s string    // "" (空文字列)
var p *int      // nil
```

✅ ゼロ値があるため、未初期化変数によるバグが起きにくい。

### 定数

```go
const Pi = 3.14159
const MaxRetry = 3
const Greeting = "Hello"

// iota を使った列挙
const (
    Sunday = iota // 0
    Monday        // 1
    Tuesday       // 2
    Wednesday     // 3
)
```

### 型変換

Goは**暗黙の型変換がない**。明示的に変換する必要がある。

```go
var i int = 42
var f float64 = float64(i)   // int → float64
var u uint = uint(f)         // float64 → uint

// ❌ これはコンパイルエラー
// var f float64 = i
```

---

## 4. 関数

### 基本的な関数

```go
package main

import "fmt"

// 引数と戻り値の型を明示する
func add(x int, y int) int {
    return x + y
}

// 同じ型の引数はまとめて書ける
func multiply(x, y int) int {
    return x * y
}

func main() {
    fmt.Println(add(3, 4))       // 7
    fmt.Println(multiply(3, 4))  // 12
}
```

### 複数の戻り値 ⚡

Goの大きな特徴の一つ。複数の値を返せる。

```go
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, fmt.Errorf("ゼロ除算はできません")
    }
    return a / b, nil
}

func main() {
    result, err := divide(10.0, 3.0)
    if err != nil {
        fmt.Println("エラー:", err)
        return
    }
    fmt.Printf("結果: %.2f\n", result)
}
```

💡 **エラー処理のイディオム:** Goでは例外ではなく、戻り値でエラーを伝える。`(値, error)` のペアを返すパターンが標準的。

### 名前付き戻り値

```go
func minMax(nums []int) (min, max int) {
    min, max = nums[0], nums[0]
    for _, n := range nums {
        if n < min {
            min = n
        }
        if n > max {
            max = n
        }
    }
    return // 裸のreturn（名前付き変数をそのまま返す）
}
```

### 可変長引数

```go
func sum(nums ...int) int {
    total := 0
    for _, n := range nums {
        total += n
    }
    return total
}

func main() {
    fmt.Println(sum(1, 2, 3))        // 6
    fmt.Println(sum(1, 2, 3, 4, 5)) // 15

    nums := []int{1, 2, 3, 4}
    fmt.Println(sum(nums...))        // スライスを展開して渡す: 10
}
```

### 関数は第一級オブジェクト

```go
func apply(f func(int) int, x int) int {
    return f(x)
}

func main() {
    double := func(x int) int { return x * 2 }
    fmt.Println(apply(double, 5)) // 10

    // 即時実行
    result := func(x, y int) int { return x + y }(3, 4)
    fmt.Println(result) // 7
}
```

---

## 5. 制御構文

### if 文

```go
package main

import "fmt"

func classify(n int) string {
    if n > 0 {
        return "正"
    } else if n < 0 {
        return "負"
    } else {
        return "ゼロ"
    }
}

func main() {
    // if の初期化文（スコープがif内に限定される）
    if x := 10; x > 5 {
        fmt.Println("xは5より大きい:", x)
    }
    // ここでは x は使えない

    fmt.Println(classify(42))  // 正
    fmt.Println(classify(-1))  // 負
}
```

### for 文（Goで唯一のループ）

Goには `while` も `do-while` もない。すべて `for` で表現する。

```go
// C言語スタイル
for i := 0; i < 5; i++ {
    fmt.Println(i)
}

// while相当（条件のみ）
n := 1
for n < 100 {
    n *= 2
}
fmt.Println(n) // 128

// 無限ループ
for {
    // break で抜ける
    break
}

// rangeを使ったイテレーション
fruits := []string{"apple", "banana", "cherry"}
for i, fruit := range fruits {
    fmt.Printf("%d: %s\n", i, fruit)
}

// インデックスが不要なら _ で捨てる
for _, fruit := range fruits {
    fmt.Println(fruit)
}

// マップのイテレーション
scores := map[string]int{"Alice": 90, "Bob": 85}
for name, score := range scores {
    fmt.Printf("%s: %d\n", name, score)
}
```

### switch 文

```go
func dayName(d int) string {
    switch d {
    case 0:
        return "日曜日"
    case 1:
        return "月曜日"
    case 2:
        return "火曜日"
    default:
        return "その他"
    }
}

// 条件式を使ったswitch（if-elseの代替）
func grade(score int) string {
    switch {
    case score >= 90:
        return "A"
    case score >= 80:
        return "B"
    case score >= 70:
        return "C"
    default:
        return "F"
    }
}
```

💡 **他言語との違い:** Goの `switch` は各ケースに自動で `break` がある。明示的に次のケースに落としたい場合は `fallthrough` を使う。

---

## 6. 配列・スライス・マップ

### 配列（固定長）

```go
// 配列は固定長で、型の一部として扱われる
var a [5]int
a[0] = 1
fmt.Println(a) // [1 0 0 0 0]

// リテラルで初期化
b := [3]string{"Go", "Rust", "Python"}
fmt.Println(len(b)) // 3

// [...]で長さを自動推論
c := [...]int{1, 2, 3, 4, 5}
```

⚡ 実際にはスライスを使うことがほとんど。配列を直接使う機会は少ない。

### スライス（可変長・実用的）

```go
// スライスは配列への参照 + 長さ + 容量
s := []int{1, 2, 3, 4, 5}
fmt.Println(s[1:3]) // [2 3]（インデックス1から3未満）
fmt.Println(s[:2])  // [1 2]
fmt.Println(s[3:])  // [4 5]

// make で作成（長さと容量を指定）
s2 := make([]int, 3, 5) // len=3, cap=5
fmt.Println(s2)          // [0 0 0]

// append で要素追加
s3 := []int{1, 2, 3}
s3 = append(s3, 4, 5)
fmt.Println(s3) // [1 2 3 4 5]

// スライスの連結
s4 := []int{6, 7}
s3 = append(s3, s4...)
fmt.Println(s3) // [1 2 3 4 5 6 7]
```

⚡ **注意:** スライスは元の配列を参照するため、コピーと参照の挙動を理解する必要がある。

```go
original := []int{1, 2, 3}
ref := original       // 参照（同じメモリを指す）
ref[0] = 99
fmt.Println(original) // [99 2 3] ← 変わってしまう！

// 独立したコピーを作るには copy を使う
copied := make([]int, len(original))
copy(copied, original)
copied[0] = 100
fmt.Println(original) // [99 2 3] ← 変わらない
```

### マップ（ハッシュマップ）

```go
// マップの作成
m := map[string]int{
    "Alice": 90,
    "Bob":   85,
    "Carol": 92,
}

// 要素へのアクセス
fmt.Println(m["Alice"]) // 90

// 存在確認（2値受け取り）
score, ok := m["Dave"]
if !ok {
    fmt.Println("Dave は存在しない")
} else {
    fmt.Println("Dave:", score)
}

// 要素の追加・更新
m["Dave"] = 78
m["Alice"] = 95  // 更新

// 要素の削除
delete(m, "Bob")

// makeによる初期化
m2 := make(map[string][]string)
m2["fruits"] = append(m2["fruits"], "apple", "banana")
```

---

## 7. 構造体とメソッド

Goにはクラスがない。代わりに **構造体** と **メソッド** を組み合わせる。

### 構造体の定義と使用

```go
package main

import (
    "fmt"
    "math"
)

type Circle struct {
    X, Y   float64 // 中心座標
    Radius float64
}

type Rectangle struct {
    Width  float64
    Height float64
}

func main() {
    // 構造体リテラル
    c := Circle{X: 0, Y: 0, Radius: 5}
    fmt.Println(c.Radius) // 5

    // フィールド名省略（順序依存なので非推奨）
    r := Rectangle{10, 20}
    fmt.Println(r.Width, r.Height) // 10 20

    // new() でポインタを取得
    c2 := new(Circle)
    c2.Radius = 3
    fmt.Println(c2) // &{0 0 3}
}
```

### メソッド

```go
// 値レシーバ（コピーを受け取る）
func (c Circle) Area() float64 {
    return math.Pi * c.Radius * c.Radius
}

func (c Circle) String() string {
    return fmt.Sprintf("Circle(r=%.1f)", c.Radius)
}

// ポインタレシーバ（元の値を変更できる）
func (c *Circle) Scale(factor float64) {
    c.Radius *= factor
}

func main() {
    c := Circle{Radius: 5}
    fmt.Printf("面積: %.2f\n", c.Area())  // 面積: 78.54
    fmt.Println(c)                         // Circle(r=5.0)

    c.Scale(2)
    fmt.Printf("拡大後: %.1f\n", c.Radius) // 拡大後: 10.0
}
```

💡 **値 vs ポインタレシーバ:**
- **値レシーバ** — 構造体を変更しない。コピーで動作
- **ポインタレシーバ** — 構造体を変更する。大きな構造体のコピーを避けたい場合にも使う
- 一つの型に対しては、どちらかに統一するのが慣習

### 埋め込み（継承の代替）

```go
type Animal struct {
    Name string
}

func (a Animal) Speak() string {
    return a.Name + " が鳴いています"
}

type Dog struct {
    Animal        // 埋め込み（フィールドとメソッドが昇格する）
    Breed  string
}

func main() {
    d := Dog{
        Animal: Animal{Name: "ポチ"},
        Breed:  "柴犬",
    }
    fmt.Println(d.Speak()) // ポチ が鳴いています（Animalのメソッドを使える）
    fmt.Println(d.Name)    // ポチ（フィールドも昇格）
}
```

---

## 8. インターフェース

Goのインターフェースは **暗黙的** に実装される。`implements` キーワードは不要。

```go
package main

import (
    "fmt"
    "math"
)

// インターフェースの定義
type Shape interface {
    Area() float64
    Perimeter() float64
}

type Circle struct {
    Radius float64
}

func (c Circle) Area() float64 {
    return math.Pi * c.Radius * c.Radius
}

func (c Circle) Perimeter() float64 {
    return 2 * math.Pi * c.Radius
}

type Rectangle struct {
    Width, Height float64
}

func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

func (r Rectangle) Perimeter() float64 {
    return 2 * (r.Width + r.Height)
}

// Shape インターフェースを受け取る関数
func printShapeInfo(s Shape) {
    fmt.Printf("面積: %.2f, 周囲: %.2f\n", s.Area(), s.Perimeter())
}

func main() {
    shapes := []Shape{
        Circle{Radius: 5},
        Rectangle{Width: 4, Height: 6},
    }

    for _, s := range shapes {
        printShapeInfo(s)
    }
}
```

### 型アサーションと型スイッチ

```go
func describe(i interface{}) {
    switch v := i.(type) {
    case int:
        fmt.Printf("整数: %d\n", v)
    case string:
        fmt.Printf("文字列: %q\n", v)
    case bool:
        fmt.Printf("真偽値: %t\n", v)
    default:
        fmt.Printf("不明な型: %T\n", v)
    }
}

func main() {
    describe(42)
    describe("hello")
    describe(true)
    describe(3.14)
}
```

### よく使うインターフェース

```go
import (
    "fmt"
    "io"
    "strings"
)

// fmt.Stringer: String() メソッドを持つ型はfmt.Printで自動的に使われる
type Point struct{ X, Y int }

func (p Point) String() string {
    return fmt.Sprintf("(%d, %d)", p.X, p.Y)
}

// error インターフェース
type ValidationError struct {
    Field   string
    Message string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("検証エラー [%s]: %s", e.Field, e.Message)
}

func main() {
    p := Point{3, 4}
    fmt.Println(p) // (3, 4) ← Stringerが使われる

    // io.Reader を使う関数は、ファイル・文字列・ネットワークなどを統一的に扱える
    r := strings.NewReader("Hello, Go!")
    buf := make([]byte, 5)
    n, _ := r.Read(buf)
    fmt.Printf("%d bytes: %s\n", n, buf[:n]) // 5 bytes: Hello
}
```

---

## 9. エラー処理

Goでは例外を使わない。エラーは **戻り値** として扱う。

### error インターフェース

```go
package main

import (
    "errors"
    "fmt"
)

// errors.New でシンプルなエラーを作る
var ErrDivByZero = errors.New("ゼロ除算エラー")

func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, ErrDivByZero
    }
    return a / b, nil
}

func main() {
    result, err := divide(10, 3)
    if err != nil {
        fmt.Println("エラー:", err)
        return
    }
    fmt.Printf("10 / 3 = %.4f\n", result)

    _, err = divide(5, 0)
    if err != nil {
        // errors.Is でエラー種別を確認
        if errors.Is(err, ErrDivByZero) {
            fmt.Println("ゼロ除算が発生しました")
        }
    }
}
```

### カスタムエラー型

```go
type AppError struct {
    Code    int
    Message string
    Err     error // ラップしたエラー
}

func (e *AppError) Error() string {
    if e.Err != nil {
        return fmt.Sprintf("[%d] %s: %v", e.Code, e.Message, e.Err)
    }
    return fmt.Sprintf("[%d] %s", e.Code, e.Message)
}

func (e *AppError) Unwrap() error {
    return e.Err
}

func fetchUser(id int) (string, error) {
    if id <= 0 {
        return "", &AppError{
            Code:    400,
            Message: "無効なユーザーID",
        }
    }
    if id > 100 {
        return "", &AppError{
            Code:    404,
            Message: "ユーザーが見つかりません",
            Err:     fmt.Errorf("id=%d は存在しない", id),
        }
    }
    return fmt.Sprintf("User-%d", id), nil
}

func main() {
    _, err := fetchUser(-1)
    var appErr *AppError
    if errors.As(err, &appErr) {
        fmt.Printf("コード: %d, メッセージ: %s\n", appErr.Code, appErr.Message)
    }
}
```

### エラーのラップと fmt.Errorf

```go
func readConfig(path string) error {
    // fmt.Errorf の %w でエラーをラップ
    return fmt.Errorf("設定ファイルの読み込み失敗 (%s): %w", path, errors.New("ファイルが見つかりません"))
}

func initialize() error {
    if err := readConfig("/etc/app.conf"); err != nil {
        return fmt.Errorf("初期化失敗: %w", err)
    }
    return nil
}

func main() {
    err := initialize()
    if err != nil {
        fmt.Println(err)
        // 初期化失敗: 設定ファイルの読み込み失敗 (/etc/app.conf): ファイルが見つかりません
    }
}
```

💡 **Goのエラー処理の哲学:**
- エラーは特別なものではなく、普通の値として扱う
- `if err != nil` を繰り返すのはGoらしいスタイル
- 「エラーを無視しない」ことをコードが明示する

---

## 10. ゴルーチンとチャネル

Goの最大の特徴。軽量な並行処理を言語レベルでサポートする。

### ゴルーチン

```go
package main

import (
    "fmt"
    "time"
)

func say(s string) {
    for i := 0; i < 3; i++ {
        fmt.Println(s)
        time.Sleep(100 * time.Millisecond)
    }
}

func main() {
    go say("world") // ゴルーチンとして非同期に実行
    say("hello")    // メインゴルーチンで実行
}
```

⚡ `go` キーワードを付けるだけで非同期実行できる。ゴルーチンはOSスレッドより軽量（初期スタック約2KB）で、数千〜数百万個を同時に動かせる。

### チャネル

ゴルーチン間のデータのやり取りには **チャネル** を使う。

```go
func sum(nums []int, ch chan int) {
    total := 0
    for _, n := range nums {
        total += n
    }
    ch <- total // チャネルに送信
}

func main() {
    nums := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}

    ch := make(chan int)

    // 2つのゴルーチンで並列計算
    go sum(nums[:5], ch)
    go sum(nums[5:], ch)

    a, b := <-ch, <-ch // チャネルから受信（2回）
    fmt.Println(a + b)  // 55
}
```

### バッファ付きチャネル

```go
// バッファなし: 送信側が受信されるまでブロック
ch1 := make(chan int)

// バッファあり: バッファが満杯になるまで非ブロック
ch2 := make(chan int, 3)
ch2 <- 1 // ブロックしない
ch2 <- 2
ch2 <- 3
// ch2 <- 4 // バッファ満杯でブロック

fmt.Println(<-ch2) // 1
```

### select 文

複数のチャネル操作を待つ。

```go
func fibonacci(n int, ch chan int, quit chan bool) {
    a, b := 0, 1
    for {
        select {
        case ch <- a: // ch に送信できる場合
            a, b = b, a+b
        case <-quit: // quit チャネルから受信した場合
            fmt.Println("終了")
            return
        }
    }
}

func main() {
    ch := make(chan int)
    quit := make(chan bool)

    go func() {
        for i := 0; i < 10; i++ {
            fmt.Println(<-ch)
        }
        quit <- true
    }()

    fibonacci(100, ch, quit)
}
```

### sync.WaitGroup

```go
import (
    "fmt"
    "sync"
)

func worker(id int, wg *sync.WaitGroup) {
    defer wg.Done() // 終了時に Done を呼ぶ
    fmt.Printf("Worker %d 開始\n", id)
    // ... 何か処理 ...
    fmt.Printf("Worker %d 完了\n", id)
}

func main() {
    var wg sync.WaitGroup

    for i := 1; i <= 5; i++ {
        wg.Add(1)        // カウンタをインクリメント
        go worker(i, &wg)
    }

    wg.Wait() // すべてのゴルーチンが Done を呼ぶまで待つ
    fmt.Println("全ワーカー完了")
}
```

### sync.Mutex（ミューテックス）

```go
type SafeCounter struct {
    mu sync.Mutex
    v  map[string]int
}

func (c *SafeCounter) Inc(key string) {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.v[key]++
}

func (c *SafeCounter) Value(key string) int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.v[key]
}
```

💡 **Goの並行処理の哲学:**
> "Don't communicate by sharing memory; share memory by communicating."
> メモリを共有して通信するのではなく、通信してメモリを共有せよ。

チャネルを使ってデータを渡すことで、ミューテックスによるロックを減らせる。

---

## 11. パッケージとモジュール

### パッケージの基本

```
myapp/
├── go.mod
├── main.go
└── calculator/
    └── calc.go
```

```go
// calculator/calc.go
package calculator

// 大文字始まり = 公開（exported）
func Add(a, b int) int {
    return a + b
}

func Subtract(a, b int) int {
    return a - b
}

// 小文字始まり = パッケージ内プライベート
func validate(a, b int) bool {
    return a >= 0 && b >= 0
}
```

```go
// main.go
package main

import (
    "fmt"
    "myapp/calculator" // モジュール名/パッケージパス
)

func main() {
    fmt.Println(calculator.Add(3, 4))      // 7
    fmt.Println(calculator.Subtract(10, 3)) // 7
}
```

### go.mod ファイル

```
module myapp

go 1.22

require (
    github.com/some/library v1.2.3
)
```

### 外部パッケージの追加

```bash
# パッケージを追加
go get github.com/fatih/color

# 不要なパッケージを削除
go mod tidy
```

### init 関数

```go
package mypackage

import "fmt"

// パッケージがインポートされたときに自動実行される
func init() {
    fmt.Println("mypackage が初期化されました")
}
```

### テスト

Goにはテストが標準で組み込まれている。

```go
// calculator/calc_test.go
package calculator

import "testing"

func TestAdd(t *testing.T) {
    result := Add(2, 3)
    if result != 5 {
        t.Errorf("Add(2, 3) = %d; want 5", result)
    }
}

func TestSubtract(t *testing.T) {
    tests := []struct {
        a, b, want int
    }{
        {10, 3, 7},
        {5, 5, 0},
        {0, 1, -1},
    }

    for _, tt := range tests {
        got := Subtract(tt.a, tt.b)
        if got != tt.want {
            t.Errorf("Subtract(%d, %d) = %d; want %d", tt.a, tt.b, got, tt.want)
        }
    }
}
```

```bash
go test ./...          # すべてのテストを実行
go test -v ./...       # 詳細出力
go test -run TestAdd   # 特定のテストのみ
```

---

## 12. 練習課題

### 課題 1: FizzBuzz（基礎）

1から100までの数を出力せよ。
- 3の倍数のとき: `Fizz`
- 5の倍数のとき: `Buzz`
- 両方の倍数のとき: `FizzBuzz`

```go
package main

import "fmt"

func fizzBuzz(n int) string {
    // ここを実装してください
    return ""
}

func main() {
    for i := 1; i <= 100; i++ {
        fmt.Println(fizzBuzz(i))
    }
}
```

---

### 課題 2: 単語カウンター（スライス・マップ）

テキストに含まれる各単語の出現回数を数えて、多い順に表示せよ。

```go
package main

import (
    "fmt"
    "sort"
    "strings"
)

func countWords(text string) map[string]int {
    // ここを実装してください
    return nil
}

func main() {
    text := "the quick brown fox jumps over the lazy dog the fox"
    counts := countWords(text)

    // 出現回数の多い順にソートして表示
    type wordCount struct {
        word  string
        count int
    }
    var wcs []wordCount
    for w, c := range counts {
        wcs = append(wcs, wordCount{w, c})
    }
    sort.Slice(wcs, func(i, j int) bool {
        return wcs[i].count > wcs[j].count
    })

    for _, wc := range wcs {
        fmt.Printf("%s: %d\n", wc.word, wc.count)
    }
}
```

---

### 課題 3: スタックの実装（ジェネリクス）

Go 1.18以降のジェネリクスを使って、任意の型を格納できるスタックを実装せよ。

```go
package main

import (
    "errors"
    "fmt"
)

type Stack[T any] struct {
    // ここを実装してください
}

func (s *Stack[T]) Push(v T) {
    // 実装してください
}

func (s *Stack[T]) Pop() (T, error) {
    // 実装してください
    var zero T
    return zero, errors.New("スタックが空です")
}

func (s *Stack[T]) Peek() (T, error) {
    // 実装してください
    var zero T
    return zero, errors.New("スタックが空です")
}

func (s *Stack[T]) IsEmpty() bool {
    // 実装してください
    return true
}

func main() {
    s := &Stack[int]{}
    s.Push(1)
    s.Push(2)
    s.Push(3)

    for !s.IsEmpty() {
        v, _ := s.Pop()
        fmt.Println(v) // 3, 2, 1 の順
    }

    // 文字列スタック
    ss := &Stack[string]{}
    ss.Push("Go")
    ss.Push("Rust")
    v, err := ss.Pop()
    if err == nil {
        fmt.Println(v) // Rust
    }
}
```

---

### 課題 4: 並行ファイルダウンローダー（ゴルーチン・チャネル）

複数のURLを並行してダウンロードし、すべての結果をまとめて表示せよ。
（実際のHTTPリクエストの代わりに `time.Sleep` でシミュレートする）

```go
package main

import (
    "fmt"
    "time"
)

type Result struct {
    URL     string
    Content string
    Err     error
}

// 実際はHTTPリクエストをするが、ここではシミュレート
func fetch(url string) (string, error) {
    time.Sleep(100 * time.Millisecond) // ネットワーク遅延のシミュレート
    return fmt.Sprintf("content of %s", url), nil
}

func fetchAll(urls []string) []Result {
    // ゴルーチンとチャネルを使って並行ダウンロードを実装してください
    return nil
}

func main() {
    urls := []string{
        "https://example.com/page1",
        "https://example.com/page2",
        "https://example.com/page3",
        "https://example.com/page4",
        "https://example.com/page5",
    }

    start := time.Now()
    results := fetchAll(urls)
    elapsed := time.Since(start)

    for _, r := range results {
        if r.Err != nil {
            fmt.Printf("エラー [%s]: %v\n", r.URL, r.Err)
        } else {
            fmt.Printf("成功 [%s]: %s\n", r.URL, r.Content)
        }
    }

    fmt.Printf("\n合計時間: %v\n", elapsed)
    // 並行処理なら ~100ms（逐次処理なら ~500ms）
}
```

---

## 次のステップ

### 公式ドキュメント
- [A Tour of Go](https://go.dev/tour/) — インタラクティブなGo入門
- [Effective Go](https://go.dev/doc/effective_go) — Goらしい書き方のガイド
- [Go by Example](https://gobyexample.com/) — 例題集

### 学習リソース
- [The Go Programming Language（書籍）](https://www.gopl.io/) — 定番の入門書
- [Go標準ライブラリのドキュメント](https://pkg.go.dev/std)

### 練習サイト
- [Exercism Go Track](https://exercism.org/tracks/go)
- [LeetCode](https://leetcode.com/)（Goで挑戦）

### 発展トピック（この教材の次）
- 標準ライブラリ（`net/http`, `encoding/json`, `database/sql`）
- テスト（`testing`, テーブル駆動テスト, ベンチマーク）
- コンテキスト（`context.Context`）
- リフレクション
- CGo（C言語との連携）
