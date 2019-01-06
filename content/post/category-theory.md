---
title: "Notes on Category Theory"
date: 2019-01-04T13:48:08+01:00
---

During the holiday season I finally decided to learn something about [Category
theory][category-theory] which seems to be *the* hot topic in the Haskell
community. The book I used to guide me in this journey is [Category theory for
programmers][cats-for-programmers] by Bartosz Milewski.

I cannot say I fully understood everything, I'm far away from doing so, but I
want to write down what I think I learned nonenthless. I hope this will clarify
my thoughts and maybe help someone else.

## What is category theory

Bartosz likes to say that Category theory is the science of structure and
composition. Since functional programmers believe, rightly imho, that programs
are nothing but a huge chain of function composition it might be worthwhile to
take a look at it.

Generally speaking though, category theory is a branch of maths that tries to
define the structure of "things" by only studying the "relationships" between
them. It's closely related to [Set theory][set-theory] but the core difference
is that in category theory it's not allowed to define anything relying on
properties on the elements themselves, but only in terms of their connectivity.

## Category

A category is a bunch of objects connected by arrows with two properties:

- every object always has an identity arrow that points to itself;
- the arrows, also called morphism, must be composable that is if there's a
  function from object **A** to **B** and from **B** to **C** then there must be
  also an arrow from **A** to **C**. Moreover, the composition must be
  associative that is `(f . g) . h = f . (g . h)`.

If the second property sounds like the Haskell definition of function
composition then you're absolutely right! In fact, it turns out there is a
category called **Hask** where the objects are types and the arrows are the
functions between them. This concepts can be translated to other type systems
with no almost no changes.

Here's an example of a simple category

<img src="/post/category-theory/simple-category.svg" alt="simple category" class="image-centered">

## Isomorphism

Another core concept of category theory is the concept of *isomorphism*. This
buzzword means that there's an invertible arrow(a morphism) between two objects
in a category and therefore the two objects are *isomorphic*.

Simply put, this means that the composition between the two arrows is the
identity morphism. In pseudo Haskell notation this could be expressed as `f . g
= id`.

As a trivial example, let's show how `Maybe ()` is isomorphic to `Bool`.

```haskell
isJust :: Maybe () -> Bool
isJust (Just _) = True
isJust Nothing  = False

justIf :: Bool -> Maybe ()
justIf True  = Just ()
justIf False = Nothing

-- function comparison is not implemented in Haskell, but the following
-- statements shouldn't be controversial
-- isJust . justIf = id
-- justIf . isJust = id
```

<img src="/post/category-theory/isomorphism-example.svg" alt="isomorphism example" class="image-centered">

Clearly `Maybe ()` is not semantically the same as `Bool`(i.e. they're not
equal), but they actually carry the same amount of information only in a
different shape.

A use case I find quite interesting for isomorphisms is in compiler
optimizations because theoretically a compiler could substitute a data type with
another data type that is isomorphic to the original one but that's more
efficient.

<!-- TODO: mention that a product type is isomorphic to a function? -->

## Duality

Since a category is just a collection of objects and arrows it's always possible
to create the dual of a category by inverting the direction of the arrows. The
dual category is also called the *opposite* category or **C**<sup>_op_</sup>.
This duality is interesting because it also means that structures built on a
category **C** also have a dual shape in **C**<sup>_op_</sup>. Usually the dual
shapes are called with the same name with a *co* prefix.

For example, here's the opposite category of first example category

<img src="/post/category-theory/simple-category-op.svg" alt="simple category" class="image-centered">

Note how the identity arrows are preserved since their source and target are the
same object.

Also note that the dual of the dual of a category is the category itself that is
(**C**<sup>_op_</sup>) <sup>_op_</sup> = **C**. This is why you can totally give
a nut to a category theorist who asked you a *coco*-nut.

## Product

At this point we have the tools to understand one of the core shapes that is
*Product*s. A *product* between two objects is an object that has two morphisms
to the original objects. Intuitively, a *Product* between two objects is another
object from which it's possible to retrieve the original objects. In Haskell
pseudo-notation these two morphisms, also called projections, can be written as

```haskell
f :: p -> a
g :: p -> b
```

Using this definition there could be multiple valid product objects for any two
objects. To solve this ambiguity there's an additional constraint the product
must satisfy: it must have a unique morphism to any other candidate product
object. I know it sounds scary, but all it means is that the product object
cannot be built from any other candidate product object. I like to think that
it's the "original" object.

Here's the graph for the product **P**. Note that I've omitted the identity
morphisms for simplicity only.

<img src="/post/category-theory/product.svg" alt="product" class="image-centered">

In this diagram, there are two candidate product objects: **P** and **P'**.
However, only **P** is the product between **A** and **B** because there's a
morphism _h_ that can be used to go from **P'** to **P**.

In Haskell, the product of two types is the two element tuple `(a, b)` which can
also be expressed as `data Product a b = Product a b`. The projections are
simply the `fst` and `snd` functions.

As an example of a Haskell type that is a candidate for being the product type
let's consider `(a, b, Int)`. It's easy to show that the morphism that builds
`(a, b)` from it is

```haskell
h :: (a, b, Int) -> (a, b)
h (a, b, _) = (a, b)
```

Therefore we can say that `(a, b, Int)` is not a product of `(a, b)`.

## Coproduct

In category theory, as soon as a pattern is found, the next thing to do is to
apply the duality concept to find its dual pattern. In our case let's build the
dual of the product: the *coproduct*.

This is the product diagram with the arrows inverted.

<img src="/post/category-theory/sum.svg" alt="sum" class="image-centered">

The intuition behind this is that a *coproduct* is an object that can be built
either by **A** or by **B** that is there are two morphism, also known as
injections, that create a coproduct. Sure enough, there could be multiple
coproducts for the same pair of objects, but just like a product, the coproduct
cannot be built from any other coproduct candidate.

This might be a bit harder to recognize, but if we try hard enough we can
recognize the pattern of algebraic data types! In fact, in Haskell we can define
the *coproduct* in the following way

```haskell
data Coproduct a b = CoproA a | CoproB b
```

The two injections are the value constructors.

You might recognize this declaration as the declaration of the `Either` data
type and you'd be right because they're isomorphic to each other!

## Algebra

Both product and coproduct, also known as *sum*, have names of arithmetic
operations but they don't seem to relate to the concept of addition and
multiplication. Accident? As Master Oogway says, there is no accident.

As a refresher here are the definitions of `Product` and `Sum`

```haskell
data Product a b = Product a b
data Sum a b = SumA a | SumB b
```

One of the questions we can ask is how many ways are there to create one of these patterns.

Let's start with the product. If it had a single type parameter then it'd be the
number of distinct values in that type. Since there are more type parameters,
the total number of ways to build a product is the product of the number of
distinct values in each type parameter!

We can apply the same reasoning to sum. However, to build a sum we either need
an _a_ or a _b_ but not both. Therefore the number of ways of building a sum is
just the sum of the values in the type parameters.

Now that we have operations to combine (positive) numbers it might be worthwhile
to find an encoding for them.

Let's start with one. In this line of reasoning one means that there should be
only a value for that type. Not too hard, in Haskell something like the
following will work

```haskell
data One = One
```

There's just one way of instantiating the type One that is by using the _value_
constructor One. Note how One is isomorphic to `()`.

Now that we have One we can two, three, etc.. simply by using sums and products!
Take a look at the following type definitions

```haskell
type Two = Sum One One
type Four = Product Two Two
```

Let's check if this actualy works by enumerating in how many ways we can build
those types:

- we can build a Two with `SumA One` and with `SumB One`;
- a Four instead can be built with `Product (SumA One) (SumA One)`, `Product
  (SumA One) (SumB One)`, `Product (SumB One) (SumA One)` and `Product (SumB
  One) (SumB One)`.

It seems to work, neat!

However we haven't yet encoded zero. Let's fix that now, here's the definition

```haskell
data Zero
```

In Haskell it's possible to define a type without any value constructors which
actually means that we can never create any value of this type. This is also
known as the `Void` type (which is not `void` in c/c++).

Note how Zero is the identity for Sum as well as One is for Product. Also note
how any Product with Zero ends up in no ways to build that type.

If we substitute numbers with types it should become clear that creating new
data types is just a composition over Sums and Products (if we can get any
number we can get any type). This allows the creation of a generic framework to
manipulate data types which I think is how the Haskell compiler is able to
implement a lot of typeclasses automatically.

## Curry-Howard isomorphism

<!-- All of this can be translated to logic where Product is & and Sum is |. -->

## Functor

A *functor* is a mapping between categories which maps both objects and
morphisms. However, in programming we don't usually have multiple categories
because we live only in the category of types. When the target category of the
functor is the same as the source then the functor is called *endofunctor*.

In Haskell a functor can be defined in the following way:

```haskell
class Functor f where
  fmap :: (a -> b) -> (f a -> fb)
```

The mapping between objects is in the fact that `f` must be a type constructor
that is a type that needs another type to be instantiated. For example, `Maybe`
and `Either String` are type constructors because they require only another type
parameter but `Either` is not because it requires 2 type parameters. Note that
also `Either String Int` or `Int` are not type constructors because they're
types which require no additional type parameters.

If we try hard enough we can also see how `fmap` is doing the mapping between
morphism, that is functions, from one category to another one. In fact it takes
the morphism `a -> b` which it maps to the morphism `f a -> f b` in category
**F**.

## Monoid

<!-- Monoid builds
Comonoid destroys -->

## Monad

<!-- > A monad is just a monoid in the category of endofunctors.

Clearly. Well, I still don't think I understand that.

In the category of endofunctors the objects are represented by functors and the
morpshism by natural transformations. A monad can be seen as a monoid where
`join` is the `mappend` rule.

Monad creates values in context
Comonad evaluates context -->

## Yoneda lemma

Every object is completely defined by the morphism towards the other objects.

I find this quite fascinating because if we apply it to the category of the real
world where objects are people and arrows are the relationships between them
then it means that everyone of us can be defined by the people connected to us.
I don't think that's too far away from the truth.

## Haskell things I learned

Along the way I learned why some Haskell-y terms are named like that.

For instance it's called *point free notation* because it doesn't mention the
arguments which can also be called "points" (e.g. calculate the fuction at the
given points). The fact that to achieve this we must use the composition
operator which is represened by a point is a coincidence.

## Category theory is hard

Some of the things I didn't get: T-Algebras, Adjunctions, Kan extensions,
Lawvere theory and a lot more!

<!-- General idea is too abstract for me and I need examples or something I can
relate to in order to better understand things. In fact, the parts that I
understood the most are the ones that provided code examples. Generally speaking
I wasn't able to follow all the demonstrations completely but I think I
understand only some parts of them. -->

## Conclusions


<!-- My favourite chapter was "F-Algebras" because it showed how it's possible to use
*F-Algebras* to factor out explicit recursion in data types using fixed points
making the data type more reusable. I think it's the foundation of
`recursion-schemes` which seems really cool!

Eventually I understood how a lot of problems share the same "structure".

Mathematicians like greek notation and complicated names. -->

[category-theory]: https://en.wikipedia.org/wiki/Category_theory
[cats-for-programmers]:
https://bartoszmilewski.com/2014/10/28/category-theory-for-programmers-the-preface/
[set-theory]: https://en.wikipedia.org/wiki/Set_theory
