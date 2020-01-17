## About this document

This document is not an attempt at explaining category theory in full. Its primary purpose is to explain the interfaces, functions, etc. that exist in the [`fp-ts` library](https://gcanti.github.io/fp-ts/), many of which depict abstract concepts of category theory and function programming.

The goal of this article is to weed through that, so that you can better understand how to use the library in practice.

While this document is not aimed at people who understand category theory already, if you do already have a basis in it (or just don't have any interest), you can skip down to the [Utility Functions](#utility-functions) section to get started on some of the functions that are provided by the library.

## First, Some Words

This is a glossary of words which I feel is better to define early, as they are used to define other terms below but don't have any explicit definition in `fp-ts`. Feel free to skip over them for now if they feel too abstract. I will like back to them below when they are first introduced in a more concrete definition.

### Counting Numbers and Counting Numbers

There is some ambiguity over whether Natural and Counting Numbers include the number 0. Since this article uses these terms frequently, we are going to disambiguate them here. We are TypeScript programmers after all.

When you think of Counting Numbers, think of a child. If you tell them to count to 3 they say "1, 2, 3", not "0, 1, 2, 3". So Counting Numbers will exclude 0, while Counting Numbers will include it. Easy pneumonic.

### Binary Operation

> an operation that combines two elements to form another element

In `fp-ts`, when we discuss binary operations, we are generally speaking of _closed_ binary operations.

> an operation that combines two elements to form another element WITHIN THE SAME SET

    // our set
    Counting Numbers { 1, 2, 3, 4, 5, ... }

    // binary operations (take to parameters)
    +, -

    // closed binary operation
    2 + 2 // returns 4, which is in our set

    // open binary operation
    2 - 2 // returns 0, which is not in our set

#### Morphism

> a structure-preserving map from one mathematical structure to another one of the same type

As a more concrete example, you can think of JavaScript functions as morphisms, which take an input and return an output (though that output can certainly be the same input!)

![morphism chart example](https://jeremykun.files.wordpress.com/2013/02/pairs-category.png)

Here the Z, B, and C are our mathematical structures and f and g are our morphisms.

#### Composition

Composition is represented with `∘`

Composition of morphisms means that if you have morphism of type `A -> B` and morphism of type `B -> C`, you can compose them into one morphism `A -> C`.

For example, if we have a function `addTwo` and a function `multiplyByFour`, we can compose them into one function `addTwoThenMultiplyByFour`. Of course, composition is not commutative. `addTwoThenMultiplyByFour` will return a different result than `multiplyByFourThenAddTwo`.

#### Mathematical Properties

| Property     | Definition                                                                          |
| ------------ | ----------------------------------------------------------------------------------- |
| Identity     | a = a                                                                               |
| Commutative  | a + b = b + a                                                                       |
| Associative  | ( a + b ) + c = a + ( b + c )                                                       |
| Distributive | a \* ( b + c ) = ( a \* b ) + ( a \* c )                                            |
| Annihilation | a \* 0 = 0                                                                          |
| Inverse      | In the set, there exists some element `x` for `y` that results in `y = y \* x \* y` |
| Idempotency  | x ^ x = 0                                                                           |
| Absorbtion   | meet: `x ^ (x v y) = x` join: `x V (x ^ y) = x`                                     |

## Who am I? Where do I fit in?

### [`Identity.ts`](https://gcanti.github.io/fp-ts/modules/Identity.ts.html)

One of the foundations of mathematics (and indeed all reality!) is that everything is equal to itself.

This is called the identity property.

This module simply creates a `type alias` to represent this:

`Identity<A> = A`

This module also contains a number of functions (`map`, `chain`, `ap`, etc.) that work on `Monad`s, which `Identity` is member of (more on `Monad`s later).

### [`Eq.ts`](https://gcanti.github.io/fp-ts/modules/Eq.ts.html)

You can read the "Getting started" documentation on `Eq` on the `fp-ts` [documentation page](https://gcanti.github.io/fp-ts/getting-started/Eq.html) as well.

Closely connected with `Identity` is `Eq`. `Eq` contains types that admit _equality_. They must have a function defined on them named `equals`. This function must adhere to the laws of:

1. reflexivity (`equals(x, x) === true`)
2. symmetry (`equals(x, y) === equals(y, x)`)
3. transitivity(`equals(x, y) === true` and `equals(y, z) === true`, then `equals(x, z) === true`)

### [`Ord.ts`](https://gcanti.github.io/fp-ts/modules/Ord.ts.html) and [`Ordering.ts`](https://gcanti.github.io/fp-ts/modules/Ordering.ts.html)

Again, like `Eq` you can read the documentation on `Ord` on the [documentation page](https://gcanti.github.io/fp-ts/getting-started/Ord.html).

`Ord` is used to represent the concept of order. In order to compare things, we must first know if they are equal, so `Ord` must also be of type `Eq`. It must contain a definition for `equals`, as well as its own `compare`.

We say that:

1. `x < y` if and only if `compare(x, y)` is equal to -1
2. `x === y` if and only if `compare(x, y)` is equal to 0
3. `x > y` if and only if `compare(x, y)` is equal to 1

`fp-ts` defined `-1`, `0`, and `1` in the `Ordering` type.

Instance of `Ord` must adhere to the following laws:

1. reflexivity (`compare(x, x) === 0`)
2. antisymmetry (`compare(x, y) <= 0` and `compare(y, x) <= 0` then `x === y`)
3. transitivity(`compare(x, y) <= 0` and `compare(y, z) <= 0`, then `compare(x, z) <= 0`)

### [`Bounded.ts`](https://gcanti.github.io/fp-ts/modules/Bounded.ts.html)

The `Bounded` type class is exactly what it sounds like: it contains an lower _and_ upper limit, such that `lower <= a <= upper`. It must also be a member of `Ord`. After all, how can we know if a member of the `Bounded` is between the lower and upper if we don't know how to compare them it.

### [`Show.ts`](https://gcanti.github.io/fp-ts/modules/Show.ts.html)

The `Show` type class is also aptly named. It represents types that can be converted into a human-readable string.

`show(true) === "true"`
`show(2) === "2"`
`show([]) === ""`

---

## Category Theory: The Nitty-Gritty

Let's start with the modules that represent abstract concepts and are perhaps less useful in everyday programming.

### [Set.ts](https://gcanti.github.io/fp-ts/modules/Set.ts.html)

Disregard the previous comment for a second. This module contains a lot of useful functions for the [JavaScript `Set` object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set).

But before we go there, let's explain what a set is.

|            |                                             |
| ---------- | ------------------------------------------- |
| Definition | a well-defined collection of unique objects |
| Extends    | N/A                                         |

##### Example

    { 2, 4, 6}
    { true, false }
    { 'Hi', 'Hello' }

##### Counterexample

    { 2, 4, 2 } // not a set since not all elements are unique

##### What this module contains

Functions for operating on a `Set`, for example: `every`, `toArray`, `remove`, etc, all of which are fairly self-explanatory.

### [Magma.ts](https://gcanti.github.io/fp-ts/modules/Magma.ts.html)

Okay, back to the ethereal stuff. Yay!...
| | |
| ------------ | ------------------ |
| Definition | Set equipped with a single [**closed** binary operation](#binary-operation) |
| Extends | Set |
| What it Adds | A single closed binary operation (`concat` in `fp-ts`) |

##### Example

    (Counting Numbers, +)
    1 + 2 = 3 // 3 is a Counting Number
    (All Strings, concat)
    "Hello" concat " " concat "World" = "Hello World" // "Hello World" is a string

##### Counterexample

    (Counting Number, -)
    1 - 2 = -1 // -1 is not a Counting Number

##### What does this module contain?

A single interface that defines a Magma containing a non-empty set with the binary operation `concat`, which combines them into a single set, removing all duplicates.

### [Semigroupoid.ts](https://gcanti.github.io/fp-ts/modules/Semigroupoid.ts.html)

Semigroupoid? Sounds sexy 🤔

|              |                                                                                        |
| ------------ | -------------------------------------------------------------------------------------- |
| Definition   | Set containing 1+ [morphisms](#morphism) which can be composed associatively           |
| Extends      | Magma                                                                                  |
| What it Adds | [Composition](#composition) of morphisms which is [associative](#associative-property) |

In this case, `f ∘ ( g ∘ h ) = ( f ∘ g ) ∘ h = f( g( h ) )`

##### What this module contains

Several `Semigroupoid` interfaces that all implement the `compose` morphism.

### [Category.ts](https://gcanti.github.io/fp-ts/modules/Category.ts.html)

`fp-ts` has a [page](https://gcanti.github.io/fp-ts/getting-started/Category.html) dedicated to `Category`s in "Getting Started".

|              |                                                                   |
| ------------ | ----------------------------------------------------------------- |
| Definition   | Semigroupoid with an identity morphism.                           |
| Extends      | Semigroupoid                                                      |
| What it Adds | Identity morphism, which will return `a` when given `a`: `a -> a` |

Identity morphisms require the existance of an identity element

> a special type of element of a set with respect to a binary operation on that set, which leaves any element of the set unchanged when combined with it (Wikipedia)

##### Example

    Set
        all strings including an empty string ('a', 'b', 'supercalifrag'...)
    Morphisms
        concat (a, b) => a + b;
        id = concat(a, '');
        // 'Hello' === 'Hello'

##### Counterexample

    Set
        all strings excluding an empty string ('a', 'b', 'supercalifrag'...)
    Morphisms
        concat (a, b) => a + ' ' + b;
        notId => concat(a, ' ');
        // 'Hello' !== 'Hello '

##### What this module contains

Several `Category` interfaces that all implement the `id` morphism.

### [Semigroup.ts](https://gcanti.github.io/fp-ts/modules/Semigroup.ts.html)

`fp-ts` has a detailed explanation of `Semigroup`s on their ["Getting Started" page](https://gcanti.github.io/fp-ts/getting-started/Semigroup.html).

|              |                                                                                    |
| ------------ | ---------------------------------------------------------------------------------- |
| Definition   | A group of objects and morphisms that allows for composition, which is associative |
| Extends      | Semigroupoid, Magma                                                                |
| What it Adds | N/A (combines rules governing Semigroupoid and Magma)                              |

Semigroups, like semigroupoids and magmas, do not require an identity element like categories do. Conversely, categories don't require totality/closure (think closed operations), like semigroups do.

![group-like structures table](http://xahlee.info/math/i/group-like_structures_2019-07-25_wd7p2.png)

##### Examples

See [Category.ts](#categorytshttpsgcantigithubiofp-tsmodulescategorytshtml)

##### Counterexamples

See Category.ts

##### What does this module contain?

A single `Semigroup` interface extending `Magma` and its `concat`, several constants of various types of `Semigroup`s, and several functions for creating `Semigroup`s of a given type or converting type classes to `Semigroup`s.

Additionally, there is a `fold` function, which is similar to the JavaScript `Array.prototype.reduce`, but instead of taking a function, it uses the `concat` that it inherits from `Magma`.

### [Monoid.ts](https://gcanti.github.io/fp-ts/modules/Monoid.ts.html)

Like `Semigroup`, `fp-ts` dedicated a [page](https://gcanti.github.io/fp-ts/getting-started/Monoid.html) for `Monoid` as well.

> an algebraic structure with a single associative binary operation and an identity element.

|              |                                                                                     |
| ------------ | ----------------------------------------------------------------------------------- |
| Definition   | Semigroupoid with an identity morphism, or category whose morphisms exhibit closure |
| Extends      | Semigroup, Category                                                                 |
| What it Adds | N/A (combines rules governing Semigroup and Category)                               |

You'll notice that all of these build on each other. That's why I didn't just say, "Don't bother with these, you'll never use them." Yeah, you probably won't, but since they affect some of the modules you _will_ use, let's burn the midnight oil and get through these concepts.

##### What this module contains

A single `Monoid` interface extending `Semigroup` with an `empty` property (this is the identity element), several constants of various types of `Monoid`s, and several functions for creating `Monoid`s of a given type or converting type classes to `Monoid`s.

Additionally, there is a `fold` function, which is similar to the JavaScript `Array.prototype.reduce`, but instead of taking a function, it uses the `concat` that it inherits from `Semigroup`.

### [Group.ts](https://gcanti.github.io/fp-ts/modules/Group.ts.html)

|              |                                                             |
| ------------ | ----------------------------------------------------------- |
| Definition   | Monoid with an [inverse](#mathematical-properties) property |
| Extends      | Monoid                                                      |
| What it Adds | Inverse property                                            |

![algebraic structures between magmas and groups](https://upload.wikimedia.org/wikipedia/commons/thumb/d/d0/Magma_to_group2.svg/300px-Magma_to_group2.svg.png)

##### Example

    (Integers excluding 0, +)
    5 + -5 + 5 = 5
    // If we included 0, this would still hold as there is +0 and -0.
    // However, we are excluding 0 to distinguish from Semirings,
    // which will be explained next.

##### Counterexample

    (Counting Numbers, *)
    5 * ??? * 5 = 5
    // Counting Numbers don't include fractions

##### What this module contains

A single `Group` interface extending `Monoid` with an `inverse` property, which calculates the inverse of an element in the `Group`.

### [Semiring.ts](https://gcanti.github.io/fp-ts/modules/Semiring.ts.html)

|              |                                                                                                                    |
| ------------ | ------------------------------------------------------------------------------------------------------------------ |
| Definition   | A monoid equipped with addition and multiplication, both of which exhibit the identity and associative properties. |
|              | Its multiplication also exhibits the [annihilation and distributive](#mathematical-properties) properties.         |
|              | Its addition additionally exhibits the commutative properties.                                                     |
| Extends      | Monoid                                                                                                             |
| What it Adds | Inverse property                                                                                                   |

##### Example

    (Integers including 0, *, +)
    5 * 1 = 5
    3 * ( 4 * 5 ) =  ( 3 * 4 ) * 5
    5 * 0 = 5
    5 * ( 1 + 2 ) = 5 * 1 + 5 * 2

    5 + 0 = 5
    5 + ( 3 + 4 ) = ( 5 + 3 ) + 4
    5 + 3 = 3 + 5

Integers actually go beyond semirings, but they do exhibit all of the semiring laws. However, you'll notice two properties that are missing

1. Multiplication does not need to be commutative, that is, a _ b !== b _ a in all cases
2. Addition does not have an inverse. This is important because it shows that a `Semiring` does not extend a `Group` but rather `Monoid`.

##### Counterexample

    Counting Numbers
    // no additive identity or multiplicative annihilation (0)

    Negative Numbers
    // no multiplicative identity(1) and no multiplicative annihilation (0)

##### What this module contains

A single `Semiring` interface which contains `add` and `mul` methods, as well as `zero` and `one` properties (our identity and multiplicative annihilation properties).

Additionally, we are provided a `getFunctionSemiring` function, which given a `Semiring<B>`, returns a `Semiring<(a: A) => B>`.

### [Ring.ts](https://gcanti.github.io/fp-ts/modules/Ring.ts.html)

|              |                                                                                           |
| ------------ | ----------------------------------------------------------------------------------------- |
| Definition   | A semiring with an additive inverse. Multiplication is still not necessarily commutative. |
| Extends      | Semiring                                                                                  |
| What it Adds | Additive Inverse (Subtraction)                                                            |

##### Examples

See [Semiring](#semiringtshttpsgcantigithubiofp-tsmodulessemiringtshtml)

##### Counterexamples

See Semiring

##### What this module contains

1. A single `Ring` interface extending `Semiring` with an `sub` property, which contains the additive inverse.
2. `getFunctionRing` for creating a `Ring` containing a function
3. `getTupleRing` for creating a `Ring` of tuples from a tuple of `Rings`, i.e. `(tra: Tuple<Ring<A>>) => Ring<Tuple<A>>`
4. `negate`, a function that uses `Ring`'s `zero` property (extended from `Semiring`) to create a function `(a: A): A => sub(zero, a)`

### [Field.ts](https://gcanti.github.io/fp-ts/modules/Field.ts.html)

|              |                                                                                            |
| ------------ | ------------------------------------------------------------------------------------------ |
| Definition   | A ring with a multiplicative inverse. Multiplication is still not necessarily commutative. |
| Extends      | Ring                                                                                       |
| What it Adds | Multiplicative Inverse (Division)                                                          |

##### Examples

    (Rational Numbers, + - * /)

##### Counterexamples

    (Rational Numbers, + - *)
    (Integers, + - * /)

##### What this module contains

1. A single `Field` interface extending `Ring` with an `div` and `mod` property, which contains the multiplicative inverse and modulo operation (%) respectively.
2. `fieldNumber` constant holding a `Ring<number>`
3. `gcd`, a function for calculating the Great Common Divisor of two numbers
4. `lcm`, a function for calculating the Least Common Multiple of two numbers

---

## Lattices

Before we delve into lattices, I should mention that understanding them will not affect your ability to understand the rest of the article `fp-ts`. That being said, they can be useful to you in certain circumstances, so the following section is worth a read. I place it here because it is still on the more abstract side of the library, but feel free to skip over it and come back.

If you have decided to continue reading or have come back to this section (welcome back! Aren't `Monad`'s awesome!?!), I should then also define a partial ordered set:

> a set together with a binary relation indicating that, for certain pairs of elements in the set, one of the elements precedes the other in the ordering.

### [JoinSemilattice.ts](https://gcanti.github.io/fp-ts/modules/JoinSemilattice.ts.html)

|              |                                                                               |
| ------------ | ----------------------------------------------------------------------------- |
| Definition   | A magma whose binary operation ^ (join) defines the least upper bound.        |
|              | Join is associative, commutative, and [idempotent](#mathematical-properties). |
| Extends      | Magma                                                                         |
| What it Adds | join                                                                          |

##### Examples

##### Counterexamples

##### What this module contains

A single interface `JoinSemilattice` which defines the type for the `join` method.

### [MeetSemilattice.ts](https://gcanti.github.io/fp-ts/modules/MeetSemilattice.ts.html)

|              |                                                                               |
| ------------ | ----------------------------------------------------------------------------- |
| Definition   | A magma whose binary operation ∨ (meet) defines the greatest lower bound.     |
|              | Meet is associative, commutative, and [idempotent](#mathematical-properties). |
| Extends      | Magma                                                                         |
| What it Adds | meet                                                                          |

##### Examples

##### Counterexamples

##### What this module contains

A single interface `MeetSemilattice` which defines the type for the `meet` method.

### [Lattice.ts](https://gcanti.github.io/fp-ts/modules/Lattice.ts.html)

|              |                                                                                                                          |
| ------------ | ------------------------------------------------------------------------------------------------------------------------ |
| Definition   | A combination of the laws of meet- and join-semilattices, i.e. it has both a greatest lower bound and least upper bound. |
|              | It must also follow the [absorbtion](#mathematical-properties) laws for meet and join.                                   |
| Extends      | MeetSemilattice, JoinSemilattice                                                                                         |
| What it Adds | N/A                                                                                                                      |

##### Examples

##### Counterexamples

##### What this module contains

A single interface `Lattice` which extends `JoinSemilattice` and `MeetSemilattice`.

### [DistributiveLattice.ts](https://gcanti.github.io/fp-ts/modules/DistributiveLattice.ts.html)

|              |                                                                                |
| ------------ | ------------------------------------------------------------------------------ |
| Definition   | A lattice which must also exhibit the distributive property for join and meet. |
| Extends      | Lattice                                                                        |
| What it Adds | N/A                                                                            |

##### Examples

##### Counterexamples

##### What this module contains

A single interface `DistributiveLattice` which extends `Lattice`.

A function `getMinMaxDistributiveLattice` which given an `Ord<A>` returns a `DistributiveLattice<A>`.

### [BoundedJoinSemilattice.ts](https://gcanti.github.io/fp-ts/modules/BoundedJoinSemilattice.ts.html)

|              |                                                                                 |
| ------------ | ------------------------------------------------------------------------------- |
| Definition   | A join-semilattice which must also exhibit the following for join: `a ∨ 0 == a` |
|              | That is, 0 is the greatest lower bound of the set.                              |
| Extends      | JoinSemilattice                                                                 |
| What it Adds | zero                                                                            |

##### Examples

##### Counterexamples

##### What this module contains

A single interface `BoundedJoinSemilattice` which extends `JoinSemilattice` and defines a property `zero`, equaling the greatest lower bound.

### [BoundedMeetSemilattice.ts](https://gcanti.github.io/fp-ts/modules/BoundedMeetSemilattice.ts.html)

|              |                                                                                 |
| ------------ | ------------------------------------------------------------------------------- |
| Definition   | A meet-semilattice which must also exhibit the following for meet: `a ^ 1 == a` |
|              | That is, 1 is the least upper bound of the set.                                 |
| Extends      | MeetSemilattice                                                                 |
| What it Adds | one                                                                             |

##### Examples

##### Counterexamples

##### What this module contains

A single interface `BoundedMeetSemilattice` which extends `MeetSemilattice` and defines a property `one`, equaling the least upper bound.

### [BoundedLattice.ts](https://gcanti.github.io/fp-ts/modules/BoundedLattice.ts.html)

|              |                                                                                                                                                 |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| Definition   | A combination of the laws of bounded-meet- and bounded-join-semilattices, i.e. its greatest upper bound is 1 and its greatest lower bound is 0. |
|              | It must also follow the [absorbtion](#mathematical-properties) laws for meet and join.                                                          |
| Extends      | BoundedJoinSemilattice, BoundedMeetSemilattice                                                                                                  |
| What it Adds | one                                                                                                                                             |

##### Examples

##### Counterexamples

##### What this module contains

A single interface `BoundedSemilattice` which extends `BoundedJoinSemilattice` and `BoundedMeetSemilattice`.

### [BoundedDistributiveLattice.ts](https://gcanti.github.io/fp-ts/modules/BoundedDistributiveLattice.ts.html)

|              |                                                                                        |
| ------------ | -------------------------------------------------------------------------------------- |
| Definition   | A combination of the laws of bounded- and distributive-semilattices. That is,          |
|              | Its greatest upper bound is 1 and its greatest lower bound is 0.                       |
|              | It must also follow the [absorbtion](#mathematical-properties) laws for meet and join. |
|              | A lattice which must exhibit the distributive property for join and meet.              |
| Extends      | BoundedJoinSemilattice, BoundedMeetSemilattice                                         |
| What it Adds | one                                                                                    |

##### Examples

##### Counterexamples

##### What this module contains

A single interface `BoundedDistributiveSemilattice` which extends `BoundedSemilattice` and `DistributiveSemilattice`.

### `HeytingAlgebra.ts`

No, it's not the emotion you're feeling after reading through all these definitions.

### `BooleanAlgebra.ts`

---

## Put the Gun Down and Step Away from the Abstractions

You made it! Congrats! You crawled out from the deep dark cave and your eyes are starting to adjust. Color! Shapes! It's all so much more beautiful than you remember! Let's explore this brave new world together!

![Trying to read an actual map when I don’t have Google Maps to tell me where I am](https://i.makeagif.com/media/6-08-2016/rHBP08.gif)

### [`Functor.ts`](https://gcanti.github.io/fp-ts/modules/Functor.ts.html) and [`FunctorWithIndex.ts`](https://gcanti.github.io/fp-ts/modules/FunctorWithIndex.ts.html)

`fp-ts` has a [page](https://gcanti.github.io/fp-ts/getting-started/Functor.html) dedicated to `Functor`s in "Getting Started".

In this beautiful brave new world, nothing is more awe-inspiring than the functor.

"Functor? Ugh, sounds complex..." Wrong! In fact, you have been using functors all your life! Okay, that's a bit of an exaggeration (but not really).

You've been using them at least as long as you have been writing JavaScript though!

Functors are simply containers that tell us how to deal with their contained value, i.e. there exists a function named `map` that applies a function to its value. Each container's map is different. An `array`'s `map` applies that function to the value stored at each index. Similarly, an `object`'s `map` might apply the function to the value stored at each key (in fact, `fp-ts` defines such a function). Remember our `Identity` type class from before? As we said, it's a `Monad` (which is a type of `Functor`), so we can `map` `Identity`. What would that look like? The same as if we had applied the function to `Identity`'s value itself. That is:

```
const fn = (x: number): number => x * 2;

map(fn)(Identity) = fn(Identity.value);
```

You may have noticed that we included a second module in the header, `FunctorWithIndex.ts`. There's nothing extremely special about this. It's simply a functor which also supports `mapWithIndex`, a function taking an additional parameter, e.g. `i`, which keeps track of the index we are mapping in our value.

```
const fn = (i: number, x: number): number => x * index;

map(fn)([1,2,3]);
// [0, 2, 6]
```

This is useful for objects like `array` or even `record`, and a lot of functors that perform iterative mappings will have an additional `WithIndex` module. We won't address these going forward, but each allows for the iterative functions in their definitions to access a function that tracks the index of the iteration.

Keep functors in mind going forward. They will build the basis of (almost) everything else we are going to look at here. We are going to step away from them for just a moment, but fret not. Absence only makes the heart grow fonder.

### [`Foldable.ts`](https://gcanti.github.io/fp-ts/modules/Foldable.ts.html) and [`FoldableWithIndex.ts`](https://gcanti.github.io/fp-ts/modules/FoldableWithIndex.ts.html)

Folding, or reducing, as it's more commonly referred to in JavaScript, is process of, well, processing all of the elements in a container and returning a value. A common example would be `Array.prototype.reduce`, which allows you to take a list of integers `[ 1, 2, 3 ]` and return its sum, `6`. What you may not have realized is that folding is done recursively. Therefore, a more formal definition of a `Foldable` might be:

> analyzing a recursive data structure and through use of a given combining operation, recombine the results of recursively processing its constituent parts, building up a return value. (Wikipedia)

However, it's importantly to realize that a `Foldable` is not necessarily a `Functor`, though the two often overlap. A `Functor`'s `map` retains the data structure of the original data structure of the `Functor`. A `Foldable`'s `reduce` does not, as seen in the `fp-ts` definition:

```
    // note: this is modified typing of the fp-ts definition for readability,
    // given the reader's assumed understanding of this library
    reduce: <A, B>(functorOfA: F<A>, acc: B, func: (acc: B, curr: A) => B) => B
```

As you can see, it returns the value of `B` not `F<B>`.

### [`Unfoldable.ts`](https://gcanti.github.io/fp-ts/modules/Unfoldable.ts.html)

Although you may have understandably assumed that an `Unfoldable` is simply a data type that cannot be folded, that is not the case. Remember, any type class that does not extend `Foldable` would fit that definition.

Instead of a type class that is _not able_ to be _folded_, an `Unfoldable` is _able_ to be _unfolded_. Latin could certainly have taken some queues from functional programming, huh?

`fp-ts`'s `unfold` function (no not `unreduce`) is the reverse of `reduce`. Where `reduce` takes a collection of values and returns a single return value, `unfold` takes a seed value (or values) and returns a collection of values. Maybe the easiest way to think of this is the (reverse) Fibonnacci sequence. Given the first two values (1, 1), `unfold` would apply a function `([a, b, ...acc]) => [ a + b, b, a, ...acc]`. An `unfold` would have some arbirary breakpoint, since it could generate this sequence forever.

### [`Traversable.ts`](https://gcanti.github.io/fp-ts/modules/Traversable.ts.html) and [`TraversableWithIndex.ts`](https://gcanti.github.io/fp-ts/modules/TraversableWithIndex.ts.html)

### [`Compactable.ts`](https://gcanti.github.io/fp-ts/modules/Compactable.ts.html)

Compactable are data structures that can be compacted/filtered. They are not functors, per se, since they cannot map any function over their value. We cna use these classes to provide the ability to operate on a data type by eliminating intermediate `None`s (see [Option.ts](#optionts)) below.

They take define two functions, `compact` and `separate` as such:

```
    readonly compact: <A>(compactableOfOptionA: C< Option<A> >) => F<A>
    readonly separate: <A, B>(compactableOfEitherAOrB: C< Either<A, B> >)
        => Separated< C<A>, C<B> >
```

In this file, we also get the `Separated` type class. We'll discuss this more when we discuss `Either` below.

### [`Filterable.ts`](https://gcanti.github.io/fp-ts/modules/Filterable.ts.html) and [`FilterableWithIndex.ts`](https://gcanti.github.io/fp-ts/modules/FilterableWithIndex.ts.html)

While a `Compactible` can only perform `compact` or `separate` on its value, a `Filterable` can perform an given `Predicate` on its value since it is a member of the `Functor` type class. This gives us greater control over what we filter.

A filterable defines two sets of function pairs: `filter` / `filterMap` and `partition` / `partitionMap`.

`filter` removes elements based on a predicate that returns a `boolean` (`true` is kept, `false` is removed), while `filterMap`'s predicate returns an `Option` (`some(x)` is kept, `none` is removed).

`partition` / `partitionMap` separate the values into a `Separated` type class with `left` and `right` values. `partition` uses a boolean return value, while `partitionMap` uses an `Either` return value. More on `Separated` when we discuss `Either` below.

### [`Witherable.ts`](https://gcanti.github.io/fp-ts/modules/Witherable.ts.html)

<!-- Combines Traversable and Filterable -->

### `Contravariant.ts`

### `Invariant.ts`

### `Bifunctor.ts`

### [`Alt.ts`](https://gcanti.github.io/fp-ts/modules/Alt.ts.html)

`Alt` is a `Functor` that can perform `alt` on a value. `alt` is similar to composing functions, except that it works on type classes instead. In fact, in `Alt`'s description, `fp-ts` draws a parallel between it and `Semigroup` (if you'll recall, a `Semigroup` had to allow for associative composition of its morphisms).

In the same way, the composition of `Alt` must follow the rules of:

1. associativity: `A.alt(A.alt(fa, ga), ha) = A.alt(fa, A.alt(ga, ha))`
2. distributivity: `A.map(A.alt(fa, ga), ab) = A.alt(A.map(fa, ab), A.map(ga, ab))`

### [`Extend.ts`](https://gcanti.github.io/fp-ts/modules/Extend.ts.html)

### [`Comonad.ts`](https://gcanti.github.io/fp-ts/modules/Comonad.ts.html)

`Comonad` is an `Extend`, but it allows for the operation of the function `extract`. To put it incredibly simply (almost too simply 🕵️‍♂️), it reverses the construction of the `Functor`, i.e. it extracts the value from the `Functor` and returns it to the world.

### [`Profunctor.ts`](https://gcanti.github.io/fp-ts/modules/Profunctor.ts.html)

### [`Strong.ts`](https://gcanti.github.io/fp-ts/modules/Strong.ts.html)

### [`Choice.ts`](https://gcanti.github.io/fp-ts/modules/Choice.ts.html)

### [`Apply.ts`](https://gcanti.github.io/fp-ts/modules/Choice.ts.html)

`Apply` is a direct decendant of a `Functor` and we're zeroing in on our ultimate goal here, so stay with me! An `Apply` defines the function `ap`, which which is used to apply a function to an argument under a type constructor.

Like I said, stay with me. `ap` is incredibly important.

In laymans terms, `ap` takes the value of the first functor passed into it and applies it to the value (i.e. function) of the second functor. Here's the type signature:

```
type AtoB = (a: A) => B
ap: <A, B>(fab:  F<AtoB>, fa: F<A>) => F<B>
```

In addition to the `Functor` laws, instance of `Apply` must follow the associative composition property for `ap`:

```
F.ap(
    F.ap(
        F.map(
            fbc,
            bc => ab => a => bc( ab( a ) )
        ), fab
    ), fa
) = F.ap( fbc, F.ap( fab, fa ) )
```

What the holy mess? Let's ignore the outer F.ab, since it remains constant on both sides of the equation. This is saying that mapping a function to a functor and the applying the function of another functor to that function and then applying the value of a functor to that function (huh???) is the same as applying the value of a functor to the function of the same functor type and then applying that result to the

# TBFinished

### [`Chain.ts`](https://gcanti.github.io/fp-ts/modules/Chain.ts.html)

`Chain` extends `Apply` with the addition `chain` operation. `chain` is an incredibly powerful tool in functional programming, so make sure you understand it. In fact, if you know `map`, `ap`, and `chain` you can do some amazing things in FP. That's where the long awaited `Monad` comes into play, but we'll get there shortly.

`map` and `chain` may seem similar at first. Both take a first `Functor` of some value and perform a second function on it. The difference is the type of function that is passed in.

    // functor
    fa: F<A>

    // map
    mapHandler: (fa: F<A>) => F<B>
    chainHandler: (a: A) => F<B>

    map(fa, mapHandler)
    // F<B>

    chain(fa, chainHandler)
    // F<B>

Do you see it? The `mapHandler` takes `F<A>` as an argument and `chainHandler` takes simply `A`. `chain` applies the function to the _value_ of the functor passed in, while `map` applies the function to the functor itself. `chain` allows us to write functions that can take simply a value and return a functor and apply them to _either_ a non-functor `chainHandler(a)` or a functor of the same value type `chain(fa, chainHandler)`.

### [`ChainRec.ts`](https://gcanti.github.io/fp-ts/modules/ChainRec.ts.html)

### `Applicative.ts`

[page](https://gcanti.github.io/fp-ts/getting-started/Applicative.html)

### [`Alternative.ts`](https://gcanti.github.io/fp-ts/modules/Alternative.ts.html)

`Alternative` extends `Alt` and `Applicative`, but like `Monoid` compared to `Semigroup`, it must now allow for an identity element, in this case a `zero` function.

`Alternative`s must follow the following rules, where `f` is a function and `fa` is a `Functor`:

1. Left identity: A.alt( zero, fa ) == fa
2. Right identity: A.alt( fa, zero ) == fa
3. Annihilation: A.map( zero, f ) == zero
4. Distributivity: A.ap( A.alt( fab, gab ), fa ) = A.alt( A.ap( fab, fa ), A.ap( gab, fa ) )
5. Annihilation: A.ap( zero, fa ) = zero

### `Monad.ts`

[page](https://gcanti.github.io/fp-ts/getting-started/Monad.html)

![fp-ts type classes](https://gcanti.github.io/fp-ts/type-classes.svg)

---

## S`Monads, Please!

![sandlot smore gif](https://media1.tenor.com/images/c5d7937f0a4bb5f12c1a0c5a62b14ecb/tenor.gif?itemid=13799567)

### `Option.ts`

### `Either.ts`, `These.ts`, and `ValidationT.ts`

<!-- Compactable's Separated -->

### `Array.ts`, `NonEmptyArray.ts`, and `Tuple.ts`

### `Records.ts` and `Map.ts`

These are type classes representing JavaScript objects and ES6 Maps respectively. For more on the differences, see [the MDN page](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map#Objects_and_maps_compared).

### `Task.ts`, `TaskEither.ts`, and `TaskThese.ts`

### `IO.ts` and `IOEither.ts`

### `IORef.ts`

### `MonadIO.ts` and `MonadTask.ts`

### `State.ts`, `Reader.ts`, and `Writer.ts`

### `ReaderEither.ts`, `ReaderTask.ts`, `ReaderTaskEither.ts`, `StateReaderTaskEither.ts`,

### `MonadThrow.ts`

### `HKT.ts`: `OptionT.ts`, `EitherT.ts`, `ReaderT.ts`, `StateT.ts`, `TheseT.ts`, `WriterT.ts`

---

## Utility Functions

### `boolean.ts`

### `Date.ts`

### `Console.ts`

### `Random.ts`

### `function.ts`

### `index.ts`

### `pipeable.ts`

### `Traced.ts`

## Misc

### `Const.ts`

### `Store.ts`

### `Tree.ts`
