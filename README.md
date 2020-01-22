## About this document

This document is not an attempt at explaining category theory in full. Its primary purpose is to explain the interfaces, functions, etc. that exist in the [fp-ts` library](https://gcanti.github.io/fp-ts/), many of which depict abstract concepts of category theory and function programming.

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

### [Identity.ts](https://gcanti.github.io/fp-ts/modules/Identity.ts.html)

One of the foundations of mathematics (and indeed all reality!) is that everything is equal to itself.

This is called the identity property.

This module simply creates a `type alias` to represent this:

`Identity<A> = A`

This module also contains a number of functions (`map`, `chain`, `ap`, etc.) that work on `Monad`s, which `Identity` is member of (more on `Monad`s later).

### [Eq.ts](https://gcanti.github.io/fp-ts/modules/Eq.ts.html)

You can read the "Getting started" documentation on `Eq` on the `fp-ts` [documentation page](https://gcanti.github.io/fp-ts/getting-started/Eq.html) as well.

Closely connected with `Identity` is `Eq`. `Eq` contains types that admit _equality_. They must have a function defined on them named `equals`. This function must adhere to the laws of:

1. reflexivity (`equals(x, x) === true`)
2. symmetry (`equals(x, y) === equals(y, x)`)
3. transitivity(`equals(x, y) === true` and `equals(y, z) === true`, then `equals(x, z) === true`)

### [Ord.ts](https://gcanti.github.io/fp-ts/modules/Ord.ts.html) and [Ordering.ts](https://gcanti.github.io/fp-ts/modules/Ordering.ts.html)

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

### [Bounded.ts](https://gcanti.github.io/fp-ts/modules/Bounded.ts.html)

The `Bounded` type class is exactly what it sounds like: it contains an lower _and_ upper limit, such that `lower <= a <= upper`. It must also be a member of `Ord`. After all, how can we know if a member of the `Bounded` is between the lower and upper if we don't know how to compare them it.

### [Show.ts](https://gcanti.github.io/fp-ts/modules/Show.ts.html)

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

![I know some of those words](https://media0.giphy.com/media/3osxYbgtOZrFW2C81O/source.gif)

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

|              |                                                                                                                       |
| ------------ | --------------------------------------------------------------------------------------------------------------------- |
| Definition   | A partially-ordered set whose binary operation v (join) defines the least upper bound for any nonempty finite subset. |
|              | Join is associative, commutative, and [idempotent](#mathematical-properties).                                         |
| Extends      | Magma                                                                                                                 |
| What it Adds | join                                                                                                                  |

##### Example

    // { 1, 2, 3, 4, 5}
    1 v 5 === 5
    2 v 3 === 3
    1 v 1 === 1

##### What this module contains

A single interface `JoinSemilattice` which defines the type for the `join` method.

### [MeetSemilattice.ts](https://gcanti.github.io/fp-ts/modules/MeetSemilattice.ts.html)

|              |                                                                                                                          |
| ------------ | ------------------------------------------------------------------------------------------------------------------------ |
| Definition   | A partially-ordered set whose binary operation ^ (meet) defines the greatest lower bound for any nonempty finite subset. |
|              | Meet is associative, commutative, and [idempotent](#mathematical-properties).                                            |
| Extends      | Magma                                                                                                                    |
| What it Adds | meet                                                                                                                     |

##### Example

    // { 1, 2, 3, 4, 5 }
    1 ^ 5 === 1
    2 ^ 3 === 2
    1 ^ 1 === 1

##### What this module contains

A single interface `MeetSemilattice` which defines the type for the `meet` method.

### [Lattice.ts](https://gcanti.github.io/fp-ts/modules/Lattice.ts.html)

|              |                                                                                                                          |
| ------------ | ------------------------------------------------------------------------------------------------------------------------ |
| Definition   | A combination of the laws of meet- and join-semilattices, i.e. it has both a greatest lower bound and least upper bound. |
|              | It must also follow the [absorbtion](#mathematical-properties) laws for meet and join.                                   |
| Extends      | MeetSemilattice, JoinSemilattice                                                                                         |
| What it Adds | N/A                                                                                                                      |

##### Example

    // { 1, 2, 3, 4, 5 }
    1 v 5 === 5
    4 ^ 3 === 3
    2 v 2 === 1

##### What this module contains

A single interface `Lattice` which extends `JoinSemilattice` and `MeetSemilattice`.

### [DistributiveLattice.ts](https://gcanti.github.io/fp-ts/modules/DistributiveLattice.ts.html)

|              |                                                                                |
| ------------ | ------------------------------------------------------------------------------ |
| Definition   | A lattice which must also exhibit the distributive property for join and meet. |
| Extends      | Lattice                                                                        |
| What it Adds | N/A                                                                            |

##### Example

    // { 1, 2, 3, 4, 5 }
    1 v (5 ^ 4) === (1 v 5) ^ (1 v 4) // 4
    1 ^ (5 v 4) === (1 ^ 5) v (1 ^ 4) // 1

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

##### Example

    // { 0, 1, 2, 3, 4, 5}
    1 v 5 === 5
    2 v 0 === 2

##### Counterexample

    // { -1, 0, 1, 2, 3, 4, 5}
    -1 v 0 !== -1

##### What this module contains

A single interface `BoundedJoinSemilattice` which extends `JoinSemilattice` and defines a property `zero`, equaling the greatest lower bound.

### [BoundedMeetSemilattice.ts](https://gcanti.github.io/fp-ts/modules/BoundedMeetSemilattice.ts.html)

|              |                                                                                 |
| ------------ | ------------------------------------------------------------------------------- |
| Definition   | A meet-semilattice which must also exhibit the following for meet: `a ^ 1 == a` |
|              | That is, 1 is the least upper bound of the set.                                 |
| Extends      | MeetSemilattice                                                                 |
| What it Adds | one                                                                             |

##### Example

    // { -2, -1, 0, 0.5, 1}
    -2 v 1 === 1
    1 v 0.5 === 1

##### Counterexample

    // { -1, 0, 1, 2 }
    2 ^ 1 !== 2

##### What this module contains

A single interface `BoundedMeetSemilattice` which extends `MeetSemilattice` and defines a property `one`, equaling the least upper bound.

### [BoundedLattice.ts](https://gcanti.github.io/fp-ts/modules/BoundedLattice.ts.html)

|              |                                                                                                                                                 |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| Definition   | A combination of the laws of bounded-meet- and bounded-join-semilattices, i.e. its greatest upper bound is 1 and its greatest lower bound is 0. |
|              | It must also follow the [absorbtion](#mathematical-properties) laws for meet and join.                                                          |
| Extends      | BoundedJoinSemilattice, BoundedMeetSemilattice                                                                                                  |
| What it Adds | one                                                                                                                                             |

##### Example

    // { 0, 0.25, 0.5, 0.75, 1 }
    1 v 0.75 === 1
    2 ^ 0 === 0

    //Absorbtion Join
    0.25 ^ (0.25 v 0.75) === 0.25

    //Absorbtion Meet
    0.5 v (0.5 ^ 1) === 0.5

##### Counterexample

See BoundedJoinLattice and BoundedMeetLattice above.

##### What this module contains

A single interface `BoundedSemilattice` which extends `BoundedJoinSemilattice` and `BoundedMeetSemilattice`.

### [BoundedDistributiveLattice.ts](https://gcanti.github.io/fp-ts/modules/BoundedDistributiveLattice.ts.html)

|              |                                                                                                                                                                                                                                                                                                                                |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Definition   | An intersection of the laws of bounded- and distributive-semilattices: <br> 1. Its greatest upper bound is 1 and its greatest lower bound is 0. <br>2. It must also follow the [absorbtion](#mathematical-properties) laws for meet and join.<br> 3. A lattice which must exhibit the distributive property for join and meet. |
| Extends      | BoundedJoinSemilattice, BoundedMeetSemilattice                                                                                                                                                                                                                                                                                 |
| What it Adds | one                                                                                                                                                                                                                                                                                                                            |

##### Example

    // { 0, 0.25, 0.5, 0.75, 1 }
    1 v 0.75 === 1
    2 ^ 0 === 0

    //Absorbtion Join
    0.25 ^ (0.25 v 0.75) === 0.25

    //Absorbtion Meet
    0.5 v (0.5 ^ 1) === 0.5

    //Distributive
    0 v (1 ^ 0.5) === (0 v 1) ^ (0 v 0.5) // 0.5
    1 ^ (0.25 v 0) === (1 ^ 0.25) v (1 ^ 0) // 0.25

##### Counterexample

See BoundedJoinLattice and BoundedMeetLattice above.

##### What this module contains

A single interface `BoundedDistributiveSemilattice` which extends `BoundedSemilattice` and `DistributiveSemilattice`.

A function `getMinMaxBoundedDistributiveLattice` which given an `Ord<A>` returns a `BoundedDistributiveLattice<A>`.

### [HeytingAlgebra.ts](https://gcanti.github.io/fp-ts/modules/HeytingAlgebra.ts.html)

No, it's not the emotion you're feeling after reading through all these definitions.

|              |                                                                                                                                                                                  |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Definition   | A bounded-distributive-semilattice that includes both a `not` and an `implies` method.                                                                                           |
|              | `implies` defines a function that returns 1 if left value is less than or equal to the right. Otherwise it returns the right.<br>`not` function can be defined as `a` implies 0. |
|              | Implication (→) must follow these laws: <br> `a → a = 1` <br> `a ∧ (a → b) = a ∧ b` <br> `b ∧ (a → b) = b` <br> `a → (b ∧ c) = (a → b) ∧ (a → c)`                                |
|              | Not (*) must follow the following laws: <br> `*a = a → 0 `                                                                                                                       |
| Extends      | BoundedDistributiveLattice                                                                                                                                                       |
| What it Adds | implies, not                                                                                                                                                                     |

It's important to note (and is defined as such in `fp-ts`) that it cannot be stated as true that `a v not(a) === 0`. This is because Heyting Algebra does not include the law of excluded middle, which excludes anything between the lowest upper bound and greatest lower bounds. See examples for a more concrete example.

##### Example

    const implies = <A>(p: A, q: A) => p >= q ? 1 : q;
    const not = implies(p, 0);

    // not can also be written as follows, but this is not how it is strictly defined in Heyting Algebra
    // const not = <A>(p: A, q: A) => p >= q ? 1 : 0;

    // Example of Heyting Algebra:
    { 0, 0.5, 1}

    implies(0, 0.5) // 1
    implies(0.5, 0) // 0
    implies(1, 0.5) // 0.5
    implies(0.5, 0.5) // 1
    not(1) // 0
    not(0) // 1
    not(0.5) // 0

    0.5 v not(0.5) // 0.5

### [BooleanAlgebra.ts](https://gcanti.github.io/fp-ts/modules/BooleanAlgebra.ts.html)

|              |                                                                                                                                                                             |
| ------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Definition   | Heyting Algebra with that includes the law of excluded middle. Put simply, it's is a set made up of only 1's and 0's. <br> As a result, the following is true: `a ∨ *a = 1` |
| Extends      | HeytingAlgebra                                                                                                                                                              |
| What it Adds | Law of excluded middle, i.e. there can be no members of the set between the least upper bound (1) and the greatest upper bound (0).                                         |

##### Example

    // Boolean set
    { 0, 1 }

    *1 // 0
    1 v *1 // 1
    *0 // 1
    0 v *0 // 1

##### Counterexample

    // most simple, non-Boolean Heyting Algebra set
    { 0, 0.5, 1 }

    *0.5 // 0
    0.5 v *0.5 // 0.5 (does not equal 1, so not a Boolean Algebra)

---

## Put the Gun Down and Step Away from the Abstractions

You made it! Congrats! You crawled out from the deep dark cave and your eyes are starting to adjust. Color! Shapes! It's all so much more beautiful than you remember! Let's explore this brave new world together!

![Trying to read an actual map when I don’t have Google Maps to tell me where I am](https://i.makeagif.com/media/6-08-2016/rHBP08.gif)

### [Functor.ts](https://gcanti.github.io/fp-ts/modules/Functor.ts.html) / [FunctorWithIndex.ts](https://gcanti.github.io/fp-ts/modules/FunctorWithIndex.ts.html)

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

### [Foldable.ts](https://gcanti.github.io/fp-ts/modules/Foldable.ts.html) / [FoldableWithIndex.ts](https://gcanti.github.io/fp-ts/modules/FoldableWithIndex.ts.html)

Folding, or reducing, as it's more commonly referred to in JavaScript, is process of, well, processing all of the elements in a container and returning a value. A common example would be `Array.prototype.reduce`, which allows you to take a list of integers `[ 1, 2, 3 ]` and return its sum, `6`. What you may not have realized is that folding is done recursively. Therefore, a more formal definition of a `Foldable` might be:

> analyzing a recursive data structure and through use of a given combining operation, recombine the results of recursively processing its constituent parts, building up a return value. (Wikipedia)

However, it's importantly to realize that a `Foldable` is not necessarily a `Functor`, though the two often overlap. A `Functor`'s `map` retains the data structure of the original data structure of the `Functor`. A `Foldable`'s `reduce` does not, as seen in the `fp-ts` definition:

```
    // note: this is modified typing of the fp-ts definition for readability,
    // given the reader's assumed understanding of this library
    reduce: <A, B>(functorOfA: F<A>, acc: B, func: (acc: B, curr: A) => B) => B
```

As you can see, it returns the value of `B` not `F<B>`.

### [Unfoldable.ts](https://gcanti.github.io/fp-ts/modules/Unfoldable.ts.html)

Although you may have understandably assumed that an `Unfoldable` is simply a data type that cannot be folded, that is not the case. Remember, any type class that does not extend `Foldable` would fit that definition.

Instead of a type class that is _not able_ to be _folded_, an `Unfoldable` is _able_ to be _unfolded_. Latin could certainly have taken some queues from functional programming, huh?

`fp-ts`'s `unfold` function (no not `unreduce`) is the reverse of `reduce`. Where `reduce` takes a collection of values and returns a single return value, `unfold` takes a seed value (or values) and returns a collection of values. Maybe the easiest way to think of this is the (reverse) Fibonnacci sequence. Given the first two values (1, 1), `unfold` would apply a function `([a, b, ...acc]) => [ a + b, b, a, ...acc]`. An `unfold` would have some arbirary breakpoint, since it could generate this sequence forever.

### [Traversable.ts](https://gcanti.github.io/fp-ts/modules/Traversable.ts.html) / [TraversableWithIndex.ts](https://gcanti.github.io/fp-ts/modules/TraversableWithIndex.ts.html)

### [Compactable.ts](https://gcanti.github.io/fp-ts/modules/Compactable.ts.html)

Compactable are data structures that can be compacted/filtered. They are not functors, per se, since they cannot map any function over their value. We cna use these classes to provide the ability to operate on a data type by eliminating intermediate `None`s (see [Option.ts](#optionts)) below.

They take define two functions, `compact` and `separate` as such:

```
    readonly compact: <A>(compactableOfOptionA: C< Option<A> >) => F<A>
    readonly separate: <A, B>(compactableOfEitherAOrB: C< Either<A, B> >)
        => Separated< C<A>, C<B> >
```

In this file, we also get the `Separated` type class. We'll discuss this more when we discuss `Either` below.

### [Filterable.ts](https://gcanti.github.io/fp-ts/modules/Filterable.ts.html) / [FilterableWithIndex.ts](https://gcanti.github.io/fp-ts/modules/FilterableWithIndex.ts.html)

While a `Compactible` can only perform `compact` or `separate` on its value, a `Filterable` can perform an given `Predicate` on its value since it is a member of the `Functor` type class. This gives us greater control over what we filter.

A filterable defines two sets of function pairs: `filter` / `filterMap` and `partition` / `partitionMap`.

`filter` removes elements based on a predicate that returns a `boolean` (`true` is kept, `false` is removed), while `filterMap`'s predicate returns an `Option` (`some(x)` is kept, `none` is removed).

`partition` / `partitionMap` separate the values into a `Separated` type class with `left` and `right` values. `partition` uses a boolean return value, while `partitionMap` uses an `Either` return value. More on `Separated` when we discuss `Either` below.

### [Witherable.ts](https://gcanti.github.io/fp-ts/modules/Witherable.ts.html)

<!-- Combines Traversable and Filterable -->

### `Contravariant.ts`

### `Invariant.ts`

### `Bifunctor.ts`

### [Alt.ts](https://gcanti.github.io/fp-ts/modules/Alt.ts.html)

`Alt` is a `Functor` that can perform `alt` on a value. `alt` is similar to composing functions, except that it works on type classes instead. In fact, in `Alt`'s description, `fp-ts` draws a parallel between it and `Semigroup` (if you'll recall, a `Semigroup` had to allow for associative composition of its morphisms).

In the same way, the composition of `Alt` must follow the rules of:

1. associativity: `A.alt(A.alt(fa, ga), ha) = A.alt(fa, A.alt(ga, ha))`
2. distributivity: `A.map(A.alt(fa, ga), ab) = A.alt(A.map(fa, ab), A.map(ga, ab))`

### [Extend.ts](https://gcanti.github.io/fp-ts/modules/Extend.ts.html)

### [Comonad.ts](https://gcanti.github.io/fp-ts/modules/Comonad.ts.html)

`Comonad` is an `Extend`, but it allows for the operation of the function `extract`. To put it incredibly simply (almost too simply 🕵️‍♂️), it reverses the construction of the `Functor`, i.e. it extracts the value from the `Functor` and returns it to the world.

### [Profunctor.ts](https://gcanti.github.io/fp-ts/modules/Profunctor.ts.html)

### [Strong.ts](https://gcanti.github.io/fp-ts/modules/Strong.ts.html)

### [Choice.ts](https://gcanti.github.io/fp-ts/modules/Choice.ts.html)

### [Apply.ts](https://gcanti.github.io/fp-ts/modules/Apply.ts.html)

`Apply` is a direct decendant of a `Functor` and we're zeroing in on our ultimate goal here, so stay with me! An `Apply` defines the function `ap`, which which is used to apply a function to an argument under a type constructor.

Like I said, stay with me. `ap` is incredibly important.

In laymans terms, `ap` takes the value of the first functor passed into it and applies it to the value (i.e. function) of the second functor. Here's the type signature:

```
type AtoB = (a: A) => B
ap: <A, B>(fab:  F<AtoB>, fa: F<A>) => F<B>
```

In addition to the `Functor` laws, instance of `Apply` must follow the associative composition property for `ap`:

    // F is a functor
    // bc is a function that takes b and returns c. fab is a functor of such a function
    // ab is a function that takes a and returns b. fab is a functor of such a function
    // fa  is a functor of a
    F.ap(
        F.ap(
            F.map(
                fbc,
                bc => ab => a => bc( ab( a ) ) ),
            fab
        ), fa
    ) = F.ap( fbc, F.ap( fab, fa ) )

What the holy mess? Let's first start with the problem both sides of the equation is trying to solve. We need to apply the value of `fa` to `fab` and then apply that return value to `fbc`, which is our answer.

At this point, you might be saying, Why don't we just take all of these values out and work on them, then put them back into the container. That may seem simple on paper, but let's use a metaphor for a second. We have an assembly line that make sandwiches. We need to take the bread out of the package and put two ingredients in it. The first thing that comes along is bread, ham, and cheese. Okay, take the cheese and ham out, stick it between the bread, and put it back in the container (functor). Maybe not super sanitary, and our hands are a little greasy, but the sandwich is made and packed! Okay, feeling confident, let's take sandwich two. Oh s&\$@. Peanut butter and jelly. Damn, we threw out the container, which was a nice squeeze bottle. Oh, what a mess! It's everywhere! Oh, the humanity.

Okay, so we've decided not to take things out of their containers willy-nilly. They're in there for a reason after all.

So, let's brainstorm with our fellow programmers. Jeff the new guy raises his hand. He has an idea! Let's map the functor `fbc` to a function takes `bc`, store that in a function, and apply that to the functor `fab`, and then apply that to `fa`. Done! Jeff sits back in his chair appreciates the stunned silence.

Except we're not stunned at your genius Jeff! We're stunned at how convolutedly stupid that was! Go on a coffee run Jeff and think about what you've done.

![you're wrong gif](https://cdn.business2community.com/wp-content/uploads/2013/04/Parks-and-Rec-Ben-1.gif)
Instead of all that awful nesting and writing a separate function that's just going to apply functions to it (gross!) and then mapping that to `fbc`, let's apply `fa` to `fab`, and then apply that to `fbc`.

That's all this is saying. Instead of writing a separate function that takes functions and finally applies them to some argument, let's just apply the values of the functors as needed, in an order that makes sense.

### [Chain.ts](https://gcanti.github.io/fp-ts/modules/Chain.ts.html)

`Chain` extends `Apply` with the addition `chain` operation. `chain` is an incredibly powerful tool in functional programming, so make sure you understand it. In fact, if you know `map`, `ap`, and `chain` you can do some amazing things in FP. That's where the long awaited `Monad` comes into play, but we'll get there shortly.

`map` and `chain` may seem similar at first. Both take a `Functor` of some value and a handler function and apply that functors value to the function. The difference is the type of the handler.

    // functor
    fa: F<A>

    // map
    mapHandler: (fa: F<A>) => F<B>
    chainHandler: (a: A) => F<B>

    map(fa, mapHandler)
    // F<B>

    chain(fa, chainHandler)
    // F<B>

Do you see it? The `mapHandler` takes `F<A>` as an argument and `chainHandler` takes simply `A`. `chain` applies the function to the _value_ of the functor passed in, while `map` applies the function to the functor itself. `chain` allows us to write functions that can simply take a value and return a functor and apply them to _either_ a non-functor `chainHandler(a)` or a functor of the same value type `chain(fa, chainHandler)`.

However, it is important to note that the `chainHandler` always returns the type of functor that `chain` is acting on. For example, this doesn't work:

    import { chain } from 'fp-ts/lib/Record';

    const chainHandler = (a: number): number => a * 2;
    const fa = [2, 3, 4]; // remember, an array is a functor

    chain(fa, chainHandler) // Error

### [ChainRec.ts](https://gcanti.github.io/fp-ts/modules/ChainRec.ts.html)

### [Applicative.ts](https://gcanti.github.io/fp-ts/modules/Applicative.ts.html)

|                                |                                                                         |
| ------------------------------ | ----------------------------------------------------------------------- |
| Definition                     | An `Apply` with an `of` method that wraps a value in a functor.         |
| Extends                        | Apply                                                                   |
| What it Adds                   | `of` function, which constructs a functor out of a value                |
| `fp-ts` "Getting Started" Page | [page](https://gcanti.github.io/fp-ts/getting-started/Applicative.html) |

The `Applicative` must follow these additional laws:

1. Identity: A.ap(A.of(a => a), fa) = fa
2. Homomorphism: A.ap(A.of(ab), A.of(a)) = A.of(ab(a))
3. Interchange: A.ap(fab, A.of(a)) = A.ap(A.of(ab => ab(a)), fab)

### [Alternative.ts](https://gcanti.github.io/fp-ts/modules/Alternative.ts.html)

`Alternative` extends `Alt` and `Applicative`, but like `Monoid` compared to `Semigroup`, it must now allow for an identity element, in this case a `zero` function.

|              |                                                                                                                                                                      |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Definition   | `Alternative` extends `Alt` and `Applicative`, but like `Monoid` compared to `Semigroup`, it must now allow for an identity element, in this case a `zero` property. |
| Extends      | Apply                                                                                                                                                                |
| What it Adds | `zero` property, which allows for identity                                                                                                                           |

`Alternative`s must follow the following rules, where `f` is a function, `fa` is a `Functor`, & `fab` and `gab` are functions that take `a` and return `b`:

1. Left identity: A.alt( zero, fa ) == fa
2. Right identity: A.alt( fa, zero ) == fa
3. Annihilation: A.map( zero, f ) == zero
4. Distributivity: A.ap( A.alt( fab, gab ), fa ) = A.alt( A.ap( fab, fa ), A.ap( gab, fa ) )
5. Annihilation: A.ap( zero, fa ) = zero

### [Monad.ts](https://gcanti.github.io/fp-ts/modules/Monad.ts.html)

[page](https://gcanti.github.io/fp-ts/getting-started/Monad.html)

![fp-ts type classes](https://gcanti.github.io/fp-ts/type-classes.svg)

---

## S`Monads, Please!

![sandlot smore gif](https://media1.tenor.com/images/c5d7937f0a4bb5f12c1a0c5a62b14ecb/tenor.gif?itemid=13799567)

### [Option.ts](https://gcanti.github.io/fp-ts/modules/Option.ts.html)

| Name              | Definition                                                                                                                                                                                                            |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| elem              | Test if an `Option` contains a certain value. For `None`, always returns `false`.                                                                                                                                     |
| exists            | Returns true if the predicate is satisfied by the wrapped value.                                                                                                                                                      |
| fold              | Takes a default value, a function, and an `Option` value, if the `Option` value is `None` the default value is returned, otherwise the function is applied to the value inside the `Some` and the result is returned. |
| fromNullable      | Constructs a new `Option` from a nullable type. If the value is `null` or `undefined`, returns `None`, otherwise returns the value wrapped in a `Some`                                                                |
| fromPredicate     | Returns a smart constructor based on the given predicate, that is, it returns the value wrapped in an `Option`.                                                                                                       |
| getApplyMonoid    |                                                                                                                                                                                                                       |
| getApplySemigroup |                                                                                                                                                                                                                       |
| getEq             | Creates an `Eq` for comparing the values of `Option`s.                                                                                                                                                                |
| getFirstMonoid    |                                                                                                                                                                                                                       |
| getSecondMonoid   |                                                                                                                                                                                                                       |
| getLeft           | Returns the `left` of an `Either`, wrapped in an `Option`.                                                                                                                                                            |
| getMonoid         |                                                                                                                                                                                                                       |
| getOrElse         | Extracts the value out of the `Option` if it exists. Otherwise, returns the default value.                                                                                                                            |
| getOrd            | The `Ord` instance allows `Option` values to be compared with compare, whenever there is an Ord instance for the type the `Option` contains. `None` is considered to be less than any `Some` value.                   |
| getRefinement     |                                                                                                                                                                                                                       |
| getRight          | Returns the `right` of an `Either`, wrapped in an `Option`.                                                                                                                                                           |
| getShow           | Returns a `Show<Option<A>>` of the `Option<A>`                                                                                                                                                                        |
| isNone            | Returns `true` if the option is `None`, `false` otherwise.                                                                                                                                                            |
| isSome            | Returns `true` if the option is `Some`, `false` otherwise.                                                                                                                                                            |
| mapNullable       | `chain` + `fromNullable`. It takes a function that operates on potentially nullable values, passes in the `Option`, and returns an `Option`.                                                                          |
| some              | Converts a value to a `Some`. Note, even if the value passed in is nullable, it will return a `Some`. If you aren't sure if the value is not nullable, use `fromNullable` instead.                                    |
| toNullable        | Returns the value or `null` if the `Option` is `None`.                                                                                                                                                                |
| toUndefined       | Same as `toNullable` but returns `undefined` instead of `null`.                                                                                                                                                       |
| tryCatch          | Transforms an exception into an `Option`. If f throws, returns `None`, otherwise returns the output wrapped in `Some`                                                                                                 |
| alt               |                                                                                                                                                                                                                       |
| ap                |                                                                                                                                                                                                                       |
| apFirst           |                                                                                                                                                                                                                       |
| apSecond          |                                                                                                                                                                                                                       |
| chain             |                                                                                                                                                                                                                       |
| chainFirst        |                                                                                                                                                                                                                       |
| compact           | Unpacks the value of the inner `Option` to the outer `Option`. For `Option`, accomplishes the same task as `flatten`.                                                                                                 |
| duplicate         | Nests an `Option` in another `Option`.                                                                                                                                                                                |
| extend            |                                                                                                                                                                                                                       |
| filter            | Filter an `Option` using a function that returns a boolean.                                                                                                                                                           |
| filterMap         | Filter an `Option` using a function that returns an `Option`.                                                                                                                                                         |
| flatten           | Removes one layer of `Option` nesting. For `Option`, accomplishes the same task as `compact`.                                                                                                                         |
| foldMap           |                                                                                                                                                                                                                       |
| fromEither        | Takes an `Either`'s right value and wraps it in an `Option`.                                                                                                                                                          |
| map               | Applies a function to each value of the `Option` if that `Option` is not `None`. Otherwise returns `None`.                                                                                                            |
| partition         | Separates the `Option` into a `Separated` type class with `left` and `right` values using a function that returns a boolean.                                                                                          |
| partitionMap      | Separates the `Option` into a `Separated` type class with `left` and `right` values using a function that returns an `Option`.                                                                                        |
| reduce            | Reduces an `Option` to single value.                                                                                                                                                                                  |
| reduceRight       |                                                                                                                                                                                                                       |
| separate          | Given an `Option` of `Either<A, B>`, returns a `Separated<Option<A>, Option<B>>`                                                                                                                                      |

### [Either.ts](https://gcanti.github.io/fp-ts/modules/Either.ts.html) / [These.ts](https://gcanti.github.io/fp-ts/modules/These.ts.html) / [ValidationT.ts](https://gcanti.github.io/fp-ts/modules/ValidationT.ts.html)

<!-- Compactable's Separated -->

| Name                   | Either | These | Definition                                                                                                                                                                           |
| ---------------------- | :----: | :---: | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| both                   |        |   X   | Constructs a `These` from a left and right value.                                                                                                                                    |
| elem                   |   X    |       | Tests whether an `Either` is a `Right` and has a value is equal to a given value.                                                                                                    |
| exists                 |   X    |       | If the `Either` is a `Right` returns the boolean value of the `Predicate`. If its a `Left`, returns `false`.                                                                         |
| fold                   |   X    |   X   | Takes two functions, and applies one, depending on whether the `Either` is a `Left` or `Right` respectively.                                                                         |
| fromNullable           |   X    |       | Takes a default and nullable value. If the value is not nully, returns a `Right` value, else returns the `Left` default value.                                                       |
| fromOptions            |        |   X   | Takes a pair of `Option`s and attempts to create a `These` from them                                                                                                                 |
| getApplyMonoid         |   X    |       |                                                                                                                                                                                      |
| getApplySemigroup      |   X    |       |                                                                                                                                                                                      |
| getEq                  |   X    |       | Takes an `Eq` for the left and right value, and returns an `Eq` for the `Either` / `These`.                                                                                          |
| getLeft                |        |   X   | Returns an `Option` from a `These`'s left. If there's not a left, returns a `None`.                                                                                                  |
| getLeftOnly            |        |   X   | Returns an `Option` from a `These`'s left, but only if it was constructed with `Left`, not `Both`.                                                                                   |
| getMonad               |        |   X   |                                                                                                                                                                                      |
| getOrElse              |   X    |       | Takes a function `onLeft`, and returns the `Right` value or the result of the `onLeft` function, which returns the same type as the `Either`'s right.                                |
| getRight               |        |   X   | Returns an `Option` from a `These`'s right. If there's not a right, returns a `None`.                                                                                                |
| getRightOnly           |        |   X   | Returns an `Option` from a `These`'s right, but only if it was constructed with `Right`, not `Both`.                                                                                 |
| getSemigroup           |   X    |   X   |                                                                                                                                                                                      |
| getShow                |   X    |   X   | Returns a `Show<Either<A, B>>` of the `Either<A, B>`                                                                                                                                 |
| getValidation          |   X    |       |                                                                                                                                                                                      |
| getValidationMonoid    |   X    |       | Given `Semigroup<E>` and `Monoid<A>`, constructs `Monoid<Either<E, A>>`.                                                                                                             |
| getValidationSemigroup |   X    |       | Given `Semigroup<E>` and `Semigroup<A>`, constructs `Semigroup<Either<E, A>>`.                                                                                                       |
| getWitherable          |   X    |       |                                                                                                                                                                                      |
| isBoth                 |        |   X   | Tests whether the `These` is a `Both`.                                                                                                                                               |
| isLeft                 |   X    |   X   | Tests whether the `Either` / `These` is a `Left`.                                                                                                                                    |
| isRight                |   X    |   X   | Tests whether the `Either` / `These` is a `Right`.                                                                                                                                   |
| left                   |   X    |   X   | Constructs a new `Either` with a `left` value. This usually represents a failure, due to its left-bias.                                                                              |
| leftOrBoth             |        |   X   | Takes a value for the left and an `Option`. If the `Option` is `Some`, returns a `Both`. Else returns a `Left`.                                                                      |
| orElse                 |   X    |       |                                                                                                                                                                                      |
| parseJSON              |   X    |       | Converts a JSON string into an object. Returns a `Left` on failure.                                                                                                                  |
| right                  |   X    |   X   | Constructs a new `Either` with a `right` value. This usually represents a success, due to its right-bias.                                                                            |
| rightOrBoth            |        |   X   | Takes a value for the right and an `Option`. If the `Option` is `Some`, returns a `Both`. Else returns a `Right`.                                                                    |
| stringifyJSON          |   X    |       | Converts an object into a JSON string. Returns a `Left` on failure.                                                                                                                  |
| swap                   |   X    |   X   | Swaps the left and right values of an `Either`.                                                                                                                                      |
| toError                |   X    |       | Constructs a value into an `Error`. Used in `tryCatch`.                                                                                                                              |
| toTuple                |        |   X   | Takes a default value for left and right, and takes a `These`. If its a `Both`, returns a `Tuple` whose first value is the `These` left and whose second value is the `These` right. |
|                        |        |       | Else applies the default value for the missing `Left` or `Right`.                                                                                                                    |
| tryCatch               |   X    |       | Takes a function that might throw an `Error`and a `toError` function and constructs on `Either` from it.                                                                             |
| alt                    |   X    |       |                                                                                                                                                                                      |
| ap                     |   X    |       |                                                                                                                                                                                      |
| apFirst                |   X    |       |                                                                                                                                                                                      |
| apSecond               |   X    |       |                                                                                                                                                                                      |
| bimap                  |   X    |   X   | Takes two functions, each of which operates on a left and right value, applies them to each side of the `Either` and returns the new `Either`.                                       |
| chain                  |   X    |       |                                                                                                                                                                                      |
| chainFirst             |   X    |       |                                                                                                                                                                                      |
| duplicate              |   X    |       | Nests an `Either` in another `Either`. The nested `Either` is stored as a right value.                                                                                               |
| extend                 |   X    |       |                                                                                                                                                                                      |
| filterOrElse           |   X    |       |                                                                                                                                                                                      |
| flatten                |   X    |       | Unpacks an inner `Right` into an outer `Right`.                                                                                                                                      |
| foldMap                |   X    |   X   |                                                                                                                                                                                      |
| fromOption             |   X    |       | Takes a function that returns a `Left` and converts the `Option` to a `Right` if it's a `Some`, else uses the function to return a `Left`.                                           |
| fromPredicate          |   X    |       |                                                                                                                                                                                      |
| map                    |   X    |   X   | Performs a function on the `Either`'s right value. If the `Either` is a `Left`, returns the `Left`.                                                                                  |
| mapLeft                |   X    |   X   | Performs a function on the `Either`'s left value. If the `Either` is a `Right`, returns the `Right`.                                                                                 |
| reduce                 |   X    |   X   |                                                                                                                                                                                      |
| reduceRight            |   X    |   X   |                                                                                                                                                                                      |

### [Array.ts](https://gcanti.github.io/fp-ts/modules/Array.ts.html) / [NonEmptyArray.ts](https://gcanti.github.io/fp-ts/modules/NonEmptyArray.ts.html) / [Tuple.ts](https://gcanti.github.io/fp-ts/modules/Tuple.ts.html)

When I am referring to an JavaScript array, I will use the lowercase "array". If I am referring to the `fp-ts` version, I will use `Array`. In practice, any function that can take an `Array` will also take an array.

| Function              | Array | NEA | Tuple | Definition                                                                                                                                                                                       |
| --------------------- | :---: | :-: | :---: | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| chop                  |   X   |     |       | A useful recursion pattern for processing an array to produce a new array, often used for “chopping” up the input array.                                                                         |
|                       |       |     |       | Typically chop is called with some function that will consume an initial prefix of the array and produce a value and the rest of the array.                                                      |
| chunksOf              |   X   |     |       | Splits an array into length-n pieces. The last piece will be shorter if n does not evenly divide the length of the array.                                                                        |
|                       |       |     |       | Note that `chunksOf(n)([])` is `[]`, not `[[]]`. This is intentional, and is consistent with a recursive definition of chunksOf                                                                  |
| comprehension         |   X   |     |       |                                                                                                                                                                                                  |
| cons                  |   X   |  X  |       | Attaches an element to the front of an array, creating a new non empty array                                                                                                                     |
| copy                  |   X   |  X  |       | Makes a shallow copy of an array.                                                                                                                                                                |
| deleteAt              |   X   |     |       | Delete the element at the specified index, returning an `Option` (`None` if the index is out of bounds)                                                                                          |
| difference            |   X   |     |       | Creates an array of array values not included in the other given array using a Eq for equality comparisons. The order and references of result values are determined by the first array.         |
| dropLeft              |   X   |     |       | Drop a number of elements from the start of an array, creating a new array                                                                                                                       |
| dropLeftWhile         |   X   |     |       | Remove the longest initial subarray for which all element satisfy the specified predicate, creating a new array                                                                                  |
| dropRight             |   X   |     |       | Drop a number of elements from the end of an array, creating a new array                                                                                                                         |
| elem                  |   X   |     |       | Test if a value is a member of an array. Takes a `Eq<A>` as a single argument which returns the function to use to search for a value of type `A` in an array of type `Array<A>`.                |
| findFirst             |   X   |     |       | Find the first element which satisfies a predicate (or a refinement) function                                                                                                                    |
| findFirstMap          |   X   |     |       | Find the first element returned by an option based selector function                                                                                                                             |
| findIndex             |   X   |     |       | Find the first index for which a predicate holds                                                                                                                                                 |
| findLast              |   X   |     |       | Find the last element which satisfies a predicate function                                                                                                                                       |
| findLastIndex         |   X   |     |       | Returns the index of the last element of the list which matches the predicate                                                                                                                    |
| findLastMap           |   X   |     |       | Find the last element returned by an option based selector function                                                                                                                              |
| flatten               |   X   |  X  |       | Removes one level of nesting                                                                                                                                                                     |
| foldLeft              |   X   |     |       | Break an array into its first element and remaining elements                                                                                                                                     |
| foldRight             |   X   |     |       | Break an array into its initial elements and the last element                                                                                                                                    |
| getApply              |       |     |   X   |                                                                                                                                                                                                  |
| getApplicative        |       |     |   X   |                                                                                                                                                                                                  |
| getChain              |       |     |   X   |                                                                                                                                                                                                  |
| getChainRec           |       |     |   X   |                                                                                                                                                                                                  |
| getEq                 |   X   |  X  |       | Derives an `Eq` over the Array of a given element type from the `Eq` of that type.                                                                                                               |
|                       |       |     |       | The derived `Eq` defines two arrays as equal if all elements of both arrays are compared equal pairwise with the given `E`. In case of arrays of different lengths, the result is non equality.  |
| getMonad              |       |     |   X   |                                                                                                                                                                                                  |
| getMonoid             |   X   |     |       | Returns a Monoid for `Array<A>`                                                                                                                                                                  |
| getOrd                |   X   |     |       | Derives an Ord over the Array of a given element type from the Ord of that type.                                                                                                                 |
|                       |       |     |       | The ordering between two such arrays is equal to: the first non equal comparison of each arrays elements taken pairwise in increasing order, in case of equality over all the pairwise elements. |
|                       |       |     |       | The longest array is considered the greatest, if both arrays have the same length, the result is equality.                                                                                       |
| getShow               |   X   |  X  |       | Derives a `Show` over an Array.                                                                                                                                                                  |
| head                  |   X   |  X  |       | Get the first element in an array, or `None` if the array is empty                                                                                                                               |
| init                  |   X   |  X  |       | Get all but the last element of an array, creating a new array, or `None` if the array is empty                                                                                                  |
| insertAt              |   X   |  X  |       | Insert an element at the specified index, creating a new array, or returning `None` if the index is out of bounds                                                                                |
| intersection          |   X   |     |       | Creates an array of unique values that are included in all given arrays using a Eq for equality comparisons. The order and references of result values are determined by the first array.        |
| isEmpty               |   X   |     |       | Test whether an array is empty                                                                                                                                                                   |
| isNonEmpty            |   X   |     |       | Test whether an array is non empty narrowing down the type to `NonEmptyArray<A>`                                                                                                                 |
| isOutOfBound          |   X   |     |       | Test whether an array contains a particular index                                                                                                                                                |
| last                  |   X   |  X  |       | Get the last element in an array, or `None` if the array is empty                                                                                                                                |
| max                   |       |  X  |       | Given an `Ord` and a `NonEmptyArray`, returns the maximum value of that `NonEmptyArray`.                                                                                                         |
| min                   |       |  X  |       | Given an `Ord` and a `NonEmptyArray`, returns the minimum value of that `NonEmptyArray`.                                                                                                         |
| lefts                 |   X   |     |       | Extracts from an array of `Either` all the `Left` elements. All the `Left` elements are extracted in order                                                                                       |
| lookup                |   X   |     |       | Read a value at a particular index from an array                                                                                                                                                 |
| makeBy                |   X   |     |       | Return a list of length `n` with element `i` initialized with `f(i)`                                                                                                                             |
| modifyAt              |   X   |  X  |       | Apply a function to the element at the specified index, creating a new array, or returning `None` if the index is out of bounds                                                                  |
| of                    |       |  X  |       | Given a value, returns an `Array` / `NonEmptyArray` containing the value.                                                                                                                        |
| range                 |   X   |     |       | Create an array containing a range of integers, including both endpoints                                                                                                                         |
| replicate             |   X   |     |       | Create an array containing a value repeated the specified number of times                                                                                                                        |
| reverse               |   X   |  X  |       | Reverse an array, creating a new `Array` / `NonEmptyArray`. See `swap` for `Tuple`s.                                                                                                             |
| swap                  |       |     |   X   | Reverse the two values of a `Tuple`. See reverse for `Array`s and `NonEmptyArray`s.                                                                                                              |
| rights                |   X   |     |       | Extracts from an array of `Either` all the `Right` elements. All the `Right` elements are extracted in order                                                                                     |
| rotate                |   X   |     |       | Rotate an array to the right by `n` steps                                                                                                                                                        |
| scanLeft              |   X   |     |       | Same as `reduce` but it carries over the intermediate steps                                                                                                                                      |
| scanRight             |   X   |     |       | Same as `reduceRight` but it carries over the intermediate steps                                                                                                                                 |
| snoc                  |   X   |  X  |       | Append an element to the end of an array, creating a new non empty array                                                                                                                         |
| concat                |       |  X  |       | Concatenate an `Array` with a `NonEmptyArray`, returning a `NonEmptyArray`.                                                                                                                      |
| fst                   |       |     |   X   | Returns the first value of a tuple.                                                                                                                                                              |
| snd                   |       |     |   X   | Returns the second value of a tuple.                                                                                                                                                             |
| sort                  |   X   |  X  |       | Sort the elements of an array in increasing order, creating a new array                                                                                                                          |
| sortBy                |   X   |     |       | Sort the elements of an array in increasing order, where elements are compared using first `ords[0]`, then `ords[1]`, etc…                                                                       |
| spanLeft              |   X   |     |       | Split an array into two parts: 1) the longest initial subarray for which all elements satisfy the specified predicate and 2) the remaining elements                                              |
| splitAt               |   X   |     |       | Splits an array into two parts: 1) an array of `n` length and 2) the rest of the array.                                                                                                          |
| tail                  |   X   |  X  |       | Get all but the first element of an array, creating a new array, or `None` if the array is empty                                                                                                 |
| takeLeft              |   X   |     |       | Keep only a number of elements from the start of an array, creating a new array. `n` must be a natural number                                                                                    |
| takeLeftWhile         |   X   |     |       | Calculate the longest initial subarray for which all element satisfy the specified predicate, creating a new array                                                                               |
| takeRight             |   X   |     |       | Keep only a number of elements from the end of an array, creating a new array. n must be a natural number                                                                                        |
| union                 |   X   |     |       | Creates an array of unique values, in order, from all given arrays using a `Eq` for equality comparisons                                                                                         |
| uniq                  |   X   |     |       | Remove duplicates from an array, keeping the first occurrence of an element.                                                                                                                     |
| unsafeDeleteAt        |   X   |     |       | Same as `deleteAt`, except it returns an `Array<A>` instead of `Option<Array<A>>`, even if the index is out of bounds.                                                                           |
| unsafeInsertAt        |   X   |     |       | Same as `insertAt`, except it returns an `Array<A>` instead of `Option<Array<A>>`, even if the index is out of bounds.                                                                           |
| unsafeUpdateAt        |   X   |     |       | Same as `updateAt`, except it returns an `Array<A>` instead of `Option<Array<A>>`, even if the index is out of bounds.                                                                           |
| unzip                 |   X   |     |       | The reverse of `zip`. Takes an array of pairs and return two corresponding arrays                                                                                                                |
| updateAt              |   X   |  X  |       | Change the element at the specified index, creating a new array, or returning `None` if the index is out of bounds                                                                               |
| zip                   |   X   |     |       | Takes two arrays and returns an array of corresponding pairs. If one input array is short, excess elements of the longer array are discarded                                                     |
| zipWith               |   X   |     |       | Apply a function to pairs of elements at the same index in two arrays, collecting the results in a new array. If one input array is short, excess elements of the longer array are discarded.    |
| alt                   |   X   |     |       |                                                                                                                                                                                                  |
| ap                    |   X   |  X  |       |                                                                                                                                                                                                  |
| apFirst               |   X   |  X  |       |                                                                                                                                                                                                  |
| apSecond              |   X   |  X  |       |                                                                                                                                                                                                  |
| bimap                 |       |     |   X   | Takes two functions, each of which operates on one side of a tuple, applies them to each side of tuple and returns the new tuple.                                                                |
| chain                 |   X   |  X  |       |                                                                                                                                                                                                  |
| chainFirst            |   X   |  X  |       |                                                                                                                                                                                                  |
| compact               |   X   |     |       | Takes an array of `Option`s, and returns an `Array` of the values of the `some`s.                                                                                                                |
| compose               |       |     |   X   |                                                                                                                                                                                                  |
| duplicate             |   X   |  X  |   X   | Nests the functor within a functor of the same type. For example, `duplicate([1])` would return `[[1]]`. Tuples are nested in their first spot, i.e. `duplicate([a, b])` is `[ [a, b], b ]`      |
| extend                |   X   |  X  |   X   |                                                                                                                                                                                                  |
| filter                |   X   |  X  |       | Filter an array using a function that returns a boolean.                                                                                                                                         |
| filterMap             |   X   |  X  |       | Filter an array using a function that returns an `Option`.                                                                                                                                       |
| filterMapWithIndex    |   X   |  X  |       | Same as `filterMap` but takes an additional parameter that tracks the current index.                                                                                                             |
| filterWithIndex       |   X   |     |       | Same as `filter` but takes an additional parameter that tracks the current index.                                                                                                                |
| getSemigroup          |       |  X  |       |                                                                                                                                                                                                  |
| group                 |       |  X  |       | Group equal, consecutive elements of an array into non empty arrays.                                                                                                                             |
| groupBy               |       |  X  |       | Splits an array into sub-non-empty-arrays stored in an object, based on the result of calling a string-returning function on each element, and grouping the results according to values returned |
| groupSort             |       |  X  |       | Sort and then group the elements of an array into non empty arrays.                                                                                                                              |
| foldMap               |   X   |  X  |   X   |                                                                                                                                                                                                  |
| foldMapWithIndex      |   X   |  X  |       |                                                                                                                                                                                                  |
| map                   |   X   |  X  |   X   | Applies a function to each value of the `Array` / `NonEmptyArray`. For `Tuple`s, the function is applied to the first value.                                                                     |
| mapLeft               |       |     |   X   | Applies a function to the right second value of a `Tuple`.                                                                                                                                       |
| mapWithIndex          |   X   |  X  |       | Same as `map` but takes an additional parameter which tracks the index.                                                                                                                          |
| partition             |   X   |     |       | Separates the values into a `Separated` type class with `left` and `right` values using a function that returns a boolean.                                                                       |
| partitionMap          |   X   |     |       | Separates the values into a `Separated` type class with `left` and `right` values using a function that returns an `Option`.                                                                     |
| partitionMapWithIndex |   X   |     |       | Same as `partitionMap` but with an additional parameter that tracks the index.                                                                                                                   |
| partitionWithIndex    |   X   |     |       | Same as `partition` but with an additional parameter that tracks the index.                                                                                                                      |
| reduce                |   X   |  X  |   X   | Reduces all of the values in an array into an accumulator value, starting at the first value.                                                                                                    |
| reduceRight           |   X   |  X  |   X   | Reduces all of the values in an array into an accumulator value, starting at the last value.                                                                                                     |
| reduceRightWithIndex  |   X   |  X  |       | Same as `reduceRight` but with an additional parameter that tracks the index.                                                                                                                    |
| reduceWithIndex       |   X   |  X  |       | Same as `reduce` but with an additional parameter that tracks the index.                                                                                                                         |
| separate              |   X   |     |       | Given an array of `Either<A, B>`, returns a `Separated<A[], B[]>`                                                                                                                                |

### [Record.ts](https://gcanti.github.io/fp-ts/modules/Record.ts.html) / [Map.ts](https://gcanti.github.io/fp-ts/modules/Map.ts.html)

These are type classes representing JavaScript objects and ES6 Maps respectively. For more on the differences, see [the MDN page](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map#Objects_and_maps_compared).

| function               | Record | Map | Definition                                                                                                                            |
| ---------------------- | :----: | :-: | ------------------------------------------------------------------------------------------------------------------------------------- |
| collect                |   X    |  X  | Map a record/map into an array using a handler function.                                                                              |
| deleteAt               |   X    |  X  | Delete a key and value from a map/record.                                                                                             |
| elem                   |   X    |  X  | Test whether or not a value is a member of a map/record.                                                                              |
| every                  |   X    |     | Test whether or not a condition holds true for each value of a record.                                                                |
| filterMapWithIndex     |   X    |     | `filterMap` with an additional parameter to keep track of the key.                                                                    |
| foldMapWithIndex       |   X    |     | Given a function that can fold the values of a record, returns the folded value/accumulator.                                          |
| fromFoldable           |   X    |  X  | Create a record/map from a foldable collection of key/value pairs, using the specified `Magma` to combine values for duplicate keys.  |
| fromFoldableMap        |   X    |     | Same as `fromFoldable`, but supplies additional function to map values to keys.                                                       |
| getEq                  |   X    |  X  | Create an `Eq` of a record or map, with type signature `Eq<Record<K, A>>` / `Eq<Map<K, A>>`                                           |
| getFilterableWithIndex |        |  X  |                                                                                                                                       |
| getMonoid              |   X    |  X  | Returns a `Semigroup` instance for records/maps given a `Semigroup` instance of their values.                                         |
| getShow                |   X    |  X  | Returns a `Show` instance for records/maps.                                                                                           |
| hasOwnProperty         |   X    |     | Given a string `k` and a record `R`, returns the string as `keyof R` if the record has the property.                                  |
| getWitherable          |        |  X  |                                                                                                                                       |
| insertAt               |   X    |  X  | Insert or replace a key/value pair in a record/map.                                                                                   |
| isEmpty                |   X    |  X  | Test whether a record/map is empty.                                                                                                   |
| isSubrecord / isSubmap |   X    |  X  | Test whether one record/map contains all of the keys and values contained in another record/map                                       |
| keys                   |   X    |  X  | Get an array of keys contained in a record/map. For a map, the array is sorted.                                                       |
| lookup                 |   X    |  X  | Lookup the value for a key in a record/map. Returns an `Option` of the value.                                                         |
| lookupWithKey          |        |  X  | Lookup the value for a key in a map. If the result is a `Some`, the existing key is also returned as tuple as `Option<[key, value]>`. |
| member                 |        |  X  | Test whether or not a key is a member of a map. To test for a value in a map, see `elem`.                                             |
| modifyAt               |   X    |  X  | Modifies a value at a key in a record/map given a function. Returns an `Option` of the record/map.                                    |
| pop                    |   X    |  X  | Delete a key and value from a record/map, returning the value as well as the subsequent record/map as an `Option` containing a tuple. |
| reduceRightWithIndex   |   X    |     | Same as the `reduceRight` function, but takes a first parameter that tracks the current key.                                          |
| reduceWithIndex        |   X    |     | Same as the `reduce` function, but takes a first parameter that tracks the current key.                                               |
| sequence               |   X    |     |                                                                                                                                       |
| singleton              |   X    |  X  | Create a record/map with one key/value pair                                                                                           |
| size                   |   X    |  X  | Calculate the number of key/value pairs in a record/map.                                                                              |
| some                   |   X    |     | Maps a predicate to a record, and returns true if any of the individual values return true.                                           |
| traverse               |   X    |     |                                                                                                                                       |
| traverseWithIndex      |   X    |     |                                                                                                                                       |
| toArray                |   X    |  X  | Get an array of the key/value pairs contained in a record/map. For a map, the array is sorted.                                        |
| toUnfoldable           |   X    |  X  | Unfolds a record/map into a list of key/value pairs                                                                                   |
| updateAt               |   X    |  X  | Modifies a value at a key in a record/map given a value. Returns an `Option` of the record/map.                                       |
| values                 |        |  X  | Get a sorted array of the values contained in a map                                                                                   |
| compact                |   X    |  X  | Given a record of `Option` values, returns those values which evaluate to a `some`.                                                   |
| filter                 |   X    |  X  | Filter a record using a function that returns a boolean.                                                                              |
| filterMap              |   X    |  X  | Filter a record using a function that returns an `Option`.                                                                            |
| map                    |   X    |  X  | Apply a function to each of the keys in a record/map.                                                                                 |
| mapWithIndex           |   X    |     | Map a record passing the keys to the iterating function                                                                               |
| partition              |   X    |  X  | Separates the values into a `Separated` type class with `left` and `right` values using a function that returns a boolean.            |
| partitionMap           |   X    |  X  | Separates the values into a `Separated` type class with `left` and `right` values using a function that returns an `Option`.          |
| partitionWithIndex     |   X    |     | `partition` that takes an additional parameter, which tracks the current key.                                                         |
| partitionMapWithIndex  |   X    |     | `partitionMap` that takes an additional parameter, which tracks the current key.                                                      |
| reduce                 |   X    |     | Folds all of the values in a record into an accumulator value, starting at the first value.                                           |
| reduceRight            |   X    |     | Folds all of the values in a record into an accumulator value, starting at the last value.                                            |
| separate               |   X    |  X  | Given a record whose values are of type `Either<A, B>`, returns a `Separated<Record<string, A>, Record<string, B>>`                   |

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

### [boolean.ts](https://gcanti.github.io/fp-ts/modules/boolean.ts.html)

| Name | Definition                                                                                                                                                           |
| ---- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| fold | Defines the fold over a boolean value. Takes two thunks `onTrue`, `onFalse` and a boolean value. If value is `false`, `onFalse()` is returned, otherwise `onTrue()`. |

### [Date.ts](https://gcanti.github.io/fp-ts/modules/Date.ts.html)

| Name   | Definition                                                                                 |
| ------ | ------------------------------------------------------------------------------------------ |
| create | Returns an `IO<Date>`, representing the current date.                                      |
| now    | Returns an `IO<number>`, representing the milliseconds since Midnight UTC, January 1, 1970 |

### [Console.ts](https://gcanti.github.io/fp-ts/modules/Console.ts.html)

| Name  | Definition                              |
| ----- | --------------------------------------- |
| error | Wraps `console.error` in an `IO<void>`. |
| info  | Wraps `console.info` in an `IO<void>`.  |
| log   | Wraps `console.log` in an `IO<void>`.   |
| warn  | Wraps `console.warn` in an `IO<void>`.  |

### [Random.ts](https://gcanti.github.io/fp-ts/modules/Random.ts.html)

| Name        | Definition                                                                                                       |
| ----------- | ---------------------------------------------------------------------------------------------------------------- |
| randomBool  | An `IO<boolean>` that returns a random boolean.                                                                  |
| random      | An `IO<number>` that wraps `Math.random`.                                                                        |
| randomInt   | An `IO<number>` that returns a number within a closed (includes both endpoints) interval.                        |
| randomRange | An `IO<number>` that returns a number within a range, where the minimum is included and the maximum is excluded. |

### [function.ts](https://gcanti.github.io/fp-ts/modules/function.ts.html)

| Name           | Definition                                                                                                                            |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| unsafeCoerce   | Takes a value and returns the same value assigned to a different type. By default, it returns `unknown`.                              |
| absurd         | A function which is called with one `never` argument, and throws `Error('Called 'absurd' function which should be uncallable')`       |
| constFalse     | A thunk that always returns `false`, i.e. `() => false`                                                                               |
| constNull      | A thunk that always returns `null`, i.e. `() => null`                                                                                 |
| constTrue      | A thunk that always returns `true`, i.e. `() =>`true`                                                                                 |
| constUndefined | A thunk that always returns `undefined`, i.e. `() => undefined`                                                                       |
| constVoid      | A thunk that always returns `void`, i.e. `(): void => {}`                                                                             |
| constant       | Takes a value and returns a thunk that always returns that value.                                                                     |
| decrement      | Decrements a number by one.                                                                                                           |
| flip           | Given a function that takes two arguments, reverses the order of those arguments.                                                     |
| flow           | Function composition, left to right.                                                                                                  |
| identity       | Function that always returns the function passed in.                                                                                  |
| increment      | Increments a number by one.                                                                                                           |
| not            | Takes a `Predicate` (unary function that returns a boolean) and returns its inverse. `not(x => x === true)` is `x => x === false`     |
| tuple          | Returns the arguments as an `Array<any>`.                                                                                             |
| tupled         | Creates a tupled version of this function: instead of `n` arguments, it accepts a single tuple argument.                              |
| untupled       | Takes a tupled function and returns a function with multiple arguments: instead of a single tuple argument, it accepts `n` arguments. |

There are several useful interfaces defined here as well:

| Interface    | Definition                                                                                                                                                                            |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Endomorphism | Defines a function where the input type is same as the output.                                                                                                                        |
| FunctionN    | Defines a function of `n` arguments.                                                                                                                                                  |
| Lazy         | Defines a [thunk](https://daveceddia.com/what-is-a-thunk/), i.e. a deferred function                                                                                                  |
| Predicate    | Defines a function that returns a boolean.                                                                                                                                            |
| Refinement   | Defines a function that asserts a type on its return value, using TypeScript's `is`. Though the interface doesn't include it, `Refinement`s in `fp-ts` typically returns an `Option`. |

### `index.ts`

### `pipeable.ts`

### `Traced.ts`

## Misc

### `Const.ts`

### `Store.ts`

### `Tree.ts`
