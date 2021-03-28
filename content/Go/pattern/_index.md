---
title: "パターン"
date: 2021-02-09T22:14:00+09:00
draft: false
hide:
- toc
- nextpage
weight: 4
---

Go らしい書き方や処理パターンについて記載する。

1. Go らしいコード
2. エラー処理
3. インターフェース
4. channel
5. その他

<!--more-->

## 1. Go らしいコード

[参考](https://github.com/golang/go/wiki/CodeReviewComments)（ [日本語](https://gist.github.com/knsh14/0507b98c6b62959011ba9e4c310cd15d) ）

### 1.1 静的解析

|ツール|機能|
|:---|:---|
|gofmt, goimports|フォーマッタ|
|go vet, golint|チェッカ, リンタ|
|gocode|コード補完|
|errcheck|エラー処理チェッカ|

VS Code などのエディタ・IDE を利用する場合は保存時などに勝手に実行されるものが多いが、以下のコマンドの実行は意識しておいたほうがいい。

- `go fmt` （ = `gofmt -w -l` ）
    - VS Code ではコード保存時に勝手に実行してくれる
- `go vet ./...`
- `golint ./...`
    - これは VS Code が教えてくれたりもするが、コマンド叩くと一覧表示されるのでコマンドを併用してもいい
- `errcheck ./...`
    - `go get -u github.com/kisielk/errcheck`

### 1.2 コメント

コメントは、記述されているものの名​​前で始まり、ピリオドで終わる必要がある。

```go
// Request represents a request to run a command.
type Request struct { ...

// Encode writes the JSON encoding of req to w.
func Encode(w io.Writer, req *Request) { ...
```

### 1.3. Context

一連オンライントランザクション処理の流れなどで保持しておく必要のある値をコンテキストで管理する。（セキュリティ資格情報、トレース情報、期限、キャンセルのシグナルなど）  
Contextを利用するほとんどの関数で一番最初の引数に指定して引き回す。

```go
func F(ctx context.Context, /* other arguments */) {}
```

### 1.4. スライス

```go
var t []string  // Good

t := []string{} // Bad
```

前者は `nil` スライスを返し、後者は非 `nil` スライスを返却する。どちらも `cap` , `len`ともにゼロだが、前者が優先スタイル。  
しかしJSONオブジェクトをエンコードするときなど理由がある場合は後者も可。

### 1.5. panic は使わない

通常のエラー処理に `painc` を使用せず、エラーと複数の戻り値を使用する。

### 1.6. エラーの文字列

エラー文字列は、通常、他のコンテキストに従って出力されるため、大文字にしない。ただし、固有名詞や頭文字で始まる場合は大文字。

```go
// Bad
fmt.Errorf("Something bad")
// 通常はこのようにフォーマットされて出力される
fmt.Errorf("something bad")
log.Printf("Reading %s: %v", filename, err)
```

### 1.7. エラーハンドリング

関数の戻り値としてエラーが返ってくるときに、絶対に受け取り、処理をする必要がある。本当に例外の時 のみ panicを利用。  
また、エラーが発生したときは先にエラーハンドリングを書く。

```go
// Bad
if x, err := f(); err != nil {
    // error handling
    return
} else {
    // use x
}

// Good
x, err := f()
if err != nil {
    // error handling
    return
}
// use x
```

ただし、エラーのみを返却する関数を実行する場合は「 `err` 変数の受け取りを複数回繰り返していると `:=` だけで書けないという状況」に落ちいってしまうため、以下の記載を許容する。

```go
// Good
if err := f(); err != nil {
    // error handling
    return
}
```

### 1.8. import

名前の衝突を避ける以外の目的でインポートの名前を変更しない。  
衝突を避ける場合には、プロジェクト固有の名前にする。

### 1.9. レシーバのタイプ

- ポインタレシーバを利用する場合
  - メソッドがレシーバを変更する場合
  - レシーバが `sync.Mutex` または同様の同期フィールドを含む構造体である場合（コピーを避けるため）
  - レシーバが大きな構造体または配列の場合
  - レシーバが構造体、配列、またはスライスであり、その要素のいずれかが変化している可能性のある場合
  - その他判断に迷う場合
- 値レシーバを利用する場合
  - レシーバが map、func、または chan の場合
  - レシーバがスライスであり、メソッドがスライスの再スライスまたは再割り当てを行わない場合
  - レシーバが小さな配列または構造体の場合や可変フィールドとポインターがない場合、またはintやstringなどの単純な基本型である場合

### その他参考

- [Effective Go](https://golang.org/doc/effective_go)
  - [日本語1](http://go.shibu.jp/effective_go.html)
  - [日本語2](https://xn--go-hh0g6u.com/doc/effective_go.html)

## 2. エラー処理

- [Golangのエラー処理とpkg/errors](https://deeeet.com/writing/2016/04/25/go-pkg-errors/)

## 3. インターフェース

- [インタフェースの実装パターン #golang](https://qiita.com/tenntenn/items/eac962a49c56b2b15ee8)
- [Goらしいコードを業務でも書くために](https://speakerdeck.com/kender_09/gorasiikodowoye-wu-demoshu-kutameni)

### まとめ

`io.Reader` `io.Writer` `io.Closer`

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}
type Writer interface {
    Write(p []byte) (n int, err error)
}
type Closer interface {
    Close() error
}
type ReadWriter interface { // 埋め込み
    Reader
    Writer
}
type WriteCloser interface { // 埋め込み
    Writer
    Closer
}
```

## 4. channel

- [Go の channel 処理パターン集](https://hori-ryota.com/blog/golang-channel-pattern/)

```go
package main

import (
	"fmt"
	"time"
)

func add(c chan int, a, b int) {
	// 疑似処理時間
	time.Sleep(1*time.Second)
	// 本処理
	fmt.Printf("お仕事%d: %d + %d = %d\n", a/2, a, b, a+b)
	c <- 0
}

func main() {
	c, quit := make(chan int), make(chan int)
	// お仕事の作成
	nums := []int{0,1,2,3,4,5,6,7,8,9}
	// お仕事の分配
	for i := 0; i < 5; i++ {
		go add(c, nums[2*i], nums[2*i+1])
	}
	sum := 0
	for {
		select {
		case <-c:
			sum++
			if sum >= 5 {
				close(quit)
			}
		case <-quit:
			fmt.Println("お仕事おしまい。")
			return
		}
	}
}
```

```zsh
お仕事1: 2 + 3 = 5
お仕事2: 4 + 5 = 9
お仕事4: 8 + 9 = 17
お仕事0: 0 + 1 = 1
お仕事3: 6 + 7 = 13
お仕事おしまい。
```

`sync.WaitGroup` を利用したら以下も同じ結果。

```go
package main

import (
	"fmt"
	"time"
	"sync"
)

func add(wg *sync.WaitGroup, a, b int) {
	defer wg.Done()
	// 疑似処理時間
	time.Sleep(1*time.Second)
	// 本処理
	fmt.Printf("お仕事%d: %d + %d = %d\n", a/2, a, b, a+b)
}

func main() {
	wg := new(sync.WaitGroup)
	quit := make(chan int)
	// お仕事の作成
	nums := []int{0,1,2,3,4,5,6,7,8,9}
	// お仕事の分配
	wg.Add(5)
	for i := 0; i < 5; i++ {
		go add(wg, nums[2*i], nums[2*i+1])
	}
	// お仕事の終わり待ち
    	go func() {
		wg.Wait()
		close(quit)
	}()
	// お仕事終わり検知
	select {
	case <-quit:
		fmt.Println("お仕事おしまい。")
		return
	}
}
```

## 5. その他

### 5.1. 設計単位

```go
package trace

import (
    "fmt"
    "io"
)
// インターフェース
type Tracer interface {
    Trace(...interface{})
}
// 構造体
type tracer struct {
    out io.Writer
}
// メソッド
func (t *tracer) Trace(a ...interface{}) {
    t.out.Write([]byte(fmt.Sprint(a...)))
    t.out.Write([]byte("\n"))
}
// New
func New(w io.Writer) Tracer {
    return &tracer{out: w}
}
```

なお、 `interface` は必須ではない。