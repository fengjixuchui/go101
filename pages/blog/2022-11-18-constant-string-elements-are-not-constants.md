
# It Is a Pity That Byte Elements of Go Constant Strings Are Not Constants. And It Is a Luck

Does the following Go program compile okay?

```Go
package main

const S = "Go"

const G = S[0]

func main() {}
```

Many gophers might think it should compile okay.
But it fails to compile.
Because element values of a string are always non-constant in Go,
even if the string is a constant.

It is a pity that Go 1.0 didn't specify that elements of constant strings are also constants.

For backward-compatibility, the pity is hard to make up.
For example, currently, the following program compiles okay.
It prints `185` (-71 + 256).

```Go
package main

const S = "Go" // S[0] == 71

func main() {
	println(-S[0])
}
```

But if elements of constant strings become into constants, then the program will not compile,
because the type of `-S[0]` is `byte` (aka. `uint8`), whereas `-71` owerflows the range of `byte`.

It is a pity, it is also a luck.

If elements of constant strings are constants, then there will be parsing ambiguities in Go custom generic age.
Since Go 1.18, the following line will be treated as generic type declaration,
in which `S` is a type parameter and `[4]*int` is its constraint.

```Go
type T [S [4]*int] struct{}
```

However, if elements of constant strings are constants,
then `S [4]` may be treated as constant `byte` value if `S` is a constant string.
In the parsing phase, the compiler doesn't know what the identifier `int` denotes.
A type or an integer constant? If it denotes an integer constant,
and `S [4]` may be treated as constant `byte` value,
then the above line is a valid ordinary array type declaration.

Luckily, now, Go compiler knows `S [4]` will never be constant,
so it always tries to think the above line is not an ordinary array type declaration.
No ambiguities happen here.
Were elements of constant strings constants, the simplified type parameter declaration syntax couldn't be possible.

There is [a proposal] to let Go support constant arrays (and other constant composite values).
However, the Go custom syntax design has almost sentenced that proposal to death.
Because, similarly, the following code will lead to parsing ambiguities
if array values may be declared as constants.

```Go
const A = [2]int{1, 2}
type BoolArray [A [1] * int]bool
```

To avoid absolutely sentencing the constant array proposal to death,
elements of constant array must not be treated as constants,
just as elements of constant strings are not treated as constants.

[a proposal]: https://github.com/golang/go/issues/6386

### BTW, substrings are also never constants

The following code line doesn't compile in Go:

```Go
const _ = "Google"[:2]
```

Instead, the following line compiles:

```Go
var _ = "Google"[:2]
```

By this fact, the following program will print `128 0` (the reason is explained [here]).

```Go
package main

const S = "Go"

var a byte = 64 << len(S) / 2
var b byte = 64 << len(S[:]) / 2

func main() {
	println(a, b) // 128 0
}
```

So [making substrings of constant strings constant will also break backward-compatibility][issues#28591].

[here]: https://go101.org/quizzes/operator-3.html
[issues#28591]: https://github.com/golang/go/issues/28591#issuecomment-697357430



