---
title: The future is algebraic
---

# The future is algebraic

Vesa Karvonen

---

## Poll

<!-- .element: class="fragment" -->Heard of free monads and composable handlers?

<!-- .element: class="fragment" -->Have you heard of *algebraic effects* before?

<!-- .element: class="fragment" -->See why someone would want algebraic effects?

<!-- .element: class="fragment" -->Have you tried programming with them?

<!-- .element: class="fragment" -->Do you know theory behind them?

---

## Motivation

<!-- .element: class="fragment" -->[What Color is Your Function?](http://journal.stuffwithstuff.com/2015/02/01/what-color-is-your-function/)

<!-- .element: class="fragment" -->[Monad tutorials timeline](https://wiki.haskell.org/Monad_tutorials_timeline)

---

## Effects

* Mutable state
* IO (Http, Db, Disk, ...)
* Exceptions
* Threads
* Asynchrony
* Generators
* ...

---

## And more effects

* Randomness
* Reader
* Parsing
* Non-determinism
* Reactive values
* ...

---

## Algebraic?

<small>*Paraphrased* from
[Leijen](https://www.youtube.com/watch?v=BfsnvKggmes&t=04m47s):</small>

<blockquote>
  <small>Signature of effect operations forms a free algebra giving rise to a
  free monad.  Free monads provide way to give semantics to effects, where
  handlers describe a fold over the algebra of operations.</small>

  <small>Operationally one can see algebraic effects as resumable
  exceptions.</small>
</blockquote>

<small>Personally I think of them as more structured delimited
continuations.</small>

---

## Exceptions 1/4

<small>(Snippets from [Algebraic effects for Functional
Programming](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/08/algeff-tr-2016-v3.pdf).)</small>

```
effect exc {
  fun raise( s : string ) : a
}
```

```
fun safediv( x, y ) {
  if (y == 0) then raise("divide by zero") else x / y
}
```

---

## Exceptions 2/4

```
fun catch(action, h) {
  handle(action) {
    raise(s) -> h(s)
  }
}
```

```
catch : ( action : () → ⟨exc | e⟩ a, h : string → e a) → e a
```

---

## Exceptions 3/4

```
fun zerodiv(x, y) {
  catch( { safediv(x, y) }, fun(s) { 0 } )
}
```

---

## Exceptions 4/4

```
fun to-maybe(action) {
  handle(action) {
    return x -> Just(x)
    raise(s) -> Nothing
  }
}
```

```
to-maybe : (() → ⟨exc | e⟩ a) → e maybe⟨a⟩.
```

---

## State 1/2

```
effect state<s> {
  fun get() : s
  fun put( x : s ) : ()
}
```

```
fun counter() {
  val i = get()
  if (i <= 0) then () else {
    println("hi")
    put(i - 1)
    counter()
  }
}
```

---

## State 2/2

```
val state = handler(s) {
  return x -> (x, s)
  get() -> resume(s, s)
  put(st) -> resume(st, ())
}
```

```
state : (x : s, action : () → ⟨state⟨s⟩| e⟩ a) → e (a,s)
```

```
> state(2, counter)
hi
hi
```

---

## Let's try some Koka!

---

## A list mapping function

```
fun elems(fn: a -> e b) : (list<a> -> e list<b>) {(
  fun(xs) { map(xs, fn) }
)}
```

<small>(The above reuses the overloaded `map` function from Koka's library.  The
type annotations are needed because of the overloading.)</small>

---

## Map 1st or 2nd elem of a pair

```
fun e1o2(fn) {(
  fun(t2) {
    match(t2) {(x1, x2) -> (fn(x1), x2)}
  }
)}

fun e2o2(fn) {(
  fun(t2) {
    match(t2) {(x1, x2) -> (x1, fn(x2))}
  }
)}
```

---

## Let's get real

That was some boring stuff from the 20th century.

What about stuff from this millennium like optics?

<!-- .element: class="fragment" -->Well, actually we just implemented them...

---

## Effect polymorphism

<small>The previous mapping functions weren't just polymorphic in the types of
the elements of the lists and pairs being manipulated.</small>

<small>The functions were also polymorphic in the *effects* that could be
performed.</small>

So, let's create an effect:

```
effect optic<x, y> {
  fun focus(x: x) : y
}
```

---

## View

```
fun view(optic, data) {
  (handler {
    return x -> error("no focus")
    focus(x) -> x
  }){ (optic(focus))(data) }
}
```

<small>(Could also return an option.)</small>

---

## Over

```
fun over(optic, data, fn) {
  optic(fn)(data)
}
```

<small>(No effects needed here.)</small>

---

## Collect

```
fun collect(optic, data) {
  reverse((handler(xs) {
     return x -> xs
     focus(x) -> resume(x, Cons(x, xs))
   })([]){ (focus.optic)(data) })
}
```

---

## So...

```
> val optic = o(elems, o(elems, e1o2))
optic : forall<a,b,c,e> (x : (a) -> e b) -> ((list<list<(a, c)>>) -> e list<list<(b, c)>>)

> val data = [[(3,"a")],[(1,"b"),(4,"c")]]
data : list<list<(int, string)>>

> view(optic, data)
3

> collect(optic, data)
[3,1,4]

> over(optic, data, negate)
[[(-3,"a")],[(-1,"b"),(-4,"c")]]
```

---

## That is all folks!

---

## Links

* [Asynchrony with Algebraic Effects](https://www.youtube.com/watch?v=hrBq8R_kxI0)
* [Freer Monad, More Extensible Effects](https://www.youtube.com/watch?v=3Ltgkjpme-Y)
* [Structured asynchrony with algebraic effects](https://www.youtube.com/watch?v=BfsnvKggmes)
* [Effective Programming: bringin algebraic effects and handlers to OCaml](https://www.youtube.com/watch?v=ibpUJmlEWi4)
* [Algebraic effects and handlers](https://www.youtube.com/watch?v=atYp386EGo8&list=PL0DsGHMPLUWX4YrLzwJGcqd-XsmXrE_Vn)
* ...lots [more](https://www.youtube.com/results?search_query=algebraic+effects)
