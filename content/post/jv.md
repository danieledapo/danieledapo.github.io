---
title: "JV"
date: 2019-03-17T10:35:15+01:00
draft: true
---

JSON is a really common format and I think it gets the job done 90% of the time.
However, it's not perfect and it lacks some features that would be handy
sometimes.

In particular, it doesn't have a way to specify a reference to a value in the
document. A common solution is to store the id of the referenced object, but
it's not always possible (or convenient) to do so.

Anyway, it's trivial to come up with a custom string format that represent a
path to a value. For example, consider the following JSON document

```json
{
  "people": [{ "name": "Dan" }, { "name": "Luke" }, { "name": "Frank" }]
}
```

A very simple reference format could support the following queries:

- `#/` is the document root;
- `#/people` is the path to the "people" array;
- `#/people/0` maps to the first object inside the "people" array;
- `#/people/0/name` maps to the "name" of the first object inside the "people"
  array;

We could then store these paths inside the JSON document to express
relationships between values. For example, we could add a "brothers" key to
every object in the "people" array.

```json
{
  "people": [
    { "name": "Dan", "brothers": ["#/people/2"] },
    { "name": "Luke" },
    { "name": "Frank", "brothers": ["#/people/0"] }
  ]
}
```

At work I have to deal with JSON documents like the above and I can say they're
particularly annoying to manually inspect because I cannot easily jump to a
path. I could use [jq] to query the document but, besides having to convert the
reference format to the one jq likes, there doesn't seem to exist a sort of
"interactive jq" that supports moving inside the document without entering a
query first.

Therefore, I decided to shave this yak by myself and I started working on [jv]:
a terminal application to quickly inspect json documents that provides a quick
jump to path functionality.

## Viewer

To start things off, I started implementing all the basic features like cursor
movement, paging, line truncation, JSON syntax highlighting, formatting, etc...
and things were going smoothly until one night I realized I was doing it wrong.

All the code I wrote up until that moment was assuming that each character would
take up exactly one column when printed to the terminal, but that is not always
the case! In fact, a lot of Unicode characters, namely emojis or [CJK
ideographs][cjk ideographs], can span multiple columns!

Armed with hope, I tried to understand if there was some sort of table that
defines the width for each unicode symbol. I went through the [Unicode Annex
#11][unicode annex #11]] page which describes how to interoperate with Asian
character sets and I realized that the thing is tricky to handle to say the
least.

Since I don't have to deal with Unicode documents at work (and I'm lazy) I took
the shortest route that is to enforce that the text contains only ASCII
characters.

## Tabs

I can live without supporting Unicode characters, but I do want to support tabs
mostly for a matter of honour. I think tabs should not be used anymore because
they're a thing of the past, but lots of software(e.g. Go, Make, the Linux
kernel, etc...) do not agree and instead they make heavy use of tabs.

Initially I thought that I could just simply replace each tab with 8 spaces and
get away with it. Poor me.

Tabs are not always of a fixed size, because they depend on the position they're
rendered from! In fact a tab spans all the columns until the next tab stop which
is usually 8 columns. Hopefully the following examples will make it more clear.

```shell
$ printf '########\tciaociao\t########'
# output: ########        ciaociao        ########

$ printf '########\tciao\t########'
# output: ########        ciao    ########

$ printf '#######\tX\t########'
# output: ####### X       ########
```

This was really annoying to implement because I don't like that the model has to
know how and where it will be rendered in order to change its representation. It
doesn't sound right to me. Anyway, with enough hacks I was able to get something
working.

## JV

After I managed to implement all of this, adding the real core functionality was
pretty much straightforward.

Here's a small demo that shows some of what [jv] can do.

[![asciicast](https://asciinema.org/a/233199.svg)](https://asciinema.org/a/233199)

I plan to add some more features in the near-ish future, namely it really needs
a search mode where you can search for plain strings inside the document.

## Conclusions

I'm not proud of the code I've written. I think there are a lot of abstractions
that I could have come up with to make the code cleaner and my life easier, but
as soon as I got something working I basically stopped working on it because I
was frustrated by all the ugly hacks I've put in the code.

Anyway, I think it works quite well now and it's really useful for my daily job
so I'm ok with it. It's not pretty, but it kinda works and after all, an ugly
tool is still bettern than nothing.

[jq]: https://stedolan.github.io/jq/
[jv]: https://github.com/d-dorazio/jv
[cjk ideographs]: https://en.wikipedia.org/wiki/CJK_Unified_Ideographs
[unicode annex #11]: http://www.unicode.org/reports/tr11/
