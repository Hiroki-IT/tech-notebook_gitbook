# Go

## Goとは

### 特徴

手続き型言語．構造体と関数を組み合わせて処理を実装する．言語としてオブジェクトという機能を持っていないが，構造体に関数を関連付けることで，擬似的にオブジェクトを表現することもできる．

<br>

### GOPATH

#### ・構造

```
$GOPATH
│
├── bin（ビルドされたバイナリファイル）
│   └── fuga
├── pkg（ビルドされたパッケージ群）
│   └── darwin_amd64
│       └── hoge.go
│
└── src
    ├── dir_a（mainパッケージ）
    │   └── main.go
    └── dir_b（自作パッケージ）
        └── hoge.go
```



<br>

## データ型

### 基本型

#### ・基本型に属するデータ型

各データ型で，値が代入されていない時，初期値が代入されている．

| データ型 | 表記                   | 初期値             |
| -------- | ---------------------- | ------------------ |
| 数値     | ```int```，```float``` | ```0```            |
| 文字列   | ```string```           | ```""```（空文字） |
| 真偽値   | ```boolean```          | ```false```        |

**＊実装例＊**

```go
// 定義（初期値として「0」が割り当てられる）
var number int

// 代入
number = 5
```

#### ・独自の基本型

type宣言を使用して，独自の基本型を定義する．

**＊実装例＊**

int型を元に，Age型を定義する．

```go
type Age int
```

**＊実装例＊**

パッケージの型を元に，MyAppWriterを定義する．

```go
type MyAppWriter io.Writer
```

<br>

### 合成型

#### ・合成型に属するデータ型

各データ型で，値が代入されていない時，初期値が代入されている．

| データ型 | 表記         | 初期値 |
| -------- | ------------ | ------ |
| 構造体   | ```struct``` |        |
| 配列     | ```[n]```    |        |

#### ・構造体

他の言語でいう『データのみを保持するオブジェクト』に相当する．

**＊実装例＊**

構造体を定義し，変数に代入する．

```go
var person struct {
    Name string
}
```

#### ・配列

他の言語でいう『固定長配列』に相当する．

**＊実装例＊**

配列を定義し，変数に代入する．

```go
// 定義と代入を同時に行う．また，型推論と要素数省略を行う．
names := [...]string {"Hiroki", "Gopher"}

// 定義と代入を同時に行う．また，要素数の定義が必要．
var names[2] string = [2]string {"Hiroki", "Gopher"}

// 定義と代入を別々に行う．また，要素数の定義が必要．
var names[2] string
names[0] = "Hiroki"
names[1] = "Gopher"
```

#### ・独自の合成型

type宣言を使用して，独自の構造体を定義する．

**＊実装例＊**

構造体を元に，Person型を定義する．

```go
type Person struct {
    Name string
}
```

<br>

### 参照型

#### ・参照型に属するデータ型

各データ型で，値が代入されていない時，初期値が代入されている．

| データ型 | 表記     | 初期値                   |
| -------- | -------- | ------------------------ |
| ポインタ | ```*```  | ```(nil)```              |
| スライス | ```[]``` | ```<nil>```（要素数は0） |
| マップ   |          |                          |
| チャネル |          |                          |
| 関数     |          |                          |

#### ・ポインタ

メモリアドレスを代入できるデータ型のこと．定義された変数に対して，&（アンパサンド）を宣言すると，メモリアドレスを抽出できる．抽出したメモリアドレス値は，ポインタ型の変数に代入する必要があるが，型推論で記述すればこれを意識しなくてよい．

**＊実装例＊**

```go
package main
import "fmt"

func main(){
    x := "a"
    
    // ポインタ型の変数を定義代入
    var p *string = &x
    // p := &x と同じ
    
    // メモリアドレスを抽出しない場合
    fmt.Printf("%#v\n", x) // "a"
    
    // メモリアドレスを抽出する場合
    fmt.Printf("%#v\n", p) // (*string)(0xc0000841e0)
}
```

#### ・スライス

他の言語でいう『可変長配列』に相当する．

**＊実装例＊**

```go
// 定義と代入を同時に行う．
names := []string{"Hiroki", "Gopher"}

// 定義と代入を同時に行う．また，型推論を行う．
var names []string = []string{"Hiroki", "Gopher"}
```

スライスを使用して，バイト配列を定義できる．

```go
package main
import "fmt"

func main(){
    x := []byte("abc")
    
    fmt.Printf("%#v\n", x) // []byte{0x61, 0x62, 0x63}
}
```

<br>

### インターフェース型

####  ・特徴

Goでは，様々な値をインターフェース型として定義できる．また，メソッドを定義できる．

**＊実装例＊**

様々な値をインターフェースに変換できる．

```go
var x interface{}

x = 1
x = 3.14
x = "Hiroki"
x = [...]uint8[1, 2, 3, 4, 5]
```

なお，インターフェース型データは演算できない．

```go
var x, y interface{}

x,y = 1, 2

// エラーになる．
z := x + y
```

**＊実装例＊**

Animalインターフェースに変換すると，```Eat```メソッド，```Sleep```メソッド，```Mating```メソッド，の実装が強制される．

```go
package main  
import "fmt"

// インターフェースとそのメソッドを定義する．
type Animal interface {
    Eat()
    Sleep()
    Mating()
}

// 構造体に関数を定義する．
type Insect struct {
    Name string
}

type Fish struct {
    Name string
}

type Mammal struct {
    Name string    
}

// 構造体に関数を関連付ける．インターフェースのメソッドの関連付けが強制される．
func (insect Insect) Eat(){
    fmt.Println("雑食")
}

func (insect Insect) Sleep(){
    fmt.Println("眠る")    
}

func (insect Insect) Mating(){
    fmt.Println("単為生殖")       
}

func main() {
    // Animalインターフェース型の変数を定義する．
    var animal Animal
    
    // 構造体の変数を定義する．
    insect := Insect {Name : "Ant"}
    
    // 構造体をインターフェースに変換する．
    animal = insect
    
    // メソッドを実行する．
    animal.Eat()
    animal.Sleep()
    animal.Mating()
}
```

もし，構造体に関連付けられたメソッドに不足があると，エラーが起こる．

```sh
# Eatメソッドを関連付けていない場合
cannot use insect (type Insect) as type Animal in assignment:
Insect does not implement Animal (missing Eat method)
```

<br>

### nil

#### ・ポインタ型の場合

**＊実装例＊**

```go
package main
import "fmt"

func main(){
    
    x := "x"
    
    // ポインタ型の定義のみ
    var p1 *string
    
    // ポインタ型の変数を定義代入
    var p2 *string = &x

    fmt.Printf("%#v\n", p1) // (*string)(nil)
    fmt.Printf("%#v\n", p2) // (*string)(0xc0000841e0)
}
```

####  ・インターフェース型の場合

**＊実装例＊**

```go
package main
import "fmt"

func main(){
    var x interface{}
    
    fmt.Printf("%#v\n", x) // <nil>
}
```

<br>

### 構造体の機能

#### ・初期化

すでに値が代入されている構造体を初期化する場合，いくつか記法がある．その中では，タグ付きリテラルが推奨される．

**＊実装例＊**

まずは，タグ付きリテラル表記．

```go
package main
import "fmt"

type Person struct {
    Name string
}

func main() {
    // タグ付きリテラル表記
    person := Person{Name: "Hiroki"}
    
    fmt.Printf("%#v\n", person.Name) // "Hiroki"
}
```

二つ目に，タグ無しリテラル表記がある．

```go
package main
import "fmt"

type Person struct {
    Name string
}

func main() {
    // タグ無しリテラル表記
    person := Person{"Hiroki"}
    
    fmt.Printf("%#v\n", person.Name) // "Hiroki"
}
```

三つ目に，```new```関数とフィールド代入による初期化がある．```new```関数は，構造体以外のデータ型でも使用できるが，あまり使わない．

```go
package main
import "fmt"

type Person struct {
    Name string
}

func main(){
    // new関数を使用する
    person := new(Person)
    
    // フィールドに代入する
    person.Name = "Hiroki"
    
    fmt.Printf("%#v\n", person.Name) // "Hiroki"
}
```

#### ・JSONとのマッピング

構造体をJSONにパースする時，事前に構造体の各フィールドと，JSONのキー名を，マッピングしておくことができる．

**＊実装例＊**

```go
package main
import (
    "encoding/json"
    "fmt"
    "log"
)

type Person struct {
    Name string `json:"Name"`
}

func main() {
    person := Person{Name: "Hiroki"}
    
    json, err := json.Marshal(person)
    if err != nil {
        log.Println("JSONエンコードに失敗しました。")
    }
 
    // エンコード結果を出力
    fmt.Printf("%#v\n", string(json))// "{\"Name\":\"Hiroki\"}"
}
```

<br>

## 関数

### main関数

#### ・```main```関数

goのエントリポイントとなる．goのプログラムが起動したときに，各パッケージの```init```関数が実行された後，```main```関数が実行される．

**＊実装例＊**

```go
package main
import "fmt"

func main(){
    fmt.Printf("%#v\n", "Hello world!")
}
```

当然，```main```パッケージや```main```関数が無いと，goのプログラムの起動時にエラーが発生する．

```sh
$ go run server.go

go run: cannot run non-main package
```

```sh
$ go run server.go

# command-line-arguments
runtime.main: call to external function main.main
runtime.main: main.main: not defined
runtime.main: undefined: main.main
```

<br>

### 独自関数

#### ・関数とは

構造体に関連付けられていない関数のこと．

**＊実装例＊**

```go
package main
import "fmt"

func example(x string) string {
    fmt.Println(x)
}

func main(){
    example("Hello world!")
}
```

#### ・Closure（無名関数）とは

名前のない関数のこと．

#### ・即時関数とは

定義したその場でコールされる無名関数のこと．

**＊実装例＊**

main関数で即時関数を実行する．

```go 
package main
import "fmt"

func main() {
    result := func() string {
        return "Closure is working!"
    }()
    
    fmt.Printf("%#v\n", result)
}
```

**＊実装例＊**

即時関数に引数を設定できる．その場合，仮引数と引数の両方を設定する必要がある．

```go
package main
import "fmt"

func main() {
    // 仮引数を設定
    result := func(x string) string {
        
        return x
        
    // 引数に値を渡す
    }("Closure is working!")
    
    fmt.Printf("%#v\n", result)
}
```

<br>

### メソッド

#### ・メソッドとは

データ型や型リテラルに関連付けられている関数のこと．

#### ・レシーバによる関連付け

データ型や型リテラルなどを関数のレシーバとして渡すことによって，それに関数を関連づけられる．関連付け後，関数はメソッドと呼ばれるようになる．

**＊実装例＊**

int型を値レシーバとして渡し，構造体に関数を関連付ける．

```go
package main
import "fmt"

type Age int

func (age Age) printAge() string {
    return fmt.Sprintf("%dです．", age)
}

func main() {
    var age Age = 20
    
    fmt.Printf("%#v\n", age.printAge())
}
```

**＊実装例＊**

構造体を値レシーバとして渡し，構造体に関数を関連付ける．

```go
package main
import "fmt"

// 構造体を定義
type Person struct {
    Name string
}

// 構造体に関数を関連付ける．
func (person Person) GetName() string {
    return person.Name
}

// 構造体から関数をコール
func main() {
    // 構造体を初期化
    person := Person{Name: "Hiroki"}
    
    fmt.Printf("%#v\n", person.GetName()) // "Hiroki"
}
```

#### ・値レシーバ

関連付け後，関数はメソッドと呼ばれるようになる．レシーバとして渡された引数をメソッド内でコピーしてから使用する．値レシーバによって関連付けられると，そのメソッドは構造体の状態を変えられなくなるので，構造体をイミュータブルにしたい場合は，値レシーバを使うと良い．

**＊実装例＊**

構造体を値レシーバとして渡し，構造体に関数を関連付ける．

```go
package main
import "fmt"

type Person struct {
	Name string
}

// 値レシーバ
func (person Person) SetName(name string) {
    // 引数の構造体をコピーしてから使用
	person.Name = name
}

func (person Person) GetName() string {
    return person.Name
}

func main() {
	person := Person{Name: "Gopher"}

	person.SetName("Hiroki")
    
    fmt.Printf("%#v\n", person.GetName()) // "Gopher"
}
```

#### ・ポインタレシーバ

関連付け後，関数はメソッドと呼ばれるようになる．レシーバとして渡された引数をメソッド内でそのまま使用する．ポインタレシーバによって関連付けられると，そのメソッドは構造体の状態を変えられるようになるので，構造体をミュータブルにしたい場合は，ポインタレシーバを使うと良い．

**＊実装例＊**

```go
package main
import "fmt"

type Person struct {
	Name string
}

// ポインタレシーバ
func (person *Person) SetName(name string) {
    // 引数の構造体をそのまま使用
	person.Name = name
}

func (person *Person) GetName() string {
    return person.Name
}

func main() {
	person := Person{Name: "Gopher"}

	person.SetName("Hiroki")
    
	fmt.Printf("%#v\n", person.GetName()) // "Hiroki"
}
```

<br>

## 変数

### 定義

#### ・明示的な定義

**＊実装例＊**

```go
// 一つの変数を定義（宣言と代入が同時でない）
var number int
number = 5

// 一つの変数を定義（宣言と代入が同時）
var number int = 5

// 複数の変数を定義
var x, y, z int
x, y, z = 1, 3, 5
```

#### ・暗黙的な定義（型推論）

**＊実装例＊**

```go
// データ型が自動的に認識される
w := 1
x := true
y := 3.14
z := "abc"

var w = 1

var (
    x = true
    y = 3.14
    z = "abc"
)
```

<br>

### 変数の破棄

#### ・アンダースコア

関数から複数の値が返却される時，使わない値をアンダースコアに代入することで，これを破棄できる．

```go
package main
import (
    "fmt"
    "os"
)

func main() {
    // errorインターフェースを破棄
    file, _ := os.Open("filename.ext")
    
    // エラーキャッチする必要がなくなる
    fmt.Printf("%#v\n", flle)
}
```

<br>

## エラーキャッチ，エラー返却，ロギング

### エラーキャッチとエラー返却

#### ・例外スローのある言語の場合

例外スローの意義は，以下の参考にせよ．

参考：https://hiroki-it.github.io/tech-notebook_gitbook/public/backend_logic_validation.html

#### ・Goには例外スローが無い

エラーをキャッチした場合に，例外をスローするべきであるが，Goには例外が無い．

<br>

### 返却されるエラー

#### ・標準エラー

Goでは複数の値を返却できるため，多くの関数では標準で，最後にerrorインターフェースが返却されるようになっている．これから，エラーメッセージを取り出せる．

```go
type error interface {
    Error() string
}
```

**＊実装例＊**

osパッケージの```Open```メソッドからerrorインターフェースが返却される．これから，エラーメッセージを取り出し，標準エラー出力に出力する．

```go
package main
import (
    "fmt"
    "log"
    "os"
)

func main() {
    // 処理結果とerrorインターフェースが返却される．
    file, err := os.Open("filename.ext")
    
    if err != nil {
        // エラーの内容を出力する．
        log.Printf("ERROR: %#v\n", err)
    }
    
    fmt.Printf("%#v\n", flle)
}
```

#### ・独自エラー

errorsパッケージの```New```メソッドにエラーメッセージを設定し，エラーをキャッチできた時に，設定したエラーを返却するようにする．

参考：https://golang.org/pkg/errors/#New

**＊実装例＊**

```go
package main
import (
    "errors"
    "fmt"
    "log"
    "os"
)

func throwErrorsNew() error {
    return errors.New("<エラーメッセージ>")
}

func main() {
    file, err := os.Open("filename.ext")
    
    if err != nil {
        // 独自エラーメッセージを設定する．
        myErr := throwErrorsNew()
        log.Printf("ERROR: %#v\n", myErr)
    }
    
    fmt.Printf("%#v\n", flle)
}
```

他には，```Errorf```メソッドでも独自エラーを作成できる．事前に定義したフォーマットを元にエラーメッセージを設定する．

参考：https://golang.org/pkg/fmt/#Errorf

**＊実装例＊**

```go
package main
import (
    "errors"
    "fmt"
    "log"
    "os"
)

func throwErrorf() error {
    return fmt.Errorf("%s %s", x, y)
}

func main() {
    file, err := os.Open("filename.ext")
    
    if err != nil {
        // 独自エラーメッセージを設定する．
        myErr := throwErrorf()
        log.Printf("ERROR: %#v\n", myErr)
    }
    
    fmt.Printf("%#v\n", flle)
}
```

<br>

### エラーキャッチ

#### ・nilの比較検証

関数から返却されたerrインターフェースが，```nil```出なかった場合に，エラーであると見なすようにする．

```go
if err != nil {
    // 何らかの処理
}
```

<br>

### ロギング

#### ・logパッケージ

Goには標準で，ロギング用パッケージが用意されている．ただし，機能が乏しいので，外部パッケージ（例：logrus）も推奨である．

参考：

- https://pkg.go.dev/log
- https://github.com/sirupsen/logrus

#### ・接尾辞```Print```メソッド

引数に渡されたエラーを標準出力に出力する．

```go
if err != nil {
    log.Printf("ERROR: %#v\n", err)
}
```

#### ・接尾辞```Fatal```メソッド

引数に渡されたエラーを標準出力に出力し，```os.Exit(1)```を実行して処理を停止する．

```go
if err != nil {
    // 内部でos.Exit(1)を実行する．
    log.Fatalf("ERROR: %#v\n", err)
}
```

#### ・接尾辞```Panic```メソッド

引数に渡されたエラーを標準出力に出力し，予期せぬエラーが起きたと見なして```panic```メソッドを実行する．ちなみに，```panic```メソッドによって，エラーメッセージ出力，スタックトレース出力，処理停止が行われる．

```go
if err != nil {
    // panicメソッドを実行する．
    log.Panicf("ERROR: %#v\n", err)
}
```

<br>

## 標準パッケージ

### パッケージのソースコード

参考：https://golang.org/pkg/

<br>

### encoding/json

#### ・Marshal

構造体をJSONに変換する．変換前に，マッピングを行うようにする．

参考：https://golang.org/pkg/encoding/json/#Marshal

**＊実装例＊**

```go
package main
import (
    "encoding/json"
    "fmt"
    "log"
)

type Person struct {
    Name string `json:"Name"`
}

func main() {
    person := Person{Name: "Hiroki"}
    
    json, err := json.Marshal(person)
    
    if err != nil {
        log.Println("JSONへの変換に失敗しました．")
    }
 
    // エンコード結果を出力
    fmt.Printf("%#v\n", string(json)) // "{\"Name\":\"Hiroki\"}"
}
```

#### ・Unmarshal

JSONを構造体に変換する．

参考：https://golang.org/pkg/encoding/json/#Unmarshal

**＊実装例＊**

```go
package main
 
import (
	"encoding/json"
	"fmt"
	"log"
)
 
type Person struct {
	Name string
}
 
func main() {
    byte := []byte(`{"name":"Hiroki"}`)
    
    var person Person
    
    err := json.Unmarshal(byte, &person)
    
	if err != nil {
		log.Println("構造体への変換に失敗しました．")
	}
    
	fmt.Printf("%#v\n", person) // main.Person{Name:"Hiroki"}
}
```

<br>

### bytes

#### ・```Buffer```メソッド

渡された文字列を結合し，標準出力に出力する．

**＊実装例＊**

```go
package main
import (
    "bytes"
    "fmt"
)

func main() {
    var buffer bytes.Buffer

    buffer.WriteString("Hello ")
    
    buffer.WriteString("world!")

    fmt.Printf("%#v\n", buffer.String()) // "Hello world!"
}
```

<br>

### fmt

#### ・接頭接尾辞無しメソッド

接頭接尾辞の無いメソッド（```Print```メソッド，```Sprint```メソッド，```Fprint```メソッド，など）が属する．複数の引数をスペースを挟んで繋ぐ．

参考：

- https://golang.org/pkg/fmt/#Print
- https://golang.org/pkg/fmt/#Fprint
- https://golang.org/pkg/fmt/#Sprint

**＊実装例＊**

```go
package main
import "fmt"

func main() {
    fmt.Print("Hello world!") // Hello world! 
}
```

**＊実装例＊**

```go
package main
import "fmt"

func main() {
    
    // 複数の引数をスペースで挟んで繋ぐ
    fmt.Print(1, 2, 3) // 1 2 3
}
```

ただし，引数のいずれかが文字列の値の場合，スペースが挿入されない．

```go
package main
import "fmt"

func main() {
    // いずれかが文字列
    fmt.Print("Hello", "world!", 12345) // Helloworld!12345
}
```

また，連続で使用しても，改行が挿入されない．

```go
package main
import "fmt"

func main() {
    fmt.Print("Hello", "world!")
    fmt.Print("Hello", "world!")
    
    // Hello world!Hello world!
}
```

#### ・接頭辞```S```メソッド

接頭辞に```S```のあるメソッド（```Sprint```メソッド，```Sprintf```メソッド，```Sprintln```メソッド，など）が属する．処理結果を標準出力に出力せずに返却する．標準出力に出力できる他の関数の引数として渡す必要がある．

参考：

- https://golang.org/pkg/fmt/#Sprint
- https://golang.org/pkg/fmt/#Sprintf
- https://golang.org/pkg/fmt/#Sprintln

**＊実装例＊**

```go
package main
import "fmt"

func main() {
    
    // Sprintは返却するだけ
    fmt.Print(fmt.Sprint(1, 2, 3)) // 1 2 3
}
```

#### ・接尾辞```ln```メソッド

接尾辞に```ln```のあるメソッド（```Println```メソッド，```Fprintln```メソッド，```Sprintln```メソッド，など）が属する．複数の引数をスペースを挟んで繋ぎ，最後に改行を挿入して結合する．

参考：

- https://golang.org/pkg/fmt/#Println
- https://golang.org/pkg/fmt/#Fprintln
- https://golang.org/pkg/fmt/#Sprintln

**＊実装例＊**

文字を連続で標準出力に出力する．

```go
package main
import "fmt"

func main() {
    fmt.Println("Hello", "world!")
    fmt.Println("Hello", "world!")
    
    // Hello world!
    // Hello world!
}
```

#### ・接尾辞```f```メソッド

渡された引数を，事前に定義したフォーマットにも基づいて結合する．

| よく使う識別子 | 標準出力に出力されるもの     | 備考                                                 |
| -------------- | ---------------------------- | ---------------------------------------------------- |
| ```%s```       | 文字列またはスライスとして   |                                                      |
| ```%p```       | ポインタとして               |                                                      |
| ```%+v```      | フィールドを含む構造体として | データの構造を確認できるため，デバッグに有効である． |
| ```%#v```      | Go構文として                 | データの構造を確認できるため，デバッグに有効である． |

参考：

- https://golang.org/pkg/fmt/#Printf
- https://golang.org/pkg/fmt/#Fprintf
- https://golang.org/pkg/fmt/#Sprintf

**＊実装例＊**

渡された引数を文字列として結合する

```go
package main
import "fmt"

func main() {
    fmt.Printf("String is %s", "Hello world!")
}
```

また，連続して使用しても，改行は挿入されない．

```go
package main
import "fmt"

func main() {
    fmt.Printf("String is %s", "Hello world!")
    fmt.Printf("String is %s", "Hello world!")
    
    // String is Hello world!String is Hello world!
}
```

**＊実装例＊**

渡された引数をポインタとして結合する．

```go
package main

import "fmt"

type Person struct {
    Name     string
}

func main() {
    person:= new(Person)
    
    person.Name = "Hiroki"
    
    fmt.Printf("Pointer is %p", person) // 0xc0000821e0
}
```

**＊実装例＊**

渡された複数の引数を文字列として結合する．

```go
package main
import "fmt"

func main() {
    var first string = "Hiroki"
    
    var last string = "Hasegawa"
    
    fmt.Printf("I'm %s %s", first, last)// I'm Hiroki Hasegawa
}
```

<br>

### strings

#### ・```Builder```メソッド

渡された文字列を結合し，標準出力に出力する．

**＊実装例＊**

```go
package main
import (
    "fmt"
    "strings"
)

func main() {
    var builder strings.Builder
    
    builder.WriteString("Hello ")
    
    builder.WriteString("world!")
    
    fmt.Println(builder.String()) // Hello world! 
}
```

