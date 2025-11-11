---
title: 🍿 Slides
---

Hello everyone! ☺️

In this page, you can find the slides from my talk "Retro Meets Modern: Commodore 64 live coding with C# and .NET 9+" presented at .NET Conf 2025.

<div id="retroc64-carousel" class="carousel slide" data-bs-theme="light">
  <div class="carousel-indicators">
    {{ for i in 1..16 }}
    <button 
        type="button"
        data-bs-target="#retroc64-carousel"
        data-bs-slide-to="{{ i - 1}}"
        class="{{ i == 1 ? "active" : "" }}"
        aria-current="{{ i == 1 ? "true" : ""}}"
        aria-label="Slide {{ i }}">
    </button>
    {{ end; }}
  </div>
  <div class="carousel-inner">

<div class="carousel-item active">
<img src="/slides/retroc64_slide1.jpg" class="d-block w-100" alt="RetroC64 Slide 1">
<div class="d-none slide-caption-source">

Today we'll time-travel from the 1980s 8-bit world … straight into .NET 9.

<br></div></div>
<div class="carousel-item">
<img src="/slides/retroc64_slide2.jpg" class="d-block w-100" alt="RetroC64 Slide 2">
<div class="d-none slide-caption-source">

My name is Alexandre Mutel - VP of Engineering at DataGalaxy. In my spare time, I love contributing to open-source projects in the .NET ecosystem. Some of them became popular… But most are pretty niche, but I like them.
And above all, I'm a .NET performance enthusiast.

<br></div></div>
<div class="carousel-item">
<img src="/slides/retroc64_slide3.jpg" class="d-block w-100" alt="RetroC64 Slide 3">
<div class="d-none slide-caption-source">

So today, I'm going to talk about this project,
what made the 8-bit era so magical, and why the Commodore 64 in particular.
Then I'll introduce the .NET SDK I've built for it,
with, hopefully, an entertaining live demo.
And if time allows, I'll wrap up with a look at the architecture and what's coming next.

<br></div></div>
<div class="carousel-item">
<img src="/slides/retroc64_slide4.jpg" class="d-block w-100" alt="RetroC64 Slide 4">
<div class="d-none slide-caption-source">

The very first computer I ever touched was a Texas Instruments TI-99/4A.
I didn't own it, it was at my uncle's place.
Later, I was lucky enough to get my own Amstrad CPC 464,
and then... my beloved Amiga 500.

<br></div></div>
<div class="carousel-item">
<img src="/slides/retroc64_slide5.jpg" class="d-block w-100" alt="RetroC64 Slide 5">
<div class="d-none slide-caption-source">

One of the magical things about the 8-bit era
was that you could jump straight into coding
right there, on the machine itself.
You'd start with BASIC, maybe move on to assembly...
or, of course, you could just play games.
It was a pure moment of creativity - and I loved doing both.

<br></div></div>
<div class="carousel-item">
<img src="/slides/retroc64_slide6.jpg" class="d-block w-100" alt="RetroC64 Slide 6">
<div class="d-none slide-caption-source">

Back then, there were so many different 8-bit machines
each with its own charm and quirks.
So why did I choose the Commodore 64 for this experiment?

<br></div></div>
<div class="carousel-item">
<img src="/slides/retroc64_slide7.jpg" class="d-block w-100" alt="RetroC64 Slide 7">
<div class="d-none slide-caption-source">

And the reason is simple - the Commodore 64 is actually coming back!
Thanks to Peri Fractic, a well-known YouTuber in the retro-computing scene,
who bought the Commodore brand and brought together the community
to design a brand-new machine - fully compatible with the original,
but built with modern hardware.

So I ordered one - it's actually shipping this month!
And while waiting, I thought it was the perfect opportunity
to start learning about the Commodore 64
and see what I could already do with it.

<br></div></div>
<div class="carousel-item">
<img src="/slides/retroc64_slide8.jpg" class="d-block w-100" alt="RetroC64 Slide 8">
<div class="d-none slide-caption-source">

Let's take a quick look at what's inside the Commodore 64.
At its heart, there's the 6510 CPU - running at about one megahertz.
Then you have the VIC-II graphics chip, responsible for all the colors, sprites, and smooth scrolling.
The SID chip - the sound synthesizer - gave the C64 its legendary music.
Two CIA chips handled all the input and I/O.
And, of course, the system had a whole 64 kilobytes of RAM and 20 kilobytes of ROM.

<br></div></div>
<div class="carousel-item">
<img src="/slides/retroc64_slide9.jpg" class="d-block w-100" alt="RetroC64 Slide 9">
<div class="d-none slide-caption-source">

Now let's look at how memory is organized on the Commodore 64.
The address space is 64 kilobytes - but it's not all RAM.
By default, the system maps part of that space to ROM:
the BASIC interpreter, the I/O registers, and the KERNAL -
which acts as a kind of built-in operating system layer.

If we look down at the very bottom of RAM,
we find two important areas - the zero page and the stack.
The zero page is special - it's the first 256 bytes of memory, 
and many instructions can access it with shorter opcodes,
which makes them faster and smaller.

Then there's the stack, another page of memory,
which is interesting because it wraps around automatically when you push or pull data.

And finally, if we zoom in even more,
at addresses $00 and $01 -
we find two tiny but powerful I/O registers, which makes the 6510 CPU unique.

These ports control which parts of the memory map are active -
so by flipping a few bits here,
you can switch between showing the ROMs or accessing the underlying RAM instead.

<br></div></div>
<div class="carousel-item">
<img src="/slides/retroc64_slide10.jpg" class="d-block w-100" alt="RetroC64 Slide 10">
<div class="d-none slide-caption-source">

And programming the C64 requires typically

to write assembly code in one or more .asm files -
but that's rarely enough.

You often need to add some C or C++ code to precompute data -
for example, cosine tables for demo effects.

Then you have to glue everything together with a makefile -
assemble, link, and generate .prg or .d64 images.

And once you've built it,
you still need to manually load it into the VICE emulator
-or a real machine- before you can actually test it.

And I'm also eluding the challenges of debugging…

<br></div></div>
<div class="carousel-item">
<img src="/slides/retroc64_slide11.jpg" class="d-block w-100" alt="RetroC64 Slide 11">
<div class="d-none slide-caption-source">

That's why I have developed RetroC64, an SDK for modern Commodore 64 development straight from .NET

It comes with a zero-friction dev loop with auto-launch live coding into VICE Emulator.

I have developed a fluent 6510 assembler with Asm6502 - Labels, sections, data blocks, helpers, and source mapping back to C#.

BASIC integration - Ideal for quick demos and prototyping and Core helpers - C64Assembler with Zero-Page allocator, raster IRQ setup, Sprites classes…etc.

SID tooling - Loader, relocator, and player

Disk and program formats - D64 and PRG support.

First-class VS Code debugging: inspect registers, memory, VIC/SID registers, breakpoints, execution control (step-in- step-over…), view disassembly…

And, of course, supporting all OS thanks to .NET 9.0+
<br></div></div>
<div class="carousel-item">
<img src="/slides/retroc64_slide12.jpg" class="d-block w-100" alt="RetroC64 Slide 12">
<div class="d-none slide-caption-source">

And now let's experience live this project.

<br></div></div>
<div class="carousel-item">
<img src="/slides/retroc64_slide13.jpg" class="d-block w-100" alt="RetroC64 Slide 13">
<div class="d-none slide-caption-source">

The SDK is architecture around 4 main packages:

The Main package that contains:
- The App Layer that manages the life cycle of the build, dotnet watch and live sync with the emulator
- A debugger that connects to the VICE emulator through a remote interface

The Core package that contains all the core functionalities:
- A Basic compiler
- A C64 assembler with a zero-page allocator
- A fast-loader code
- Packer routines
- Graphics helpers - like the sprite class integrated with SkiaSharp
- Music helpers - that can load, relocate and play SID files
- Storage of D64/PRG files
- And various other things like C64 registers definitions

We have the Asm6502 project that powers:
- The assembler
- The disassembler used by the debugger
- And an emulator used by the SID relocator (it is a port of the famous sidreloc project made by Linus Åkesson)

And finally, Vice package that implements the custom socket binary protocol to communicate with the VICE emulator.

<br></div></div>
<div class="carousel-item">
<img src="/slides/retroc64_slide14.jpg" class="d-block w-100" alt="RetroC64 Slide 14">
<div class="d-none slide-caption-source">

I'd like to zoom in on the debugger - because I think it was a really interesting challenge, and I learned a lot from it.

In particular, I discovered the Debug Adapter Protocol - a protocol that lets IDEs talk to debuggers through a common interface, without needing to know how each one works internally.

So, when you create a RetroC64 application, there's actually a debug adapter inside it. I lied earlier - it's not a real debugger, it's just an adapter. Sorry!

When VS Code connects to the VICE emulator, it uses the DAP to communicate with that adapter inside RetroC64. The adapter then proxies those calls to the VICE monitor, and also enriches them with information only the RetroC64 app knows - like variable names, or source mappings.
To make all this work, you have to implement dozens of callback functions. It feels like a mountain of work at first - but it turned out to be totally manageable.
After all, in our case, we only have one thread and one frame in the call stack - so some things are actually much simpler.
And honestly, I found it really cool that adding full debugger support in C# to this project was ultimately that straightforward.

<br></div></div>
<div class="carousel-item">
<img src="/slides/retroc64_slide15.jpg" class="d-block w-100" alt="RetroC64 Slide 15">
<div class="d-none slide-caption-source">

First, the Fast Loader. I've already ported a lot of assembly code... but it's actually not finished yet. I lied again - sorry!

Next, the ZX0 depacker. That still needs to be written in assembly - but that part will be easy, since the packer itself was the tricky one.
The SDK also doesn't support multi-part demos yet - that's what the Fast Loader and ZX0 will enable.
And of course, it'll need proper multi-part debugging support too.

Documentation… I've done the bare minimum so far. 

Graphics - I haven't really played with bitmaps yet, and there's a lot more to explore there: using bitmaps, converting them from real images, and making that process easier.

And then… a bunch of tiny things all over the place.

So yes - there's still plenty of work ahead... and contributors are, of course, more than welcome!

<br></div></div>
<div class="carousel-item">
<img src="/slides/retroc64_slide16.jpg" class="d-block w-100" alt="RetroC64 Slide 16">
<div class="d-none slide-caption-source">

Thank you for watching,

Happy coding!

And don't forget to buy the new Commodore 64 Ultimate and play with it!

<br></div></div>

  <button class="carousel-control-prev" type="button" data-bs-target="#retroc64-carousel" data-bs-slide="prev">
    <span class="carousel-control-prev-icon" aria-hidden="true"></span>
    <span class="visually-hidden">Previous</span>
  </button>
  <button class="carousel-control-next" type="button" data-bs-target="#retroc64-carousel" data-bs-slide="next">
    <span class="carousel-control-next-icon" aria-hidden="true"></span>
    <span class="visually-hidden">Next</span>
  </button>

</div>

</div>

<!-- External caption area -->
<div id="retroc64-caption"
     class="text-start text-body bg-body px-3 py-3 px-md-4 border-top"
     aria-live="polite"
     style="min-height: 5rem;">
</div>

<script>
  document.addEventListener('DOMContentLoaded', function () {
    const carouselEl = document.getElementById('retroc64-carousel');
    const captionEl = document.getElementById('retroc64-caption');

    function updateCaption() {
      const active = carouselEl.querySelector('.carousel-item.active');
      const src = active && active.querySelector('.slide-caption-source');
      captionEl.innerHTML = src ? src.innerHTML : '';
    }

    updateCaption();
    carouselEl.addEventListener('slid.bs.carousel', updateCaption);
  });
</script>