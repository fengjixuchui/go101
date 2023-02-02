
# Go Optimizations 101 Update History

### v1.20.a (2023/Feb/01)

* In the "Function inlining" section of the "Functions" chapter, mentions containing type delcarations will not prevent a function from being inlined since Go toolchain 1.20. Many examples using local type declaration `type _ int` are modified accordingly, including the one in the last section ("Grow stack in less times") of the "Stack and Escape Analysis" chapter.
* In the "Arrays and Slices" chapter, mentions that `copy(s, s)` is a no-op, but `append(s[:0], s...)` is not.
* In the "BCE (Bound Check Eliminate)" chapter, mentions that `-gcflags="-d=ssa/check_bce"` doesn't work for some generic functions.

## v1.19 (2022/Aug/28)

* the `Garbage Collection` chapter is updated much. The update contents are mainly in the last 4 sections (starting the `Use new heap memory percentage strategy to control automatic GC frequency` section). Some descriptions in the first section `GC pacer` are also modified a bit.
* in the `Stack and Escape Analysis` chapter, the descriptions about initial stack sizes in the `Stacks growth and shrinkage` section are modified.
* in the `BCE` chapter, a new `Example 4` section is added. The example was in the `Sometimes, the compiler needs some hints to remove some bound checks` section.
* in the `Functions` chapter, the `Which functions are inline-able?` section states that functions containing `select` code blocks may be inlined since Go toolchain 1.19.

### v1.17 (2021/Dec/22)

* published
