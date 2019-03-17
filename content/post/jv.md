---
title: "JV"
date: 2019-03-17T10:35:15+01:00
draft: true
---

JSON is a really common format and I think it gets the job done 90% of the time.
However, it's not perfect and it lacks some features that would be handy. In
particular, it doesn't have a way to specify a reference to a value in the
document. A common solution is to store the id of the referenced object, but
it's not always possible (or convenient) to do so.

Anyway, it's trivial to come up with a custom string format that represent a
path to a value inside the JSON document. For example, consider the following
JSON document

```json
{
    "people": [
        {"name": "Dan"},
        {"name": "Luke"},
        {"name": "Frank"}
    ]
}
```

A naive reference format reference could express paths like the following:

- `#/` is the document root;
- `#/people` is the path to the "people" array;
- `#/people/0` maps to the first object inside the "people" array;
- `#/people/0/name` maps to the "name" of the first object inside the "people"
  array;

We could then store these paths inside the JSON document to express
relationships between values. For example, we could add a "brothers" key to
store the paths to each person's brothers.

```json
{
    "people": [
        {"name": "Dan", "brothers": ["#/people/2"]},
        {"name": "Luke"},
        {"name": "Frank", "brothers": ["#/people/0"]}
    ]
}
```

At work I have to deal with JSON documents like the above and I can say they're
particularly annoying to manually inspect because I cannot easily jump to a
path. I could use [jq] to query the document but, besides having to convert the
reference format to the one jq likes, there doesn't seem to exist an interactive
jq that supports moving inside the document without entering a query first.

I then decided to shave this yak by myself and I started working on [jv]: a
terminal application to quickly inspect json documents providing jump to path
functionality.

## Viewer

You can think of JV as a very simple read-only vim that allows jumping to a
reference.

I started implementing all the core features like cursor movement, paging, line
truncation to avoid line wrapping, JSON syntax highlighting, formatting, etc...
until one night I realized I was doing it all wrong.

All the code I wrote up until that point was assuming that each character would
take up exactly one column when printed to the terminal, but that is not always
the case! A lot of Unicode characters, namely emojis or [CJK ideographs], can
span multiple columns!

After all, terminals were built way before Unicode was a thing so that makes
sense. Armed with hope, I tried to understand if there was some sort of list
that defines the width for each unicode symbol. I went through the [Unicode
Annex #11] page which describes how to interoperate with Asian character sets
and I realized that the thing is tricky to say the least.

Since I don't have to deal with Unicode documents at work, I took the shortest
route that is enforce only ASCII characters.

## Tabs

tabs are not always 8, they depend on the position they're rendered to.
a lot of frameworks do not handle them.

Tabs are a thing of the past, right? Make and Go still use them...


## JV

Demo

## Conclusions

I'm not proud of the code I've written. I think there are a lot of abstractions
that I could have come up with to make the code cleaner and my life easier, but
as soon as I got something working I didn't enjoy writing it anymore and so I
didn't want to invest more time in it.

Anyway, I think it works quite well now and it's really useful for my daily job
so I'm ok with it. After all, an ugly tool is still better that nothing.

[jq]: https://stedolan.github.io/jq/
[jv]: https://github.com/d-dorazio/jv
[CJK ideographs]: https://en.wikipedia.org/wiki/CJK_Unified_Ideographs
[Unicode Annex #11]: http://www.unicode.org/reports/tr11/
