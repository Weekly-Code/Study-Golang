1. 구조체
- 메소드
- 인터페이스
- 고루틴
- 채널
- 패키지
- 문서화
- 테스트

===

## 구조체

```go
// struct 정의
type person struct {
    name string
    age  int
}

// 객체 생성 1
p1 := person{}  // 0, 0.0, "", nil 등으로 초기화

// 객채 생성 2
p2 := person{"Bob", 20}

// 객채 생성 3
p3 := person{name: "Sean", age: 50}

// 객채 생성 4
p4 := new(person)
p4.name = "Lee"  // p가 포인터라도 . 을 사용한다 -> 아님.
```

===

## 메소드

``` go
//Rect - struct 정의
type Rect struct {
    width, height int
}
 
//Rect의 area() 메소드
func (r Rect) area() int {
    return r.width * r.height   
}

// 포인터 Receiver
func (r *Rect) area2() int {
    r.width++
    return r.width * r.height
}
```

---
Embedding / Promoting

```go
type Person struct {
    Name string
}

func (p *Person) Talk() {
    fmt.Println("안녕, 내 이름은", p.Name, "야")
}

type Android struct {
    Person  // 익명 필드
    Model string
}

a := new(Android)  // a := Android{}
a.Name = "철수"  // a.Person.Name = "철수"
a.Talk()  // a.Person.Talk()
```

===

## 인터페이스

```go
// 인터페이스 선언
type Shape interface {
    area() float64
    perimeter() float64
}

// 인터페이스 구현
type Rect struct {
    width, height float64
}
 
func (r Rect) area() float64 { return r.width * r.height }
func (r Rect) perimeter() float64 {
     return 2 * (r.width + r.height)
}
 
var shape Shape = Rect{1, 2}
fmt.Println(shape.area())
```

---

### empty interface

```go
func main() {
    var x interface{}
    x = 1 
    x = "Tom"
 
    printIt(x)
}
 
func printIt(v interface{}) {
    fmt.Println(v) //Tom
}
```

---

### type assertion

```go
// type assertion
func main() {
    var a interface{} = 1
 
    i := a       // a와 i 는 dynamic type, 값은 1
    j := a.(int) // j는 int 타입, 값은 1
 
    println(i)  // 포인터주소 출력
    println(j)  // 1 출력

    if _, ok := a.(int); ok {
        fmt.Println("valid integer")
    }
}
```

---

### type switch

```go
switch f := arg.(type) {
	case bool:
		p.fmtBool(f, verb)
	case float32:
		p.fmtFloat32(f, verb)
	case float64:
		p.fmtFloat64(f, verb)
	case complex64:
		p.fmtComplex64(f, verb)
	case complex128:
		p.fmtComplex128(f, verb)
	case int:
		p.fmtInt64(int64(f), verb)
	case int8:
		p.fmtInt64(int64(f), verb)
	...
	case string:
		p.fmtString(f, verb)
		wasString = verb == 's' || verb == 'v'
	case []byte:
		p.fmtBytes(f, verb, nil, depth)
		wasString = verb == 's'
	case reflect.Value:
		return p.printReflectValue(f, verb, depth)
	default:
		// If the type is not simple, it might have methods.
		if handled := p.handleMethods(verb, depth); handled {
			return false
		}
		// Need to use reflection
		return p.printReflectValue(reflect.ValueOf(arg), verb, depth)
	}
	```

---

### go로 oop하기

- https://medium.com/behancetech/oop-and-go-sorta-c6682359a41b#.f7x7j0ksb
  - Encapsulation: Packages
  - Inheritance: Composition
  - Polymorphism: Interface
  - Abstraction: Embedding
  - [슬라이드](https://docs.google.com/presentation/d/1kzPtqk9ZlVo_harFEasa3TKoJzr8DWUDcoRElV6xAYg)

===

## Go루틴

```go
func say(s string) {
    for i := 0; i < 10; i++ {
        fmt.Println(s, "***", i)
    }
}
 
func main() {
    // 함수를 동기적으로 실행
    say("Sync")
 
    // 함수를 비동기적으로 실행
    go say("Async1")
    go say("Async2")
    go say("Async3")
 
    // 3초 대기
    time.Sleep(time.Second * 3)
}
```
---

### Concurrency is not parallelism

 ```go
import (
    "runtime"  
)
 
func main() {
    // 4개의 CPU 사용
    runtime.GOMAXPROCS(4)
 
    //...
}
```

===
## 채널

1. 데이터를 주고 받는 통로
1. 데이터를 쓸 수 있거나 읽을 수 있을 때까지 대기함
  - synchronous
  - lock을 사용할 필요가 없어짐

```go
package main
 
import "fmt"
 
func main() {
    done := make(chan bool)
    go func() {
        for i := 0; i < 10; i++ {
            fmt.Println(i)
        }
        done <- true
    }()
 
    // 위의 Go루틴이 끝날 때까지 대기
    <-done
}
```

---

### Buffered Channel

```go
package main
 
import "fmt"
 
func main() {
  c := make(chan int)
  c <- 1   //수신루틴이 없으므로 데드락 
  fmt.Println(<-c) //코멘트해도 데르락
}
```

```go
package main
 
import "fmt"
 
func main() {
    ch := make(chan int, 1)
 
    //수신자가 없더라도 보낼 수 있다.
    ch <- 101
 
    fmt.Println(<-ch)
}
```
---
### 채널 파라미터

```go
package main
 
import "fmt"
 
func main() {
    ch := make(chan string, 1)
    sendChan(ch)
    receiveChan(ch)
}
 
func sendChan(ch chan<- string) {
    ch <- "Data"
    // x := <-ch // 에러발생
}
 
func receiveChan(ch <-chan string) {
    data := <-ch
    fmt.Println(data)
}
```

---

### 채널 닫기

1. 채널을 닫으면 송신 불가 (수신은 가능)
- <- 는 2개의 값을 리턴: 메시지, 수신성공

```go
func main() {
    ch := make(chan int, 2)
     
    // 채널에 송신
    ch <- 1
    ch <- 2
     
    // 채널을 닫는다
    close(ch)
 
    // 채널 수신
    println(<-ch)
    println(<-ch)
     
    if _, success = <-ch; !success {
        println("더이상 데이타 없음.")
    }
}
```

---
### 채널 range 문

```go
func main() {
    ch := make(chan int, 2)
 
    // 채널에 송신
    ch <- 1
    ch <- 2
 
    // 채널을 닫는다
    close(ch)

    for i := range ch {
        println(i)
    }
}
```

---

### 채널 select 문

1. 복수 채널을 기다리면서 먼저 데이터를 보내온(준비된) 채널을 실행
- 준비될때까지 대기
  - default문이 있으면 대기하지 않음
- 동시에 도착하면 랜덤하게 선택됨

```go
select {
    case msg1 := <- c1:
        fmt.Println("Message 1", msg1)
    case msg2 := <- c2:
        fmt.Println("Message 2", msg2)
    case <- time.After(time.Second):
        fmt.Println("timeout")
}
```

---
### 연습문제
time.After를 이용해 Sleep 함수를 직접 작성하라.

```go
func main() {
	for i := 1; i <= 10; i++ {
		sleep(1000 * 1)
		fmt.Println(i, "Mississippi")
	}
}

func sleep(amt time.Duration) {
	<-time.After(time.Millisecond * amt)
}
```

---

### Go Concurrency Patterns
- [Go Concurrency Patterns: Pipelines and cancellation](https://blog.golang.org/pipelines)
- [Go Concurrency Patterns: Context](https://blog.golang.org/context)
- [Go Concurrency Patterns: Timing out, moving on](https://blog.golang.org/go-concurrency-patterns-timing-out-and)
- [Advanced Go Concurrency Patterns](https://blog.golang.org/advanced-go-concurrency-patterns)
===
## 패키지

1. main 패키지
  1. 패키지명이 main인 경우 공유 라이브러리 대신 executable을 생성
  - main() 함수가 entry point가 됨
-  패키지 Import
  1. 표준 패키지는 GOROOT/pkg 에서
  - 사용자 패키지나 3rd Party 패키지의 경우 GOPATH/pkg에서 찾음
  - vendoring
- 패키지 Scope
  1. 대문자는 Public, 소문자는 private

---

### init / alias

1. 패키지내 init() 함수는 import시 자동 실행됨
- init만 실행시키고 싶을 때
```go
import _ "other/xlib" 
```
- package이름이 동일한 경우
```go 
import (
  mongo "other/mongo/db"
  mysql "other/mysql/db"
)
func main() {
  mondb := mongo.Get()
  mydb := mysql.Get()
  //...
}```

---

### 사용자 정의 패키지 생성

```go
// GOPATH/src/github.com/weekly-code/syntax/custompackage/package.go
package custompackage

// ...
```
1. 폴더명과 패키지명은 동일해야함 (*달라도 되네?*)
- 하나의 폴더엔 하나의 패키지만
- go install을 실행하면 /pkg에 .a파일 생성 (main이면 /bin에)

---
### popular packages
[awesome go](https://github.com/avelino/awesome-go)

=== 
## 문서화
```go
// golang-book/chapter11/math/math.go

package math

// 인자로 전달된 숫자의 평균을 구한다
func Average(xs []float64) float64 {}
```

```bash
godoc golang-book/chapter11/math Average

use 'godoc cmd/golang-book/chapter11/math' for documentation on the golang-book/chapter11/math command

func Average(xs []float64) float64
    인자로 전달된 숫자의 평균을 구한다
```

```bash
godoc -http=":6060"
open http://localhost:6060/pkg/golang-book/chapter11/math/
```


===
## 테스트

```go
// math_test.go

package math
 
import "testing"
 
type testpair struct {
    values []float64
    average float64
}
 
var tests = []testpair{
    { []float64{1,2}, 1.5 },
    { []float64{1,1,1,1,1,1}, 1 },
    { []float64{-1,1}, 0 },
}
 
func TestAverage(t *testing.T) {
    for _, pair := range tests {
        v := Average(pair.values)
        if v != pair.average {
            t.Error(
                "For", pair.values,
                "expected", pair.average,
                "got", v,
            )
        }
    }
}
```

---

## examples

```go
//  math_test.go

func ExampleAverage() {
    average := Average([]float64{1, 2, 3})
    fmt.Println(average)
    // Output: 2
}
```

---

## Assertion

- 그런거 없다
- [FAQ](https://golang.org/doc/faq#assertions)
  - Q: Why does Go not have assertions?
  - A: proper error handling을 하지않고 assertion을 남용하는 경향이 있음
- https://github.com/stretchr/testify
