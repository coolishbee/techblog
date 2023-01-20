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

>https://golangcode.com/uint64-to-string/

## [strconv] convert uint8 to string

```
strconv.Itoa(int(123))
```

>https://stackoverflow.com/questions/19223277/how-to-convert-uint8-to-string

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

>출처: https://www.darrencoxall.com/golang/executing-commands-in-go

## [iconv-go] byte to string 변환시 한글깨짐 해결

간혹 byte를 string 으로 출력시 한글깨짐 현상을 겪을 수 있다.
이때 가장 합리적인 해결 방안
example:
```
result, _ := iconv.ConvertString(string(output), "euc-kr", "utf-8")
```

>https://github.com/djimenez/iconv-go

윈도우 사용자:
윈도우는 gcc 활성화를 위해서 [MinGW](https://sourceforge.net/projects/mingw-w64/files/Toolchains%20targetting%20Win64/Personal%20Builds/mingw-builds/8.1.0/threads-win32/seh/)를 설치해야 한다.(키보드 관련 go 라이브러리도 마찬가지)
![res](https://velog.velcdn.com/images/james-chun-dev/post/8a4d7e17-d24a-462b-8a8e-12ca2c704f55/image.png)
인스톨 파일이 작동되지 않는다면 압축파일 받아서 ProgramFiles 에 압축해제해서 떨궈놓고 환경변수(mingw64/bin)만 셋팅하면 된다.

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

## Datetime Format Layout

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


> 출처: https://gosamples.dev/date-format-yyyy-mm-dd/