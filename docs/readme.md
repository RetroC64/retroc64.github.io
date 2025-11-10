---
title: Documentation
---

## Prerequisites

In order to run RetroC64, you need the following prerequisites installed on your system:

- The [RetroC64 VSCode Debugging Extension](https://marketplace.visualstudio.com/items?itemName=xoofx.retro-c64) to enable debugging support in Visual Studio Code.
- The [.NET SDK 9.0+](https://dotnet.microsoft.com/download/dotnet/9.0) or later installed
- The [C64 VICE emulator](https://vice-emu.sourceforge.io/) (**version 3.7** or later) to run C64 programs and live programming and debugging.
- OS: `Windows`/`macOS`/`Linux`

{{TIP do}}

When installing VICE, you should ensure that:

- `x64sc` is in your `PATH` environment variable so that RetroC64 can find it.
- Or set `RETROC64_VICE_BIN` environment variable to the x64sc binary
  - Windows (x64sc.exe):
    - Example: `$env:RETROC64_VICE_BIN=C:\Program Files\c64\GTK3VICE-3.9-win64\bin\x64sc.exe`
  - macOS / Linux:
    - Example: `RETROC64_VICE_BIN=/usr/bin/x64sc`
{{end}}

## Your First RetroC64 Assembly Program

Create a new console application:

```shell-session
$ dotnet new console -n HelloAsm
$ cd HelloAsm
```

Add the required NuGet packages:

```shell-session
$ dotnet add package RetroC64
```

Replace the content of `Program.cs` with the following code:

```csharp
using Asm6502;
using RetroC64;
using RetroC64.App;
using static RetroC64.C64Registers;

// A program is a command line app that builds and runs a 6510 assembly program.
return await C64AppBuilder.Run<HelloAsm>(args);

/// <summary>
/// A simple assembler program that changes the background and border colors.
/// </summary>
public class HelloAsm : C64AppAsmProgram
{
    protected override Mos6502Label Build(C64AppBuildContext context, C64Assembler asm)
    {
        asm.Label(out var start)
            .BeginCodeSection("Main")
            .LDA_Imm(COLOR_RED)
            .STA(VIC2_BG_COLOR0)
            .LDA_Imm(COLOR_GREEN)
            .STA(VIC2_BORDER_COLOR)
            .InfiniteLoop()
            .EndCodeSection();
        return start;
    }
}
```

Launch dotnet watch to build and run the program with live coding support:

```shell-session
$ dotnet watch -- run
```

It will output build information and launch the VICE C64 emulator:

![HelloAsm Build Output](/img/RetroC64-HelloAsm-output.png)

And it will display the following screen in the emulator:

![HelloAsm Example](/img/RetroC64-HelloAsm.png)

{{NOTE do}}
Thanks to `dotnet watch`, you can modify the assembly code in `Program.cs`, save the file, and see the changes reflected immediately in the running VICE emulator.
{{end}}

## Generated Files

When you run the RetroC64 application above, it generates the following files in the `bin/Debug/net9.0` (or corresponding) directory:

- `.retroC64/build/helloasm.prg`: The assembled C64 program in PRG format.
- `.retroC64/cache/` directory: Contains cached files for faster builds (e.g. SID relocated files).

The `.retroC64` directory is created automatically by the RetroC64 SDK.

The `helloasm.prg` file can be loaded and run directly in the VICE C64 emulator or can be copied to a physical C64 disk image for use on real hardware.

In the [core helpers](helpers.md) documentation, you can find more information about the available APIs to build D64 image files.

## More Examples

You can find more example projects in the [RetroC64 Examples GitHub repository](https://github.com/RetroC64/RetroC64-Examples).

## Next Steps

Now that you have created your first RetroC64 assembly program, you can explore more advanced features of the RetroC64 SDK, such as:

- Using the C# Asm6502 [assembler](assembler.md) to create more complex assembly programs.
- Use [graphics and sprites](graphics.md) to create visual effects.
- Play SID [music](music.md) and sound effects.
- Use core [helpers](helpers.md) to e.g. create D64 disk images.
- [Debugging](debugging.md) with VS Code


