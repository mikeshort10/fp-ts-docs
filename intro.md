## Intro to FP-TS and (Functional) Functional Programming

### What the Functor?!

Think for a moment about a time that you ate soup. You may have bought that soup in a can, then transferred it to a pot to boil it, and then finally to a bowl for consumption. In fact, to transferred it again... to your stomach!

Now think of that soup as your code, or at least the values stored in your code. That can, pot, bowl, and full tummy are all functors, that is, they contain some value or values and have a way to for you to get at and deal with those values. You use the pull tab on the can to get to the cold soup, you use pot handles to pour the hot soup, a spoon to eat it, and so on. Did it ever occur to you to perform any action on that soup without placing it in a container first? No! That would be ridiculous. Well, it's equally as ridiculous to do that with code. JavaScript is full of `undefined`s and `null`s and `Errors` and other problematic pieces that we simply have to place them in containers to ensure that at every step of them process, these values are being dealt with properly.

A very basic but apt description of a functor is simply a container that (at minimum) a way to access its value, which we'll refer to here as the `map` function. If `map` sounds familiar to you, it should! That's right, an array is a functor! In fact, `Array` is such an important functor and easy bridge into functional programming that, whenever possible, I will include examples with `Array`'s in going over the numerous methods and functions provided by the `fp-ts` library.

Write functional code is not hard. It simply involves breaking your code down into smaller and smaller testable steps so that you can plug and play more complex functions later on. To you give you one every practical example, how many times have you written this in your code?

```
if (x) {
    // then do something with x cause it's defined
}
```

Now, how would you love to never have to do that again? That's write! Your friendly neighborhood `Option` will make all the bad `undefined`s and `null`s disappear. This is the second functor that I will be using regularly as I find it's one of the most useful ones provided by `fp-ts`.

### Your Friendly Neighborhood Map Function

As with Functors, let's start with the FP functions that you're used to, before we dive into anything that's new and strange. `map` is perhaps the most significant FP function, as it's what most of the others are based off of. Hopefully, you're familiar with the `Array.prototype.map` function, which, according to MDN:

> creates a new array populated with the results of calling a provided function on every element in the calling array

So:

    [1,2,3].map(n => n + 1) // [2, 3, 4]

This is probably the definition that you're used to, but if you've never had to map another functor before, you may never have truly understood the power of the map function. What it is really doing is allowing us to manipulate with the value of the functor WITHOUT EVER LEAVING IT! Think of the example above for a second. How would you do this if `map` did not exist. Probably something like this:

```
const arr = [];
const newArray = [];
for (let i = 0; i < arr.length; i++) {
    newArray[i] = arr[i] + 1;
}
```

Oh god, I think I'm gonna be sick...

Writing that once is bad enough, but what if you had to map over that array multiple times with different functions. You'd have to rewrite that code over and over. You could, of course, put that into a function for reuse, but that's exactly what the `map` method is!

Let's now take a look at mapping another functor, `Option`. If you have read any other FP literature, you might have seen this more commonly referenced as `Maybe`. Essentially, it helps us to deal with the nullable values, `null` and `undefined`. `fp-ts`'s module `fp-ts/lib/Option` has two functions that I'm going use here. The first is `map`, and the second is `fromNullable`. `fromNullable` will go over a value and return one of the two subtypes of `Option`: `Some<T>` and `None`. The first simply holds a non-nullable value, while the second **represents** a nullable value. You could implement this with the following:

```
type Some<T> = {
    tag: 'Some';
    value: T;
}

type None = {
    tag; 'None';
}

const none: None = { tag: 'None' }
const some = <T>(t: T): Some<T> => ({ tag: 'Some', value: t })

fromNullable = <T>(t: T) => t == null ? none : some(t);
// none and some are a constant and function that return their respective types.
// I would recommend against using some(t) outside of this, since some(null) does not return `None` but rather `Some(null)`, which would obviously introduce bugs.
// Use fromNullable to ensure type safety.
```

Luckily, all of this boilerplate code is defined for you, so you only need to import `fromNullable`, as well as `map`. Then you can perform the following:

```
const someone = fromNullable(1); // Some(1);
const noOne = fromNullable(undefined); // None;

const addOne = (n: number): number => n + 1;

map(someone, addOne); // Some(2);
map(noOne, addOne); // None;
```

Whoa, what was that?! Didn't you see it?! Well let's run that code without functors and see what would have happened:

```
addOne(1); // 2, no problem
addOne(undefined) // NaN
```

Oh jeez, so nauseous.

We didn't take into account that the number could have been undefined. Actually, that would only have been the result with pure JavaScript, in TypeScript, our code wouldn't have even compiled because `addOne` can't take anything but a number as its argument. We would have had to introduce code that checked for that:

    const addOne = (n: number | undefined): number | undefined => n != null ? n + 1 : n

Okay, safe code, but still ugly. And consider that we now have to introduce this type check into EVERY function that could possibly take the same `n` as a parameter. I can feel the carpal tunnel already.

With `Option`, we can simply write:

```
function increaseIfNumber = <X>(x): Option<X> => {
    const optX = fromNullable(x);
    return map(optX, addOne);
}
```

Notice, we don't have to do anything to our addOne function to make it work with map. It just does! The value (or lack thereof) stays in the container of `Option`, and we will never see that monster `NaN` ever again without having to type check every parameter ever again!

However, I understand your skepticism here. "But there's still so much code to write...". Granted, but that's because we've only just started on FP! You can't expect to complete a marathon after a brisk morning jog. Actually, that's a terrible analogy, since unlike exercise, FP requires you to do less work in the long run! (see what I did there)

Next we will step away from functors explicitly and look at some helper functions that will make our job easier.

### Composing the way Mozart would (but with a lot more ones and zeros)

Composition is critical to functional programming. We're going into category theory a little. Uh oh...

No, this bit is fairly simple. A set, an object, a morphism, and a category.

A set, as you may know already, is a collection of unique objects, such as { 1, 2, 3 }. For example, { 1, 2, 3, 3 } is not a set, because not all of its objects are unique, while { } is a set, despite having no objects.

An object is something that exists within a set. If we equate that with TypeScript, these are our types. Anything from `string` or `number` to custom types like `FigTree` or `RickSanchez`. In fact, because "object" has an alternate meaning in TypeScript, let's just call these types for now.

A morphism (intimidating though it may sound) is simply a function in JavaScript. It is a **structure-preserving map** from one type to another. So if we were to type a morphism:

    type Morphism = <A, B>(a: A) => B

Keep in mind that A and B do not necessarily have to be different type. This simply means that they can be. Some examples might be:

```
const wordLength = (str: string): number => str.length;
const isGreaterThanTen = (n: number): boolean => str.length > 10;
```

Finally, a category, equated with TypeScript, is... well... TypeScript. It defines it's own functions and types, all of which are unique.

Actually, what truly makes it a category is the fact that it obeys two important laws:

1. It has an identity function, which transforms one type back to itself. We can write this simple as:

   const identity = x => x

In fact, `fp-ts` defines this very function as well.

2. The morphisms can be composed. What this means is that if one function's parameter is of the same type as another function's return type, we can compose them. We can write the compose function as follows:

```
const compose = (...fas: Function[]) => (a) => {
    let answer = a;
    for (const fn of fns) {
        answer = fn(answer);
    }
    return answer;
}
```

Actually, we would probably implement this in recursion in keeping with FP best practices, but this is more explicitly stated for those unaccustomed to recursive functions.

While in typical FP libraries, this function is indeed called compose, in `fp-ts`, we have two alternate functions that are slightly different, `flow` and `pipe`. `flow` is actually very similar to this. We could write:

```
const isTooFriendly = flow(wordLength, isGreaterThanTen);

isTooFriendly('hi!') // false;
isTooFriendly('hihihihihihihihi!') // true
```

Great! In fact, we can string together as many functions as we want, and pass the single composed function around wherever it is needed. So