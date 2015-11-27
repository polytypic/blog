# Freedom of Expression has a Cost.

It is hard to argue against giants like Harold Abelson:

> Programs must be written for people to read, and only incidentally for
> machines to execute.

and Donald Knuth:

> Premature optimization is the root of all evil.

In fact, I don't, but there is an inherent trade-off: *introducing expressive
abstractions to make programs easier for people to read often also tends to make
the same programs more resource consuming for machines to execute*.

## Optimization is the elimination of abstraction

Ideally everyone would be using
[a sufficiently smart compiler](http://www.mlton.org) that would effectively
eliminate abstraction penalty, but in
[the absence of such technology](http://fsharp.org/), you will need to eliminate
abstraction penalty manually.  But therein lies the other side of the trade-off:
*optimization is often about eliminating the very abstractions that make
programs more readable*.

## Ubiquitous abstractions

If you are lucky, your profiler tells that you only need to optimize a few
localized spots of your program.  But what if that isn't the case?  What if the
source of inefficiency is the very *abstraction mechanism* that is *used
throughout your program* and there is no easy to way to make it faster?  That
can very well be the case with, for example, *monadic abstractions* such as
parser combinators and user space threads.

The performance of a parser written with monadic parser combinators depends
crucially on the performance of the basic monadic operations.  If the basic
operations are slow and cannot be optimized further then there isn't much that
can be done except to rewrite the whole parser.  This is why the internal
implementation of
[FParsec](http://www.quanttec.com/fparsec/about/changelog.html#v0_9.background-on-low-level-api-changes)
is manually optimized to death.  *Has the manual optimization of FParsec been
worth the trouble?*

## Performance can be an enabling feature

I've written a few parsers with FParsec, including a parser for a simple
programming language with a layout sensitive syntax.  In that project,
performance was important and I needed to manually optimize the parser.  In
terms of FParsec combinators, I mainly just made sure that parts of parsers were
not unnecessarily being reevaluated.  Mostly, however, I could just use the high
level monadic parser combinators to implement the parser in a manner that I
found fairly readable.  If the internals of FParsec weren't optimized as
aggressively as they are, I probably would have had to completely rewrite the
parser.  I'd argue that *performance can be an enabling feature*.  Without the
aggressive internal optimization, FParsec would at best be an interesting toy
and only suitable for prototyping.

## The cost of everything

Similarly, the performance of a program written with monadic user space thread
combinators depends crucially on the performance of the basic monadic
operations.  If the basic operations are slow and cannot be optimized further
then there isn't much that can be done except to rewrite the whole program.

As Alan Perlis has said

> LISP programmers know the value of everything and the cost of nothing.

Unfortunately I'm not a LISP programmer and I would like to be able to express
parallel, asynchronous and concurrent programs without having to worry about the
various costs.  For example, if it seems useful to
[spawn a new thread for every item to build](https://github.com/VesaKarvonen/Recalled/#background-material),
I'd rather do that than redesign the core of the system in another, possibly
less powerful, way.

It isn't the case that I would have optimized
[Hopac](https://github.com/VesaKarvonen/Hopac#performance), because it was fun.
It was fun for a day or two and meant eliminating much of the readability of the
internals of Hopac&mdash;a costly trade-off I don't particularly like, but pay
so that I can elsewhere *stop worrying about costs and gain freedom of
expression*.
