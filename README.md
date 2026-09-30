# Zevox 8.4 AMP — Linux & Web DSP Editor

Native Linux and web-based DSP control software for the **Zevox ZP 8.4 AMP** car amplifier.

Official product page:  
https://voxresearch.id/katalog/digital-signal-processors/zp-8-4-amp/

<p align="center">
  <img src="https://voxresearch.id/wp-content/uploads/2021/04/ZP-8.4-AMP-3-web.jpg"
       alt="Zevox ZP 8.4 AMP"
       width="400">
</p>

This project provides a native Linux DSP editor without requiring the original vendor application, Electron, or heavy runtime dependencies.

The USB protocol was reverse engineered from the vendor Windows software, and the parameter map has been verified against real **Zevox ZP 8.4 AMP** hardware.

<p align="center">
  <img src="docs/screenshots/main-ui.png"
       alt="ZP 8.4 AMP DSP Editor"
       width="100%">
</p>

| Item | Information |
|---|---|
| Device | `Bus 003 Device 005: ID 4084:4357 Nuvoton HID Transfer` |
| Linux device | `/dev/hidraw*` |
| udev symlink | `/dev/zp84amp` |
| Verified on | Linux x86_64 — 29 September 2026 |
| Language | C11 |
| Codebase | ~15k lines |
| Desktop GUI | Xlib |
| USB interface | Linux hidraw |
| Web UI | Built-in HTTP server |
| License | MIT |

---

## Features

### Native Linux DSP Editor

The application provides a console-style interface inspired by the original Zevox DSP software.

Implemented controls include:

- Input mixer
- Channel mixer
- High-pass filter
- Low-pass filter
- Crossover configuration
- Live EQ response curve
- 31-band EQ table
- 31-band EQ faders
- EQ frequency
- EQ gain
- EQ Q factor
- Channel selector
- Channel overlay
- Car speaker diagram
- Master volume
- Per-channel output level
- Phase control
- Delay control
- Delay unit selection
- Eight output channel strips

---

## Direct USB Communication

The application communicates directly with the amplifier through Linux `hidraw`.

Verified parameters include:

- HPF frequency
- LPF frequency
- HPF filter code
- LPF filter code
- EQ frequency
- EQ gain
- EQ Q
- Channel level
- Phase
- Delay

The program can both **read DSP state from the amplifier** and **write DSP parameters back to the device**.

---

## Browser DSP Editor

A browser-based interface is also included.

Start the web server:

```sh
zp84web
```

Then open:

```text
http://127.0.0.1:8085
```

The web server is implemented as a small native C binary.

No Node.js, Electron, PHP, Python, or external web runtime is required.

---

## Simulation Mode

The DSP interface can run without the amplifier connected.

This is useful for:

- UI development
- Testing controls
- DSP layout development
- Demonstrations
- Debugging parameter handling

---

## Snapshot and Diff Tool

The `zpsniff` utility can scan the DSP parameter address space.

Currently the tool can read all:

```text
1,934 parameter IDs
```

A complete DSP state can be saved as a plain-text snapshot.

Example:

```sh
zpsniff snap -o baseline.txt
```

Change one parameter using the original vendor software and create another snapshot:

```sh
zpsniff snap -o changed.txt
```

Then compare them:

```sh
diff -u baseline.txt changed.txt
```

This method was used extensively during reverse engineering to identify the parameter map.

---

# Requirements

## Runtime

- Linux
- Kernel with `hidraw` support
- Zevox ZP 8.4 AMP connected over USB

## Build tools

- GCC or Clang
- GNU Make
- C11 compiler

For the native GUI:

```text
libx11-dev
```

or the equivalent X11 development package for your Linux distribution.

The following tools do **not** require X11:

```text
zp84web
zpsniff
```

---

# Build

Clone the repository:

```sh
git clone <repository-url>
cd <repository-directory>
```

Build the native GUI and USB tools:

```sh
make
```

Build only the web version:

```sh
make web
```

Build and run the tests:

```sh
make test
```

Compiled binaries are written to:

```text
build/
```

---

# Prebuilt Linux Binaries

Prebuilt stripped binaries for Linux x86_64 are available in:

```text
release/linux-x86_64/
```

The release directory also contains:

```text
release/SHA256SUMS
```

Verify downloaded binaries with:

```sh
sha256sum -c release/SHA256SUMS
```

---

# Installation

Run:

```sh
./scripts/install-app.sh
```

The installer:

- builds the application
- installs binaries into `~/.local/bin`
- installs the desktop launcher
- installs the web UI assets

Make sure `~/.local/bin` is available in your `PATH`.

Example:

```sh
export PATH="$HOME/.local/bin:$PATH"
```

---

# USB Permissions

The Zevox amplifier appears as a vendor HID device.

Example:

```text
Bus 003 Device 005: ID 4084:4357 Nuvoton HID Transfer
```

Linux exposes the interface through:

```text
/dev/hidraw*
```

A udev rule is included to create:

```text
/dev/zp84amp
```

Install the rule once:

```sh
sudo install -m 0644 scripts/99-zp84.rules /etc/udev/rules.d/
```

Reload udev:

```sh
sudo udevadm control --reload-rules
```

Trigger the new rule:

```sh
sudo udevadm trigger --subsystem-match=hidraw
```

Disconnect and reconnect the amplifier if necessary.

Verify:

```sh
ls -l /dev/zp84amp
```

Until the udev rule is installed correctly, the utilities may need to be run with `sudo`.

---

# Running

## Native GUI

Start the DSP editor:

```sh
zp84gui
```

Open the application and immediately read the amplifier:

```sh
zp84gui --read-dsp
```

Run with UI scaling:

```sh
zp84gui --read-dsp --scale 1.5
```

Supported scale range:

```text
0.75 – 2.5
```

The application window is resizable.

All panels are recalculated dynamically so the 31 EQ bands remain accessible and are not clipped when the window size changes.

---

## Browser UI

Start:

```sh
zp84web
```

Open:

```text
http://127.0.0.1:8085
```

---

## USB Test

Check whether the amplifier responds:

```sh
zpsniff ping
```

---

## DSP Snapshot

Create a parameter dump:

```sh
zpsniff snap -o baseline.txt
```

---

# USB Protocol

The amplifier uses a vendor HID interface with:

```text
64-byte interrupt endpoints
```

Linux `hidraw` expects exactly:

```text
64 bytes
```

for each HID transaction.

The Windows HID API uses an additional leading report-ID byte:

```text
0x00
```

even though the device itself does not use a HID report ID.

Protocol framing currently includes structures based around:

```text
AE 1E <len> <tag>
```

and:

```text
80 <len> <command> ... CRC16
```

See:

```text
docs/PROTOCOL.md
```

for the current protocol documentation.

---

# USB Is Configuration Only

The USB interface is used for DSP control and configuration.

It is **not an audio interface**.

USB throughput is limited to approximately:

```text
64 KB/s
```

which is insufficient for normal multichannel PCM audio streaming.

Therefore:

```text
PC / Player
     │
     │ analog audio
     ▼
    RCA
     │
     ▼
Zevox ZP 8.4 AMP
```

while DSP configuration follows:

```text
Linux PC
   │
   │ USB HID
   ▼
Zevox ZP 8.4 AMP
```

In other words:

**RCA carries audio. USB carries DSP configuration.**

---

# Project Layout

```text
src/
├── hid/
│   ├── hidraw transport
│   ├── USB framing
│   ├── command handling
│   └── CRC16
│
├── dsp/
│   ├── biquad design
│   ├── filter families
│   ├── fixed-point conversion
│   └── AutoEQ solver
│
├── gui/
│   ├── Xlib console
│   ├── layout
│   ├── controls
│   ├── presets
│   └── embedded assets
│
└── web/
    ├── HTTP server
    └── embedded browser UI

tests/
├── frame tests
├── DSP math tests
└── console data tests

tools/
├── zpsniff
├── zpdecode.py
└── diffdump.py

docs/
├── PROTOCOL.md
├── CONSOLE-MAPPING.md
├── USB-VERIFICATION.md
├── DESKTOP-UI.md
└── screenshots/

web/
└── browser UI assets

examples/
└── Default Project/

release/
├── linux-x86_64/
└── SHA256SUMS
```

---

# DSP Architecture

The application separates transport, DSP processing, and user interface layers.

```text
                    ┌──────────────────┐
                    │     zp84gui      │
                    │   Xlib Desktop   │
                    └────────┬─────────┘
                             │
                             │
                    ┌────────▼─────────┐
                    │    DSP Layer     │
                    │                 │
                    │ EQ / XOVER      │
                    │ Delay / Phase   │
                    │ Level / Mixer   │
                    └────────┬─────────┘
                             │
                             │
                    ┌────────▼─────────┐
                    │ Protocol Layer   │
                    │ Frame / CRC16    │
                    │ Parameter IDs    │
                    └────────┬─────────┘
                             │
                             │
                    ┌────────▼─────────┐
                    │ Linux hidraw     │
                    └────────┬─────────┘
                             │ USB
                             ▼
                    ┌──────────────────┐
                    │ Zevox ZP 8.4 AMP│
                    └──────────────────┘
```

The web interface uses the same DSP and protocol logic:

```text
Browser
   │
   │ HTTP
   ▼
zp84web
   │
   ▼
DSP / Protocol
   │
   ▼
hidraw
   │
   ▼
ZP 8.4 AMP
```

---

# Documentation

## USB Protocol

```text
docs/PROTOCOL.md
```

Contains:

- HID frame format
- commands
- packet layout
- CRC information
- known parameter operations
- confirmed commands
- unknown commands
- reverse-engineering status

---

## Console Mapping

```text
docs/CONSOLE-MAPPING.md
```

Contains the mapping between DSP parameter IDs and user-interface controls.

Examples include:

- channel gain
- HPF
- LPF
- EQ
- phase
- delay

---

## USB Verification

```text
docs/USB-VERIFICATION.md
```

Documents what was actually observed and tested against physical hardware.

This is used to distinguish:

- confirmed behavior
- probable behavior
- assumptions
- currently unknown fields

---

## Desktop UI

```text
docs/DESKTOP-UI.md
```

Documents:

- panel behavior
- input handling
- EQ interaction
- channel selection
- resize behavior
- scaling
- DSP synchronization

---

# Testing

Run:

```sh
make test
```

The current test suite contains:

```text
46 checks
```

covering:

- packet framing
- frame parsing
- CRC handling
- DSP filter mathematics
- fixed-point conversion
- console data tables
- vendor DSP constants

Tests use constants recovered from the original vendor software and verified DSP behavior.

---

# Reverse Engineering Method

The DSP protocol was reconstructed using a combination of:

1. USB traffic observation
2. Vendor application behavior
3. Packet comparison
4. DSP parameter snapshots
5. Single-control parameter changes
6. Snapshot diffing
7. Hardware verification
8. Repeated read/write tests

A typical mapping procedure is:

```text
Read full DSP state
        │
        ▼
Save baseline
        │
        ▼
Change exactly one control
        │
        ▼
Read full DSP state again
        │
        ▼
Diff snapshots
        │
        ▼
Identify changed parameter ID
        │
        ▼
Repeat at multiple values
        │
        ▼
Determine encoding/scaling
        │
        ▼
Verify by writing from Linux
```

This avoids relying solely on assumptions derived from the Windows application.

---

# Status

The application can communicate with real Zevox ZP 8.4 AMP hardware and the primary DSP controls exposed by the vendor PC software have been mapped.

Confirmed areas currently include:

- USB communication
- DSP parameter reads
- DSP parameter writes
- EQ controls
- crossover controls
- output levels
- phase
- delay

Reverse engineering of the full parameter-ID space is still ongoing.

The complete scan contains:

```text
1,934 parameter IDs
```

Some IDs are known and mapped while others remain unnamed.

See:

```text
docs/PROTOCOL.md
```

for the current status table.

---

# Known Limitations

- USB audio streaming is not supported by the hardware interface.
- Some parameter IDs remain unidentified.
- Development and hardware verification have primarily been performed on Linux x86_64.
- The native desktop GUI currently depends on X11/Xlib.
- Wayland currently requires XWayland for the Xlib GUI.
- Hardware behavior should always be verified before assigning undocumented parameter IDs.

---

# Safety

Writing incorrect DSP parameters may result in:

- excessive output level
- incorrect crossover settings
- excessive low-frequency energy sent to tweeters
- clipping
- speaker damage

When testing newly discovered parameters:

1. Reduce master volume.
2. Disable or disconnect sensitive speakers where appropriate.
3. Change one parameter at a time.
4. Record a known-good DSP snapshot.
5. Verify read-back values after every experimental write.

---

# Bahasa Indonesia

## Zevox ZP 8.4 AMP — Editor DSP Linux & Web

Aplikasi native Linux dan web untuk mengontrol DSP amplifier mobil **Zevox ZP 8.4 AMP**.

Aplikasi ini tidak membutuhkan software vendor, Electron, Node.js, atau runtime besar lainnya.

Komunikasi dilakukan langsung melalui USB HID Linux menggunakan:

```text
/dev/hidraw*
```

atau symlink:

```text
/dev/zp84amp
```

Protokol USB direkonstruksi dari software PC vendor dan pemetaan parameter diverifikasi menggunakan perangkat Zevox ZP 8.4 AMP asli.

---

## Kemampuan

Fitur yang tersedia meliputi:

- input mixer
- mixer channel
- crossover
- HPF
- LPF
- grafik EQ langsung
- EQ 31-band
- fader EQ
- frekuensi EQ
- gain EQ
- Q EQ
- pemilih channel
- overlay channel
- diagram speaker mobil
- master volume
- level tiap output
- phase
- delay
- unit delay
- delapan output channel

Aplikasi dapat:

```text
Linux → membaca DSP → Zevox
```

dan:

```text
Linux → menulis parameter → Zevox
```

---

## Web UI

Selain GUI native Xlib tersedia juga UI berbasis browser.

Jalankan:

```sh
zp84web
```

Kemudian buka:

```text
http://127.0.0.1:8085
```

Server dibuat langsung menggunakan C.

Tidak membutuhkan:

- Node.js
- Electron
- Python
- PHP

---

## Mode Simulasi

UI dapat dijalankan tanpa amplifier terhubung.

Mode ini berguna untuk:

- pengembangan UI
- pengujian layout
- debugging
- demonstrasi
- pengembangan DSP

---

## Snapshot DSP

`zpsniff` dapat membaca seluruh ruang parameter DSP sebanyak:

```text
1.934 parameter ID
```

Contoh:

```sh
zpsniff snap -o baseline.txt
```

Setelah satu parameter diubah:

```sh
zpsniff snap -o changed.txt
```

Bandingkan:

```sh
diff -u baseline.txt changed.txt
```

Teknik ini digunakan untuk menemukan hubungan antara parameter ID dan kontrol pada software vendor.

---

## Build

```sh
make
```

Membangun:

```text
zpsniff
zp84gui
```

Untuk versi web:

```sh
make web
```

Untuk menjalankan pengujian:

```sh
make test
```

Hasil build tersedia pada:

```text
build/
```

---

## Instalasi

```sh
./scripts/install-app.sh
```

Installer akan:

- melakukan build
- menyalin binary ke `~/.local/bin`
- memasang desktop launcher
- memasang aset web

---

## Izin USB

Instal aturan udev:

```sh
sudo install -m 0644 scripts/99-zp84.rules /etc/udev/rules.d/
```

Reload:

```sh
sudo udevadm control --reload-rules
```

Trigger:

```sh
sudo udevadm trigger --subsystem-match=hidraw
```

Kemudian cek:

```sh
ls -l /dev/zp84amp
```

---

## Menjalankan

Buka GUI:

```sh
zp84gui
```

Buka sekaligus membaca DSP:

```sh
zp84gui --read-dsp
```

Dengan scaling UI:

```sh
zp84gui --read-dsp --scale 1.5
```

Web UI:

```sh
zp84web
```

Cek komunikasi amplifier:

```sh
zpsniff ping
```

Snapshot DSP:

```sh
zpsniff snap -o baseline.txt
```

---

## Audio Tidak Melewati USB

USB pada Zevox ZP 8.4 AMP digunakan untuk konfigurasi DSP.

USB bukan interface audio.

Throughput USB sekitar:

```text
64 KB/s
```

sehingga tidak cukup untuk streaming audio PCM multichannel.

Jalur audio:

```text
PC / Head Unit
      │
      ▼
     RCA
      │
      ▼
Zevox ZP 8.4 AMP
      │
      ▼
   Speaker
```

Sedangkan jalur kontrol:

```text
Linux PC
   │
   ▼
USB HID
   │
   ▼
Zevox ZP 8.4 AMP
```

Jadi:

**RCA membawa audio, USB membawa konfigurasi DSP.**

---

## Dokumentasi

Dokumentasi protokol:

```text
docs/PROTOCOL.md
```

Pemetaan parameter:

```text
docs/CONSOLE-MAPPING.md
```

Verifikasi USB pada hardware:

```text
docs/USB-VERIFICATION.md
```

Dokumentasi antarmuka desktop:

```text
docs/DESKTOP-UI.md
```

---

## Test

```sh
make test
```

Saat ini terdapat:

```text
46 pemeriksaan
```

yang mencakup:

- frame USB
- parsing protocol
- matematika filter
- fixed-point
- tabel data DSP
- konstanta dari software vendor

---

## Status Reverse Engineering

Reverse engineering ruang parameter DSP masih berlangsung.

Kontrol utama dari software vendor sudah berhasil dipetakan dan diverifikasi pada perangkat fisik.

Beberapa dari total:

```text
1.934 parameter ID
```

masih belum diketahui fungsinya.

Status terbaru dapat dilihat pada:

```text
docs/PROTOCOL.md
```

---

# License

MIT License.

See:

```text
LICENSE
```

---

## Disclaimer

This project is an independent reverse-engineering and interoperability effort.

It is not affiliated with, endorsed by, or sponsored by Zevox, Vox Research, Nuvoton, or the original DSP software vendor.

Product names and trademarks belong to their respective owners.
