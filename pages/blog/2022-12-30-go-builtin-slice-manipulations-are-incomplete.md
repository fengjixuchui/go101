
# Go Built-in Slice Manipulations Are Incomplete

## Lack of a performant way to merge 3+ slices

In Go, there are three built-in functions to manipulate slices:

1. the `make` function is used to create a slice with specified length and capacity. All the elements are set to zero values.
1. the `copy` function is used to copy some elements from one slice to another if the two slices have identical element types.
1. the `append` function is used to append some elements to a base slice and result in a new slice.
   The base slice might share all of its elements with the new slice or not,
   depending on whether or not the capacity of the base slice is large enough to hold all the appended elements.

When the capacity of the base slice if an `append` function call is not larger enough.
the call can be viewed as a way to merge two slices.
As the way is built-in, it is performant.

However, there is not a performant way to merge 3+ slices.
The following implementation might be the most performant way to do the task:

```Go
func merge[S ~[]E, E any](ss ...S) S {
	var n, allNils, k = 0, true, -1
	for i := range ss {
		if m := len(ss[i]); n != 0 {
			n += m
			if n < 0 {
				panic("sum of lengths is too large")
			}
		} else if m > 0 {
			n = m
			allNils = false
			k = i
		} else {
			allNils = allNils && ss[i] == nil
		}
	}
	if allNils {
		return nil
	}
	if n == 0 {
		return S{}
	}
	
	// Make use of this optimization:
	// https://github.com/golang/go/commit/6ed4661807b219781d1aa452b7f210e21ad1974b
	s := ss[k]
	r := make(S, n)
	copy(r, s)
	r = r[:len(s)]
	ss = ss[k+1:]

	for _, s := range ss {
		r = append(r, s...)
	}
	return r
}
```

Generally, a `make` call will zero the elements allocated during its execution, which is unnecessarily for this particular use case.
The implementation does its best to zero as few as possible elements within the `make` call,
by using the mentioned optimization, it doesn't zero the elements in `r[:len(ss[0])]`,
but it still fails to do so for the remaining elements (the ones in `r[len(ss[0]):]`).

If the `merge` function is built-in, then it is able to avoid all the unnecessary element zeroing operations.

## Lack of a clean way to clip slices

As mentioned above, an `append` function call will reuse the backing array
of the first slice argument (call it `base` here) for the result slice if the capacity of `base`
is large enough to hold all the appended elements.
For some scenarios, we want to prevent an `append` call from modifying the elements in `base[len(base):]`
and we achieve this by clipping the capacity of `base` to its length:

```Go
var x, y, z T

	... = append(base[:len(base):len(base)], x, y, z)
```

Nothing to complain about, except that it is some verbose,
in particular when the `base` expression is verbose, such as

```Go
	... = append(aValue.Field.Slice[:len(aValue.Field.Slice):len(aValue.Field.Slice)], x, y, z)
```

If the `base` expression is a function call, then we must store the result of a call in a temporary intermediate variable:

```Go
	base := aFunctionCall()
	... = append(base[:len(base):len(base)], x, y, z)
```

We can use the `Clip` function in the `golang.org/x/exp/slices` package, but this way is still not clean enough.

```Go
import "golang.org/x/exp/slices"

	... = append(slices.Clip((base), x, y, z)
	
	... = append(slices.Clip(aValue.Field.Slice), x, y, z)
	
	... = append(slices.Clip(aFunctionCall()), x, y, z)
```

I do perfer [this proposal], but it has been rejected:

```Go
aSlice[ : n : ]  // <=> aSlice[ : n : n]
aSlice[m : n : ] // <=> aSlice[m : n : n]
sSlice[ : : ]    // <=> aSlice[ : len(s) : len(s)]
aSlice[m : : ]   // <=> aSlice[m : len(s) : len(s)]
```

If the proposal is adopted, then we may just write the code as

```Go
	... = append(base[::], x, y, z)
	
	... = append(aValue.Field.Slice[::], x, y, z)
	
	... = append(aFunctionCall()[::], x, y, z)
```

which is much cleaner.

[this proposal]: https://github.com/golang/go/issues/25638


