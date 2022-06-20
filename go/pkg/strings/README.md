# Strings

## Functions

### func Clone

```go
func Clone(s string) string
```

Clone returns a fresh copy of s. 

### func Compare

```go
func Compare(a, b string) int
```

Compare returns an integer comparing two strings lexicographically. The result will be 0 if a == b, -1 if a < b, and +1 if a > b.

### func Contains

```go
func Contains(s, substr string) bool
```

Contains reports whether substr is within s.

### func ContainsAny

```go
func ContainsAny(s, chars string) bool
```

ContainsAny reports whether any Unicode code points in chars are within s.

### func ContainsRune

```go
func ContainsRune(s string, r rune) bool
```

ContainsRune reports whether the Unicode code point r is within s.

### func Count

```go
func Count(s, substr string) int
```

Count counts the number of non-overlapping instances of substr in s. If substr is an empty string, Count returns 1 + the number of Unicode code points in s.

### func Cut

```go
func Cut(s, sep string) (before, after string, found bool)
```

Cut slices s around the first instance of sep, returning the text before and after sep. The found result reports whether sep appears in s. If sep does not appear in s, cut returns s, "", false.

### func EqualFold

```go
func EqualFold(s, t string) bool
```

EqualFold reports whether s and t, interpreted as UTF-8 strings, are equal under Unicode case-folding, which is a more general form of case-insensitivity.

### func Fields

```go
func Fields(s string) []string
```

```go
func main() {
	fmt.Printf("Fields are: %q", strings.Fields("  foo bar  baz   "))
    // Fields are: ["foo" "bar" "baz"]
}
```

Fields splits the string s around each instance of one or more consecutive white space characters, as defined by unicode.

### func FieldsFunc

```go
func FieldsFunc(s string, f func(rune) bool) []string
```

FieldsFunc splits the string s at each run of Unicode code points c satisfying f(c) and returns an array of slices of s. If all code points in s satisfy f(c) or the string is empty, an empty slice is returned.

```go
import (
	"fmt"
	"strings"
	"unicode"
)

func main() {
	f := func(c rune) bool {
		return !unicode.IsLetter(c) && !unicode.IsNumber(c)
	}
	fmt.Printf("Fields are: %q", strings.FieldsFunc("  foo1;bar2,baz3...", f))
}
```

### func HasPrefix

```go
func HasPrefix(s, prefix string) bool
```

HasPrefix tests whether the string s begins with prefix.

### func HasSuffix

```go
func HasSuffix(s, suffix string) bool
```

HasSuffix tests whether the string s ends with suffix.

### func Index 

```go
func Index(s, substr string) int
```

Index returns the index of the first instance of substr in s, or -1 if substr is not present in s.

### func IndexFunc

```go
func IndexFunc(s string, f func(rune) bool) int
```

IndexFunc returns the index into s of the first Unicode code point satisfying f(c), or -1 if none do.

```go
func main() {
	f := func(c rune) bool {
		return unicode.Is(unicode.Han, c)
	}
	fmt.Println(strings.IndexFunc("Hello, 世界", f))
	fmt.Println(strings.IndexFunc("Hello, world", f))
}
```

### func Join

```go
func Join(elems []string, sep string) string
```

Join concatenates the elements of its first argument to create a single string. The separator string sep is placed between elements in the resulting string.

### func Map

```go
func Map(mapping func(rune) rune, s string) string
```

Map returns a copy of the string s with all its characters modified according to the mapping function. 

```go
func main() {
	rot13 := func(r rune) rune {
		switch {
		case r >= 'A' && r <= 'Z':
			return 'A' + (r-'A'+13)%26
		case r >= 'a' && r <= 'z':
			return 'a' + (r-'a'+13)%26
		}
		return r
	}
	fmt.Println(strings.Map(rot13, "'Twas brillig and the slithy gopher..."))
}
```

### func Replace

```go
func Replace(s, old, new string, n int) string
```

```go
func main() {
	fmt.Println(strings.Replace("oink oink oink", "k", "ky", 2))
    // oinky oinky oink
	fmt.Println(strings.Replace("oink oink oink", "oink", "moo", -1))
    // moo moo moo
}
```
### func Split

```go
func Split(s, sep string) []string
```

Split slices s into all substrings separated by sep and returns a slice of the substrings between those separators.

### func ToLower

```go
func ToLower(s string) string
```

### func ToUpper

```go
func ToUpper(s string) string
```

### func Trim

```go
func Trim(s, cutset string) string
```

Trim returns a slice of the string s with all leading and trailing Unicode code points contained in cutset removed.

```go
func main() {
	fmt.Print(strings.Trim("¡¡¡Hello, Gophers!!!", "!¡"))
    // Hello, Gophers
}
```
### func TrimFunc

```go
func TrimFunc(s string, f func(rune) bool) string
```

TrimFunc returns a slice of the string s with all leading and trailing Unicode code points c satisfying f(c) removed.

### func TrimSpace

```go
func TrimSpace(s string) string
```

TrimSpace returns a slice of the string s, with all leading and trailing white space removed, as defined by Unicode.

## Types

### type Builder

```go
type Builder struct {
	// contains filtered or unexported fields
}

// String returns the accumulated string.
func (b *Builder) String() string

// Write appends the contents of p to b's buffer.
func (b *Builder) Write(p []byte) (int, error)

// WriteString appends the contents of s to b's 
// buffer. It returns the length of s and a nil error.
func (b *Builder) WriteString(s string) (int, error)
```

A Builder is used to efficiently build a string using Write methods. It minimizes memory copying. The zero value is ready to use. Do not copy a non-zero Builder.

```go
func main() {
	var b strings.Builder
	for i := 3; i >= 1; i-- {
		fmt.Fprintf(&b, "%d...", i)
	}
	b.WriteString("ignition")
	fmt.Println(b.String())
    // 3...2...1...ignition
}
```

### type Reader

```go
type Reader struct {
	// contains filtered or unexported fields
}

func NewReader(s string) *Reader

func (r *Reader) Read(b []byte) (n int, err error)

// WriteTo implements the io.WriterTo interface.
func (r *Reader) WriteTo(w io.Writer) (n int64, err error)
```

### type Replacer

```go
type Replacer struct {
	// contains filtered or unexported fields
}

func NewReplacer(oldnew ...string) *Replacer

func (r *Replacer) Replace(s string) string

// WriteString writes s to w with all replacements performed.
func (r *Replacer) WriteString(w io.Writer, s string) (n int, err error)
```

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	r := strings.NewReplacer("<", "&lt;", ">", "&gt;")
	fmt.Println(r.Replace("This is <b>HTML</b>!"))
    // This is &lt;b&gt;HTML&lt;/b&gt;!
}
```