# Tutorial: Getting started with fuzzing

`/fuzz/main.go`
```go
package main

import "fmt"

func main() {
    input := "The quick brown fox jumped over the lazy dog"
    rev := Reverse(input)
    doubleRev := Reverse(rev)
    fmt.Printf("original: %q\n", input)
    fmt.Printf("reversed: %q\n", rev)
    fmt.Printf("reversed again: %q\n", doubleRev)
}

func Reverse(s string) string {
    b := []byte(s)
    for i, j := 0, len(b)-1; i < len(b)/2; i, j = i+1, j-1 {
        b[i], b[j] = b[j], b[i]
    }
    return string(b)
}
```

## Add a unit test

`/fuzz/reverse_test.go`
```go
package main

import (
    "testing"
)

func TestReverse(t *testing.T) {
    testcases := []struct {
        in, want string
    }{
        {"Hello, world", "dlrow ,olleH"},
        {" ", " "},
        {"!12345", "54321!"},
    }
    for _, tc := range testcases {
        rev := Reverse(tc.in)
        if rev != tc.want {
                t.Errorf("Reverse: %q, want %q", rev, tc.want)
        }
    }
}
```

## Add a fuzz test

The unit test has limitations, namely that each input must be added to the test by the developer. One benefit of fuzzing is that it comes up with inputs for your code, and may identify edge cases that the test cases you came up with didn’t reach.

`/fuzz/reverse_test.go`
```go
package main

import (
    "testing"
    "unicode/utf8"
)

func FuzzReverse(f *testing.F) {
    testcases := []string{"Hello, world", " ", "!12345"}
    for _, tc := range testcases {
        f.Add(tc)  // Use f.Add to provide a seed corpus
    }
    f.Fuzz(func(t *testing.T, orig string) {
        rev := Reverse(orig)
        doubleRev := Reverse(rev)
        if orig != doubleRev {
            t.Errorf("Before: %q, after: %q", orig, doubleRev)
        }
        if utf8.ValidString(orig) && !utf8.ValidString(rev) {
            t.Errorf("Reverse produced invalid UTF-8 string %q", rev)
        }
    })
}
```
When fuzzing, you can’t predict the expected output, since you don’t have control over the inputs. 

However, there are a few properties of the Reverse function that you can verify in a fuzz test. The two properties being checked in this fuzz test are:
- Reversing a string twice preserves the original value
- The reversed string preserves its state as valid UTF-8

__Note__ the syntax differences between the unit test and the fuzz test:
- The function begins with FuzzXxx instead of TestXxx, and takes *testing.F instead of *testing.T
- Where you would expect to see a t.Run execution, you instead see f.Fuzz which takes a fuzz target function whose parameters are *testing.T and the types to be fuzzed. The inputs from your unit test are provided as seed corpus inputs using f.Add.

## Run the code 
```sh
# without fuzzing (only seed inputs)
$ go test
# You can also run go test -run=FuzzReverse
```
```sh
$ go test -fuzz=Fuzz
```

To view the input that caused the failure, open the corpus file written to the `testdata/fuzz/FuzzReverse` directory in a text editor.

> go test fuzz v1
> string("ܠ")

The first line of the corpus file indicates the encoding version. Each following line represents the value of each type making up the corpus entry. Since the fuzz target only takes 1 input, there is only 1 value after the version.

Run go test again without the -fuzz flag; the new failing seed corpus entry will be used

## Fix the invalid string error

The current Reverse function reverses the string byte-by-byte, and therein lies our problem. In order to preserve the UTF-8-encoded runes of the original string, we must instead reverse the string rune-by-rune.

```go
package main

import (
    "testing"
    "unicode/utf8"
)

func FuzzReverse(f *testing.F) {
    testcases := []string{"Hello, world", " ", "!12345"}
    for _, tc := range testcases {
        f.Add(tc)  // Use f.Add to provide a seed corpus
    }
    f.Fuzz(func(t *testing.T, orig string) {
        rev := Reverse(orig)
        doubleRev := Reverse(rev)
        t.Logf("Number of runes: orig=%d, rev=%d, doubleRev=%d", utf8.RuneCountInString(orig), utf8.RuneCountInString(rev), utf8.RuneCountInString(doubleRev))
        if orig != doubleRev {
            t.Errorf("Before: %q, after: %q", orig, doubleRev)
        }
        if utf8.ValidString(orig) && !utf8.ValidString(rev) {
            t.Errorf("Reverse produced invalid UTF-8 string %q", rev)
        }
    })
}
```

The entire seed corpus used strings in which every character was a single byte. However, characters such as 泃 can require several bytes. Thus, reversing the string byte-by-byte will invalidate multi-byte characters.

### Fix the error

To correct the Reverse function, let’s traverse the string by runes, instead of by bytes.

func Reverse(s string) string {
    r := []rune(s)
    for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
        r[i], r[j] = r[j], r[i]
    }
    return string(r)
}

Now we can see that the string is different from the original after being reversed twice. This time the input itself is invalid unicode. 

## Fix the double reverse error

Let's replace the Reverse function with the following:
```go
func Reverse(s string) string {
    fmt.Printf("input: %q\n", s)
    r := []rune(s)
    fmt.Printf("runes: %q\n", r)
    for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
        r[i], r[j] = r[j], r[i]
    }
    return string(r)
}
```
This will help us understand what is going wrong when converting the string to a slice of runes.

### Fix the error

To fix this issue, let’s return an error if the input to Reverse isn’t valid UTF-8.

```go
package main

import (
    "errors"
    "fmt"
    "unicode/utf8"
)

func main() {
    input := "The quick brown fox jumped over the lazy dog"
    rev, revErr := Reverse(input)
    doubleRev, doubleRevErr := Reverse(rev)
    fmt.Printf("original: %q\n", input)
    fmt.Printf("reversed: %q, err: %v\n", rev, revErr)
    fmt.Printf("reversed again: %q, err: %v\n", doubleRev, doubleRevErr)
}

func Reverse(s string) (string, error) {
    if !utf8.ValidString(s) {
        return s, errors.New("input is not valid UTF-8")
    }
    r := []rune(s)
    for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
        r[i], r[j] = r[j], r[i]
    }
    return string(r), nil
}
```
Modify the `reverse_test.go` file to check for errors and skip the test if errors are generated by returning.
```go
func FuzzReverse(f *testing.F) {
    testcases := []string {"Hello, world", " ", "!12345"}
    for _, tc := range testcases {
        f.Add(tc)  // Use f.Add to provide a seed corpus
    }
    f.Fuzz(func(t *testing.T, orig string) {
        rev, err1 := Reverse(orig)
        if err1 != nil {
            return
        }
        doubleRev, err2 := Reverse(rev)
        if err2 != nil {
             return
        }
        if orig != doubleRev {
            t.Errorf("Before: %q, after: %q", orig, doubleRev)
        }
        if utf8.ValidString(orig) && !utf8.ValidString(rev) {
            t.Errorf("Reverse produced invalid UTF-8 string %q", rev)
        }
    })
}
```

Rather than returning, you can also call `t.Skip()` to stop the execution of that fuzz input.

The fuzz test will run until it encounters a failing input unless you pass the -fuzztime flag. The default is to run forever if no failures occur, and the process can be interrupted with ctrl-C.

