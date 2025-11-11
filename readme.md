---
layout: simple
title: Home
title_html: Welcome to RetroC64!
og_type: website
---

<section class="text-center py-5 text-white hero-text">
  <div class="container">
    <h1 class="fw-bold display-6">
       Programming the <span class="c64-text">Commodore</span> <span class="c64-number">64</span><br>
      with <span class="text-primary">.NET</span>
    </h1>
    <p class="lead mt-3 mb-4">
      Build, assemble, run and debug C64 programs <br>without leaving your IDE...<br>
      Retro development, reimagined through modern tooling!
    </p>
    <div class="d-flex justify-content-center gap-3 mt-4">
      <a href="/docs" class="btn btn-primary btn-lg">Get Started ></a>
      <a href="https://github.com/RetroC64/RetroC64-Examples" class="btn btn-light btn-lg">Examples ></a>
    </div>
  </div>
</section>

<section class="container my-5">
  <div class="c64-video-wrap">
    <div class="ratio ratio-16x9">
      <video class="c64-video" autoplay loop muted playsinline preload="metadata" aria-label="C64 demo video">
        <source src="/img/c64-demo.webm" type="video/webm">
        Your browser does not support WebM video.
      </video>
    </div>
  </div>
</section>

<section class="container my-5">
  <div class="row row-cols-1 row-cols-md-2 gx-5 gy-5">
    <div class="col">
        <div class="card">
        <div class="card-header display-6">
            🚀 Zero Friction Dev Loop
        </div>
        <div class="card-body">
            <p class="card-text">Emit PRG/D64 and auto-launch live coding into VICE straight from .NET.</p>
            <a href="/docs">Introduction ></a>
        </div>
        </div>
    </div>
    <div class="col">
        <div class="card">
        <div class="card-header display-6">
            🖥️ Fluent Assembler
        </div>
        <div class="card-body">
            <p class="card-text">With <a href="https://github.com/xoofx/Asm6502">Asm6502</a> Labels, sections, data blocks, helpers, and source mapping back to C#.</p>
            <a href="/docs/assembler">Assembler ></a>
        </div>
        </div>    </div>
    <div class="col">
        <div class="card">
        <div class="card-header display-6">
            🎨 Sprite Tooling
        </div>
        <div class="card-body">
            <p class="card-text">Draw in Skia and convert to C64 sprite bytes automatically.</p>
            <a href="/docs/graphics">Sprites ></a>
        </div>
        </div>    </div>
    <div class="col">
        <div class="card">
        <div class="card-header display-6">
            🎵 SID Tooling
        </div>
        <div class="card-body">
            <p class="card-text">Loader, relocator, and player (target address and ZP ranges).</p>
            <a href="/docs/music">Music ></a>
        </div>
        </div>
    </div>
    <div class="col">
        <div class="card">
        <div class="card-header display-6">
            🧪 Core Helpers
        </div>
        <div class="card-body">
            <p class="card-text">Ideal for quick demos and prototyping and Core helpers - C64Assembler with Zero-Page allocator, raster IRQ setup, Sprites classes, PRG and D64 images files...</p>
            <a href="/docs/helpers">Helpers ></a>
        </div>
        </div>    </div>
    <div class="col">
        <div class="card">
        <div class="card-header display-6">
            🐞 Debugging
        </div>
        <div class="card-body">
            <p class="card-text">With VS Code, inspect registers, memory, VIC/SID registers, breakpoints, execution control (step-in- step-over…), view disassembly…
            </p>
            <a href="/docs/debugging">Debugging ></a>
        </div>
        </div>
    </div>    
  </div>
</section>