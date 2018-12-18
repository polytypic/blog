---
title: A pattern matcher in JavaScript
---

# A pattern matcher in JavaScript

Vesa Karvonen

---

> it is possible to make the structure of the program match the structure of the
> problem being solved. — [Burge](http://jmct.cc/burge.html)


---

## Context

* A new format for exams
  * [Exams?](https://calmm-js.github.io/documentation/mental-optics/#/32)
* Old format: "JSON 1.0"
  * Missing metadata
  * Messy structure
  * Rendering rearranges DOM
* Problem: Where do we get the data from
  * Extract base structure from rendered HTML?
  * Problem: manipulating XML in JS

---

## How about

* Using an approach discussed in Neil Mitchell's
  [Uniplate](https://wiki.haskell.org/Uniplate) library
  * Used for transformations in a compiler
* Roughly
  * Bottom up transform to fixed point
  * Using pattern matching based rules
* [A sample rule](https://gist.github.com/polytypic/acfb84ae4e58a03cdd982ca412ac649b#file-sample-rule-js)

---

## First the trivial bits

---

## Bottom up transform...

```
const mapChildren = (fn, x) =>
  R.is(Object, x) ? R.map(fn, x) : x

const bottomUp = (fn, x) =>
  fn(mapChildren(x => bottomUp(fn, x), x))
```

[Playground](https://calmm-js.github.io/partial.lenses/playground.html#MYewdgzgLgBAtgQwA4GEAWBLANgEwE4CmYMAvDABQBmYANDAB4CUpAfDAEoB0GE5A8gCMAVgWBQ6TGAH4OnREiq0GzAFwMAUKEiwBIKFBBwAqklIVqE5iTbVy89NnxFy9VjF37DJxZcuN1muDQMDgIUAhmANoA3vRqMQCeagCMAL500QBe8QBMdADMALqpxYUBHgbGCq7WMFoQIFgEnFggAOYuzAA+XQx0oeH+APRDFV7VbpH0hf1hCMOjepXeNWxcPOQAgnh4CAmW0rKEAG4EeBAEnTBq9LOD6iNjVS5u67wAcgCucAJnBzIAWlcNzu8yAA)

---

## ...to fixed point

<a href="https://en.wikipedia.org/wiki/Fixed_point_(mathematics)">Fixed point</a>

```
f(x) === x
```

e.g.

```
x => x*x
```

has fixed points `0` and `1`.

---

## But as a transform...

```
const toFixedPoint = (fn, x) => bottomUp(
  x => {
    const y = fn(x)
    return R.equals(x, y) ? x : toFixedPoint(fn, y)
  },
  x
)
```

<small>(Avoiding equality might be advantageous)</small>

[Playground](https://calmm-js.github.io/partial.lenses/playground.html#MYewdgzgLgBAtgQwA4GEAWBLANgEwE4CmYMAvDABQBmYANDAB4CUpAfDAEoB0GE5A8gCMAVgWBQ6TGAH4OnREiq0GzAFwMAUKEiwBIKFBBwAqklIVqE5iTbVy89NnxFy9VjF37DJxZcuN1muDQMAYAYhj0BDgACiAYYLBkPspuHgbGCuqu1jAA3oHaMACeZrZM6oRQAK54xFwEAI5VCFi89HRFzDKuamERUbHxUMmd6gC+NFnq-gXBOAhQCGYA2rn0aqtFagCME3kAXhsATHQAzAC6Y5fnAX2RMXEJLm5cPOQAclVwAgR4ljAAMgBMAADDAADwMaQMAC02xganaMHmi0YQA)

---

## Pattern matching then

---

## Imagine

```js
const simplify = expression =>
  case expression of
    | ({type: 'sub', lhs, rhs}) =>
      ({type: 'add', lhs, rhs: {type: 'neg', arg: rhs}})
    | ({type: 'add', lhs: arg, rhs: arg}) =>
      ({type: 'mul', lhs: arg, rhs: 2})
    | ({type: 'neg', arg: {type: 'neg', arg}}) =>
      arg
    // ...
    | _ => expression
```

<small>constants - variables - strict matching - alternatives - equality</small>

---

From fantasy:

```js
const elimSum = expression =>
  case expression of
    | ({type: 'sub', lhs, rhs}) =>
      ({type: 'add', lhs, rhs: {type: 'neg', arg: rhs}})
    | _ => expression
```

To reality:

```js
const elimSub = rule((lhs, rhs) => [
  () => ({type: 'sub', lhs, rhs}),
  () => ({type: 'add', lhs, rhs: {type: 'neg', arg: rhs}})
])
```

[Playground](https://calmm-js.github.io/partial.lenses/playground.html#MYGwhgzhAEBqYCcCWYBGICm0De0C+AUAcAPYB2EALtALZiXAAW0AvNABQZkBuSC5NLpQA00AA71KGBGVEATemACUrAHw4CSAGYcJlKTOhIKlMGWAYSO+MjSYV2TTvYLT0AIQs2AVzJyMWsYYctAAZKEEnDx8AkIAdADmGJTsegZkKl4+fgFBIQA++QQASnEYAI7eYCAQUbz8ZIJklInJqZLSGfKKSr0aXPWxzXEQbWmd3aZKBAjJ3oaUCN4YBISzlPNk0FrVo6vQGDVY2hylSLUA8qgAVhjAIuIdMn2OJ+xnlzd3D67KDgRaEgIDikEzQADWRi2v2mb3GMgA2uCALqsLzQXz+QJkYLTdabba7FaA4HsUFUCFQx76Tqw5zuOgMRh1GKNISieFkJHIyZgbm9GZzQw7I6CjYLJYrNZCrYivZ4A5HDT4wylCpVGrtGkyXnTQiEYjkClLTCsaAmjAAMS2LHUjnJ1G4iBQ6AwMDYpUoSEEtXYmXUOIA7nBnXYMH7RBbrXFMGQEpRGNMHdAEZyACqMUSothRsjsOIFp22V0QJNG6ics3p5l4mXQX5qDTJgasprUNhB6AAWTAYj9Tg4jKYLIabY5T1k9Z6-2TTpAy3d0FKdD73EbLdH8SSKW4SlERZdmFLhrBCMjGDE4AsbYz2fN3kw1vzheqC9r4q2s0vYGvQgz-elD8p1MVZQJPClDm9ABlbxUDNC12HYEBGAgSMUP9FNIgw9hsEoABPMQMAALmgAByCBYNI0RkNQ80ULwPcsMbHD8MIkjSLAOQ5Co6AaLQiASNwgjiLInEEh4xAEhIhB6IYghkWmAhIJoGDUBY4T2Io1AeJokiAEZ+JIgAmBigA)

---

And further:

```js
const simplify = alternatives(
  rule((lhs, rhs) => [
    () => ({type: 'sub', lhs, rhs}),
    () => ({type: 'add', lhs, rhs: {type: 'neg', arg: rhs}})
  ]),
  rule(arg => [
    () => ({type: 'add', lhs: arg, rhs: arg}),
    () => ({type: 'mul', lhs: arg, rhs: 2})
  ]),
  rule(arg => [
    () => ({type: 'neg', arg: {type: 'neg', arg}}),
    () => arg
  ])
)
```

[Playground](https://calmm-js.github.io/partial.lenses/playground.html#MYGwhgzhAEBqYCcCWYBGICm0De0C+AUAcAPYB2EALtALZiXAAW0AvNABQZkBuSC5NLpQA00AA71KGBGVEATemACUrAHw4CSAGYcJlKTOhIKlMGWAYSO+MjSYV2TTvYLT0AIQs2AVzJyMWsYYctAAZKEEnDx8AkIAdADmGJTsegZkKl4+fgFBIQA++QQASnEYAI7eYCAQUbz8ZIJklInJqZLSGfKKSr0aXPWxzXEQbWmd3aZKBAjJ3oaUCN4YBISzlPNk0FrVo6vQGDVY2hylSLUA8qgAVhjAIuIdMn2OJ+xnlzd3D67KDgRaEgIDikEzQADWRi2v2mb3GMgA2uCALqsLzQXz+QJkYLTdabba7FaA4HsUFUCFQx76Tqw5zuOgMRh1GKNISieFkJHIyZgbm9GZzQw7I6CjYLJYrNZCrYivZ4A5HDT4wylCpVGrtGkyXnTQiEYjkClLTCsaAmjAAMS2LHUjnJ1G4iBQ6AwMDYpUoSEEtXYmXUOIA7nBnXYMH7RBbrXFMGQEpRGNMHdAEZyACqMUSothRsjsOIFp22V0QJNG6ics3p5l4mXQX5qDTJgasprUNhB6AAWTAYj9Tg4jKYLIabY5T1k9Z6-2TTpAy3d0FKdD73EbLdH8SSKW4SlERZdmFLhrBCMjGDE4AsbYz2fN3kw1vzheqC9r4q2s0vYGvQgz-elD8p1MVZQJPClqnSegkG4N0zWfOItAof1oAAD0bRwSRBctti2KxcNLJscIATzNJD2FQukOHcNVKl2CjRGIgUVS2YjQJYtCwOTCBvUvbRSLYSDOmg2DahmB9w3YEBGAgSMZJQhFIhQ9hsEoYixAwAAuaAAHIIG8VAdNEaTZPNGS8D3JTGxUtSNO0nSwDkOQjOgEy5IgbTVPUrTdJxBIXMQBJtIQcyLIIZFLItdhAsbRS-Wsry7N0xznOMmTtMC9yMoQBILOEKzbQ4RKfJ0mgHxckzsoSLLoAAJjCiL8qimLCri5TivsvyApyzzbJKrrRECvA8oK9RAvC6ZpgIHiaD4rRiJs7z7JSir0ugABmGr1osoA)

---

And to fixed point:

```js
toFixedPoint(
  simplify,
  {type: 'sub', lhs: 3, rhs: {type: 'neg', arg: 3}}
)
```

[Playground](https://calmm-js.github.io/partial.lenses/playground.html#MYGwhgzhAEBqYCcCWYBGICm0De0C+AUAcAPYB2EALtALZiXAAW0AvNABQZkBuSC5NLpQA00AA71KGBGVEATemACUrAHw4CSAGYcJlKTOhIKlMGWAYSO+MjSYV2TTvYLT0AIQs2AVzJyMWsYYctAAZKEEnDx8AkIAdADmGJTsegZkKl4+fgFBIQA++QQASnEYAI7eYCAQUbz8ZIJklInJqZLSGfKKSr0aXPWxzXEQbWmd3aZKBAjJ3oaUCN4YBISzlPNk0FrVo6vQGDVY2hylSLUA8qgAVhjAIuIdMn2OJ+xnlzd3D67KDgRaEgIDikEzQADWRi2v2mb3GMgA2uCALqsLzQXz+QJkYLTdabba7FaA4HsUFUCFQx76Tqw5zuOgMRh1GKNISieFkJHIyZgbm9GZzQw7I6CjYLJYrNZCrYivZ4A5HDT4wylCpVGrtGkyXnTQiEYjkClLTCsaAmjAAMS2LHUjnJ1G4iBQ6AwMDYpUoSEEtXYmXUOIA7nBnXYMH7RBbrXFMGQEpRGNMHdAEZyACqMUSothRsjsOIFp22V0QJNG6ics3p5l4mXQX5qDTJgasprUNhB6AAWTAYj9Tg4jKYLIabY5T1k9Z6-2TTpAy3d0FKdD73EbLdH8SSKW4SlERZdmFLhrBCMjGDE4AsbYz2fN3kw1vzheqC9r4q2s0vYGvQgz-elD8p1MVZQJPClqnSegkG4N0zWfOItAof1oAAD0bRwSRBctti2KxcNLJscIATzNJD2FQukOHcNVKl2CjRGIgUVS2YjQJYtCwOTFcAGFGCQEA5FmG0OCQ0RKMbD52CuW57nElQAH4lziFd2DEtCVAALk45NUBIfQSBoABVMR4PUiTbVw9heP4wThIoxs9IM4y+3MvcNOmcDqEoEhLSQVDggABRIYx21EycLPUJyfJc9gCHQyz7RIsi80osUCVojValQxjFLQ6BtJ8vyArkYLQrUycmNWYR4oITzkwgb1L20Ui2EgzpoNg2oZgfcN2BARgIEjQaUIRSIUPYbBKGIsQMG0gByCBvFQebRAGobzUGvA93GxtJum2aFrAOQ5FW6B1uGiBtKmma5ugeacQSM7EASbSEC27aCGRHaLXYF7GzGv09puw77uO061sG7SXsu6GEASbaaqByz9tuhaaAfM71rhhJYegAAmT7vpq37-sswGJpBu6HowJ7RBe66Dupx7nvhvBEd2yyXq+6ZPKK-ygpC5o4samhmq0YiaqphalpWyGrugABmPHpfuln6fh7TFfZuqgA)

---

## Room for improvement

* Inefficient
  * Examines pattern over and over
  * ...
* Inexpressive
  * Spread `...` patterns
  * `or`, `and`, and `not` patterns
  * ...
* Not modular
* Debuggability?
* Verbose (thunks not always needed)

---

## Efficiency: Staging

```js
const fn = (environment, pattern, data) =>
  // ...
```

```js
const fn = pattern => {
  // ...
  return (environment, data) =>
    // ...
}
```

[Playground](https://calmm-js.github.io/partial.lenses/playground.html#MYGwhgzhAEBqYCcCWYBGICm0De0C+AUAcAPYB2EALtJSQLJiXAAW0AvNAA6OUYJnsAfDgJIAZtAAU3Sr37QkFSmDLAMJCfGRpMAShEIMlAK7zJGMgDckCcgFsLlADTQAJozD62w7KImT3ZWgAQjYOYzJXDDFFDFdoADIEgnMrG3tHADoAcyNpHj4yLzDoCKiYsjjoAB9qggAlTIwAR2MwEAhU61syBzJKHLyZOSKXQM9dfV8Lboz+zIghgv4xj10CQxN5SgRjDAJCTdMBMXbFg4PoDA6scSlGpE6AeVQAKwxgZy5loqnicio0DsjBYfBgHEawM4kloDCYzBcw0K6yOZhm6V6jlWyi8Pj890yj0kL3en2xExEYhICCkpCU0AA1goBON1ndJMD4WCANoMgC67BKZWisVcKKMx2gpxuBCpNMkdMBTMUQJBzDBbP8wU5oIQEF5fK6GL6X3GBsmGwl8ml51RAh2ewudqlZ32hDwVxuBitAiNPRN5Nx0EaLTaHXyskKgYuhH+9N2mHY0ATGAAYgJvCJFdRLIgUOgMODg5lKEgHJ1JEHKgB3OB5nQYSsuFPpzKYMjZSjMdbZ6DcpH8AAqCOgAo4LbIkky09z2gLEB7AOoOvVNI4sLVEZGw8r4q2LI8QizS6uaX9jiTNegDGhmqkK74frmprWf17uZAeyLkLA0MsR-Rc95lyShJEsXQXFnfNMAXONAW5ZsME4cA1BNYcx2TYxMHTKcZ3aL890lQxkLAVDHB3dZDh9NwPBjIhe2uMsAGVjFQJMU0kSQQGYCBmx4oNuRSINJGwSgAE9OAwAAuaAAHIIFY2SXG43jkx4vAIKEo8RPEySZNksBXFcJToBUviIBk0SJOkuTKmyEzEGyGSEHUjSCD5dYCEYuwWNQHTrP0hTUBMlSZIARnMmSACYNKAA)

---

## Modularity

```js
const featureH = {
  is: any => boolean,
  toMatch: (any, toMatch) => matcher
}
```

```js
const handlers = [ variableH, objectH, constantH ]

const toMatch = pattern => {
  const handler = 
    R.find(handler => handler.is(pattern), handlers)
  if (!handler)
    throw Error("Unknown pattern")
  return handler.toMatch(pattern, toMatch)
}
```

[Playground](https://calmm-js.github.io/partial.lenses/playground.html#MYGwhgzhAEBqYCcCWYBGICm0De0C+AUAcAPYB2EALtAG6IroYAS0AvDgUhAFzQBKAOi4AKeMjSYAlABoClEgFkwlYAAteAB2WUMCMmwB80YRjI0kCcgFtTladAAmysJMMckAM2NPKYaAEJWdgBXMgcMDyQyDAdoADI4ghMzC2tbAQBzDEphLUodPVcgkLCIqJjoAB9KgkEMAEdgsBAIZPNLMhsySkzs3O1dMhlHZ0kxjlN2tO6BCD68grJ7HxcCBGzgvWhKBGCMAkJ1yk39D2a5g8vicipoElQAKwxgShZ2bE4efiFWgHlH56UGRyRTKNS8fr5Qb2eRKFSqIpGD6kCjUKxg1S6GDsQTojTCWEY+wLQaSNYbLZtVKdWzLUZuD6eYyCET-J4vOm+cYfDwkBDGFG3ADW0CiIy5nC8wnR8KxAG0hQBdNhBaChcKRaIOMlHE7QM4tfa8-nCQXUEVimVqLFkpnCfxWzEICAKxVUjpdOzisCusbk45bA0XXVbHZ7S4h07nfaEWNEM3QM1gbpvdxfQSkKwaTCe4Qsv4Al5jWSE+GaAZbVhGd3TL0rRHfBpNFqQxac1aEa6o6CqZMOTDOtjQOV0cSMJj2e7s172JMpxXxm7UUtqIckytIru3XthAdDwSahzCHf93RuE8Dn6t0n2C82yXGfx3hBkyiqSwAd2gAFEEJYEMIABEACqZBCmQJAfvo65kIBOoUvoz4CCuqjXnoMKgvCZKdgmuyYEOeEYAAYvoVYcAmo4MJg2LfJQSA2K0wgNtEX5iFRGBMfYhEkQImBkBkb5kgmcowQAKqo9jKuw3FkMIAjyZREgYBAQlLtAjpnuwKFoWQ4lMfBAb6CsDJbtQkzUp6Q4sdASj4raUoaQB5kerS3rcqZtDNHsNG4mA+I0G4zm1r0OQ0MMimMCpHlylxGDZmAwAYJ64lSdAMlyQpXnKQZerrPFiXJah2H+nqKyXDhakYCA9EAMrBKgBHBJgwjCCAqgQFx7UNnKSQNsI2CUAAnhoGC8AA5BA9VjfYbUdWl7V4MCTFuP1Q0jeNYAOA403QLNnVfANw2jdAY3RBkO2IBkvAIAti0EIqZIEFVtX1atR3jZNqA7bNvAAIz7bwABMi1AA)

---

<img src="how-to-draw-an-owl.jpg">

[Gist](https://gist.github.com/polytypic/acfb84ae4e58a03cdd982ca412ac649b)

---

## Pros and cons

* Pros
  * PM based rules are fairly easy to understand
  * No need to manually traverse the XML tree
* Cons
  * Debugging
    * Why doesn't my rule trigger?
  * Verification
    * Is the result as wanted?

---

## Questions?

---

## Background and further

* SICP pattern matcher
* Pattern matching combinator libraries
* Recursion schemes
* Prettier printer
* Expression problem
* HOAS
* ...
