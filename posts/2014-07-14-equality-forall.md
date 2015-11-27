# Equality forall and forever

Having a *correct*, *robust*, *polytypic* implementation of equality is a
prerequisite for many kinds of applications.  What does this mean?

To be correct, equality should form a relation that is

* *reflexive*,
* *symmetric*, and
* *transitive*,

and the relation should

* hold *extensionally*, and
* be *invariant*.

To be robust, equality should always return an answer or report an
error&mdash;ideally at compile time&mdash;if equality cannot be determined.

And, of course, in a typed language, equality should ideally be polytypic,
because having a different equality predicate for every type or having to
manually write equality predicates for user-defined types would be more than a
nuisance.

Why are these properties important?  They are important, because many algorithms
and data structures, built upon a notion of equality, only work correctly as
long as these properties hold.  For example, if equality does not hold
extensionally, algorithms that eliminate duplication may give subtly incorrect
results.  Also, if equality is not invariant, it may be possible to break
invariants of algorithms and data structures.  On the other hand, if we do have
an equality that has these properties, we can implement generic algorithms based
upon equality.

## Problems with F#'s built-in equality

At first glance it seems that we are in luck.  F# comes with a built-in
polytypic notion of equality:

* First of all, most built-in types, like integers, floating point numbers, and
  arrays have been given a notion of equality.

* Furthermore, F# generates equality predicates for user-defined types.

Unfortunately the built-in equality of F# is neither correct nor robust.  Let's
examine a number of problems with the built-in equality that F# provides.

### Equality of floating point numbers

The special notion of equality of IEEE floating point numbers may be useful in
carefully constructed numerical algorithms.  However, consider what happens if
an application uses such a notion of equality in other contexts.  For example,
an application might compare aggregate values for equality in order to eliminate
duplicates and reduce memory usage.  One could hope that such elimination of
duplicates would eliminate all the duplicates without changing the meaning of
the program.

One problem is that `nan` is treated as unequal to any other value, including
`nan`:

```fsharp
> nan = nan ;;
val it : bool = false
> nan <> nan ;;
val it : bool = true
```

Aggregates containing `nan` values would never compare equal and duplicate
elimination would simply not work.  What this means is that the built-in
equality is *not reflexive*.

On the other hand, `-0.0` and `0.0` are considered equal:

```fsharp
> -0.0 = 0.0 ;;
val it : bool = true
```

But they are not the same:

```fsharp
> 1.0 / -0.0 ;;
val it : float = -infinity
> 1.0 / 0.0 ;;
val it : float = infinity
```

Eliminating duplicates in that case might actually change the meaning of the
program in subtle ways.  This means that the built-in equality *does not hold
extensionally*.

### Equality of mutable data structures

If you compare two objects for equality, should the result remain the same no
matter what?  For immutable values the answer is perhaps obvious.  What about
mutable objects?  The default equality of F# for mutable objects is *not
invariant*.  For example, if you define two arrays:

```fsharp
let xs = [|1|]
let ys = [|1|]
```

and compare them for equality:

```fsharp
> xs = ys ;;
val it : bool = true
```

the result depends on what is stored in the arrays:

```fsharp
> xs.[0] <- 2 ;;
val it : unit = ()
> xs = ys ;;
val it : bool = false
```

While it is not uncommon to need to compare mutable objects for equality based
on their contents, it can be argued that the default notion of equality for
mutable objects should be based on identity rather than structure.  Consider the
following example:

```fsharp
> let a = [|1|] ;;
val a : int [] = [|1|]
> let b = [|2|] ;;
val b : int [] = [|2|]
> let m = Set<array<int>> ([a; b]) ;;
val m : Set<int array> = set [[|1|]; [|2|]]
> m.Contains a ;;
val it : bool = true
> a.[0] <- 2 ;;
val it : unit = ()
> b.[0] <- 1 ;;
val it : unit = ()
> m.Contains a ;;
val it : bool = false
```

Innocent mutations broke the `Set` data structure.  While you may argue that
mutable objects should not be used as keys, the fact is that if the notion of
equality of mutable objects would be based on their identity, rather than
contents, the problem would not exist&mdash;mutating the keys would not break
the `Set` data structure.  What should be clear, is that proper functioning of
applications may crucially depend on preserving the identities of mutable
objects.

### Equality of cyclic data structures

Cyclic data structures can occasionally be useful.  Let's see.  Here is a simple
type:

```fsharp
type Cyclic = {mutable Cycle: option<Cyclic>}
```

that allows cycles to be constructed:
 
```fsharp
> let cycle = {Cycle = None} ;;
val cycle : Cyclic = {Cycle = null;}
> cycle.Cycle <- Some cycle ;;
val it : unit = ()
```

What happens if you compare `cycle` to itself?

```fsharp
> cycle = cycle ;;
```

Unfortunately, the built-in equality of F# is *not robust* and never produces a
result nor reports an error.

## Next steps

In a future followup, we will look at an implementation of equality that
fulfills the requirements described in the beginning of this article.
