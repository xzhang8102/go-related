# bufio
ref: https://medium.com/golangspec/introduction-to-bufio-package-in-golang-ad7d1877f762
## bufio.Writer
source: https://github.com/golang/go/blob/master/src/bufio/bufio.go
### workflow:
```
data producer --> buffer --> io.Writer
```
### Definition
```go
type Writer struct {
	err error      // held the error during writing, further ops are ignored
	buf []byte     // buffer
	n   int        // writing position inside the buffer
	wr  io.Writer  // underlying writer
}
```
### APIs
#### Constructor
```go
// w - io.Writer
bufio.NewWriter(os.Stdout)  // default buffer size: 4096
bufio.NewWriterSize(os.Stdout, 4)
```
#### Reset
```go
// re-used buffer writer for different io.Writer
bw.Reset(w) // feed with a new io.Writer

// source code
func (b *Writer) Reset(w io.Writer) {
	if b.buf == nil {
		b.buf = make([]byte, defaultBufSize)
	}
	b.err = nil
	b.n = 0
	b.wr = w
}
```
#### Available
```go
bw.Available() // return the left space in buffer
```
#### Write{Byte,Rune,String}
```go
bw.WriteByte('a')
bw.WriteRune('我')
bw.WriteString("golang")
```
#### ReadFrom
```go
// implement io.ReaderFrom interface
type ReaderFrom interface {
	ReadFrom(r Reader) (n int64, err error)
}

s := strings.NewReader("onetwothree")
bw := bufio.NewWriterSize(os.Stdout, 3)
bw.ReadFrom(s)
err := bw.Flush()
if err != nil {
	panic(err)
}
```
### Tips
1. Remember to call the `Flush` method to guarantee all data has be forwarded to the underlying `io.Writer`
2. If the size of the data is larger than the buffer size, the data goes directly to the `io.Writer`