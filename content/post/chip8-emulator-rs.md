---
title: "Chip8 emulator in Rust!"
date: 2019-01-27T22:31:44+01:00
---

Some coworkers of mine are really into game development and emulators in
particular. In fact, these days they're building emulators for the Playstation 1
and the Game Boy! I decided to write an emulator too just for fun. However, I
didn't want to implement such complex consoles, but I wanted to start off with a
simpler platform first. According to the internet the simplest emulator you can
build is a Chip8 emulator so I went for it.

This blog post is not meant to be a comprehensive tutorial of how to build a
Chip8 emulator, but it's just a quick summary of me building it.

You can find the source code for my emulator at
<https://github.com/d-dorazio/chip8>.

## What is a Chip8

First things first, if we want to build a [Chip8][chip8] emulator we must first
understand what it is. The Chip8 is a really simple platform that can run simple
games such as PONG and TETRIS.

It turns out that the Chip8 is not a proper gaming console, but rather it's a
specification of a virtual machine. This virtual machine could then be
implemented on all the systems that wanted to run Chip8 games.

A proper Chip8 virtual machine has:

- 4K of memory of which the first 512 bytes are reserved for the virtual machine
  itself. I stored the font data in this section.
- 16 8-bit registers of which the last one is used as a status register.
- a stack that is only used to store return addresses when subroutines are
  called.
- two timers that count down every 60Hz stopping at 0. A first one serves as a
  general delay timer, while the other one is for sound because the emulator
  will beep when the timer is not 0.
- an input keyboard with 16 keys.
- a 64x32 monochrome display.

I won't explain all the instructions because they are not particularly
interesting and there are already tons of resources available online.

The clock of virtual machine is not specified, but it seems that most games work
best with a clock speed of 500Hz.

## The good old times

After a couple of hours implementing the virtual machine I was able to play most
of the games flawlessly, but some games were flickering a lot. I initially
thought my emulator was slow, but it turned out to not be the case. I then
investigated whether my naive main game loop was working properly

```rust
loop {
    handle_input(sdl_events());

    for _ in freq / 60 {
        chip8.emulate_cycle();
    }
    chip8.decrease_timers();

    refresh_sound();
    refresh_screen();

    thread::sleep(Duration::new(0, 1_000_000_000u32 / 60))
}
```

The general idea is quite straightforward. The screen is meant to be refreshed
at 60FPS and in each frame we first process the user input, then we run some
instructions according to the clock speed, decrease the timers, update the audio
status and eventually redraw the screen. After staring a bit at the code I was
convinced that that was not the problem.

After more time I'd like to admit, I found the cause of the flickering. It has
to do with how the Chip8 draws sprites onto the screen. The Chip8 doesn't simply
overwrite what's on the screen with the new data, but it performs a bitwise xor
between the new pixels and the old ones! This allows clearing the screen by
simply redrawing the sprites that are currently visible. Unfortunately, this
also means that unnecessarily redrawing a sprite makes it flicker. That's why
some games flicker a lot in my emulator!

I think that Chip8 consoles that do not flicker use some kind of monochrome
display that is slow to turn pixels off. In fact, this phenomena, known as
[_ghosting_][ghosting] was a known behavior of old phosphor displays.

## WASM

At this point I had the core of the emulator in pure Rust with a simple frontend
in SDL2. Since Rust boats great support for WebAssembly I decided to also write
a frontend able to run in the browser. I also imposed myself to not write any
Javascript in the process (other than the generated one).

Before doing anything wasm related I separated the core library from the SDL
frontend in different crates so that the Wasm frontend crate would depend only
on the core crate. Code reuse for the win!

I then used `cargo generate` to create a new crate for the Wasm frontend from
the official [rust wasm template][rust-wasm-template]. This crate was already
configured to use `webpack` and [`wasm-pack`][wasm-pack] to automatically
compile and bundle my wasm binary. Neat!

Now I was able to start the development server with `npm start`, write the Rust
code I needed and have Webpack recompile and refresh everything automatically.
Just like some vanilla Javascript project!

Eventually, I was able to not write any Javascript by leveraging the awesome
[`web-sys`][web-sys] crate which wraps _all_ the Apis browsers offer in Rust.
However, I cannot say I would recommend being so drastic because working with
the web Apis is way easier in Javascript than it is in Rust. The only fact that
it can be done in Rust anyway is pretty cool though!

Kudos to the Rust Wasm working group for making this whole process that easy!

To convince yourself that this actually works, just go to
<https://d-dorazio.github.io/chip8> and play some games.


[chip8]: https://en.wikipedia.org/wiki/CHIP-8
[ghosting]: https://en.wikipedia.org/wiki/Monochrome_monitor#Screen_burn
[wasm-pack]: https://rustwasm.github.io/wasm-pack/
[rust-wasm-template]: https://github.com/rustwasm/wasm-pack-template
[web-sys]: https://rustwasm.github.io/wasm-bindgen/api/web_sys/
