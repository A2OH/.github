**[English](README.md)** | **[中文](README_CN.md)**

# A2OH — Android to OpenHarmony

Running unmodified Android APKs on OpenHarmony, without containers, without API shimming — using an engine approach inspired by how Flutter runs on any platform.

## Mission

Bridge the Android app ecosystem to OpenHarmony by treating the Android framework as an **embeddable runtime engine**. Instead of mapping 57,000 Android APIs individually or running a heavy Android container, we port the Android framework as a self-contained engine that renders to OH surfaces and bridges to OH system services at ~15 HAL boundaries.

**Key insight:** 99% of Android's 57,000 APIs are pure Java that runs unchanged in any JVM. Only ~200 methods (~0.4%) need platform bridges — the same boundaries that Flutter, React Native, and Unity cross.

## Architecture

```
┌─────────────────────────────────┐
│     Android APK (unmodified)    │
├─────────────────────────────────┤
│  AOSP Framework (62K lines)     │  ← Real AOSP View, TextView, etc.
│  + MiniServer (replaces         │  ← 6 lightweight managers
│    Android SystemServer)        │
├─────────────────────────────────┤
│  Dalvik Universal VM            │  ← KitKat portable interpreter
│  (x86_64, ARM32, ARM64, RISC-V)│     64-bit ported
├─────────────────────────────────┤
│  liboh_bridge.so (~15 bridges)  │  ← Canvas→OH_Drawing, Input, Audio...
├─────────────────────────────────┤
│  OpenHarmony OS                 │  ← XComponent, NativeWindow, Services
└─────────────────────────────────┘
```

## Repositories

### [westlake](https://github.com/A2OH/westlake) — The Integration Layer

The main project. Marries Dalvik VM + OHOS platform to run Android APKs on OpenHarmony.

- **62,153 lines** of unmodified AOSP code (View, ViewGroup, TextView, LinearLayout, etc.)
- **MiniServer** replaces Android SystemServer: MiniActivityManager, MiniPackageManager, MiniWindowManager, MiniServiceManager, MiniContentResolver, ActivityThread
- **2,470+ tests** across 7 test apps (MockDonalds, Calculator, Notes, SuperApp, etc.)
- **OHBridge** JNI bridge: 106 native methods connecting Java → OHOS APIs
- **Real APK support**: manifest parsing (Facebook, Netflix, Spotify tested), resources.arsc, DEX loading
- **Pixel rendering**: Java2D PNG output for visual validation
- **Architecture docs**: EN + CN with Mermaid diagrams, performance analysis, APK gap analysis

### [dalvik-universal](https://github.com/A2OH/dalvik-universal) — Dalvik VM for Modern Platforms

### [art-universal](https://github.com/A2OH/art-universal) — ART Runtime Port (10-50x faster)

Port the Android Runtime (ART) to modern platforms. 623K lines C++, Apache 2.0, own compiler backend (no LLVM). Three strategies: ART interpreter (3-5x, 2 months), dex2oat AOT (10-50x, 3 months), full JIT (10-50x, 6 months).

KitKat Dalvik VM ported to run on any modern Linux platform, not just Android.

- **x86_64 Linux**: Hello World, MockDonalds 14/14, real APK execution
- **ARM32 OHOS**: Hello World on OpenHarmony QEMU
- **46+ fixes** for 64-bit pointer handling
- **libcore_bridge.cpp**: 30+ JNI natives (Math, ICU, regex, I/O)
- **Static binary**: 7.1MB ARM32, no shared library dependencies
- **Future**: ARM64 Ubuntu, RISC-V

### [openharmony-wsl](https://github.com/A2OH/openharmony-wsl) — OHOS on WSL2/QEMU

Build and run OpenHarmony on Windows WSL2 with QEMU ARM32 emulation. No real hardware needed.

- **98% of OHOS** built from source on WSL2
- **QEMU ARM32** with virtio devices, VNC display
- **Headless ArkUI** (ace_engine standalone)
- **Interactive shell** + Dalvik VM execution on OHOS kernel
- **VNC support** for graphical output

### Other Repos

| Repo | Description |
|------|-------------|
| [A2OH-Factory](https://github.com/A2OH/A2OH-Factory) | AI-driven shim generation playbook |
| [A2OH-skills](https://github.com/A2OH/A2OH-skills) | Android→OH conversion skill files |
| [a2oh-orchestrator](https://github.com/A2OH/a2oh-orchestrator) | Task queue orchestration |

## Current Status (March 2026)

| Milestone | Status |
|-----------|--------|
| Real APK runs on Dalvik on OHOS ARM32 QEMU | **Done** |
| 62K lines unmodified AOSP compiles + runs on Dalvik | **Done** |
| MockDonalds 14/14 tests on Dalvik x86_64 | **Done** |
| Facebook/Netflix/Spotify manifest parsing | **Done** |
| ArkUI visual rendering on QEMU | In progress |
| Run third-party F-Droid APK end-to-end | Next |

## Performance: Engine vs Container

| Metric | Engine (Westlake) | Container (Anbox) |
|--------|------------------:|------------------:|
| Memory overhead | **~15 MB** | 500 MB - 1 GB |
| App startup | **~2s** | ~5-7s |
| Touch latency | **~20ms** | ~26ms |
| JNI overhead per frame | **0.08%** | 25% (buffer copy) |
| $50 phone viable | **Yes** | No |

## Why "A2OH"

**A**ndroid **2** (to) **O**pen**H**armony. The engine approach that makes 90-95% of Android apps run on OpenHarmony with zero app modifications.

## License

Apache 2.0
