**This repo is no longer maintained,** and I'm unlikely to come back to it in the future.

-------

# ghidra-65816

This is a WDC 65816 processor module for Ghidra.

This is an early release. I've not used it much yet, so expect bugs. Feedback and pull requests are welcome!

Disassembly is probably correct. Data flow analysis is probably *mostly* correct. Decompilation is probably unusable.

## Install and Usage
Rename the root folder to `65816` and copy it to `Ghidra/Processors`.

I've only tried to use this processor module with a raw binary so far.

 1. Import a file using the "Raw Binary" format, "65816" language.
 2. Open it in the code browser. When asked if you want to analyse it now choose "no".
 3. Set up your memory map.
 4. For each entry point:
    1. Identify the mode that the processor will be running in.
    2. Right-click the first instruction byte and choose "Processor Options..." to set the `MF`, `XF`, and `EF` flags.
    3. Right-click the first instruction byte and choose "Set Register Values..." to set the `DBR` (data bank), `DF` (decimal flag), `DP` (direct page), `PBR` (program bank), and `SP` (stack pointer) registers. Perhaps others, if you're feeling keen.
    4. Cross fingers and disassemble. Watch out for limitations and unknowns.

## Limitations
 - The 65802 variant is not supported.
 - Disassembly and data flow analysis will be incorrect when an instruction and its operand(s) wrap at the end of a program bank.
 - When pulling the `MF` and `XF` flags from the stack Ghidra will not automatically set the correct processor mode. This affects the `PLP` and `RTI` instructions.
 - Interrupt vectors are not automatically recognised as entry points.

## Unknowns
I don't know how well Ghidra will pick up on changes to the processor modes. If it requires a lot of manual intervention then a custom analyser might be necessary to make it usable.

The following implemented behaviours are my interpretation of ambiguous documentation, which is likely to be wrong.
- When executing a `BRK` the processor model pushes the status register to the stack with the `BF` flag set, but leaves the flag unchanged when jumping to the handler.
- When executing a `XCE` where the emulation mode is re-switched to the current mode, the model behaves as if the new mode was entered "properly". That is:
  - `emulation -> emulation` is treated like `native -> emulation`, i.e. index registers are truncated and the stack is forced to page one.
  - `native -> native` is treated like `emulation -> native`, i.e. index registers are truncated.
