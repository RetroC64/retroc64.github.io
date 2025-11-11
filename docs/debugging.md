---
title: 🐞 Debugging
---

RetroC64 comes with an integrated debugger implemented entirely in the RetroC64 .NET runtime by using the [Debug Adapter Protocol](https://microsoft.github.io/debug-adapter-protocol/) (DAP). 

You can install the [RetroC64 VSCode extension](https://marketplace.visualstudio.com/items?itemName=xoofx.retro-c64). This extension provides only the necessary VSCode integration to connect to the RetroC64 debugger easily from within VSCode.

## Features

- **Attach to debugger**: Connect to RetroC64 debugger running on port 6503 (default)
- **Register access**: Full read-write access to CPU registers, CPU flags, stack, and zero page addresses
- **Hardware registers**: Access to VIC and SID registers
- **Breakpoints**: Set code breakpoints and data breakpoints (watchpoints)
- **Execution control**: Step-in, step-over, step-out, pause, and continue
- **Memory inspection**: View RAM contents
- **Code analysis**: View disassembly

![RetroC64 Debugger Screenshot](/img/RetroC64_Debugger.png)

## Usage

Create the following `.vscode/launch.json` file in your project and press F5 to start debugging:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "RetroC64 Attach",
      "type": "RetroC64",
      "request": "attach",
      "debugServer": 6503
    }
  ]
}
```

<img align="right" width="324px" src="/img/RetroC64_Debugger_Variables.png" alt="RetroC64 Debugger Attach" />


Only the attach request is supported since the RetroC64 debugger is started automatically when you run your C64 program during a RetroC64 run/live session (via `dotnet watch -- run`).

## Variables

The Variables tab in VSCode shows the following variable groups:

- CPU Registers: A, X, Y, PC, SP, and CPU flags (N, V, B, D, I, Z, C)
- Stack: View the contents of the stack (from $0100 to $01FF)
- Zero Page: View the contents of the zero page (from $00 to $FF)
  - The name of the variables allocated via the `C64Assembler.ZpAlloc` method are visible in the Variables pane of the debugger.
- VIC and Sprite Registers: View the contents of the VIC-II registers (from $D000 to $D02E)
- SID Registers: View the contents of the SID registers (from $D400 to $D41C)

## Breakpoints

You can set code breakpoints by clicking in the gutter of the source code editor in VSCode.

![RetroC64 Debugger Breakpoints](/img/RetroC64_Debugger_BreakpointCode.png)


## Disassembly View

By clicking on the CALL STACK frame entry, you can open the Disassembly view that shows the disassembled code around the current program counter (PC):

![RetroC64 Debugger Disassembly View](/img/RetroC64_Debugger_Disassembly.png)

## Memory View

By clicking on the `PC` register, you can open the Memory view that shows the entire memory of the C64 (64 KiB):

![RetroC64 Debugger Memory View](/img/RetroC64_Debugger_Memory.png)

## More Information

In order for the debugger to allow breakpoint on Asm6502 instructions, the Asm6502 assembler stores debug information when the C# code is compiled.

{{TIP do}}
If you are using a sub function to build your assembly code, by default, only the instructions within this function will have debug information, not the method itself.
{{end}}
