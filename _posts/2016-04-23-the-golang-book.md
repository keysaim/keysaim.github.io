---
layout:     post
title:      "The golang programming language"
subtitle:   "Go编程语言读书笔记"
date:       2016-04-23
author:     "keysaim"
header-img: ""
catalog: true
tags:
    - go
    - 读书笔记
---

# Notes
## Section 2, Program Structure
### nested block in if-else if-else block
```go
	if x, y := 100, 200; x > 1000 {
	} else if x := "hello"; y > 0 { //this x shadow the 'x' in if
		fmt.Println(x, y)
	}
```

### scope shaw issue
```go
	var cwd string
     func init() {
         cwd, err := os.Getwd() //compile error: the 'cwd' declared but not used
         if err != nil {
             log.Fatalf("os.Getwd failed: %v", err)
         }
	}
```

'cwd', 'err' are not declared in their block, so the compiler will declare them and will shadow the global 'cwd' variable.

## Section 3, Basic Data Type

### Go types
* basic types: numbers, strings, booleans
* aggregate types: arrays and structs
* reference types: slices, maps, functions, channels. (Each includes pointers that point to internal data)
* interface types 

### Types synonym
* int/uint types' size are platform and compiler dependent. ***Never make assumption for the size!***
* 'rune' is synonym to 'int32', while 'byte' to 'uint8'
* 'uintptr' is for the low level programming, like Go with C

### Operators
* the '%' for negative:

```go
    fmt.Println(-5 % -2) //-1
    fmt.Println(5 % -2) //1
    fmt.Println(-5 % 2) //-1
```

* the result is ***the same type as the operators***, so overflow may happen

```go
	var u uint8 = 255
    fmt.Println(u, u+1, u*u) // "255 0 1"
    var i int8 = 127
    fmt.Println(i, i+1, i*i) // "127 -128 1"
```

* 'unary operators': '+', '-', '^'

	** For integers, '+x' is short for '0 + x' while '-x' is short for '0 - x'
	** For floating and complex numbers, '+x' is just for 'x' while '-x' is the negative of 'x'
	** '^x' return the value with each bit inverted

* 'NaN' is from like '0/0' or 'sqrt(-1)'. Any comparation of 'NaN' always yield false

```go
    nan := math.NaN()
    fmt.Println(nan == nan, nan < nan, nan > nan) // "false false false"
```

* ***Row string literal***: no escape happens and can cross multiple lines

### Conversion and format

#### Printf

```go
	o := 0666
	fmt.Printf("%d %[1]o %#[1]o\n", o) // "438 666 0666"
	x := int64(0xdeadbeef)
	fmt.Printf("%d %[1]x %#[1]x %#[1]X\n", x)
	// Output:
	// 3735928559 deadbeef 0xdeadbeef 0XDEADBEEF
```

* The '[1]' means use the first argument, so no need to provide the same argument again and again

* The '#' is used to add the '0', '0x', '0X'

* If space is after the '%', like '% x', then it will insert the space for each hex digits, like 'e3 83 97 e3 83 ad e3 82 b0 e3 83 a9 e3 83 a0'

* The 'strconv' package includes many format functions

* The '%t' show true or false, '%T' show the type

#### Unicode

* Unicode version 8 use 4 bytes for each charactor, also knows as UTF-32/UCS-4. In Go, the 'rune' is used for this.

* Unicode wasts lots of space, so the 'UTF-8' is invented.

    ```
    00xxxxxx runes0−127 (ASCII)
    111xxxxx 10xxxxxx 128−2047 (values <128 unused)
    1110xxxx 10xxxxxx 10xxxxxx 2048−65535 (values <2048 unused)
    11110xxx 10xxxxxx 10xxxxxx 10xxxxxx 65536−0x10ffff (other values unused)
    ```

* Code point

    Same values:
    ```go
    "世界"
    "\xe4\xb8\x96\xe7\x95\x8c" //the binary using the UTF-8 coding
    "\u4e16\u754c" //the binary using the Unicode, will be encoded as UTF-8 by the compiler
    "\U00004e16\U0000754c" //the binary using the Unicode, will be encoded as UTF-8 by the compiler
    ```
    They are all valid UTF-8 encoding of code point, but in Go, when for 'rune', value below 256 can used as '\x', while above 256, must use '\u' or '\U'.

### enumeratio

```go
const (
    _ = 1 << (10 * iota)
    KiB // 1024
    MiB // 1048576
    GiB // 1073741824
    TiB // 1099511627776 (exceeds 1 << 32)
    PiB // 1125899906842624
    EiB // 1152921504606846976
    ZiB // 1180591620717411303424
    YiB // 1208925819614629174706176
)
```
The 'iota' is 'int' type, so it will overflow.

## Section 4, Composite Types

### About array type

* The size is part of the type, so '[3]int' is different type from '[5]int'

* Can init with specific indices

    ```go
    symbol := [...]string{0: "$", 2: "9", 5: "!", 9: """}
    r := [...]int{99: -1}
    ```
    All the ones not in the indices will be init as the zero value.

* Arrays are comparable, 'equal' when the two arrays have the same type and all the elements are the same

### About the slice type

* Index beyonds the capcity will cause panic, while beyonds length will expend the slice

* 's[i:j]' will share the same underlying array with slice 's'

* Slices are ***not comparable***. For '[]byte], using the 'bytes.Equal' to compare

* Nil slice has zero length and zero capcity, while the reverse is not right.

    ```go
    var s []int    // len(s) == 0, s == nil
    s = nil        // len(s) == 0, s == nil
    s = []int(nil) // len(s) == 0, s == nil
    s = []int{}    // len(s) == 0, s != nil
    ```

* Nil slice and non-nil slice with zero length should be treated in the same way, so using 'len(s) == 0' to check if a slice is empty

* Under the hood, 'make' creates an unnamed array variable and returns a slice of it

* With append, slice may enlarge its space to hold new elements. The copy will handle the overlap of the underlying array.


### About the map type

* Lookup using a key not existing will return the zero value of the value type.

    ```go
    a := make(map[string]int)
    a["bob"] = a["bob"] + 1
    fmt.Println(a["bob"]) //will output 1
    ```

* You can use the '++' to increase the value

    ```go
    a := make(map[string]int)
    a["bob"]++
    fmt.Println(a["bob"]) //will output 1
    ```

* A map element is not variable, you can never try to get its address. ***But on the different, you can get the address of a slice element***

    ```go
    _ = &a["bob"] //compile error
    ```

* Zero value of the map is nil.

    ```go
    var ages map[string]int
    fmt.Println(ages == nil)    // "true"
    fmt.Println(len(ages) == 0) // "true"
    ```

* A nil map supprts operations: lookup, delete, len, range, but not ***store value***.

    ```go
    var aa map[string]int
    aa["bob"] = 100 //panic, the map must be created before storing any value
    ```

* The maps are not comparable except for comparision with nil.

### About the struct type

* Will compile error

    ```go
    func create(id int) Employee {
    //...
    }
    create(100).Name = "bob" //compile error, change the return type to *Employee will be ok
    ```

* Fields not exported in a struct cannot be inited in another package

    ```go
    package p
    type T struct{ a, b int } // a and b are not exported

    ```
    ```go
    package q
    import "p"
    var _ = p.T{a: 1, b: 2} // compile error: can't reference a, b
    var _ = p.T{1, 2}       // compile error: can't reference a, b

    ```
    And this is also wrong:
    ```go
    package p
    type T struct{ A, b, C int } // a and b are not exported

    ```
    ```go
    package q
    import "p"
    var _ = p.T{1, 2}       // compile error: can't reference A, b
    var _ = p.T{A: 1, C: 2} // ok
    ```

* Embedding will form an **anoymous** field of the struct with the implicit field name of the embedded type.

    ```go
    type Point struct {
        X, Y int
    }

    type Circle struct {
        Point // form a anoymous field with implicit field name "Point"
        Radius int 
    }

    type Wheel struct {
        Circle // form a anoymous field with implicit field name "Circle"
        Spokes int 
    }
    ```
    So you cannot embed the same type for more than twice, as their implicit names will conflict.

* The implicit name can be optional for dot expression

    ```go
    w := new(Wheel)
    w.Circle.Radius = 100
    w.Radius = 100 // Both of the two are valid
    ```

* The embedded struct cannot init by normal struct literal

    ```go
    w = Wheel{8, 8, 5, 20}                       // compile error: unknown fields
    w = Wheel{X: 8, Y: 8, Radius: 5, Spokes: 20} // compile error: unknown fields
    ```
    You must init like this:
    ```go
    w = Wheel{Circle{Point{8, 8}, 5}, 20}
    w = Wheel{
      Circle: Circle{
           Point:  Point{X: 8, Y: 8},
      Radius: 5,
      },
      Spokes: 20, // NOTE: trailing comma necessary here (and at Radius)
    }
    fmt.Printf("%#v\n", w)
    // Output:
    // Wheel{Circle:Circle{Point:Point{X:8, Y:8}, Radius:5}, Spokes:20}
    w.X = 42
    fmt.Printf("%#v\n", w)
    // Output:
    // Wheel{Circle:Circle{Point:Point{X:42, Y:8}, Radius:5}, Spokes:20}
    ```

### About the JSON

* ***field tag*** is a string of metadata associated at compile time with the field of a struct. It's a 'key:"value"' pair.

    ```go
    Year  int  `json:"released"`
    Color bool `json:"color,omitempty"`
    ```

* The JSON field tag format: `json:"<json name>,[addition option]"`

* When Marshal, the not exported fields will be ignored; When Unmarshal, the fields of JSON that are not in the data struct will be ignored

* Associating JSON names with Go struct names during Unmarshaling is ***case-insensitive***, so no need to add the JSON field tag for simple field name. But for the Marshal, you must define the JSON field tag, or the capital name will be used.

## Section 4, Functions

### Function definition

* If return only one unamed type, the '()' can be omitted in the return result definition.

* Parameters and named results share the same level of the function outermost block.

### Go function stack

* Go has a variable size function stack, so the recursive is always safe.

### About the defer

* The 'defer' forms a stack and wil be called by stack before the function return.

* Even panic in the function, the 'defer' will be called too.

## Section 6, Methods

### About the receiver

* The compiler will perform implicit '&p' on the variable when the receiver is the pointer.

    ```go
    type Point struct {
        x int
        y int
    }

    func (p *Point) Offset(off int) {
        p.x += off
        p.y += off
    }

    //...
    p := Point{}
    p.Offset(10) // This is valid, as the compiler will perform implicit '&p' on this.
    Point{1, 2}.Offset(100) // compile error, as it's not the variable and cannot be addressed.
    ```

* It's true on these three scenarios:

    1) The type of the variable is the same as the receiver parameter
    2) The type of the variable is T, while the receiver parameter is *T (The compiler will implicitly perform '&p')
    3) The type of the variable is *T, while the receiver parameter is T (The compiler will implicitly perform '*p')

* Bind the method with the receiver and assign to a variable, then you can call the method without the receiver via the variable. This is called ***method value***

    ```go
    p := Point{1, 2}
    q := Point{4, 6}
    distanceFromP := p.Distance
    fmt.Println(distanceFromP(q))
    ```

* Assigning the method to variable directly is called ***method expression***, you can then call it by providing the receiver as the first parameter.

    ```go
    p := Point{1, 2}
    q := Point{4, 6}
    distance := Point.Distance   // method expression
    fmt.Println(distance(p, q))  // "5"
    ```

### About the embedding

* Embedding a type will inherit all its methods. In terms of implementation, the compiler will generate the wrapped methods with the type.

### About the encapsulation

* All fields in the struct is visible to any function or any method in the same package

## Section 7, Interfaces

### About the interface satisfaction

* Though you can use the type T to access `*T` method (the compiler will perform it implicitly), but you cannot use type T to satisfy the `*T` type interface.

### interface values

* An interface value is composed of: dynamic type and dynamic value. The type is the dynamic type of the value.

    ```go
    var w io.Writer // both the type and value are nil
    if w == nil { // this is true
        fmt.Println('is nil')
    }

    var buf *bytes.Buffer // buf is a nil pointer
    w = nil // both the type and value are nil
    w = buf // the type is '*bytes.Buffer' (not nil), while the value is nil
    if w == nil { // this is false, as the type of w is not nil, so the w is not nil
        fmt.Println('is nil')
    }

    ```

* interface value comparation may cause panic

    ```go
    var x interface{} = []int{1, 2, 3}
    fmt.Println(x == x) // panic: comparing uncomparable type []int
    ```
    When comparation, both the type and value must be comparable. Similar risk eixsts when using the interfaces as the map keys.

* To report the dynamic type of an interface value, using the '%T'

    ```go
    var w io.Writer
    fmt.Printf("%T\n", w) // "<nil>"
    w = os.Stdout
    fmt.Printf("%T\n", w) // "*os.File"
    w = new(bytes.Buffer)
    fmt.Printf("%T\n", w) // "*bytes.Buffer"
    ```

### Type assertion

* x.(T), if T is a concrete type, it will check if x's dynamic type is identical to T, if so the results is the dynamic value, or it will panic. If the T is a interface type, it will first check if x's dynamic type satisfies T, if so, a new interface value will be returned with the new dynamic type T and the same dynamic value.

* No matter what type was asserted, if the operand is a nil interface value, the type assertion fails

### Type switch

* take one example, you can assign the x.(type) to a new variable

    ```go
    func sqlQuote(x interface{}) string {
             switch x := x.(type) {
             case nil:
                 return "NULL"
             case int, uint:
                 return fmt.Sprintf("%d", x) // x has type interface{} here.
             case bool:
                 if x {
                     return "TRUE"
                }
                 return "FALSE"
             case string:
                 return sqlQuoteString(x) // (not shown)
             default:
                panic(fmt.Sprintf("unexpected type %T: %v",wxw,wx.i)t)-ebooks.info }
    }
    ```

## Section 8, Goroutines and Channels

### Channels

* The channel is a reference.

* channels are comparable, it's true when both are references to the same channel data structure.

* A channel can be closed, send to a closed channel will cause panic, while receive on a closed channel, it will yield zero value of the channel element type after the channel is empty.

* To test on a closed channel, use like `x, ok := <- c`, to be comvinent, you can use the 'range' to loop the channel until it's drain and closed.

    ```go
    for x := range naturals { // the loop will exit when the channel 'naturals' is drained and closed.
        squares <- x * x
    }
    ```

* It's not necessary to close the channel when you finish with it. Only close it when you want to tell the receive that you will sent anymore data. It's different from the file, you must always close a file after finishing with it.

* Unidirectional channels: the `chan<- int` is send-only channel, the `<-chan int` is the receive-only channel. And the `close` must not be applied on the receive-only channel. It will implicitly convert `chan int` to `chan<- int` and `<-chan int`.

* To get the capcity of the channel, using the `cap(c)`, while the `len(c)` returns the currently number of the buffered elements.

* An typical example for loop paralle

    ```go
    func makeThumbnails6(filenames <-chan string) int64 {
             sizes := make(chan int64)
             var wg sync.WaitGroup // number of working goroutines
             for f := range filenames {
                 wg.Add(1)
                 // worker
                 go func(f string) {
                     defer wg.Done()
                     thumb, err := thumbnail.ImageFile(f)
                     if err != nil {
                         log.Println(err)
                             return
                             }
                     info, _ := os.Stat(thumb) // OK to ignore error
                     sizes <- info.Size()
                 }(f)
             }
             // closer
             go func() {
                 wg.Wait()
                 close(sizes)
             }()
             var total int64
             for size := range sizes {
                 total += size
             }
             return total
         }
    ```

* Using a buffered channel as a semaphore

    ```go
    var sema = make(chan struct{}, 10)
    func dowork() {
        sema <- struct{}{} // acquire the sema
        defer func() { <-sema }() // release the sema
        // do something here
        //...
    }
    ```

* Using select with a closeable channel to cancel the goroutines. Closing the channel is like broadcasting the goroutines to let them exit gracefully.

    ```go
    func main() {
        done := make(chan struct{})
        jc := make(chan string)
        go func() {
            select {
            case <-done:
                return
            case job := <-jc:
                //do something with the job
            }
        }()

        jc<- "hello"
        close(done)
    }
    ```


## Section 9, Concurrency with Shared Variables
### Mutex

* defer will be be executed even the func panic

* The mutex is not ***re-entrant***, which means that it can not be locked recursively.

* Different goroutines may run on different CPU, different statements may be reordered by the modern compiler and CPU when they are not dependent on each other.

    ```go
         var x, y int
         go func() {
             x = 1 // A1
             fmt.Print("y:", y, " ") // A2
         }()
         go func() {
             y = 1                   // B1
             fmt.Print("x:", x, " ") // B2
         }()
    ```
    The result may be "x = 0, y = 0", as the CPU or compiler may reorder the statements; the different goroutines may run on different CPU, and the update may happen on each CPU cache and doesn't sync with main memory on time. The mutex will make sure the right order.

* Duplicate suppression

    ```go
    type entry struct {
             res   result
             ready chan struct{} // closed when res is ready
         }

    func New(f Func) *Memo {
         return &Memo{f: f, cache: make(map[string]*entry)}
    }

    type Memo struct {
        f     Func
        mu    sync.Mutex // guards cache
        cache map[string]*entry
    }

    func (memo *Memo) Get(key string) (value interface{}, err error) {
        memo.mu.Lock()
        e := memo.cache[key]
        if e == nil {
            // This is the first request for this key.
            // This goroutine becomes responsible for computing
            // the value and broadcasting the ready condition.
            e = &entry{ready: make(chan struct{})}
            memo.cache[key] = e
            memo.mu.Unlock()
            e.res.value, e.res.err = memo.f(key)
            close(e.ready) // broadcast ready condition
        } else {
            // This is a repeat request for this key.
            memo.mu.Unlock()
            <-e.ready // wait for ready condition
        }
        return e.res.value, e.res.err
    }
    ```

### Goroutines vs OS threads

* Goroutine has dynamic size of stack, which is up to 1GB while the OS thread is typical 2MB.

* Goroutines schedule implicitly by certain Go language constructs, like time.Sleep, channel block, mutex block, etc, no need to switch kernel context. While the OS threads schedule is invoked every few ms, need to switch kernel context.

* GOMAXPROCS is the default the number of the CPU on the machine.

## Section 10, Packages and The Go Tool
### Go build

* Using flag '-u' in 'go get' will update the current repo to the latest version.

* The 'go build -i' will install the dependent packages and will decrease the compile time next time.

* Cross compile

    ```
    $ GOARCH=386 go build -i gopl.io/ch10/cross // it will install in $GOPATH/pkg/386
    ```

* Files like `net_linux.go` or `asm_amd64.s`, the compiler will compile it based on the env.

* Comment *before* the file package declaration can also control the compiling. `// +build linux darwin` means only compile it for linux and darwin, `// +build ignore` means never compile this file.

### Go doc

* The first sentence is usually a summary that starts with the declared name.

    ```go
    // Fprintf formats according to a format specifier and writes to w.
    // It returns the number of bytes written and any write error encountered.
    func Fprintf(w io.Writer, format string, a ...interface{}) (int, error)
    ```

* Check the doc

    ```
    $ go doc json.encode
    func (dec *Decoder) Decode(v interface{}) error
        Decode reads the next JSON-encoded value from its input and stores
        it in the value pointed to by v.
    ```

* Export the doc to html and serve it

    ```
    $ godoc -http :8000
    ```
    And then, access the doc via http://localhost:8000/pkg

### Internal packages

* package under the 'internal' directory is the internal package

* Can be seen by the packages under the parent of the 'internal' directory

    ```
    net/http
    net/http/internal/chunked
    net/http/httputil
    net/url
    ```
    The 'net/http/internal/chunked' can be seen by 'net/http', 'net/http/httputil', while not seen by 'net/url'

### Go list

* list the packages

* List all the packages under the workspace.

    ```
    $ go list ...
    ```

* List all under path

    ```
    $ go list gopl.io/ch3/...
    ```

* List by matched pattern

    ```
    $ go list ...xml...
    encoding/xml
    gopl.io/ch7/xmlselect
    ```

* `go list -json` to show by json format of the package detail info.

    ```
    $ go list -json hash
    {
        "Dir": "/home/gopher/go/src/hash",
            "ImportPath": "hash",
            "Name": "hash",
            "Doc": "Package hash provides interfaces for hash functions.",
            "Target": "/home/gopher/go/pkg/darwin_amd64/hash.a",
            "Goroot": true,
            "Standard": true,
            "Root": "/home/gopher/go",
            "GoFiles": [
                "hash.go" ],
                "Imports": [
                    "io"
                ],
                "Deps": [
                    "errors",
                "io",
                "runtime",
                "sync",
                "sync/atomic",
                "unsafe"
                ] 
    }
    ```
* '-f ' to customize the output format

    ```
    $ go list -f '{{join .Deps " "}}' strconv
     errors math runtime unicode/utf8 unsafe
    ```
    It lists all the dependencies. Actually, the '-f' is like using the text template to format the output.

## Section 11, Testing

* Within `*_test.go` files, three kinds of functions are treated specially : tests, benchmarks, and examples. Function starts with 'Test' is for tests, 'Benchmark' is for benchmarks, 'Example' is for examples.

### Tests

* '-run' to run specific cases using the regular expression

    ```
    $ go test -v -run="French|Canal"
         === RUN TestFrenchPalindrome
         --- FAIL: TestFrenchPalindrome (0.00s)
             word_test.go:28: IsPalindrome("été") = false
         === RUN TestCanalPalindrome
         --- FAIL: TestCanalPalindrome (0.00s)
             word_test.go:35: IsPalindrome("A man, a plan, a canal: Panama") = false
         FAIL
         exit status 1
         FAIL    gopl.io/ch11/word1  0.014s
    ```

#### External Test Package

* Sometimes, in case of a cycle of dependencies in test files, use the `_test` to declare a external test package. For example, in 'net/url', the test file may declare package like `package url_test`, the compiler will create an external package for this implicitly.

    ```
    $ go list -f={{.GoFiles}} fmt
         [doc.go format.go print.go scan.go]
    $ go list -f={{.TestGoFiles}} fmt
         [export_test.go]
    $ go list -f={{.XTestGoFiles}} fmt
         [fmt_test.go scan_test.go stringer_test.go]
    ```
    The 'XTestGoFiles' is the external test package files.

* In external package test file, you cannot access the private variable or functions in the tested package. In order to do while-box testing in external package test files, using a export file to export the private things. And this file is always called `export_test.go`

    For example, the `export_test.go` file for the 'fmt' package is:
    ```go
    package fmt
    var IsSpace = isSpace
    ```

### Benchmark Testing

* Test and check the memory

    ```
    $ go test -bench=. -benchmem
         PASS
              BenchmarkIsPalindrome    2000000    807 ns/op  128 B/op  1 allocs/op
    ```

### Profiling

* Profiling when testing

    ```
    $ go test -cpuprofile=cpu.out
    $ go test -blockprofile=block.out
    $ go test -memprofile=mem.out
    ```

    And then run the pprof tool
    ```
        $ go test -run=NONE -bench=ClientServerParallelTLS64 \
                 -cpuprofile=cpu.log net/http
         PASS
         BenchmarkClientServerParallelTLS64-8  1000
            3141325 ns/op  143010 B/op  1747 allocs/op
         ok      net/http       3.395s
         $ go tool pprof -text -nodecount=10 ./http.test cpu.log
         2570ms of 3590ms total (71.59%)
         Dropped 129 nodes (cum <= 17.95ms)
         Showing top 10 nodes out of 166 (cum >= 60ms)
             flat  flat%   sum%     cum   cum%
           1730ms 48.19% 48.19%  1750ms 48.75%  crypto/elliptic.p256ReduceDegree
            230ms  6.41% 54.60%   250ms  6.96%  crypto/elliptic.p256Diff
            120ms  3.34% 57.94%   120ms  3.34%  math/big.addMulVVW
            110ms  3.06% 61.00%   110ms  3.06%  syscall.Syscall
             90ms  2.51% 63.51%  1130ms 31.48%  crypto/elliptic.p256Square
             70ms  1.95% 65.46%   120ms  3.34%  runtime.scanobject
             60ms  1.67% 67.13%   830ms 23.12%  crypto/elliptic.p256Mul
             60ms  1.67% 68.80%   190ms  5.29%  math/big.nat.montgomery
             50ms  1.39% 70.19%    50ms  1.39%  crypto/elliptic.p256ReduceCarry
             50ms  1.39% 71.59%    60ms  1.67%  crypto/elliptic.p256Sum
    ```
    The `-nodecount=10` means showing only 10 rows

### Example Testing

* For example test `ExampleFuncName`, the `go doc` will add this example to the function `FuncName` automatically.

## Section 12, Reflection
### reflect, type and value

* `reflect.TypeOf` return the dynamic type of the interface value. The '%T' in `fmt` is using this.

    ```go
    var w io.Writer = os.Stdout
    fmt.Println(reflect.TypeOf(w)) // "*os.File"
    ```

* `reflect.ValueOf` to get the value

    ```go
    v := reflect.ValueOf(3) // a reflect.Value
    fmt.Println(v)          // "3"
    fmt.Printf("%v\n", v)   // "3"
    fmt.Println(v.String()) // NOTE: "<int Value>"
    t := v.Type() // a reflect.Type
    fmt.Println(t.String()) // "int"
    v := reflect.ValueOf(3) // a reflect.Value
    x := v.Interface() // an interface{}
    i := x.(int) // an int 
    fmt.Printf("%d\n", i) // "3"
    ```

* `reflect.Value.Kind` usage. Kind of zero value is `reflect.Invalid`

    ```go
	func formatAtom(v reflect.Value) string {
			 switch v.Kind() {
			 case reflect.Invalid:
				 return "invalid"
			 case reflect.Int, reflect.Int8, reflect.Int16,
				 reflect.Int32, reflect.Int64:
				 return strconv.FormatInt(v.Int(), 10)
			 case reflect.Uint, reflect.Uint8, reflect.Uint16,
				 reflect.Uint32, reflect.Uint64, reflect.Uintptr:
				 return strconv.FormatUint(v.Uint(), 10)
			 // ...floating-point and complex cases omitted for brevity...
			 case reflect.Bool:
				 return strconv.FormatBool(v.Bool())
			 case reflect.String:
				 return strconv.Quote(v.String())
			 case reflect.Chan, reflect.Func, reflect.Ptr, reflect.Slice, reflect.Map:
				 return v.Type().String() + " 0x" +
					 strconv.FormatUint(uint64(v.Pointer()), 16)
			 default: // reflect.Array, reflect.Struct, reflect.Interface
				 return v.Type().String() + " value"
	} }
    ```

* reflect for the Composite Types

    ```go
	func display(path string, v reflect.Value) {
			 switch v.Kind() {
			 case reflect.Invalid:
				 fmt.Printf("%s = invalid\n", path)
			 case reflect.Slice, reflect.Array:
				 for i := 0; i < v.Len(); i++ {
					 display(fmt.Sprintf("%s[%d]", path, i), v.Index(i))
				 }
			 case reflect.Struct:
				 for i := 0; i < v.NumField(); i++ {
					 fieldPath := fmt.Sprintf("%s.%s", path, v.Type().Field(i).Name)
					 display(fieldPath, v.Field(i))
				 }
			 case reflect.Map:
				 for _, key := range v.MapKeys() {
					 display(fmt.Sprintf("%s[%s]", path,
						 formatAtom(key)), v.MapIndex(key))
				 }
			 case reflect.Ptr:
				 if v.IsNil() {
					 fmt.Printf("%s = nil\n", path)
				 } else {
					 display(fmt.Sprintf("(*%s)", path), v.Elem())
				 }
			 case reflect.Interface:
				 if v.IsNil() {
					 fmt.Printf("%s = nil\n", path)
				 } else {
					 fmt.Printf("%s.type = %s\n", path, v.Elem().Type())
					 display(path+".value", v.Elem())
				 }
			 default: // basic types, channels, funcs
				 fmt.Printf("%s = %s\n", path, formatAtom(v))
			 }
	}
    ```

* Set value

    ```go
	x := 1
	rx := reflect.ValueOf(&x).Elem()
	rx.SetInt(2) // OK, x = 2
	rx.Set(reflect.ValueOf(3)) // OK, x = 3
	rx.SetString("hello") // panic: string is not assignable to int
	rx.Set(reflect.ValueOf("hello")) // panic: string is not assignable to int
	var y interface{}
	ry := reflect.ValueOf(&y).Elem()
	ry.SetInt(2) // panic: SetInt called on interface Value
	ry.Set(reflect.ValueOf(3)) // OK, y = int(3)
	ry.SetString("hello") // panic: SetString called on interface Value
	ry.Set(reflect.ValueOf("hello")) // OK, y = "hello"
    ```

* We can use reflect to access the unexport fields of a struct, but you cannot update it, because the reflect will records whether it's exported or not.

    ```go
	stdout := reflect.ValueOf(os.Stdout).Elem() // *os.Stdout, an os.File var
    fmt.Println(stdout.Type())                  // "os.File"
    fd := stdout.FieldByName("fd")
    fmt.Println(fd.Int()) // "1"
    fd.SetInt(2)          // panic: unexported field
    fmt.Println(fd.CanAddr(), fd.CanSet()) // "true false"
    ```

* access the struct tags

    ```go
	// Build map of fields keyed by effective name.
		fields := make(map[string]reflect.Value)
		v := reflect.ValueOf(ptr).Elem() // the struct variable
		for i := 0; i < v.NumField(); i++ {
			fieldInfo := v.Type().Field(i) // a reflect.StructField
			tag := fieldInfo.Tag           // a reflect.StructTag
			name := tag.Get("http")
			if name == "" {
				name = strings.ToLower(fieldInfo.Name)
			}
			fields[name] = v.Field(i)
		}
    ```

* access the methods. Using `reflect.Value.Call` is possible to call the method

    ```go
	// Print prints the method set of the value x.
     func Print(x interface{}) {
         v := reflect.ValueOf(x)
         t := v.Type()
         fmt.Printf("type %s\n", t)
         for i := 0; i < v.NumMethod(); i++ {
             methType := v.Method(i).Type()
             fmt.Printf("func (%s) %s%s\n", t, t.Method(i).Name,
                 strings.TrimPrefix(methType.String(), "func"))
        }
    }
    ```

* ***Reflect is fragile and will cause terrible performance issue is it's in the critical code path.***
