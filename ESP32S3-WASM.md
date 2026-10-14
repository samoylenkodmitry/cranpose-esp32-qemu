# ESP32-S3 in the browser

This fork runs Espressif's ESP32-S3 emulator in a web page. It is
[ktock/qemu-wasm](https://github.com/ktock/qemu-wasm) (QEMU with a TCG backend
that emits WebAssembly), branch `wasm64-tcg-b-demo-2`, with the ESP32 machines
of [espressif/qemu](https://github.com/espressif/qemu) tag
`esp-develop-9.2.2-20260417` on top. It serves
[Cranpose on an emulated ESP32-S3](https://github.com/samoylenkodmitry/cranpose-esp32).

Changes beyond the Espressif machines, branch `esp32s3-wasm`:

- QEMU 10 API: `system/` headers, `const` class data and properties, the
  watchpoint header.
- The ESP32-S3 board builds without libgcrypt: its AES, RSA, digital
  signature and XTS-AES engines are left out and the cache runs without
  XTS-AES. `configs/devices/xtensa-softmmu/esp32s3.mak` builds that board only.
- The board selects `I2C` and `CAN_SJA1000`, which its devices need when no
  other board brings them.
- `esp_sha` calls the compress functions through wrappers of the table's type:
  WebAssembly traps on a call through a cast that drops their `int` result.
- `esp_rgb` copies an area when the guest sets the update flag, so no display
  refresh is needed, and in browser builds hands each frame to
  `Module.onEspRgbFrame(pixels, width, height, bpp)`.
- The cache copies mapped flash pages from the flash model
  (`m25p80_get_storage`) instead of reading the drive on the vCPU thread.
- `meson.build` leaves slirp not found when libslirp is missing.
- The Emscripten container downloads zlib from its GitHub release.

`.github/workflows/wasm.yml` builds the browser emulator for a `v*` tag and
attaches `qemu-system-xtensa.js`, `qemu-system-xtensa.wasm` and the ESP32-S3
ROM to the release. Run it with the guest's serial port on a file chardev
(`-chardev file,id=uart0,path=/dev/stdout -serial chardev:uart0`): a stdio
serial port makes the main loop wait for input while it holds the iothread
lock.
