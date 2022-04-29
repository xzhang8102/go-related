# bufio.Reader
ref: https://medium.com/golangspec/introduction-to-bufio-package-in-golang-ad7d1877f762  
source code: https://github.com/golang/go/blob/master/src/bufio/bufio.go
## workflow:
```
io.Reader --> buffer --> data consumer
```
## Definition
```go
type Reader struct {
	buf          []byte
	rd           io.Reader  // reader provided by the client
	r, w         int        // buf read and write positions
	err          error
	lastByte     int        // last byte read for UnreadByte; -1 means invalid
	lastRuneSize int        // size of last rune read for UnreadRune; -1 means invalid
}
```
## APIs
### Creation
```go
br := bufio.NewReaderSize(os.Stdin, 16) // min buffer size 16
br := bufio.NewReader(os.Stdin)         // default buffer size 4096
```
### `Peek`
```go
// see first n bytes of buffered data without consuming
// tips: returned slice shares the same underlying array with the buffer, maybe overwritten by further reads

// 1. buffer isn't full and holds less than n bytes, more data will be read from io.Reader
s1 := strings.NewReader(strings.Repeat("a", 20))
r := bufio.NewReaderSize(s1, 16)
b, err := r.Peek(3)   // b: "aaa"
// 2. n is bigger than buffer size, bufio.ErrBufferFull will be returned
b, err := r.Peek(17)  // bufio: buffer full

// 3. n is bigger than stream size, io.EOF will be returned
s2 := strings.NewReader("aaa")
r.Reset(s2)
b, err = r.Peek(10)   // EOF
```
### `Read`
```go
// implement the io.Reader interface
type Reader interface {
	Read(p []byte) (n int, err error)
}
```
### {Read,Unread}{Byte,Rune}
either read single byte/rune or return last byte/rune back to the buffer
### `ReadSlice`
```go
// return bytes till first occurrence of passed byte
// returned slice shares the same underlying array with the buffer
s := strings.NewReader("abcd|efg")
br := bufio.NewReader(s)
token, err := br.ReadSlice('|')  // token: abcd|

// if the delimiter cannot be found and EOF has been reached, io.EOF will be returned
// if the delimiter cannot be found and the buffer is full, io.ErrBufferFull will be returned
```
### `ReadBytes`
```go
// use ReadSlice underneath
// not restricted by buffer size like ReadSlice does
// returned slice is independent of the buffer
s := strings.NewReader(strings.Repeat("a", 20) + "|")
br := bufio.NewReaderSize(s, 16)
token, err := br.ReadBytes('|')  // token: aaaaaaaaaaaaaaaaaaaa|
```
### `ReadString`
wrapper over `ReadBytes`
### `ReadLine`
use ReadSlice('\n') underneath  
remove newline characters from returned slice  
most caller should use `ReadBytes('\n')` or `ReadString('\n')` or use a Scanner instead
### `WriteTo`
```go
// read from the data source and send to the passed consumer which is a io.Writer

// implements io.WriterTo interface
type WriterTo interface {
	WriteTo(w Writer) (n int64, err error)
}
```
## Tips
1. If internal buffer holds at least one byte, then the data will be read from buffer no mather the size of the input slice
2. If internal buffer is empty, the `io.Reader` will be triggered to fill the buffer
3. If internal buffer is empty but the passed slice is bigger than buffer, then `bufio.Reader` will skip buffering and read directly into passed slice