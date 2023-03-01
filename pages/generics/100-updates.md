
# Go Generics 101 Update History

### v1.20.d (2023/Feb/23)

* since Go 1.20, local type declarations are allowed within generic function bodies.
  So the limitation is removed from the "The Status Quo of Go Custom Generics" chapter.

### v1.20.a (2023/Feb/01)

* add [comparable vs. strictly comparable](555-type-constraints-and-parameters.md#strictly-comparable) and
  [type implementation vs. type satisfaction](555-type-constraints-and-parameters.md#implementation-vs-satisfaction) sections in the
  [constraints and type parameters]((555-type-constraints-and-parameters.md) chapter.

{#implementation-vs-satisfaction}
## Type implementation vs. type satisfaction

### v1.19 (2022/Aug/29)

* removed the "Aliases to non-basic interface types are not supported currently" section from [The Status Quo of Go Custom Generics](https://go101.org/generics/888-the-status-quo-of-go-custom-generics.html), because the limitation was removed from Go 1.19.

### v1.18 (2022/Apr/06)

* Published.
