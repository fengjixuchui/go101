
# Some Undocumented Changes in Go 1.18 and 1.19

When a new Go version is released, the changes in this version are generally
listed the release notes of the version.
However, occasionally, some changes were missing in the release notes.

This article lists several missing changes in Go [1.18] and [1.19] release notes.

[1.18]: https://go.dev/doc/go1.18
[1.19]: https://go.dev/doc/go1.19

## GoTv

We use [GoTV] to demonstrate the missing changes in this article.
GoTV (Go Toolchain Version) is a tool used to manage and
use multiple coexisting installations of official Go toolchain versions
harmoniously and conveniently.

[GoTV]: https://go101.org/apps-and-libs/gotv.html

If you haven't, you may install GoTV by running

```
go install go101.org/gotv@latest
```

then run the following commands to install the latest releases of Go 1.17,
Go 1.18 and Go 1.19 toolchains.

```
gotv 1.17. version
gotv 1.18. version
gotv 1.19. version
```

The first execution of the `gotv` command needs to pull the Go language project,
so it might need several minutes to finish.

### The definition of the terminology "slice of bytes" is clarified

Go specification states "a non-constant value `x` can be converted to type `T`
if `x` is a string and `T` is a slice of bytes, and vice versa".

But what is a slice of bytes? There are two interpretations:

1. any type whose underlying type is `[]byte`.
1. any slice type whose element type's underlying type is `byte`.

Before Go 1.18, the official standard Go compiler sometimes adopted
the first interpretation, sometimes adopted the second interpretation.
Since Go 1.18, the official standard Go compiler always adopts
the second interpretation.
And Go 1.19 formally admitted the 1.18+ implementation is correct.

For example, the official standard Go compiler 1.17 thinks
the function `f` in the following code is valid
but thinks the function `g` is invalid.

```Go
package main

type T byte

//go:noinline
func f(s string) []T {
	return []T(s)
}

//go:noinline
func g(x []T) string {
	return string(x) // error (line 12)
}

func main() {
	_ = f("Go")
	_ = g([]T{71, 111})
}
```

Since Go toolchain 1.18, both of the two functions compile okay.
The followings are the outputs of the official standard
Go compiler 1.17 and 1.18 for the above program.

```
$ gotv 1.17. run main.go
[Run]: $HOME/.cache/gotv/tag_go1.17.13/bin/go run main.go
# command-line-arguments
./main.go:12:15: cannot use <node SPTR> (type *T) as type *byte in argument to runtime.slicebytetostring

$ gotv 1.18. run main.go
[Run]: $HOME/.cache/gotv/tag_go1.18.7/bin/go main main.go
```

The official standard Go compiler 1.17 sometimes views `[]T` as a "slice of bytes"
(in the function `f`), sometimes doesn't (in the function `g`).
If the `type T byte` type definition is changed into a type alias declaration `type T = byte` ,
then the function `g` is also viewed as valid by the official standard Go compiler 1.17.

The same situation happened for "slices of runes" (or "rune slice").

Note, up to now (Go 1.19), the `reflect` standard package has not been updated accordingly.
For example, the following program prints two `false`, but should print two `true` instead.

```Go
package main

import "reflect"

type T byte

func main() {
	var s string
	var x []T
	var sType = reflect.TypeOf(s)
	var xType = reflect.TypeOf(x)
	println(sType.ConvertibleTo(xType))
	println(xType.ConvertibleTo(sType))
}
```

References:

* https://github.com/golang/go/issues/23536
* https://github.com/golang/go/issues/23814
* https://github.com/golang/go/issues/53523

### A subtle detail in local constant declarations

What should the following program print?

```Go
package main

func main() {
	const (
		iota = iota
		Y
	)
	println(Y)
}
```

Some gophers think it should print `1`, which is actually the answer of
the official standard Go compiler 1.17.13-.
However, since the official standard Go compiler 1.18, the answer is `0`.
The followings are the outputs with different compiler versions:

```
$ gotv 1.17. run main.go
[Run]: $HOME/.cache/gotv/tag_go1.17.13/bin/go run main.go
1

$ gotv 1.18. run main.go
[Run]: $HOME/.cache/gotv/tag_go1.18.7/bin/go run main.go
0
```

Which answer is correct?

The Go specification says:

> Within a parenthesized const declaration list the expression list may be omitted from any but the first ConstSpec. Such an empty list is equivalent to the textual substitution of the first preceding non-empty expression list and its type if any. Omitting the list of expressions is therefore equivalent to repeating the previous list.

The description states the 1.18+ answer is correct, because the above program
is equivalent to

```Go
package main

func main() {
	const (
		iota = iota
		Y    = iota
	)
	println(Y)
}
```

The last `iota` is the local declared `iota`, which value is `0`.

Go 1.19 confirmed it was a bug in 1.17.13- versions.

Ref: https://github.com/golang/go/issues/49157

### A long-time type alias related bug was fixed

With the introduction of type alias in Go 1.8, a comparison bug was also introduced.

The following program should print `false`, but it prints `true`
when using the official standard Go compiler from version 1.8 to 1.17.13.

```Go
package main

type T struct{}
type S = T

type A = struct{ T }
type B = struct{ S }

func main() {
	var a A
	var b B
	println(interface{}(b) == interface{}(a)) 
}
```

The type aliases `A` and `B` denote different types (with different field names),
which is why the comparison should result `false`.

The bug was fixed in Go 1.18.

The followings are the outputs with different compiler versions:

```
$ gotv 1.17. run main.go
[Run]: $HOME/.cache/gotv/tag_go1.17.13/bin/go run main.go
true

$ gotv 1.18. run main.go
[Run]: $HOME/.cache/gotv/tag_go1.18.7/bin/go run main.go
false
```

Ref: https://github.com/golang/go/issues/24721

### The "named type" terminology came back to Go specification

With the introduction of type alias in Go 1.8, the "named type" terminology
was removed from Go specification. A new terminology "defined type" was
introduced to fill in the gaps caused by the removal of "named type".
However, [this made many old Go tutorials and articles become obsolete
and brought several other drawbacks][nt].

[nt]: https://github.com/golang/go/issues/35091

With the introduction of custom generics,
the "named type" terminology came back since Go 1.18,
which is helpful to clearly explain [many conversion and assignment rules] in Go.

[many conversion and assignment rules]: https://go101.org/article/value-conversions-assignments-and-comparisons.html

Now, a named type may be

* a predeclared type;
* a defined (non-custom-generic) type;
* an instantiated type (of a generic type);
* a type parameter type (used in custom generics).

Note, a type alias of an unnamed type is not a named type.

### Goexit signals may cancel already happened (recoverable) panics

Should the following program crash for panicking or exit normally?

```Go
package main

import "runtime"

func main() {
	c := make(chan struct{})
	go func() {
		defer close(c)
		defer runtime.Goexit()
		panic("bye")
	}()
	<-c
}
```

Before Go 1.19, there was not a clear answer.
Some gophers think it should crash, for the `"bye"` panic is never recovered.
But in the implementation of the official standard Go runtime,
a Goexit signal will cancel already happened (recoverable) panics in the current goroutine.
So, with the official standard compiler, the above program will exit normally.
[The behavior was decided as correct in Go 1.19][goexit].

[goexit]: https://github.com/golang/go/issues/35378

In my personal opinion, Goexit signals should not cancel panics,
so the above program should crash instead.
But I respect the final decision of the Go core team.

The final decision makes `runtime.Goexit` calls act as super `recover` calls.
For example, the following two goroutine functions behave the same.

```Go
func goroutine1() {
	defer runtime.Goexit()
	panic(1)
}

func goroutine2() {
	defer func() {
		recover()
	}()
	panic(1)
}
```

The official Go documentation explains the panic/recover mechanism so simply
that there are still several other details which don't get covered,
which is why the Go 101 book includes
[a special article to explain the panic/recover mechanism in detail][pr].

[pr]: https://go101.org/article/panic-and-recover-more.html

### More small compiler and runtime changes

There are also some undocumented small compiler and runtime optimizations made in recent Go toolchain versions.
Some of the small optimizations are mentioned in the [Go Optimizations 101] book.

[Go Optimizations 101]: https://go101.org/optimizations/101.html

