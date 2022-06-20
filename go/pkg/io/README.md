# IO

## Overview

Package io provides _basic interfaces_ to I/O primitives. 

## Variables

```go
var EOF = errors.New("EOF")
```

`EOF` is the error returned by Read when no more input is available.

```go
var ErrClosedPipe = errors.New("io: read/write on closed pipe")
```

`ErrClosedPipe` is the error used for read or write operations on a closed pipe.

```go
var ErrShortBuffer = errors.New("short buffer")
```

`ErrShortBuffer` means that a read required a longer buffer than was provided.

```go
var ErrUnexpectedEOF = errors.New("unexpected EOF")
```

`ErrUnexpectedEOF` means that EOF was encountered in the middle of reading a fixed-size block or data structure.

## Functions

### func Copy

```go
func Copy(dst Writer, src Reader) (written int64, err error)
```

Copy copies from src to dst until either EOF is reached on src or an error occurs. It returns the number of bytes copied and the first error encountered while copying, if any.

```go
package main

import (
	"io"
	"log"
	"os"
	"strings"
)

func main() {
	r := strings.NewReader("some io.Reader stream to be read\n")

	if _, err := io.Copy(os.Stdout, r); err != nil {
		log.Fatal(err)
	}

}
```

### func CopyBuffer

```go
func CopyBuffer(dst Writer, src Reader, buf []byte) (written int64, err error)
```

CopyBuffer is identical to Copy except that it stages through the provided buffer (if one is required) rather than allocating a temporary one.

```go
import (
	"io"
	"log"
	"os"
	"strings"
)

func main() {
	r1 := strings.NewReader("first reader\n")
	r2 := strings.NewReader("second reader\n")
	buf := make([]byte, 8)

	// buf is used here...
	if _, err := io.CopyBuffer(os.Stdout, r1, buf); err != nil {
		log.Fatal(err)
	}

	// ... reused here also. No need to allocate an extra buffer.
	if _, err := io.CopyBuffer(os.Stdout, r2, buf); err != nil {
		log.Fatal(err)
	}

}
```

### func CopyN 

```go
func CopyN(dst Writer, src Reader, n int64) (written int64, err error)
```

CopyN copies n bytes (or until an error) from src to dst. It returns the number of bytes copied and the earliest error encountered while copying. On return, written == n if and only if err == nil.

### func Pipe

```go
func Pipe() (*PipeReader, *PipeWriter)
```
It is safe to call Read and Write in parallel with each other or with Close. 

```go
package main

import (
	"fmt"
	"io"
	"log"
	"os"
)

func main() {
	r, w := io.Pipe()

	go func() {
		fmt.Fprint(w, "some io.Reader stream to be read\n")
		w.Close()
	}()

	if _, err := io.Copy(os.Stdout, r); err != nil {
		log.Fatal(err)
	}

}
```

### func ReadAll

```go
func ReadAll(r Reader) ([]byte, error)
```

### func ReadFull

```go
func ReadFull(r Reader, buf []byte) (n int, err error)
```

```go
package main

import (
	"fmt"
	"io"
	"log"
	"strings"
)

func main() {
	r := strings.NewReader("some io.Reader stream to be read\n")

	buf := make([]byte, 4)
	if _, err := io.ReadFull(r, buf); err != nil {
		log.Fatal(err)
	}
	fmt.Printf("%s\n", buf)

	// minimal read size bigger than io.Reader stream
	longBuf := make([]byte, 64)
	if _, err := io.ReadFull(r, longBuf); err != nil {
		fmt.Println("error:", err)
	}

}
```
```
<!-- Output: -->

some
error: unexpected EOF
```

### func WriteString

```go
func WriteString(w Writer, s string) (n int, err error)
```

```go
package main

import (
	"io"
	"log"
	"os"
)

func main() {
	if _, err := io.WriteString(os.Stdout, "Hello World"); err != nil {
		log.Fatal(err)
	}

}
```

## Types

### type ByteReader

```go
type ByteReader interface {
	ReadByte() (byte, error)
}
```

ByteReader is the interface that wraps the ReadByte method. ReadByte reads and returns the next byte from the input or any error encountered.

ReadByte provides an efficient interface for byte-at-time processing.

### type ByteScanner

```go
type ByteScanner interface {
	ByteReader
	UnreadByte() error
}
```

### type ByteWriter

```go
type ByteWriter interface {
	WriteByte(c byte) error
}
```

### type Closer

```go
type Closer interface {
	Close() error
}
```

### type LimitedReader 

```go
type LimitedReader struct {
	R Reader // underlying reader
	N int64  // max bytes remaining
}

func (l *LimitedReader) Read(p []byte) (n int, err error)
```

A LimitedReader reads from R but limits the amount of data returned to just N bytes. Each call to Read updates N to reflect the new amount remaining. Read returns EOF when N <= 0 or when the underlying R returns EOF.

### type PipeReader

```go
type PipeReader struct {
	// contains filtered or unexported fields
}

func (r *PipeReader) Close() error

func (r *PipeReader) CloseWithError(err error) error

// Read implements the standard Read interface: it reads 
// data from the pipe, blocking until a writer arrives or 
// the write end is closed.
func (r *PipeReader) Read(data []byte) (n int, err error)
```

### type PipeWriter 

```go
type PipeWriter struct {
	// contains filtered or unexported fields
}

func (w *PipeWriter) Close() error

func (w *PipeWriter) CloseWithError(err error) error

func (w *PipeWriter) Write(data []byte) (n int, err error)
```

### type Reader

```go
type Reader interface {
	Read(p []byte) (n int, err error)
}
```

```go
func LimitReader(r Reader, n int64) Reader
```

```go
func MultiReader(readers ...Reader) Reader
```