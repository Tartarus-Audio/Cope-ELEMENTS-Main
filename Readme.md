# COPE Elements - Audio Library Collection

## Overview

COPE Elements is a modular, open-source audio library collection designed as an alternative to proprietary solutions like BASS. Each library handles a specific domain of audio functionality and can be used independently or combined.

The collection follows a naming convention with layered meanings - professional acronyms for public presentation, with subtle references for those who recognize them.

All code is written in **Zig 0.15.2**.
Designed to be platform agnostic, but currently only tested on Windows.

COPE: Chrysalis Open Platform Engine -  Helping you COPE with audio.

---

## The Libraries

### HARMONY
**Runtime Orchestration**

The context that ties everything together, AKA the force that unites the Elements. Required to enter the portal to... Something.

| Aspect | Description |
|--------|-------------|
| Purpose | Runtime context and initialization - the gateway to all COPE libraries |
| Features | Audio thread management, global clock, memory environment, library coordination |
| Responsibility | Thread priority (MMCSS), lock-free memory pools, transport state, session lifecycle |
| Priority | **Required** - Must be initialized before using any COPE library |

Platform notes:
- **Windows**: Handles COM init, MMCSS priority, memory locking
- **Linux**: RT scheduling, memory locking, thread priority (Not implemented yet)

Usage:
```zig
// First thing in your application
harmony.init();
defer harmony.deinit();

// Now you can use any COPE library
const decoder = butter.open("song.mp3");
const stream = magic.openStream(...);
```

---

### MAGIC
**Multi-API Gateway for Integrated Connectivity**

Core audio output abstraction layer.

| Aspect | Description |
|--------|-------------|
| Purpose | Unified interface for audio output across platforms |
| Backends | WASAPI, ASIO, ALSA, CoreAudio, DirectSound |
| Responsibility | Device enumeration, buffer management, sample rate handling, latency optimization |
| Priority | **Primary** - Foundation for all audio output |

---

### DASH
**Digital Audio Signal Handler**

Real-time DSP and effects processing.

| Aspect | Description |
|--------|-------------|
| Purpose | Audio transformation and effects |
| Features | Filters, EQ, compression, reverb, delay, gain, panning |
| Responsibility | Sample-accurate processing, effect chains, parameter automation |
| Priority | Secondary - Builds on MAGIC output |

---

### BUTTER
**Binary Universal Transcoder for Transforming Encoded Resources**

Audio format encoding and decoding.

| Aspect | Description |
|--------|-------------|
| Purpose | Read and write audio file formats |
| Formats | WAV, OGG/Vorbis, MP3, FLAC, AIFF |
| Responsibility | Decoding, encoding, metadata handling, streaming decode |
| Priority | Secondary - Provides audio data to other libraries |

---

### DARLING
**Dynamic Audio Runtime for Loading Integrated Node Graphs**

Plugin hosting framework.

| Aspect | Description |
|--------|-------------|
| Purpose | Load and manage audio plugins |
| Formats | VST3, CLAP |
| Responsibility | Plugin discovery, sandboxxing, parameter management, preset handling |
| Priority | Tertiary - Extends processing capabilities |

---

### APPLES
**Adaptive Pipeline for Processing Low-latency Endpoint Streams**

Audio mixing and routing.

| Aspect | Description |
|--------|-------------|
| Purpose | Combine and route multiple audio streams |
| Features | Multi-channel mixing, routing matrix, submixes, sends |
| Responsibility | Level management, channel routing, bus architecture |
| Priority | Tertiary - Orchestrates multiple sources |

---

### ADHD
**Async Data Handling and Distribution**

Utility library and kitchen sink - optional enhancement for all libraries.

| Aspect | Description |
|--------|-------------|
| Purpose | Shared utilities that enhance other libraries |
| Features | Advanced memory pools, unified logging, lock-free structures, math utilities, buffer sharing |
| Responsibility | Cross-library coordination, shared resources, enhanced implementations |
| Priority | Optional Enhancement - Makes everything better when present |

Design philosophy:
- Each library works standalone with minimal internal utilities
- When ADHD is present, libraries can use its superior implementations
- Enables shared memory pools, unified logging, and cross-library buffer sharing
- Not required, but recommended when using multiple libraries together

---

### SLUMBER
**Synthesis Layer for Unified MIDI-Based Event Rendering**

MIDI processing and soundfont synthesis.

| Aspect | Description |
|--------|-------------|
| Purpose | MIDI playback and instrument synthesis |
| Features | MIDI file parsing, MIDI I/O, soundfont synthesis (SF2/SF3/SFZ), real-time event processing |
| Responsibility | MIDI device communication, note events, instrument synthesis, controller mapping |
| Priority | Tertiary - Standalone or integrated with other libraries |

Platform notes:
- Works independently - doesn't require other COPE libraries
- Can output directly to MAGIC, or generate samples for other uses

Note: For tracker formats (MOD/XM/S3M/IT), see OCTAVIA.

---

### OCTAVIA
**Orchestrated Comprehensive Tracker And Virtual Instrument Architecture**

Tracker module playback engine.

| Aspect | Description |
|--------|-------------|
| Purpose | Play tracker music formats |
| Formats | MOD, XM, S3M, IT |
| Features | Pattern playback, sample interpolation, effect processing, tempo/speed control |
| Responsibility | Module parsing, tick-based sequencing, channel mixing, tracker effects |
| Priority | Tertiary - Standalone or integrated with other libraries |

Note: Separate from SLUMBER (MIDI) - trackers have their own playback model with patterns, samples, and effects built-in.

---

### WUBS
**Waveform Utilities for Beat Synthesis**

Audio analysis and beat detection.

| Aspect | Description |
|--------|-------------|
| Purpose | Analyze audio for visualization and synchronization |
| Features | FFT/spectrum analysis, beat detection, BPM detection, key detection, loudness metering (LUFS/RMS), waveform generation, onset detection |
| Responsibility | Real-time and offline audio analysis, tempo tracking, frequency analysis |
| Priority | Tertiary - Enhances visualization and sync capabilities |

Future goals:
- Variable tempo tracking
- Key change detection
- Segment analysis

---

### CADENCE
**[Acronym TBD]**

System-wide audio routing and processing.

| Aspect | Description |
|--------|-------------|
| Purpose | Route and process audio between applications and devices |
| Features | Virtual routing, system-wide effects (via APO), ASIO-to-WASAPI bridge, per-app audio control |
| Responsibility | Audio pipeline interception, effect injection, device bridging |
| Priority | Standalone application/service - uses COPE libraries internally |

Components:
- **APO Plugin**: System-wide effect chain injection (Windows 10+)
- **CLOP-Audio**: DLL proxy for legacy apps (DirectSound, WinMM)
- **ASIO Bridge**: Expose WASAPI as ASIO (without locking everything like ASIO4ALL)
- **Virtual Driver**: Optional kernel driver for virtual audio endpoints (requires signing)

Think: Voicemeeter replacement, but open source and not shit.

---

### CELESTIA
**Complete Environment for Leveraging Every Sound Technology In Audio**

The unified high-level API - rules over all other libraries.

| Aspect | Description |
|--------|-------------|
| Purpose | Simple, unified interface for common audio tasks |
| Features | One-liner playback, recording, format conversion, automatic library coordination |
| Responsibility | Abstracts complexity, provides BASS-like simplicity, manages library interop |
| Priority | Optional - For users who want simplicity over control |

Usage philosophy:
- Power users: Use individual libraries (MAGIC, BUTTER, DASH, etc.) directly
- Simple use cases: Use CELESTIA and let it handle everything

---

## Architecture

```
                                                    ┌─────────────┐
┌─────────────────────────────────────────────────┐ │    ADHD     │
│                    Application                  │ │ (Optional)  │
├─────────────────────────────────────────────────┤ │             │
│                     CELESTIA                    │ │  Enhances   │
│            (Unified High-Level API)             │ │  any/all    │
├─────────────────────────────────────────────────┤ │  libraries  │
│                     HARMONY                     │ │  when       │
│         (Runtime Context / Orchestration)       │ │  present    │
├─────────────────────────────────────────────────┤ │             │
│  APPLES (Mixer) │ DARLING (Plugins) │ DASH (DSP)│<┤             │
├─────────────────────────────────────────────────┤ │             │
│ BUTTER (Codecs) │ WUBS (Analysis)               │<┤             │
├─────────────────────────────────────────────────┤ │             │
│ SLUMBER (MIDI)  │ OCTAVIA (Trackers)            │<┤             │
├─────────────────────────────────────────────────┤ │             │
│             MAGIC (Core Output)                 │<┤             │
└─────────────────────────────────────────────────┘ └─────────────┘

CADENCE: Standalone system-level routing (uses COPE internally)
```

Note: HARMONY init is required to enter the Portal. After that, each library works independently. ADHD sits alongside as an optional enhancement. CELESTIA wraps everything for simplicity when you want the full experience.

---

## What HARMONY Actually Does

```
HARMONY
├── Audio Thread Management
│   ├── Spawns dedicated RT threads
│   ├── Priority elevation (MMCSS on Windows)
│   └── Thread affinity / pinning
│
├── Global Clock
│   ├── Sample-accurate timing
│   ├── Transport state (play/pause/stop)
│   ├── Tempo/BPM (optional)
│   └── Sync source selection
│
├── Memory Environment
│   ├── Pre-allocated buffers
│   ├── Lock-free pools
│   ├── No allocations in RT path
│   └── Shared memory regions
│
├── Context / Session
│   ├── Init/shutdown lifecycle
│   ├── Error handling
│   ├── Callback registration
│   └── State machine
│
└── Library Coordination
    ├── Active MAGIC backend selection
    ├── DASH effect chain routing
    ├── APPLES mixer configuration
    └── Global sample rate / buffer size
```

---

## Dependency Graph

```
HARMONY (required - the gateway to Equestria)
    |
    +-- MAGIC (audio output)
    |
    +-- BUTTER (codecs)
    |
    +-- DASH (DSP)
    |
    +-- APPLES (mixer)
    |
    +-- DARLING (plugins)
    |
    +-- SLUMBER (MIDI/synthesis)
    |
    +-- OCTAVIA (tracker playback)
    |
    +-- WUBS (audio analysis)
    |
    v
CELESTIA (optional high-level wrapper)

ADHD: Optional enhancement for any/all libraries
      - When present: shared pools, unified logging, better utilities
      - When absent: each library uses its own internal utilities

CADENCE: Standalone application (uses COPE libraries internally)
```

---

## Development Order

1. **ADHD** - Utilities, optional enhancement (Complete)
2. **MAGIC** - Core output, can't test audio without output (Complete)
3. **HARMONY** - Runtime context, the gateway (minimal version for init)
4. **BUTTER** - Codecs, need audio data to play
5. **DASH** - DSP, process audio between decode and output
6. **APPLES** - Mixer, combine multiple sources
7. **DARLING** - Plugins, extend with third-party processing
8. **SLUMBER** - MIDI/synthesis, can be developed in parallel
9. **OCTAVIA** - Tracker playback, can be developed in parallel
10. **WUBS** - Audio analysis, can be developed in parallel
11. **CELESTIA** - Unified API, built last once all libraries exist

HARMONY can start minimal (just init/deinit with platform setup) and grow as other libraries need more features.

---

## Design Principles

- **HARMONY First**: Initialize HARMONY to enter Equestria, then use any library
- **Standalone After Init**: Each library works independently after HARMONY init
- **Optional Enhancement**: ADHD enhances but is never required
- **Zero lock-in**: Open source, swappable components
- **Cross-platform ready**: Platform-specific code isolated
- **Performance first**: Hot paths optimized, features opt-in
- **Clean API**: Simple common cases, power when needed
- **Graceful Integration**: Libraries detect and leverage each other when present

---

## File Naming

```
cope-adhd.dll / libcope-adhd.so
cope-magic.dll / libcope-magic.so
cope-butter.dll / libcope-butter.so
cope-harmony.dll / libcope-harmony.so
cope-dash.dll / libcope-dash.so
cope-apples.dll / libcope-apples.so
cope-darling.dll / libcope-darling.so
cope-slumber.dll / libcope-slumber.so
cope-octavia.dll / libcope-octavia.so
cope-wubs.dll / libcope-wubs.so
cope-celestia.dll / libcope-celestia.so

Note: CADENCE is a standalone application, not a library.
```

---

## Public Positioning

When people ask what the names mean:

| Library | What to say |
|---------|-------------|
| ADHD | Does everything, can't focus on just one thing |
| MAGIC | It's like magic how it abstracts different audio APIs |
| BUTTER | Smooth format conversion, like butter |
| HARMONY | Brings all the audio components into harmony |
| DASH | Fast processing, like a dash |
| APPLES | Apples to apples comparison of audio streams |
| DARLING | Treats your plugins like darlings |
| SLUMBER | Quietly synthesizes in the background while you sleep |
| CELESTIA | Raises the sun on your audio - everything just works |
| OCTAVIA | Conducts your tracker music with precision |
| WUBS | Drops the beat - analysis and detection |
| CADENCE | Sets the rhythm for your entire system |

---

## Repository Structure

```
github.com/Tartarus-Audio/
+-- cope-elements/ (This repository, information and documentation)
+-- cope-adhd/
+-- cope-magic/
+-- cope-butter/
+-- cope-harmony/
+-- cope-dash/
+-- cope-apples/
+-- cope-darling/
+-- cope-slumber/
+-- cope-octavia/
+-- cope-wubs/
+-- cope-celestia/
+-- cadence/ (standalone application)
+-- cope/ (meta-repo or combined distribution)
```

---

## Technology Stack

All code is written in **Zig 0.15.2**.

| Component | Technology | Rationale |
|-----------|------------|-----------|
| All libraries | Zig 0.15.2 | C interop, cross-compile, small binaries, single toolchain |
| UI (if any) | Undecided | ??? |

---

## Platform Support

| Platform | Status |
|----------|--------|
| Windows x64 | Primary target, actively tested |
| Linux x64 | Designed for, not yet tested |
| macOS | Future consideration maybe? Idk |

---

## Related Projects

- **CLOP** (Chrysalis Low-latency Output Protocol) - WinMM wrapper for Windows, supports both MIDI and Audio routing
- **CADENCE** - System-wide audio routing application built on COPE
- **COPE** (Chrysalis Open Platform Engine) - Main platform umbrella

---

## License

LGPL-3.0 - See [LICENSE](LICENSE) for details.

*Eww, ponies!* *COPE harder.*