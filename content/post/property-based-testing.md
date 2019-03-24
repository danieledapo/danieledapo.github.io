---
title: "Use Property based testing whenever you can!"
date: 2018-09-13T23:03:31+02:00
---

Recently I had to implement some geometric data structures and algorithms in
Rust so that I could use them in [matto]. In particular, I built a simple [K-d
tree][kdtree], an algorithm to find the [convex hull], the [K-means
clustering][k-means] algorithm and a couple of polygon related utilities.
Building such things is relatively easy for the most part, but the corner cases
are really tricky to handle properly. In this post I'd like to show how I used
property based testing to even recognize those corner cases.

## Corner cases are hard

The first thing I had to implement was the [K-d tree][kdtree] so I guess the
story begins from there. To start off, let's briefly discuss what a K-d tree is
and what is used for. A K-d tree is a tree that's used to partition
k-dimensional points in a way that allows for fast range querying and nearest
neighbors searching. Glossing over the details, a K-d tree is built by
recursively partitioning the points by a plane of a different axis at each level
of the tree. The algorithms for performing a range search or a nearest neighbors
search are a bit more involved, but they both exploit the partitioning nature of
the tree to prune which branches of the tree to visit in order to speed up the
search. For a more detailed introduction Wikipedia has a good [article on
this][kdtree].

The way I approached this problem was to first come up with an unoptimized, but
readable version of a K-d tree, write some basic unit tests and eventually
perform some optimizations. That's what I did and I was feeling confident about
the code being correct since all the unit tests I wrote were passing and the
implementation was quite straightforward. There could be no bugs, I thought. I
was wrong. As soon as I started using the K-d tree I immediately noticed some
wrong results. I looked at the implementation again, hoping that I made some
stupid errors that I could catch by just reading the code. I found nothing.
After all, I wrote the code to be as straightforward as possible. I had to find
some examples for which the K-d tree was misbehaving then. I don't know about
you, but I'm really lazy and searching for inputs that my code doesn't handle
properly is always annoying to me. If only there was something that could
automatically find those corner cases for me...

Well, it turns out there is and it's called property based testing!

This technique consists of wring tests that describe some properties of the
thing being tested no matter what the actual value of the input is. For example,
a property about the `reverse` operation on a vector could be that calling
`reverse` twice should give the exact same input that we started with. In Python
this might look like the following:

```python
def prop_reverse_twice_is_input(arr: List[int]):
    assert list(reversed(list(reversed(arr)))) == arr
```

This might seem like nothing, but it's a really powerful way to write tests
because it shifts the focus from what the test data is to what we're testing.

What's super damn cool is that since you specify a type or model of inputs for
which the property holds, the test framework is allowed to generate multiple
different input values conforming to the model to test your function. Moreover,
most of the frameworks also implement some form of input shrinking so that they
try to find the minimal failing input that breaks the test.

This is exactly what I needed. I could write some simple properties about the
K-d tree and let the testing framework do all the hard work of finding some
inputs that break my tests. I needed a test framework though, but fear not. Rust
has a great library ecosystem and it turns out there are multiple crates for
property based testing. I decided to try [proptest] which seemed the most
active. Spoiler alert: it's really good!

I quickly came up with a test to test the nearest neighbor algorithm. After all,
I know how to find the nearest point of another: I can iterate over all points
in the tree keeping the ones with the lowest distance to the target point. This
is really inefficient for large numbers of points(that's why I needed a K-d tree
after all!) but I don't care about speed(too much) when running tests. The code
I wrote looks like the following:

```rust
proptest! {
    #![proptest_config(proptest::test_runner::Config::with_cases(500))]
    #[test]
    fn prop_kdtree_nearest_neight_same_as_loop(
        points in proptest::collection::hash_set((0_u32..255, 0_u32..255), 1..100),
        to_search in (0_u32..255, 0_u32..255)
    ) {
        same_as_nn_brute_force_loop(points, to_search);
    }
}

fn same_as_nn_brute_force_loop(points: HashSet<(u32, u32)>, to_search: (u32, u32)) {
        let points = points
            .into_iter()
            .map(|(x, y)| (PointU32::new(x, y), ()))
            .collect::<Vec<_>>();

        let tree = KdTree::from_vector(points.clone());
        let to_search = PointU32::new(to_search.0, to_search.1);

        let tree_closest_point = tree.nearest_neighbor(to_search);

        let brute_force_closest_point = points
            .iter()
            .min_by_key(|(pt, _)| pt.squared_dist::<i64>(&to_search));

        assert!(tree_closest_point.is_some());
        assert!(brute_force_closest_point.is_some());

        let brute_force_closest_point = brute_force_closest_point.unwrap().0;
        let tree_closest_point = tree_closest_point.unwrap().0;

        assert_eq!(
            brute_force_closest_point.squared_dist::<i64>(&to_search),
            tree_closest_point.squared_dist::<i64>(&to_search),
        );
}
```

I know it's a lot of code to digest, but the idea should be easy to understand.
Given a set of points and a point to find the nearest neighbor of, then the K-d
tree implementation of the nearest neighbor should match the brute force
implementation. Note that the implementation doesn't assert that the two methods
return the same point, since there could be more points at the same distance
from the target one.

I ran the test suite again and bam! The test failed because `proptest` found
multiple inputs for which the K-d tree was giving the wrong results. Awesome! I
debugged the problem with the smallest input that `proptest` showed me and then
I quickly realized and fixed the bugs.

I did the exact same thing to test the range query function and the results were
the same: `proptest` found inputs for which the tests failed, I debugged those
minimal failing inputs and fixed all the things! Here's the test if you're
curious:

```rust
proptest! {
    #![proptest_config(proptest::test_runner::Config::with_cases(500))]
    #[test]
    fn prop_kdtree_in_range_same_as_loop(
        points in proptest::collection::hash_set((0_u32..255, 0_u32..255), 0..100),
        rect in (0_u32..255, 0_u32..255, 0_u32..255, 0_u32..255)
    ) {
        same_as_range_brute_force_loop(points, rect);
    }
}

fn same_as_range_brute_force_loop(points: HashSet<(u32, u32)>, rect: (u32, u32, u32, u32)) {
    let points = points
        .into_iter()
        .map(|(x, y)| (PointU32::new(x, y), ()))
        .collect::<Vec<_>>();

    let tree = KdTree::from_vector(points.clone());
    let range =
        BoundingBox::from_dimensions_and_origin(&PointU32::new(rect.0, rect.1), rect.2, rect.3);

    let mut contained_points = tree.in_range_iter(&range).collect::<Vec<_>>();
    let mut brute_force_contained_points = points
        .iter()
        .filter_map(|(pt, val)| {
            if range.contains(pt) {
                Some((pt, val))
            } else {
                None
            }
        })
        .collect::<Vec<_>>();

    sort_points(&mut contained_points);
    sort_points(&mut brute_force_contained_points);

    assert_eq!(contained_points, brute_force_contained_points);
}
```

## Document by testing

Then I had to implement the algorithm to find the [convex hull] of a given set
of points. The algorithm was easy to implement, but as usual the devil lies in
the details. Especially if we're talking about geometry ;) .

I wrote a first implementation and then I started wondering about which
properties I could use to test it with. It turns out the definition of the
convex hull is itself the best property I could use! Let me quote the relevant
parts from Wikipedia:

> the convex hull of a set X of points is the smallest convex set that contains X

Unfortunately that property relies on a function that I didn't have at the time
that is the function to check whether a point lies inside a polygon or not.
Since I'm a lazy guy, I procrastinated and implemented a simpler property that
stated that all the points of the convex hull should be in the original set of
points.

```rust
proptest! {
    #![proptest_config(proptest::test_runner::Config::with_cases(100))]
    #[test]
    fn prop_convex_hull_lies_on_boundary(
        points in prop::collection::vec((0_u8..255, 0_u8..255), 1..100)
    ) {
        _prop_convex_hull_lies_on_boundary(points)
    }
}

fn _prop_convex_hull_lies_on_boundary(points: Vec<(u8, u8)>) {
    let points = points
        .into_iter()
        .map(|(x, y)| Point::new(x, y))
        .collect::<Vec<_>>();

    let hull = convex_hull(points.iter().map(|p| p.cast::<f64>()));

    for pt in hull {
        let pt = Point::new(pt.x as u8, pt.y as u8);

        let on_boundary = points.iter().find(|h| **h == pt).is_some();

        assert!(on_boundary);
    }
}
```

It's not as comprehensive as the definition since it doesn't ensure the convex
hull is correct, but `proptest` was still able to find a nasty bug when there
are multiple points on the same y. Impressive.

Eventually I had to implement the polygon `contains` method anyway, so I was
also able to add the following test which closely matches the definition of a
convex hull.

```rust
proptest! {
    #![proptest_config(proptest::test_runner::Config::with_cases(500))]
    #[test]
    fn prop_convex_contains_all_the_points(
        points in prop::collection::hash_set((0_u8..255, 0_u8..255), 3..100)
    ) {
        let points = points
            .into_iter()
            .map(|(x, y)| Point::new(f64::from(x), f64::from(y)))
            .collect::<Vec<_>>();

        let hull = convex_hull(points.clone());
        prop_assume!(hull.len() > 2);

        let hull = Polygon::new(hull).unwrap();

        for pt in &points {
            assert!(hull.contains(pt));
        }
    }
}
```

Look at that! Not only it's an incredibly small and valuable test, but it's
awesome documentation too! It's exactly what the users of the function should
expect from a convex hull, nothing more and nothing less. I'm [not a big fan of
documentation][my-take-on-docs] but I'm more than happy to write tests that are
also good documentation.

Luckily this test didn't find any bugs in my convex hull implementation, but it
did find some in my polygon implementation! Wonderful, not only it consolidated
my convex hull implementation that I was already using, but it found bugs in the
new algorithm I was developing(which also had some unit tests)! This was one of
the few occasions that I felt happier than sad after having seen my tests fail
:) .

This also shows another advantage of property based testing: the same test can
actually cover multiple algorithms. In fact this test ensures that the polygon
`contains` method is coherent with my `convex_hull` implementation. This means
that I don't need to write a bunch of other tests for `contains` because a lot
of cases are already covered by the `convex_hull` test. Less code to write and
maintain, yay!

## Conclusions

I already used property based testing before in Haskell and I always liked the
idea. However, the problem domain was different and I don't think I fully
appreciated the power of this tool. After this experience, I can definitely say
that I'll empower this technique as often as possible because it allows me to
use a mentality where I can just write the algorithm and then let the test
framework verify it holds the properties. It's a primitive version of a formal
proof if you wish.

Property based testing isn't perfect though, there are a couple of gotchas to
keep in mind:

- it's not deterministic. Given that inputs are pseudo-random it's totally
  possible to see the test fail and succeed without any code changes at all.
  Most of the people don't like this, because nobody likes flaky tests. I think
  this is actually a good thing though, because the test is failing for a real
  bug and not because the test is not robust enough.
- it's not exhaustive. It's possible that the properties we are able to test do
  not cover all the code paths that we'd like to. Besides fine tuning the test
  model I think the best way to go is to create different models to exercise
  even the same property under different inputs. If that's still not possible,
  unit testing is still an option.

To put an end to this disconnected rambling, I don't think you should use
property based testing for everything you write, but I encourage you to try at
least. I think that having a test suite that's composed by unit tests and
property tests is really worth it and it can find bugs you never thought of in
the first place!

[my-take-on-docs]: {{< relref "my-take-on-documenting-code.md" >}}
[matto]: https://github.com/d-dorazio/mattors
[kdtree]: https://en.wikipedia.org/wiki/K-d_tree
[convex hull]: https://en.wikipedia.org/wiki/Convex_hull
[k-means]: https://en.wikipedia.org/wiki/K-means_clustering
[proptest]: https://github.com/AltSysrq/proptest
