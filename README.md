# Traversy

[![Build Status](https://travis-ci.org/ctford/traversy.png)](https://travis-ci.org/ctford/traversy)

[![Clojars Project](http://clojars.org/traversy/latest-version.svg)](http://clojars.org/traversy)

An experimental encoding of multilenses in Clojure.

## What are multilenses?

Simply put, multilenses are generalisations of `sequence` and `update-in`. Traversy's `view` and `update`
accept a lens that determines how values are extracted or updated.

`update-in` provides a way to apply a function within a nested map:

```clojure
(-> {:x 2 :y 4} (update-in [:x] inc))
=> {:x 3 :y 4}
 ```
 
This works great so long as the value you want to update can be addressed by a single path. However,
there is no function for updating every value in a map in clojure.core. Here's how it looks with the
`all-values` lens:

```clojure
(require ['traversy.lens :refer :all])
    
(-> {:x 2 :y 4} (update all-values inc))
=> {:x 3 :y 5}
```

The same lens can also be used for viewing the foci:

```clojure
(-> {:x 2 :y 4} (view all-values))
=> (2 4)
```

Lenses can be easily composed, so it's easy to build one that suits your particular data structure:

```clojure
(-> [{:x 1 :y 2} {:x 2 :y 7}] (update (*> each all-values) inc))
=> ({:x 2 :y 3} {:x 3 :y 8})
```

And as viewing also composes:

```clojure
(-> [{:x 1 :y 2} {:x 2 :y 7}] (view (*> each all-values)))
=> (1 2 2 7)
```

As lenses are first class, once you have one that suits your needs, you can name it and put it in a var.

## Usage

See the [examples](test/traversy/test/lens.cljc).

There is [API documentation](https://ctford.github.io/traversy/traversy.lens.html), which describes the operations and provided lenses. This was generated by [Codox](https://github.com/weavejester/codox).

## Background

At the 2014 Clojure eXchange [I gave a talk about Lenses in general, and Traversy
specifically](https://skillsmatter.com/skillscasts/6034-journey-through-the-looking-glass).

## Laws

Lenses follow some rules that make them behave intuitively. The first two rules are the [Traversal Laws](http://hackage.haskell.org/package/lens-2.3/docs/Control-Lens-Traversal.html#t:Traversal).
The final rule governs the relationship between `update` and `view`.

An `update` has no effect if passed the `identity` function:

```clojure
(-> x (update l identity)) === x
```

Fusing two updates together is the same as applying them separately:

```clojure
(-> x (update l f1) (update l f2)) === (-> (update l (comp f2 f1)))
```

`update` then `view` is the same as `view` then `map`:

```clojure
(-> x (update l f) (view l x)) === (->> x (view l) (map f))
```

These should hold for any lens `l` that applies to a data structure `x`.

The second rule can be violated when the foci of a lens change after an `update`. An example of
this is when `only` is used with a predicate and function that interact.

These two expressions should have the same value, but as incrementing an odd number makes it even,
the second update in the first example has no targets:
```clojure
(-> [1 2 3] (update (only odd?) inc) (update (only odd?) inc)) => [2 2 4]
(-> [1 2 3] (update (only odd?) (comp inc inc))) => [3 2 5]
```

Careful when doing this - and please document any lenses that have this behaviour as unstable. Traversy
comes with three unstable lenses: `only`, `maybe` and `conditionally`.

## FAQs

### Aren't these just degenerate Lenses?

Yes! In fact, they're degenerate
[Traversals](http://hackage.haskell.org/package/lens-2.3/docs/Control-Lens-Traversal.html), with the `Foldable` and
`Functor` instances and without the generality of traversing using arbitrary `Applicatives`.


### Will updates preserve the structure of the target?

Yes. Whether you focus on a map, a set, a vector or a sequence, the structure of the target will remain
the same after an update.

### Can I compose these Lenses with ordinary function composition?

No. Unlike [Haskell Lenses](http://hackage.haskell.org/package/lens), these are not represented as functions.
You can, however, use `combine` (variadic form `*>`) and `both` (variadic form `+>`) to compose lenses.

### Can I use Traversy with ClojureScript?

Yup!

### How do I run the tests?

Clojure: `lein test`

ClojureScript: `lein test-cljs` (you'll need phantomjs)

both: `lein test-all`

### Is this stable enough to use in production?

Traversy is in production use on the project it originated from, but the API may yet change.
