# Lesson 1

----------------------
## Table of contents
- [Packages](#packages)
- [Imports](#imports)
- [Functions](#functions)
- [Variables](#variables)
- [Basic types](#basic-types)
- [Type conversions](#type-conversions)
- [Constants](#constants)

----------------------

# Packages
Every Go program is made up of packages.

Programs start running in package `main`.

This program is using the packages with import paths "`fmt`" and "`math/rand`".

By convention, the package name is the same as the last element of the import path. For instance, the "`math/rand`" package comprises files that begin with the statement `package rand`.
```go
package main

import (
	"fmt"
	"math/rand"
)

func main() {
	fmt.Println("My favorite number is", rand.Intn(10))
}
```

# Imports

This code groups the imports into a parenthesized, "factored" import statement.

You can also write multiple import statements, like:

```go
import "fmt"
import "math"
```
But it is good style to use the factored import statement.

# Exported names

In Go, a name is exported if it begins with a capital letter. For example, `Pizza` is an exported name, as is `Pi`, which is exported from the math package.

`pizza` and `pi` do not start with a capital letter, so they are not exported.

When importing a package, you can refer only to its exported names. Any "unexported" names are not accessible from outside the package.

# Functions

A function can take zero or more arguments.
In this example, `add` takes two parameters of type int.
Notice that the type comes after the variable name.
__TODO__ : [Go's Declaration Syntax](https://go.dev/blog/declaration-syntax)
```go
package main

import "fmt"

func add(x int, y int) int {
	return x + y
}

func main() {
	fmt.Println(add(42, 13))
}
```
When two or more consecutive named function parameters share a type, you can omit the type from all but the last.
```go
func add(x, y int) int {
	return x + y
}
```
## Multiple results

A function can return any number of results.

The `swap` function returns two strings.
```go 
package main

import "fmt"

func swap(x, y string) (string, string) {
	return y, x
}

func main() {
	a, b := swap("hello", "world")
	fmt.Println(a, b)
}
```
## Named return values

Go's return values may be named. If so, they are treated as variables defined at the top of the function.

These names should be used to document the meaning of the return values.

A return statement without arguments returns the named return values. This is known as a `"naked" return`.

Naked return statements should be used only in short functions, as with the example shown here. They can harm readability in longer functions.
```go
package main

import "fmt"

func split(sum int) (x, y int) {
	x = sum * 4 / 9
	y = sum - x
	return
}

func main() {
	fmt.Println(split(17))
}
```

# Variables

The `var` statement declares a list of variables; as in function argument lists, the type is last.

A `var` statement can be at package or function level. We see both in this example.

```go
package main

import "fmt"

var c, python, java bool

func main() {
	var i int
	fmt.Println(i, c, python, java)
}
```
## Variables with initializers
A var declaration can include initializers, one per variable.

If an initializer is present, the type can be omitted; the variable will take the type of the initializer.
```go
package main

import "fmt"

var i, j int = 1, 2

func main() {
	var c, python, java = true, false, "no!"
	fmt.Println(i, j, c, python, java)
}
```
_(smth like C++ auto)_

## Short variable declarations

Inside a function, the := short assignment statement can be used in place of a var declaration with implicit type.

Outside a function, every statement begins with a keyword (var, func, and so on) and so the := construct is not available.
```go
package main

import "fmt"

func main() {
	var i, j int = 1, 2
	k := 3
	c, python, java := true, false, "no!"

	fmt.Println(i, j, k, c, python, java)
}
```
# Basic types

> bool
> 
> string
> 
> int  int8  int16  int32  int64
> uint uint8 uint16 uint32 uint64 uintptr
> 
> byte // alias for uint8
> 
> rune // alias for int32
>      // represents a Unicode code point
> 
> float32 float64
> 
> complex64 complex128

## Zero values
Variables declared without an explicit initial value are given their zero value.

The zero value is:

0 for numeric types,
false for the boolean type, and
"" (the empty string) for strings.

## Type conversions
The expression `T(v)` converts the value `v` to the type `T`.

Some numeric conversions:
```go
var i int = 42
var f float64 = float64(i)
var u uint = uint(f)
```
Unlike in C, in Go assignment between items of different type requires an explicit conversion.

## Type inference

When declaring a variable without specifying an explicit type (either by using the := syntax or var = expression syntax), the variable's type is inferred from the value on the right hand side.

When the right hand side of the declaration is typed, the new variable is of that same type:
```go
var i int
j := i // j is an int
```
But when the right hand side contains an untyped numeric constant, the new variable may be an int, float64, or complex128 depending on the precision of the constant:
```go
i := 42           // int
f := 3.142        // float64
g := 0.867 + 0.5i // complex128
```
## Constants
Constants are declared like variables, but with the `const` keyword.

Constants can be character, string, boolean, or numeric values.

Constants cannot be declared using the := syntax.
```go
package main

import "fmt"

const Pi = 3.14

func main() {
	const World = "世界"
	fmt.Println("Hello", World)
	fmt.Println("Happy", Pi, "Day")

	const Truth = true
	fmt.Println("Go rules?", Truth)
}
```
## Numeric Constants
Numeric constants are high-precision values.

An untyped constant takes the type needed by its context.
```go
package main

import "fmt"

const (
	// Create a huge number by shifting a 1 bit left 100 places.
	// In other words, the binary number that is 1 followed by 100 zeroes.
	Big = 1 << 100
	// Shift it right again 99 places, so we end up with 1<<1, or 2.
	Small = Big >> 99
)

func needInt(x int) int { return x*10 + 1 }
func needFloat(x float64) float64 {
	return x * 0.1
}

func main() {
	fmt.Println(needInt(Small))
	fmt.Println(needFloat(Small))
	fmt.Println(needFloat(Big))
}
```