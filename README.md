# FAP EMU

![FAP EMU](./logo.svg)

**A browser-based emulator for Flipper Zero `.fap` applications.** Drop in a `.fap` file, run it against an emulated ARM core, and see its display, inputs, and SDK calls — no Flipper hardware required.

🔗 **Live demo:** [joelewis012.github.io/flipper-EMU](https://joelewis012.github.io/flipper-EMU)

---

## What it does

FAP EMU loads a compiled Flipper Zero application (`.fap`, an ELF binary) directly in your browser and emulates it using [Unicorn.js](https://github.com/AlexAltea/unicorn.js), a WebAssembly port of the Unicorn CPU emulator. It parses the ELF structure, maps the binary into emulated memory, resolves calls into the Flipper SDK, and intercepts those calls with JavaScript stubs that drive a virtual 128×64 monochrome display and button input queue.

This makes it possible to:

- Quickly sanity-check a `.fap` build without flashing it to a device
- Step through SDK call activity (GUI, ViewPort, Furi primitives, IR, etc.) via a live log
- Inspect a binary's imports, sections, and relocation data before running it

## How it works

- **ELF parsing** — reads section headers, program headers, symbol tables, and both standard (`.rel.text`) and Flipper's custom hash-based (`.fast.rel.text`) relocation sections.
- **Unicorn.js core** — emulates an ARM processor in THUMB mode entirely in WebAssembly.
- **SDK stubs** — common Flipper SDK functions (canvas drawing, ViewPort, GUI, Furi records, message queues, timers, memory allocation, ARM EABI float helpers, IR) are reimplemented in JavaScript and wired in via call-site hooks.
- **Virtual hardware** — a 128×64 canvas represents the device LCD, with on-screen D-pad controls mapped to Furi input events.

## Status

This is an active work-in-progress hobby project. Many SDK functions are stubbed as no-ops, hardware-dependent features (SubGHz, NFC, BT, IR transmission) are simulated rather than real, and some binaries — particularly those relying on deep RTOS behaviour — may not run to completion. The LOG tab shows detailed diagnostics (`[REL]`, `[FAST]`, `[EMU]`, `[FURI]`, `[GUI]`, `[DRAW]`) for debugging unsupported calls.

## Usage

1. Open the [live demo](https://joelewis012.github.io/flipper-EMU) (or serve `index.html` locally — it needs `unicorn.js` in the same directory).
2. Drop a `.fap` file onto the page, or use the file picker.
3. Check the **IMPORTS** tab to see which SDK calls are stubbed, partial, or unknown.
4. Hit **RUN**. Use the on-screen D-pad (or arrow keys / Enter / Esc) to send input.
5. Switch to the **LOG** tab for emulation diagnostics.

## Running locally

```bash
git clone https://github.com/joelewis012/flipper-EMU.git
cd flipper-EMU
# download unicorn-arm.min.js from https://github.com/AlexAltea/unicorn.js/releases
# rename it to unicorn.js and place it alongside index.html
python3 -m http.server
```

## Safety

Everything runs sandboxed inside Unicorn.js in your browser. There is no real hardware access — SubGHz, NFC, BT, and IR calls are no-ops. User-provided `.fap` binaries are not verified or endorsed; you are responsible for what you load.

## Disclaimer

FAP EMU is an independent, unofficial project and is **not affiliated with, endorsed by, or supported by Flipper Devices Inc.** "Flipper Zero" and related names/trademarks belong to their respective owners.

## License

This project is licensed under the [MIT License](./LICENSE).

It depends on [Unicorn.js](https://github.com/AlexAltea/unicorn.js) (GPLv2), which must be obtained separately and is not bundled in this repository.

## Contributing

Issues and pull requests are welcome — particularly additional SDK stub implementations, relocation format fixes, and compatibility reports for specific `.fap` builds.
