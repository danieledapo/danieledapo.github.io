---
title: "Signals from the past"
date: 2017-03-26T01:30:06+01:00
description: "A brief history of signals and how to handle them easily"
---

It's just common knowledge that pressing Ctrl-C will usually terminate a long running command in a shell.
How cool is that? But, how does it work?

Every developer that has done some little system programming on Linux knows that the answer is *signals*.
They are defined in the POSIX standard and are a (very) limited form of inter-process communication.
The idea is really simple: a signal is sent to a program to notify it that an event has occured.
It could be the Ctrl-C event(aka `SIGINT`) or the request of program termination(`SIGTERM`).

The really interesting part is that programs can react to those events providing really cool features.
It's extremely convenient to abort the compilation(without corrupting anything)
of the Linux kernel just by pressing Ctrl-C, isn't it? Therefore one would think that such a useful
thing to have would be *trivial* to implement, right? Wrong. It's really a pain. Let's see why.

## At the beginning there was signal

As defined in the POSIX standard, the primitive for registering a callback to a signal is
[`signal`](http://man7.org/linux/man-pages/man7/signal.7.html). The semantic is pretty easy,
just call it with the signal you're interested in and provide the proper callback.

It turns out writing a correct signal handler is way more harder than writing a multi-threaded
program. The problem is that your program stops as soon as it catches the signal, no matter what it was
doing. That means that *everything* could possibly be in an unconsistent state(yes, even if you protect
your code with locks). At this point I'm hearing you saying: "hey, what can I do in the handler then?". Well,
there are some system calls that are *guaranteed* to work(`open` for example), but they're really
just a few and even basic functions like `printf` are not in there because they acquire locks internally.

The probably safest way to write a signal handler is to set a `sig_atomic_t` flag(that is guaranteed to work)
once a signal arrives and then poll that flag and do something useful when it's set. Polling is always your friend.

But let's ignore the above for now, there is another pitfall in using `signal`: the signal handler gets reset
after the signal is received. Ok, you say, I'm gonna register the handler again in the handler itself and that's it.
Unfortunately no, because it's possible that another signal arrives in the meantime and that's a race condition.

Ok, `signal` is broken, let's build something new.

## Then sigaction came

`signal` was replaced by [`sigaction`](http://man7.org/linux/man-pages/man2/rt_sigaction.2.html) and
obviously that works perfectly...\</sarcasm\> It suffers the same usability problem of `signal`. However,
it introduced some interesting ideas. For example it's possible to retrieve some useful information
about the signal like the sender process pid or the user id(see the `siginfo_t` struct
and the `sa_sigaction` attribute in `sigaction`).

## ...along with sigwait

Then some clever folks thought that we could just block the current thread until a signal arrives;
[`sigwait`](http://man7.org/linux/man-pages/man3/sigwait.3.html) was born. It turns out that's a pretty usable
and easy API. Just spawn another thread that blocks until one of a given set of signals arrives and do something
according to the signal received. This way the code would run "normally" without interrupting the normal execution
of the code eliminating the limitation of the signal handler. That's awesome.

Though, with threads come great responsibility. You need to properly protect shared variables with locks
and so forth, but we're more confident with these kind of problems than with weird interrupt ones.
Moreover, threads have their own sigmask and that can cause some problems, because a thread could
ignore some signals that are worth catching.

This works, but we can do better.

## ...and signalfd eventually killed them all

Some other folks then thought that we could just expose the events via a file descriptor following
the UNIX philosophy. That rocks, UNIX rocks. Once we have a file descriptor we can rule the world!
We can integrate with existing event loops such as [`epoll`](http://man7.org/linux/man-pages/man7/epoll.7.html)
or [`select`](http://man7.org/linux/man-pages/man2/select.2.html) or we can just call the
sync functions. Everything that's left to us is just reading the `siginfo_t` from the file descriptor
and call the appropriate function for the signal received. That's what I mean for "easy".

---

### Funny things

- signals are aggregated, that means that on the imaginary "signals queue" there
  will be at most one instance of any signals. This problem affects
  [`signalfd`](http://man7.org/linux/man-pages/man2/signalfd.2.html) as well as
  `signal` and `sigaction`, but I don't really see where the problem is, because
  you eventually get the signal...;

- most system calls can be interrupted by a signal and in those cases they
  return `EINTR` that is "the call was interrupted, please retry but consider
  what we've done so far"; I always wondered why there are that big warnings in
  several programming languages docs saying that the API could return less
  things than it was asked for. This is certainly a reason.

### Literature

- <https://en.wikipedia.org/wiki/Unix_signal>
- <https://ldpreload.com/blog/signalfd-is-useless> <- this is really interesting
- <http://www.linuxprogrammingblog.com/all-about-linux-signals?page=1>
