---
title: "My Take on Documenting Code and Readability"
date: 2018-04-30T12:08:32+02:00
description: "Here I present my tips I use to write readable and well commented code."
---

Recently we had a workshop at my workplace about how to document code.
Unfortunately I wasn't able to attend it, but the topic was in the air in the
following days and I chatted about it a few times with my colleagues. I thought
this was a solved problem because a lot of ink has already been spilled on it,
but after a lot of discussions it became clear to me that it is not the case.

I'm going to use the following snippet(that I slightly adapted from real
production code) to show the guidelines I tend to use in most cases.

```python
def should_analyze(obj):
    # ...business logic
    return True

def get_category(obj):
    # ...business logic
    return "cat1"

def get_last_event_timestamp(obj):
    # ...business logic
    return datetime.datetime(2010, 1, 1)

def analyze_all(objects):
    res = {}

    for obj in objects:
        category, analysis = analyze(obj)

        if category not in res:
            res[category] = []

        res[category].append(analysis)

    return res

def analyze(obj):
    # Categorize the object
    category = get_category(obj)
    # Find object's last event timestamp
    last_event_timestamp = get_last_event_timestamp(obj)
    # Find out if the object should be analyzed
    analyze = should_analyze(obj)
    if not analyze or not last_event_timestamp:
        return None, None
    # Calculate the delta and add the result to the category
    timestamp = datetime.datetime.now()
    analysis = {
        "timestamp": timestamp,
        "delta": timestamp - last_event_timestamp,
        "last_event_timestamp": last_event_timestamp,
    }

    return category, analysis
```

## First things first

The snippet from above has multiple functions in it and we'll assume it's part
of a single file. Let's also assume that you didn't write this code and that
you're trying to understand what it does. You'll have to read the code of a lot
of functions: `get_category`, `get_last_event_timestamp`, `should_analyze` and
`analyze_all` before reaching the meat of the module that is the `analyze`
function. In this simple example it's not a big deal since the amount of code is
relatively small, but imagine how much code you should read(or skip) before
reaching the main functions in a much bigger file. By simply putting the entry
points or main functions at the very top of the file we can make the purpose of
the file more clear automatically documenting it.

This reasoning has further implications though. Note how the `analyze` function
calls `get_category`, `get_last_event_timestamp` and `should_analyze` in its
body in this very specific order. On the other hand, the function definitions
are not in the same order. This means that while reading the body of `analyze`
you'll have to jump at random locations in the file to find the other functions.
This gets out of hand pretty quickly. Instead, if the functions follow the same
order they're called then finding the function will be so much simpler. This
reasoning can also be applied to the `analyze_all` function that relies on the
`analyze` function.

Are we done with reordering things? Nope. Note that the `category` variable
inside `analyze` is used only in the very last `return` statement. Besides being
inefficient, declaring a variable at a location far from where it's first used
is annoying because it's information that you have to store in your brain for a
long time before it can be useful. I don't know about you, but if I don't use
notions my brain will likely forget them. Therefore, I think it's best to define
variables near to the location of first use.

Let's reorganize the above snippet following this idea.

```python
def analyze_all(objects):
    res = {}

    for obj in objects:
        category, analysis = analyze(obj)

        if category not in res:
            res[category] = []

        res[category].append(analysis)

    return res

def analyze(obj):
    # Find object's last event timestamp
    last_event_timestamp = get_last_event_timestamp(obj)
    # Find out if the object should be analyzed
    analyze = should_analyze(obj)
    if not analyze or not last_event_timestamp:
        return None, None
    # Calculate the delta and add the result to the category
    timestamp = datetime.datetime.now()
    analysis = {
        "timestamp": timestamp,
        "delta": timestamp - last_event_timestamp,
        "last_event_timestamp": last_event_timestamp,
    }
    # Categorize the object
    category = get_category(obj)
    return category, analysis

def get_category(obj):
    # ...business logic
    return "cat1"

def get_last_event_timestamp(obj):
    # ...business logic
    return datetime.datetime(2010, 1, 1)

def should_analyze(obj):
    # ...business logic
    return True
```

I think that at this point, we:

- made clear where's the meat of the file that is in the `analyze` function because it's at very top;
- optimized the amount of scrolling you have to do to read the entire file.

These might seem a small things, but once you'll get accustomed to it you'll
find yourself reading code much more quickly and easily.

Now that the overall snippet structure satisfies me, it's time to concentrate on
the `analyze` function.

## Code Comments

I'm not a big fan of comments mainly because of two reasons:

- they get outdated pretty quickly and if they're not updated diligently they'll
  do exactly the opposite of what they're meant to;
- to me they are often abused to try to improve the code clarity without
  actually having to change any code. This happens especially when the developer
  isn't confident enough with the changes he'd be making. There are plenty of
  reasons why it'd be the case, but the lack of a test suite should be a clear
  example.

Before I put any comments in the code I take a step back, I read the code from
scratch again and ask myself what I could do to improve the code first. Most of
the time this involves renaming things and moving stuff around as described
above. If the code still looks too complex then eventually I'll write some
comments.

> Comments should explain why not how.

In any case I tend to not write comments to explain what the code is doing, but
_why_ is doing so. Comments are also often used to give an overview of an
algorithm or a complex idea but it should still be more focused on the _why_ and
not the _how_ because the latter is already expressed in the code.

Let's take a look at the snippet now. You could say that the snippet is actually
well commented and the comments are there for a purpose. Well, imho the comments
in the `analyze` function are quite useless. They do not provide any additional
information I don't already know just by looking at the code. Let's consider the
very first comment for example:

```python
# Find object's last event timestamp
last_event_timestamp = get_last_event_timestamp(obj)
```

What is the comment telling us that the code is not already expressing? The only
thing I could think of is that maybe `get_last_event_timestamp` is not a simple
getter function, but it needs to perform some heavier computation to find the
timestamp of the last event. If that's the case then I'd rather change the
function name to `find_last_event_timestamp` to make it explicit. In any case, I
think the comment is just superfluous and can be removed. A similar reasoning
can be applied to the second and fourth comments.

The third comment is worse imho and I'll go as far as to say that it's actually
misleading and is harming code clarity.

```python
# Calculate the delta and add the result to the category
timestamp = datetime.datetime.now()
analysis = {
    "timestamp": timestamp,
    "delta": timestamp - last_event_timestamp,
    "last_event_timestamp": last_event_timestamp,
}
```

We can break the comment into two propositions: `Calculate the delta` and `add
the result to the category`. I won't spend too much time on the former because
it's of the same "type" of the comments we have already considered. I'd like to
concentrate on the latter. The proposition doesn't make any sense, because the
code is not appending anything to anything. I think the comment got outdated and
I think I also see how. At the origin there was no `analyze_all` function and
everything was done in `analyze` and as such there was some code like the
following:

```python
# Calculate the delta and add the result to the category
timestamp = datetime.datetime.now()
analysis = {
    "timestamp": timestamp,
    "delta": timestamp - last_event_timestamp,
    "last_event_timestamp": last_event_timestamp,
}
res[category].append(analysis)
```

which aligns to what the comment is saying. The problem is that eventually the
code was rightly refactored into an `analyze_all` and `analyze` functions but
the comments were not updated. This is a major problem because it leads to more
confusion than other. Remember, the source of truth is _always_ the code. I
think that at this point we all agree to nuke that misleading comment.

The resulting code of the `analyze` function looks like the following now.

```python
def analyze(obj):
    last_event_timestamp = get_last_event_timestamp(obj)
    analyze = should_analyze(obj)
    if not analyze or not last_event_timestamp:
        return None, None
    timestamp = datetime.datetime.now()
    analysis = {
        "timestamp": timestamp,
        "delta": timestamp - last_event_timestamp,
        "last_event_timestamp": last_event_timestamp,
    }
    category = get_category(obj)
    return category, analysis
```

I can say we definitely didn't lost any clarity with the benefit of having less
text to read.

## Whitespace matters

As of now the `analyze` function looks like a complex algorithm because
everything is so dense. Well, it turns out that's not the case. The problem is
that there is not even a single empty line in that function. Empty lines are
powerful because they allow us to group text into meaningful sections(aka
paragraphs). I usually group lines in the following sections:

- variables initialization;
- computations with related side effects;
- any `for`, `while`, `if` or whatever constructs your language has to offer.

Let's modify the `analyze` function to follow this principle.

```python
def analyze(obj):
    last_event_timestamp = get_last_event_timestamp(obj)
    analyze = should_analyze(obj)

    if not analyze or not last_event_timestamp:
        return None, None

    category = get_category(obj)

    timestamp = datetime.datetime.now()
    analysis = {
        "timestamp": timestamp,
        "delta": timestamp - last_event_timestamp,
        "last_event_timestamp": last_event_timestamp,
    }

    return category, analysis
```

Even though the diff is small, I think the benefits are quite big because the
logical sections in the functions are crystal clear now. Also, did I mention
that refactoring the different sections into separate functions is as easy as
cut and paste?

## Conclusions

At this point I'm quite satisfied with the results and I don't think there are
other improvements I'd make. I'd argue that by following simple steps we were
able to greatly improve the quality of the code.

I'd like to leave you with a consideration. The snippet that I used through out
this post is trivial and most of real world code is not. This doesn't mean that
the code should look complex though.

> Code should look simple even when it's doing complex stuff.
