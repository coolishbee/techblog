
## 문법 25개 키워드
### var
변수 선언
```go
func main() {
	var x int
	println(x)
	x = 2
	println(x)
	var a, b, c int = 1, 2, 3
	println(a, b, c)
}

//OUTPUT
0
2
1 2 3
```

### const
상수 선언
```
func main() {
	const x int = 1
	println(x)
	const (
		a = 2
		b = 3
		c = 4
	)
	println(a + b + c)
}
//OUTPUT
1
9
```

### for
반복문
```go
func main() {
	sum := 0
	for i := 1; i <= 10; i++ {
		sum += i
	}
	println("sum :", sum)

	i, sum2 := 0, 0
	for i < 10 {
		i++
		sum2 += i
	}
	println("sum2 :", sum2)
}

//OUTPUT
sum : 55
sum2 : 55
```

### range
컬렉션에서 각 요소의 인덱스와 값을 반환
```go
func main() {
	numbers := []int{1, 2, 3}

	for index, num := range numbers {
		println(index, num)
	}
}

//OUTPUT
0 1
1 2
2 3
```

### break
`for`, `switch`, `select` 에서 빠져나올 때 사용
```go
func main() {
	i := 0
	for i < 10 {
		i++
		if i == 5 {
			break
		}
	}

	println("i :", i)
}

//OUTPUT
i : 5
```

### continue
`for` 루프 시작 부분으로 이동
```go
func main() {
	i := 0
	for i < 5 {
		i++
		if i == 3 {
			continue
		}
		println("i :", i)
	}
}

//OUTPUT
i : 1
i : 2
i : 4
i : 5
```

### switch
여러 값을 비교하는 조건문 표현
```go
func main() {
	num := 5
	switch {
	case num == 10:
		println("num is 10")
	case num == 8:
		println("num is 8")
	case num == 5:
		println("num is 5")
	default:
		println("num is less than 5")
	}
}

//OUTPUT
num is 5
```

### case
조건문을 작성
```go
func main() {
	num := 1
	switch num{
	case 1, 2:
		println("num is 1 or 2")
	case 3, 4:
		println("num is 3 or 4")
	default:
		println("num is more than 5")
	}
}

//OUTPUT
num is 1 or 2
```

### default
모든 `case`에 부합하지 않을 때 실행
```go
func main() {
	num := 3
	switch {
	case num == 10:
		println("num is 10")
	case num == 8:
		println("num is 8")
	case num == 5:
		println("num is 5")
	default:
		println("num is less than 5")
	}
}

//OUTPUT
num is less than 5
```

### fallthrough
`case`를 만족해도 아래의 `case` 들을 실행하기 위해 fallthrough 를 사용
> go 컴파일러가 자동으로 `break` 문을 각 `case`문 블럭 마지막에 추가하므로, 조건문이 해당하는 경우 해당 `case`에서 `switch` 문이 종료된다.

```go
func main() {
	num := 1
	switch {
	case num < 10:
		println("num is less than 10")
		fallthrough
	case num < 8:
		println("num is less than 8")
		fallthrough
	default:
		println("num is less than 5")
	}
}

//OUTPUT
num is less than 10
num is less than 8
num is less than 5
```

### if
조건문
```go
func main() {
	a := 1
	if a == 0 {
		println("a is 0")
	}
	println("a is", a)
}

//OUTPUT
a is 1
```

### else
`if` 조건식이 모두 거짓일 때 실행
```go
func main() {
	a := 1
	if a == 0 {
		println("a is 0")
	} else if a == 2 {
		println("a is 2")
	} else {
		println("a is 1")
	}
}

//OUTPUT
a is 1
```

### func
함수 선언
```go
func main() {
	sum(5, 10)
}

func sum(a int, b int) {
	println("sum :", a+b)
}
```

### return
값을 반환
```go
func main() {
	println(sum(5, 10))
	println(multiply(5, 10))
}

func sum(a int, b int) (sum int) {
	sum = a + b
	return
}

func multiply(a int, b int) int {
	return a * b
}

//OUTPUT
15
50
```

### defer
함수 내에서 제일 마지막에 실행
```go
func main() {
	defer println("last")
	println("first")
}

//OUTPUT
first
last
```

### package
코드의 모듈화, 코드의 재사용 가능
* main 패키지 : 실행 프로그램
* 그 외 패키지 : 공유 패키지(라이브러리)

#### import
다른 패키지를 사용하기 위해 포함시킬 것을 선언
```go
import "fmt"

func main() {
	fmt.Println("fmt package")
}

//OUTPUT
fmt package
```

### type
새로운 타입 정의
#### struct
변수를 묶어서 새로운 자료형 정의 (Custom Data Type)
```go
type hotel struct {
	name  string
	price int
}

func main() {
	var h1 = hotel{}
	h1.name = "abc"
	h1.price = 5000

	h2 := hotel{name: "cde", price: 1000}

	fmt.Println(h1, h2)
}

//OUTPUT
{abc 5000} {cde 1000}
```

#### interface
메소드들의 집합체
```go
type reserve interface {
	twoday() int
}

type hotel struct {
	name  string
	price int
}

type airbnb struct {
	name   string
	price  int
	coupon int
}

func (h hotel) twoday() int {
	return h.price * 2
}

func (a airbnb) twoday() int {
	return a.price*2 - a.coupon
}

func main() {
	a := hotel{"aaa", 1000}
	b := airbnb{"bbb", 1000, 500}

	makeReserve(a, b)
}

func makeReserve(r ...reserve) {
	for _, v := range r {
		fmt.Println(v.twoday())
	}
}

//OUTPUT
2000
1500
```

### map
해시테이블을 구현한 자료구조
```go
func main() {
	m := map[string]int{
		"one": 1,
		"two": 2,
	}

	m["five"] = 5

	for k, v := range m {
		println(k, v)
	}
}

//OUTPUT
one 1
two 2
five 5
```

### go
`go` 키워드로 함수를 호출하면, goroutine 실행
```go
func main() {
	go num(1)
	go num(2)
	go num(3)

	time.Sleep(time.Second * 3)
}

func num(n int) {
	println(n)
}

//OUTPUT
3
1
2
```

### chan
채널 선언
* 채널 : 고루틴 사용시 값을 주고 받는 통로
```go
func main() {
	ch := make(chan int)

	go func() {
		ch <- 1
	}()

	num := <-ch

	println(num)
}

//OUTPUT
1
```

### select
다중 채널사용시 채널 분기문으로 사용
```go
func main() {

    c1 := make(chan string)
    c2 := make(chan string)

    go func() {
        time.Sleep(1 * time.Second)
        c1 <- "one"
    }()
    go func() {
        time.Sleep(2 * time.Second)
        c2 <- "two"
    }()

    for i := 0; i < 2; i++ {
        select {
        case msg1 := <-c1:
            fmt.Println("received", msg1)
        case msg2 := <-c2:
            fmt.Println("received", msg2)
        }
    }
}
```

### goto
특정 레이블로 이동
```go
func main() {
	num := 1
	if num == 1 {
		goto ONE
	} else {
		goto OTHER
	}

ONE:
	println("num is 1")
	goto END
OTHER:
	println("num is not 1")
END:

//OUTPUT
num is 1
}
```

!!! Reference
	[https://miryang.dev/blog/go-keyword](https://miryang.dev/blog/go-keyword)



## [strconv] Convert uint64 to string

```
package main

import (
    "fmt"
    "strconv"
)

func main() {

    // Create our number
    var myNumber uint64
    myNumber = 18446744073709551615
    
    // Format to a string by passing the number and it's base.
    str := strconv.FormatUint(myNumber, 10)

    // Print as string
    // Will output: 'The number is: 18446744073709551615'
    fmt.Println("The number is: " + str)
}
```

!!! Reference
	https://golangcode.com/uint64-to-string/

## [strconv] convert uint8 to string

```
strconv.Itoa(int(123))
```

!!! Reference
	https://stackoverflow.com/questions/19223277/how-to-convert-uint8-to-string

## [os/exec] 커맨드 명령어 실행하기 in go

```
package main

import (
	"fmt"
	"os"
	"os/exec"
	"strings"
)

func main() {

	// 1. Create an *exec.Cmd
	// cmd := exec.Command("go", "version")

	// // Stdout buffer
	// cmdOutput := &bytes.Buffer{}
	// // Attach buffer to command
	// cmd.Stdout = cmdOutput

	// // Execute command
	// printCommand(cmd)
	// err := cmd.Run() // will wait for command to return
	// printError(err)
	// // Only output the commands stdout
	// printOutput(cmdOutput.Bytes())

	// 2. Create an *exec.Cmd
	cmd := exec.Command("go", "version")

	// Combine stdout and stderr
	printCommand(cmd)
	output, err := cmd.CombinedOutput()
	printError(err)
	printOutput(output)
}

func printCommand(cmd *exec.Cmd) {
	fmt.Printf("==> Executing: %s\n", strings.Join(cmd.Args, " "))
}

func printError(err error) {
	if err != nil {
		os.Stderr.WriteString(fmt.Sprintf("==> Error: %s\n", err.Error()))
	}
}

func printOutput(outs []byte) {
	if len(outs) > 0 {
		fmt.Printf("==> Output: %s\n", string(outs))
	}
}
```

!!! Reference
	출처: https://www.darrencoxall.com/golang/executing-commands-in-go

## [iconv-go] byte to string 변환시 한글깨짐 해결

간혹 byte를 string 으로 출력시 한글깨짐 현상을 겪을 수 있다.<br>
이때 가장 합리적인 해결 방안.

```go
result, _ := iconv.ConvertString(string(output), "euc-kr", "utf-8")
```

윈도우 사용자:<br>
윈도우는 gcc 활성화를 위해서 [MinGW](https://sourceforge.net/projects/mingw-w64/files/Toolchains%20targetting%20Win64/Personal%20Builds/mingw-builds/8.1.0/threads-win32/seh/)를 설치해야 한다.(키보드 관련 go 라이브러리도 마찬가지)

![res](https://velog.velcdn.com/images/james-chun-dev/post/8a4d7e17-d24a-462b-8a8e-12ca2c704f55/image.png)

인스톨 파일이 작동되지 않는다면 압축파일 받아서 ProgramFiles 에 압축해제해서 떨궈놓고 환경변수(mingw64/bin)만 셋팅하면 된다.

!!! Reference
	https://github.com/djimenez/iconv-go

## aes256 cbc 암호화 복호화

### PKCS5Padding
base64:
```go
import (
	"bytes"
	"crypto/aes"
	"crypto/cipher"
	"encoding/base64"
	"fmt"
)

func AESEncrypt(src string, key []byte, iv string) string {
	block, err := aes.NewCipher(key)
	if err != nil {
		fmt.Println("key error1", err)
	}
	if src == "" {
		fmt.Println("plain content empty")
	}
	ecb := cipher.NewCBCEncrypter(block, []byte(iv))
	content := []byte(src)
	content = PKCS5Padding(content, block.BlockSize())
	crypted := make([]byte, len(content))
	ecb.CryptBlocks(crypted, content)

	return base64.StdEncoding.EncodeToString(crypted)
}

func AESDecrypt(cryptedStr string, key []byte, iv string) []byte {
	encryptedData, _ := base64.StdEncoding.DecodeString(cryptedStr)
	block, err := aes.NewCipher(key)
	if err != nil {
		fmt.Println("key error1", err)
	}
	if len(encryptedData) == 0 {
		fmt.Println("plain content empty")
	}
	ecb := cipher.NewCBCDecrypter(block, []byte(iv))
	decrypted := make([]byte, len(encryptedData))
	ecb.CryptBlocks(decrypted, encryptedData)

	return PKCS5Trimming(decrypted)
}

func PKCS5Padding(ciphertext []byte, blockSize int) []byte {
	padding := blockSize - len(ciphertext)%blockSize
	padtext := bytes.Repeat([]byte{byte(padding)}, padding)
	return append(ciphertext, padtext...)
}

func PKCS5Trimming(encrypt []byte) []byte {
	padding := encrypt[len(encrypt)-1]
	return encrypt[:len(encrypt)-int(padding)]
}
```

hex:
```go
func Ase256Encode(plaintext string, key string, iv string, blockSize int) string {
	bKey := []byte(key)
	bIV := []byte(iv)
	bPlaintext := PKCS5Padding([]byte(plaintext), blockSize, len(plaintext))
	block, err := aes.NewCipher(bKey)
	if err != nil {
		panic(err)
	}
	ciphertext := make([]byte, len(bPlaintext))
	mode := cipher.NewCBCEncrypter(block, bIV)
	mode.CryptBlocks(ciphertext, bPlaintext)
	return hex.EncodeToString(ciphertext)
}

func Ase256Decode(cipherText string, encKey string, iv string) (decryptedString string) {
	bKey := []byte(encKey)
	bIV := []byte(iv)
	cipherTextDecoded, err := hex.DecodeString(cipherText)
	if err != nil {
		panic(err)
	}

	block, err := aes.NewCipher(bKey)
	if err != nil {
		panic(err)
	}

	mode := cipher.NewCBCDecrypter(block, bIV)
	mode.CryptBlocks([]byte(cipherTextDecoded), []byte(cipherTextDecoded))
	return string(cipherTextDecoded)
}

func PKCS5Padding(ciphertext []byte, blockSize int, after int) []byte {
	padding := (blockSize - len(ciphertext)%blockSize)
	padtext := bytes.Repeat([]byte{byte(padding)}, padding)
	return append(ciphertext, padtext...)
}
```

## [time] YYYY-MM-DD date format 사용하기

### Datetime Format Layout

```
const (
	Layout      = "01/02 03:04:05PM '06 -0700" // The reference time, in numerical order.
	ANSIC       = "Mon Jan _2 15:04:05 2006"
	UnixDate    = "Mon Jan _2 15:04:05 MST 2006"
	RubyDate    = "Mon Jan 02 15:04:05 -0700 2006"
	RFC822      = "02 Jan 06 15:04 MST"
	RFC822Z     = "02 Jan 06 15:04 -0700" // RFC822 with numeric zone
	RFC850      = "Monday, 02-Jan-06 15:04:05 MST"
	RFC1123     = "Mon, 02 Jan 2006 15:04:05 MST"
	RFC1123Z    = "Mon, 02 Jan 2006 15:04:05 -0700" // RFC1123 with numeric zone
	RFC3339     = "2006-01-02T15:04:05Z07:00"
	RFC3339Nano = "2006-01-02T15:04:05.999999999Z07:00"
	Kitchen     = "3:04PM"
	// Handy time stamps.
	Stamp      = "Jan _2 15:04:05"
	StampMilli = "Jan _2 15:04:05.000"
	StampMicro = "Jan _2 15:04:05.000000"
	StampNano  = "Jan _2 15:04:05.000000000"
)
```


#### YYYY-MM-DD
```
package main

import (
    "fmt"
    "time"
)

const (
    YYYYMMDD = "2006-01-02"
)

func main() {
    now := time.Now().UTC()
    fmt.Println(now.Format(YYYYMMDD))
}
//output: 2022-03-14
```

#### DD/MM/YYYY
```
package main

import (
    "fmt"
    "time"
)

const (
    DDMMYYYY = "02/01/2006"
)

func main() {
    now := time.Now().UTC()
    fmt.Println(now.Format(DDMMYYYY))
}
//output: 14/03/2022
```

#### YYYY-MM-DD hh:mm:ss
```
package main

import (
    "fmt"
    "time"
)

const (
    DDMMYYYYhhmmss = "2006-01-02 15:04:05"
)

func main() {
    now := time.Now().UTC()
    fmt.Println(now.Format(DDMMYYYYhhmmss))
}
//output: 2022-03-14 05:41:33
```

!!! Reference
	[https://gosamples.dev/date-format-yyyy-mm-dd](https://gosamples.dev/date-format-yyyy-mm-dd)


## [time] 문자열 파싱

```go
input := "2020-10-24"
layout := "2006-01-02"

t, _ := time.Parse(layout, input)
fmt.Println(t)   // 2020-10-24 00:00:00 +0000 UTC
```


## [ioutil.TempFile] 임시파일 생성하는 방법

[IO / ioutil](https://golang.org/pkg/io/ioutil/) 패키지는 두 가지 기능을 포함하고 임시 디렉토리 [TempDir](https://golang.org/pkg/io/ioutil/#TempDir) 및 임시 파일 [TempFile](https://golang.org/pkg/io/ioutil/#TempFile) 을 만들 수 있습니다.

유닉스 머신에서 기본적으로 TempDir 함수는/tmp/임시 디렉토리로반환됩니다.접두사가있는 디렉토리를 작성하기 위해 추가 매개 변수를 전달할 수 있습니다.

TempFile 함수는 고유하고 임의의 숫자를 만듭니다.pattern매개 변수를 사용하여 파일의 접두사와 접미사를 제어 할 수 있습니다. 예로 \*.tmp 를 전달하면 임시 파일이 생성됩니다.프로그램에서 임시 파일을 삭제하지 않고 나중에 임시 파일을 정리하는 일괄 삭제 작업을 원하는 경우에 유용합니다.

파일을 만든 후에는 파일 형식 그대로 파일에 무엇이든 쓸 수 있습니다

os.File.os.File은 io.ReadWriter 인터페이스를 구현하므로 Write메소드를 사용하여 파일에 쓸 수 있습니다.

```
package main

import (
    "fmt"
    "io/ioutil"
    "os"
)

func main() {

    // Create a Temp File:  This will create a filename like /tmp/pre-23423234
    // If we use a pattern like "pre-*.ext", you can get a file like: /tmp/pre-23423234.ext
    tmpFile, err := ioutil.TempFile(os.TempDir(), "pre-")
    if err != nil {
        fmt.Println("Cannot create temporary file", err)
    }

    // cleaning up by removing the file
    defer os.Remove(tmpFile.Name())

    fmt.Println("Created a Temp File: " + tmpFile.Name())

    // Write to the file
    text := []byte("Writing some text into the temp file.")
    if _, err = tmpFile.Write(text); err != nil {
        fmt.Println("Failed to write to temporary file", err)
    }

    // Close the file
    if err := tmpFile.Close(); err != nil {
        fmt.Println(err)
    }
}
```

## io/ioutil -> io 옮겨간 메서드

- ioutil.ReadAll => io.ReadAll
- ioutil.ReadDir => os.ReadDir
- ioutil.ReadFile => os.ReadFile
- ioutil.TempDir => os.MkdirTemp
- ioutil.TempFile => os.CreateTemp
- ioutil.WriteFile => os.WriteFile

!!! Reference
	https://runebook.dev/ko/docs/go/io/ioutil/index

## bool 에서 int8 로 변환하기

```
package main

import "fmt"

func B2i(b bool) int8 {
    if b {
        return 1
    }
    return 0
}
func main() {
    fmt.Println(B2i(true))  // 1
    fmt.Println(B2i(false)) // 0    
}
```

!!! Reference
	https://stackoverflow.com/questions/38627078/how-to-convert-bool-to-int8-in-golang