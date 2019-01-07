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
known as the `Void` type (which is not the same as `void` in c/c++).

Note how Zero is the identity for Sum as well as One is for Product. Also note
how any Product with Zero ends up in no ways to build that type.

If we substitute numbers with types it should become clear that creating new
data types is just a composition over Sums and Products (if we can get any
number we can get any type). This allows the creation of a generic framework to
manipulate data types which I think is how the Haskell compiler is able to
implement a lot of typeclasses automatically.

## Curry-Howard isomorphism

It's possible to show that products and sums can be transformed into constructs
in logic. In particular a product would be *logical and* or `&&` and a sum
would be a *logical or* or `||`. Moreover, Zero would correspond to false and
One to _true_.

It's quite easy to show that the familiar properties about such constructs hold.
For example `a && false = false` holds because if we translate the proposition
to sums and products it eventually evaluates to `Product a Zero` which we know
is always `Zero`. This same approach can be used to show that `a || true =
true`.

We can also show how functions can be mapped to *logical implication* or `a ->
b` meaning that if _a_ is _true_ then we can say _b_. The obvious question is
here is what happens when _a_ is _false_? Well, the function would be of type
`Zero -> b` but if we stare long at it we might realize that we can never call
that function because Zero doesn't have any values! Therefore we cannot conclude
the proposition with _b_.

An interesting interpretation of this is that since all the possible types are a
combination of products and that we can map sums and products to logical
constructs then we can say that there's an isomorphism between types and logic!
This is known as the [Curry-Howard isomorphism][curry-howard-isomorphism]. Under
this point a view, checking if a program type checks is no different than
checking whether a logical proposition holds or not.

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

<!-- TODO: provide an example implementation and refine this -->

If we try hard enough we can also see how `fmap` is doing the mapping between
morphisms, that is functions, from one category to another one. In fact it takes
the morphism `a -> b` which is mapped to the morphism `f a -> f b` in category
**F**.

The dual of a functor isn't particularly exciting though, because by inverting
the direction of the mapping we still get a functor.

## Monoid

Another interesting pattern is the *Monoid*. A *Monoid* is an object equipped
with a unit arrow and an associative multiplication arrow.

In Haskell this could be expressed as

```haskell
class Monoid m where
  mempty :: m
  mappend :: m -> m -> m
```

Note that in Haskell `mempty` is not actually an arrow but a value of type `m`
because it's quite easy to show that `m` is isomorphic to `() -> m`.

These morphisms must satisfy the following laws

```haskell
mappend mempty m = m
mappend m mempty = m

(mappend x . mappend y) . mappend z = mappend x . (mappend y . mappend z)
```

To put it simply, mappend is associative "multiplication-like" morphism and
mempty is the identity value under mappend.

I know this definition isn't very clear, hopefully an example helps. Probably
the simplest example of a Monoid is the product of numbers. In fact it's easy to
show that the mempty value is 1 and that mappend is simply the `*` operator.

Here's a possible implementation in Haskell

```haskell
data ProductM = ProductM Int

instance Monoid ProductM where
  mempty = ProductM 1
  mappend (ProductM a) (ProductM b) = ProductM (a * b)
```

The real definition is a bit different because ProductM must also be a
*Semigroup* in order to be a Monoid. Let's ignore this detail by just saying
that a Semigroup is a Monoid without the identity value that is it just has an
associative morphism.

Besides the usual unwrapping and rewrapping this should be quite straightforward
to follow. I won't bother demonstrating that `ProductM` indeed satisfies the
monoid requirements because I think we can all agree that it's just
multiplication.

However numbers form a monoid over addition too. In this case the mempty value is
0 and mapped is simply the `+` operator.

Monoids are everywhere though! For example basically every container I can think
of is a Monoid under the concatenation morphism. Here's an hopefully easy to
understand implementation for lists in Haskell

```haskell
instance Monoid [a] where
  mempty = []
  mappend = concat
```

It should be quite evident how this definition can be mapped to nearly every
container.

Also note how the implementation of Monoid for lists is totally generic over the
contained value. That's why lists are also known as *Free monoids* because they
give you a Monoid _for free_.

To show you how much monoids are useful let's pick a simple task: find the sum
and product of a sequence of numbers in a single pass. Here's how it could be
done in Haskell

```haskell
f :: [Int] -> (Product Int, Sum Int)
f = foldl1 mappend . map (\n -> (Product n, Sum n))
```

The realization here is that the tuple is also a Monoid as long as its
components do!

## Comonoid

If there is a Monoid there must be also a *Comonoid* in the opposite category,
right? Indeed there is. Unfortunately I don't understand it fully.

After reversing the arrows eventually we get to a definition like the following

```haskell
class Comonoid m where
  destroy :: m -> ()
  dup :: m -> (m, m)
```

where dup must be associative and must return an exact copy of the input. In
Haskell the only possible definitions of these morphisms are

```haskell
destroy :: a -> ()
destroy _ = ()

dup :: a -> (a, a)
dup a = (a, a)
```

And that's why it's hard for me to understand. With only this possible
definition I'm not able to see a general pattern here. My intuition is that
since a Monoid builds a somewhat bigger object at each mappend the Comonoid, its
dual, must gradually destroy the already built object. I don't think that's 100%
accurate though.

It seems [that a Comonoid could be used to model a linear type
system][comonoid-linear-types] upon which many new languages are based upon.
Rust and Idris are examples of languages with a linear type system. It seems
interesting but I haven't found the time to explore this topic yet.


## Monad

It's time for the big one: the *Monad*.

> A monad is _just_ a monoid in the category of endofunctors.

If the definition doesn't say anything to you then I'll try my best to explain
my intuition. First of all, the definition says we're in the category of
*endofunctors* which means that objects in this category are functors. Since a
monad is a monoid by definition, it means that there must be an arrow that
somehow multiplies two endfunctors to get a new one. This arrow is called a
*natural transformation*. Moreover, there must also be the unit arrow too. I
know this is still a bit nebulous, so let's take a look at how this definition
looks in Haskell

```haskell
class Functor m => Monad m where
  unit :: a -> m a
  join :: m (m a) -> m a
```

If you're familiar with Haskell this doesn't seem the typical monad
definition(there's no *bind* operator), but the two are equivalent. I think we
can vaguely see how *unit* is similar to mempty, it just takes a value and wraps
it inside the monad. The join function is a bit more difficult to relate to
mappend, but the intuition is that join is actually doing the product with
itself essentially flattening the structure.

<!-- TODO: complete me -->

The canonical Haskell definition of the Monad requires the definition of the
bind operator or `>>=`. We can show how it's possible to automatically implement
it for any monad defined the "category theory" way. Here's how it could be
automatically implemented

```haskell
(>>=) :: Monad m => (a -> m b) -> m a -> m b
f (>>=) m = join . fmap f $ m
```

My intuition for the Monad is that it's just a way of composing functions
running in a given context.

## Comonad

This is the last pattern we'll talk about: the *Comonad* that is the dual of the
Monad.

<!-- TODO: complete me -->

My intuition for the Comonad is that since the Monad builds up a computation in
a context by composing functions then a Comonad runs such a computation.

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
[curry-howard-isomorphism]: https://en.wikipedia.org/wiki/Curry%E2%80%93Howard_correspondence
[comonoid-linear-types]: https://stackoverflow.com/questions/23855070/what-does-a-nontrivial-comonoid-look-like
