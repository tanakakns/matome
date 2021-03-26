---
title: "文法"
date: 2021-02-09T22:14:00+09:00
draft: false
hide:
- toc
- nextpage
weight: 2
---

基本文法についてメモる。

1. パッケージ
2. 関数
3. 変数
4. ループ
5. 条件分岐
6. 配列とスライス
7. map
8. ポインタ
9. 構造体
10. インターフェース
11. 型
12. 並列処理

<!--more-->

[A Tour of Go](https://go-tour-jp.appspot.com/list)を一通りやるといい。  
とりあえず、こんにちは世界。

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, 世界")
}
```

[ideone.com](https://ideone.com/vPpbZ5)

```zsh
% go run hello.go
Hello, 世界
```

なお、[The Go Playground](https://play.golang.org/) というサービスを使うと Web でお試し実行できる。  
以降は、[A Tour of Go](https://go-tour-jp.appspot.com/list) を読みつつも、**自分向けに** 補足したり省略したりしてまとめたもの。

## 1. パッケージ

### 1.1. インポート

標準パッケージのインポートは以下。

```go
import (
	"fmt"
	"math/rand"
)
```

```go
import "fmt"
import "math"
```

インポートしたら `fmt.Println` のようなインポート名、 `rand.Intn(100)` のような一番後ろのパッケージ名で使用できる。  
また、インポート名を変更もできる。（ `f.Println` ）

```go
import (
	f "fmt"
)
```

また、独自や OSS ライブラリのインポートは Go modules を利用する場合は `${GOPATH}/packages/pkg/mod` 、しない場合は `${GOPATH}/src` からのパスを指定する。

```go
import (
    "github.com/spf13/cobra"
)
```

### 1.2. パッケージ外からの参照

Go では、最初の文字が **大文字で始まる名前** の変数・関数は、外部のパッケージから参照できる公開された名前( **exported name** )。  
例えば、 `Pi` は `math` パッケージでエクスポートされている。  
`pi` （小文字）ではないことに注意。

```go
package main

import (
	"fmt"
	"math"
)

func main() {
	//fmt.Println(math.pi)  // Error になる
    fmt.Println(math.Pi)
}
```

## 2. 関数

### 2.1. 宣言

```go
package main

import "fmt"

// func add(x , y int) int { // 引数の型定義、省略可能
func add(x int, y int) int {
	return x + y
}

func main() {
	fmt.Println(add(42, 13))
	
	// 関数も値として扱える
	sub := func(x, y int) int {
		return x - y
	}
	fmt.Println(sub(42, 13))
	
	// 即時関数
	v := func(x, y int) int {
		return x * y
	}(42, 13)
	fmt.Println(v)
}
```

### 2.2. 複数の return

```go
package main

import "fmt"

func swap(x, y string) (string, string) {
	return y, x
}

func main() {
	a, b := swap("hello", "world")
	fmt.Println(a, b)
}
```

### 2.3. return の変数名

返り値となる変数に名前をつけることができる。  
そして、 `return` と書くだけでよくなる。

```go
package main

import "fmt"

func split(sum int) (x, y int) {
	x = sum * 4 / 9
	y = sum - x
	return
}

func main() {
	fmt.Println(split(17))
}
```

### 2.4. クロージャ

Go の関数は **クロージャ** （関数オブジェクトの一種）。  
関数の引数として渡したり、遅延実行させたりできる。

```go
package main

import "fmt"

func adder() func(int) int {
	sum := 0
	return func(x int) int {
		sum += x
		return sum
	}
}

func main() {
	pos := adder() // 無名関数をオブジェクトとして受け取る
	for i := 0; i < 5; i++ {
		// 関数オブジェクト内で sum が保持されているので
		// 呼び出す度に加算されていく
		fmt.Println(pos(i))
	}
}
// 0
// 1
// 3
// 6
// 10
```

### 2.5. defer

`defer` ステートメントは、 `defer` へ渡した関数の実行を、呼び出し元の関数の終わり( `return` 後)まで遅延させる。  
`defer` へ渡した関数の引数は、すぐに評価されるが、その関数自体は呼び出し元の関数が `return` するまで実行されない。

```go
package main

import "fmt"

func main() {
	defer fmt.Println("world") // main 関数の終わりまで実行が遅延される

	fmt.Println("hello")
}
// hello
// world
// と出力される
```

`defer` が複数ある場合、その呼び出しはスタックされ、 LIFO の順番で実行される。（後から順に実行される）

```go
package main

import "fmt"

func main() {
	fmt.Println("counting")

	for i := 0; i < 10; i++ {
		defer fmt.Println(i)
	}

	fmt.Println("done")
}
// counting
// done
// 9
// 8
// 7
// 6
// 5
// 4
// 3
// 2
// 1
// 0
```

`defer` は `panic` の `recover` によく用いられる。（ Java で言う `try-catch` 的な）

```go
package main

import (
    "fmt"
)

func hoge() {
    defer func() {
        err := recover()
        if err != nil {
            fmt.Println("Recover!:", err)
        }
    }()
    panic("Panic!")
}

func main() {
    hoge()
}
```

`panic` は Java でいう `Runtime Exception` 。
エラーハンドリングでは使っちゃダメ。 `Error` インターフェースを使おう。

- https://qiita.com/nayuneko/items/3c0b3c0de9e8b27c9548

## 3. 変数

### 3.1. 宣言

`var` は **変数宣言** 。

```go
package main

import "fmt"

var c, python, java bool

func main() {
	var i int
	fmt.Println(i, c, python, java)
}
```

初期化子が与えられている場合、型を省略できる。

```go
package main

import "fmt"

var i, j int = 1, 2

func main() {
	var c, python, java = true, false, "no!"
	fmt.Println(i, j, c, python, java)
}
```

関数の中では、 `var` 宣言の代わりに、短い `:=` の代入文を使い、暗黙的な型宣言ができる。  
この場合、変数の型は右側の変数から型推論される。  

```go
i := 42           // int
f := 3.142        // float64
g := 0.867 + 0.5i // complex128
```

関数の外では、キーワードではじまる宣言( `var` , `func` など)が必要で、 `:=` での暗黙的な宣言は利用できない。

```go
package main

import "fmt"

func main() {
	var i, j int = 1, 2
	k := 3
	c, python, java := true, false, "no!"

	fmt.Println(i, j, k, c, python, java)
}
```

### 3.2. 基本型

- `bool`
- `string`
- `int` 、 `int8` 、 `int16` 、 `int32` 、 `int64`
- `uint` 、 `uint8` 、 `uint16` 、 `uint32` 、 `uint64` 、 `uintptr`
- `byte`
    - `uint8` の別名
- `rune`
    - `int32` の別名
    - Unicode のコードポイントを表す
	- rune とは古代文字を表す言葉( runes )、 Go では文字そのものを表すためにruneという言葉を使う
- `float32` 、 `float64`
- `complex64` 、 `complex128`

```go
package main

import (
	"fmt"
	"math/cmplx"
)

var (
	ToBe   bool       = false
	MaxInt uint64     = 1<<64 - 1
	z      complex128 = cmplx.Sqrt(-5 + 12i)
)

func main() {
	fmt.Printf("Type: %T Value: %v\n", ToBe, ToBe)
	fmt.Printf("Type: %T Value: %v\n", MaxInt, MaxInt)
	fmt.Printf("Type: %T Value: %v\n", z, z)
}
```

`int` や `uint` は OS や CPU などの実装系に依存して、 `int32` になったり `int64` になったりする。

|型|サイズ(bit)|符号|最小値|最大値|
|:---|:---|:---|:---|:---|
|int8|8|あり|-128|127|
|int16|16|あり|-32768|32767|
|int32|32|あり|-2147483648|2147483647|
|int64|64|あり|-9223372036854775808|9223372036854775807|
|uint8(byte)|8|なし|0|255|
|uint16|16|なし|0|65535|
|uint32|32|なし|0|4294967295|
|uint64|64|なし|0|18446744073709551615|

また、 Go の小数は浮動小数点数の `float32` 、 `float64` 、 `math.BitFloat` が用意されてりるが、 `float` や固定小数点数は存在しない。

### 3.3. ゼロ値

変数に初期値を与えずに宣言すると、ゼロ値( **zero value** )が与えられる。  
ゼロ値は型によって以下のように与えられる。

- 数値型(int,floatなど): 0
- bool型: false
- string型: "" (空文字列( empty string ))

```go
package main

import "fmt"

func main() {
	var i int
	var f float64
	var b bool
	var s string
	fmt.Printf("%v %v %v %q\n", i, f, b, s) // 0 0 false ""
}
```

### 3.4. 型変換

従来の **キャスト** とほぼ同じ。

```go
var i int = 42
var f float64 = float64(i)
var u uint = uint(f)
```

ただし、C言語とは異なり、Goでの型変換は **明示的な変換が必要** 。

### 3.5. 定数型

定数は、 `const` キーワードを使って変数と同じように宣言。  
定数は、文字(character)、文字列(string)、boolean、数値(numeric)のみで使える。  
なお、定数は `:=` を使って宣言できない。

## 4. ループ

### 4.1. 繰り返し for

所謂 `for` ループ。Go に `while` はない。  
他言語とは異なり、 for ステートメントの3つの部分を括る括弧 `( )` はない。なお、中括弧 `{ }` は必要。

```go
package main

import "fmt"

func main() {
	sum := 0
	for i := 0; i < 10; i++ {
		sum += i
	}
	fmt.Println(sum)
}
```

### 4.2. while っぽい for

`for ` はセミコロン( `;` )を省略することもできる。  
つまり、C言語などにある `while` は、Goでは `for` だけを使う。

```go
package main

import "fmt"

func main() {
	sum := 1
	for sum < 1000 {
		sum += sum
	}
	fmt.Println(sum)
}
```

### 4.3. range

Python っぽい。

```go
package main

import "fmt"

var pow = []int{1, 2, 3}

func main() {
	for i, v := range pow { // インデックスと値
		fmt.Printf("%d : %d\n", i, v)
	}
	for i := range pow {   // インデックスだけ
		fmt.Println(i)
	}
	for _, v := range pow { // "_"で捨てて値だけ
		fmt.Println(v)
	}
}
```

## 5. 条件分岐

### 5.1. 条件文 if

Go 言語の `if` ステートメントは、 `for` ループと同様に、括弧 `( )` は不要で、中括弧 `{ }` は必要。  
また、 `if` ステートメントは、 `for` のように、条件の前に、評価するための簡単なステートメントを書くことができる。  
ここで宣言された変数は、 if のスコープ内だけで有効。

```go
package main

import (
	"fmt"
	"math"
)

func pow(x, n, lim float64) float64 {
	if v := math.Pow(x, n); v < lim { // ここ注目
		return v
	}
	return lim
}

func main() {
	fmt.Println(
		pow(3, 2, 10),
		pow(3, 3, 20),
	)
}
```

`if` ステートメントで宣言された変数は、 `else` ブロック内でも使うことができる。

```go
package main

import (
	"fmt"
	"math"
)

func pow(x, n, lim float64) float64 {
	if v := math.Pow(x, n); v < lim {
		return v
	} else {
		fmt.Printf("%g >= %g\n", v, lim) // v を参照できる
	}
	// can't use v here, though
	return lim
}

func main() {
	fmt.Println(
		pow(3, 2, 10),
		pow(3, 3, 20),
	)
}
```

### 5.2. switch 文

Go の `switch` は C や C++、Java、JavaScript、PHP の `switch` と似ているが、 Go では選択された `case` だけを実行してそれに続く全ての `case` は実行されない。   
これらの言語の各 `case` の最後に必要な `break` ステートメントが Go では **自動的に提供される** 。   
もう一つの重要な違いは Go の `switch` の `case` は定数である必要はなく、 関係する値は整数である必要はない。

```go
package main

import (
	"fmt"
	"runtime"
)

func main() {
	fmt.Print("Go runs on ")
	switch os := runtime.GOOS; os {
	case "darwin":
		fmt.Println("OS X.")
	case "linux":
		fmt.Println("Linux.")
	default:
		// freebsd, openbsd,
		// plan9, windows...
		fmt.Printf("%s.", os)
	}
}
```

## 6. 配列とスライス

```go
package main

import "fmt"

func main() {
	var a [2]string // 配列
	a[0] = "Hello"
	a[1] = "World"
	fmt.Println(a[0], a[1]) // Hello World
	fmt.Println(a)          // [Hello World]

	primes := [6]int{2, 3, 5, 7, 11, 13} // 配列の値付き宣言
	fmt.Println(primes)                  // [2 3 5 7 11 13]
	
	var s []int = primes[1:4] // スライス
	fmt.Println(s)            // [3 5 7]
	
	q := []int{2, 3, 5, 7, 11, 13} // スライスの値付き宣言
	fmt.Println(q)                 // [2 3 5 7 11 13]
	fmt.Println(q[1:3])            // Python のスライスっぽくアクセスできる：[3 5]
	
	// 構造体のスライス
	ss := []struct {
		i int
		b bool
	}{
		{2, true},
		{3, false},
		{5, true},
		{7, true},
		{11, false},
		{13, true},
	}
	fmt.Println(ss) // [{2 true} {3 false} {5 true} {7 true} {11 false} {13 true}]
}
```

多次元配列・スライスも作成可能。  
また、スライスは要素の追加（ `s = append(s, 0)` 、複数 `s = append(s, 2, 3, 4)` ）が可能。

## 6.1. make と new

変数の作成に用いる `make` と `new` の違いは以下の通り。

- `make(T)`
  - 対象の型：`slice` 、 `map` 、 `channel`
  - 初期化：初期化する
  - 返り値： `T` （値）
- `new(T)`
  - 対象の型：任意の型
  - 初期化：初期化しない(ゼロ値になる)
  - 返り値： `*T` （ポインタ）

`make` は `list := make([]int, len, cap)` のようにサイズを指定できる。  
`len` は長さでゼロ値が存在する値の場合は実態を作成。 `cap` （省略可）はメモリだけ確保し、各要素の実態作成は行われない。  
なお、 **多次元で作成する場合は注意** が必要。

```go
# 2 次元スライス（2x2）を作成する場合
list := make([][]int, 2)
# 上記で [][]int は作成しても []int は作成されていない！
for i := 0; i < 2; i++ {
	list[i] = make([]int, 2)
}
```

## 7. map

```go
package main

import "fmt"

type vertex struct {
	x, y int
}

func main() {
	var m map[string]vertex     // mapの宣言
	m = make(map[string]vertex) // mapはmakeで作る
	m["hoge"] = vertex{
		1, 2,
	}
	fmt.Println(m["hoge"]) // {1 2}
	
	mm := map[string]vertex{ // 値付きで宣言
		"hoge": {1, 2},
		"fuge": {3, 4},
	}
	fmt.Println(mm["fuge"]) // {3 4}
	
	delete(mm, "fuge")        // 要素の削除
	v, ok := mm["fuge"]       // 要素の存在を確認
	fmt.Println(v, ok)        // {0 0} false
	mm["fuge"] = vertex{3, 4} // 要素の追加
	v, ok = mm["fuge"]
	fmt.Println(v, ok)        // {3 4} true
}
```

## 8. ポインタ

Go では **ポインタ** （値が格納されているメモリのアドレス）を扱える。

```go
package main

import "fmt"

func main() {
	i := 10

	p := &i         // i のポインタ
	fmt.Println(p)  // i が格納されてるメモリのアドレス：0x416020
	fmt.Println(*p) // ポインタ経由で i の値にアクセス：10
	*p = 20         // ポインタ経由で i の値を変更
	fmt.Println(i)  // もちろん i から参照しても値は変わってる：20
}
```

## 9. 構造体

`type` で作成する。  
`type` 自体は `struct` 専用ではなく、独自の型を定義できるもの。

```go
package main

import "fmt"

type vertex struct {
	x int // var 不要
	y int
}

func main() {
	v := vertex{1, 2} // vertex{x:1, y:2} でフィールド明記もできる

	fmt.Println(v)   // {1 2}
	fmt.Println(v.x) // ドットでフィールドアクセス：1
	
	pv := &v           // v のポインタ
	p := &vertex{1, 2} // いきなりポインタで作成も可能
	
	fmt.Println((*pv).y)// 2
	fmt.Println(p.y) // * を省略してもコンパイラが良しなに解釈してくれる：2
}
```

## 10. インターフェース

**インタフェース** は、メソッドのシグニチャの集まりを定義する。  
何も定義していないもの `interface{}` 型は **空のインターフェース** といい、任意の値を保持できる。

```go
package main

import "fmt"

type vertex struct {
	x, y int
}

func (v vertex) add() int {
	return v.x + v.y
}

func (v *vertex) sub() int {
	return v.x - v.y
}

type adder interface {
	add() int
}

type suber interface {
	sub() int
}

func main() {
	var a adder
	v1 := vertex{1, 2}

	a = v1
	fmt.Println(a.add()) // 3
	
	var s suber
	v2 := &vertex{3, 4}
	
	s = v2
	fmt.Println(s.sub()) // -1
	
	s = v1 // コンパイルエラーになる、sub() は vertex のメソッドではなく *vertex のメソッドなので、「s = &v1」ならOK
	fmt.Println(s.sub())
	
	var n interface{}              // 空のインターフェース
	n = v1                         // 空のインターフェースは任意の値を保持できる
	fmt.Printf("(%v, %T)\n", n, n) // ({1 2}, main.vertex)
	n = v2                         // 空のインターフェースは任意の値を保持できる
	fmt.Printf("(%v, %T)\n", n, n) // (&{3 4}, *main.vertex)
}
```

標準ライブラリには [**Stringer** インターフェース](https://go-tour-jp.appspot.com/methods/17)があり、 `String()` メソッドが定義されている。  
例えば、 `fmt` に `String()` メソッドを実装したオブジェクトを渡すと、その定義通りに標準出力してくれる。  
他にも以下の様なものがある。

- [error インターフェース](https://go-tour-jp.appspot.com/methods/19)
- [Reader インターフェース](https://go-tour-jp.appspot.com/methods/21)
- [Image インターフェース](https://go-tour-jp.appspot.com/methods/24)

## 11. 型

### 11.1. 型定義

`type` で作成する。

```go
type 識別子 型
```

「識別子」は，ここで宣言する型の名前、「型」は，型名または型リテラル。（ ex. `type Hex int` ）  
構造体やインターフェースも型定義しているのと同じ。  
型リテラルは以下のような型をリテラルで書いたもの。（リテラル ＝ ソースコードの中に直接書きこんである文字とか数字とかのこと）

```go
// 配列型
[10]int

// 構造体型
struct {}

// ポインタ型
*int

// 関数型
func(s string) int

// インタフェース型
interface {}

// スライス型
[]int

// マップ型
map[string]int

// チャネル型
chan bool
```

### 11.2. メソッド

Go には **class** は無いが、**型に対してメソッドを定義** できる。  
構造体だけにメソッド定義できるのではなく、型に対して定義できることに注意。  
Go ではメソッドを定義する型を **レシーバ** と呼び、 **値レシーバ** と **ポインタレシーバ** がある。  
レシーバは `func` と関数名の間に定義する。

- 値レシーバ
    - `(r Type)`
	- メソッド呼び出しの度に `r` （値）が作成されるのでメモリ多用に注意
	- 基本的にはポインタレシーバを使ってればいい。
- ポインタレシーバ
    - `(r *Type)`
	- 値レシーバと異なり、メソッド呼び出しの度に値は作成されない（ポインタなので）
	- ただし、 `nil` には注意（オブジェクト化されてなくても呼べちゃう）

```go
package main

import (
	"fmt"
)

type user struct {
	name  string
	score int
}

// ポインタレシーバ
func (u *user) hit() {
	u.score++
}

// 参照渡しでないと
func (u user) hit2() {
	u.score++
}

func main() {
	u1 := &user{name: "pepese", score: 100}
	fmt.Println(u1) // &{pepese 100}

	u1.hit()
	fmt.Println(u1.score) // カウントアップされてる：101

	u1.hit2()
	fmt.Println(u1.score) // カウントアップされてない：101 （102になると思った？）
	/*
		hit2 はメソッド実行時に u1 のコピーが作成され、その値に対して処理が行われている。
		なので、u1 のコピーに処理が行われているのであった、u1 に対しては処理は行われていない。
	*/
}
```

### 11.3. 型アサーション

`インターフェース.(型)` で型を確認できる。

```go
package main

import "fmt"

func main() {
	var i interface{} = "hello"

	s := i.(string) // string 型で空のインターフェースから値を取得する
	fmt.Println(s)  // hello

	s, ok := i.(string) // 第2引数で string 型かどうかチェックできる
	fmt.Println(s, ok)  // hello true

	f, ok := i.(float64) // 型が異なる場合は false
	fmt.Println(f, ok)   // 0 false

	f = i.(float64) // 型が異なるのに第2引数が無いと panic する
	fmt.Println(f)
}
```

### 11.4. 型 switch

`インターフェース.(type)` で型が取得でき、型に応じて switch できる。

```go
package main

import "fmt"

func do(i interface{}) {
	switch v := i.(type) {
	case int:
		fmt.Printf("int: %v\n", v)
	case string:
		fmt.Printf("string: %v\n", v)
	default:
		fmt.Printf("unknown type: %T\n", v)
	}
}

func main() {
	do(21)      // int: 21
	do("hello") // string: hello
	do(true)    // unknown type: bool
}
```

## 12. 並列処理

### 12.1. goroutine

**goroutine** （ゴルーチン）は Go ランタイムが管理する軽量スレッド。  
OS が管理するスレッドではなく、 Go ランタイムなのがポイント。  
関数の前に `go` と記載すれば新しい goroutine 上でその関数が実行される。  
main 関数自体も goroutine で実行されており、 `go` キーワードで実行する関数自体はメイン goroutine 上で評価される。

```go
package main

import (
	"fmt"
	"time"
)

func say(s string) {
	fmt.Println(s)
}

func main() {
	// goroutine 実行
	go say("hello")
	
	// 関数であれば実行できるので、即時関数でもよい
	go func(s string) {
		fmt.Println(s)
	}("world")
	
	// 上記の goroutine が実行される前にメイン goroutine が終了するためスリープ
	time.Sleep(1*time.Second)
}
```

### 12.2. チャネル（ channel ）

Go では **チャネル** （ **channel** ）を用いて goroutine 間のデータの送受信およびブロックを実現する。  
チャネルは `make` で作成（ `c := make(chan int)` ）し、送受信するデータの型を指定する。  
また、データの送信（ `c <- 0` 入れる）・受信（ `<-c` 取り出す）はアロー（？）で表現する。 **キュー** みたいなものだ。

```go
package main

import "fmt"

func main() {
	c := make(chan string) // チャネルの作成

	go func(c chan string) {
		c <- "hello world"
	}(c)
	
	fmt.Println(<-c) // チャネルに値が入って読みだせるまでメイン goroutine はブロックされる
}
```

```go
package main

import "fmt"

func add(c chan int, x, y int) {
	c <- x + y
}

func main() {
	c := make(chan int) // チャネルの作成

	nums := [4]int{1, 2, 3, 4}
	go add(c, nums[0], nums[1]) // 足し算のお仕事を goroutine で分割
	go add(c, nums[2], nums[3])
	
	r1, r2 := <-c, <-c
	fmt.Println(r1, r2) // 7 3
	// 終わった方から先に入るので順番に保証は無い
}
```

チャネルには **バッファ** （チャネルに入るデータの数） を定義でき、 **バッファがいっぱいのときはチャネルへの送信をブロック** し、 **バッファが空の時はチャネルの受信をブロック** する。

```go
package main

import "fmt"

func main() {
	ch := make(chan int, 2) // バッファサイズ 2 のチャネル
	ch <- 1
	ch <- 2
	fmt.Println(<-ch)
	fmt.Println(<-ch)
}
```

チャネルは閉じる（ **`close()`** ）ことができ、以下のように検知可能。

- `v, ok := <-ch`
    - `ok` が `false` の場合チャネルが閉じている
- `for i := range c`
    - チャネルが閉じられるまで値を繰り返し受信する

```go
package main

import "fmt"

func count(n int, c chan int) {
	x := 0
	for i := 0; i < n; i++ {
		x += 1
		c <- x
	}
	close(c) // チャネルを閉じる
}

func main() {
	c := make(chan int, 10)
	go count(cap(c), c)
	for i := range c { // チャネルが閉じられるまでループ＆ブロック
		fmt.Print(i, " ")
	} // 1 2 3 4 5 6 7 8 9 10
}
```

**`select`** を使用することで複数のチャネルを評価できる。  
複数ある `case` のいずれかが準備できるようになるまでブロックし、準備ができた `case` を実行する。  
もし、複数の `case` の準備ができている場合、 `case` は **ランダムに選択・実行** される。  
どの `case` も準備できていない場合は `default` が実行される。

```go
package main

import (
	"fmt"
	"time"
)

func count(c, quit chan int) {
	x := 0
	for {
		select { // どちらかの case が for 内で評価され続ける
		case c <- x:
			x += 1
		case <-quit:
			fmt.Println("quit")
			return
		default:
			fmt.Println("wait")
			time.Sleep(1*time.Second)
		}
	}
}

func main() {
	c, quit := make(chan int), make(chan int)
	go func() {
		for i := 0; i < 5; i++ {
			fmt.Print(<-c, " ")
		}
		quit <- 0
	}()
	count(c, quit)
}
// wait
// wait
// 0 wait
// 1 wait
// 2 wait
// 3 wait
// 4 quit
```

### 12.3. sync.Mutex

チャネルは goroutine 間でデータの送受信とブロックを実現するものだが、データ送受信が不要な場合は **sync.Mutex** （排他制御・ミューテックス： mutual exclusion の略）を利用する。  
所謂ロック機構（ `Lock` `Unlock` ）の機能を提供し、 **クリティカルセッション** （他の処理の介入抑止し、データの生合成を守る必要のある一連の一まとまりの処理）を保護する。

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

type counter struct {
	num int
	mux sync.Mutex
}

// クリティカルセッション
func (c *counter) countup(roop int) {
	c.mux.Lock()
	defer c.mux.Unlock() // deferの利用、ほぼクリティカルセッションの構文
	
	for i:=0; i<roop; i++ {
		c.num++
	}
	fmt.Println(c.num)
}

func main() {
	c := counter{num: 0}
	for i := 0; i < 5; i++ {
		go c.countup(10)
	}

	time.Sleep(time.Second)
}
```

上記はロック機構によりきちんと 10 単位でカウントアップ・表示されている。

## 参考

- [Where to Go from here...](https://go-tour-jp.appspot.com/concurrency/11)
    - この先の学習のヒント
- [Go入門](https://www.slideshare.net/takuyaueda967/2016-go)
    - slideshare
- [Mac OS X で Golang に入門してみる](http://blog.amedama.jp/entry/2015/10/06/231038)
    - Go の文法ではなく、標準の構造がよくわかる
- [シュッと golang に入門する話](http://blog.nishimu.land/entry/2015/03/16/032222)
    - 文法がわかる
- [Go言語入門](https://gist.github.com/hayajo/9559874)
- [はじめてのGo言語](http://cuto.unirita.co.jp/gostudy/)