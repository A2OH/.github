# A2OH — Android to OpenHarmony

Running unmodified Android APKs on OpenHarmony, without containers, without API shimming — using an engine approach inspired by how Flutter runs on any platform.

---

# A2OH — Android 到 OpenHarmony

在 OpenHarmony 上运行未修改的 Android APK，无需容器，无需逐个 API 映射——采用类似 Flutter 跨平台运行的引擎方案。

---

## Mission | 使命

Bridge the Android app ecosystem to OpenHarmony by treating the Android framework as an **embeddable runtime engine**. Instead of mapping 57,000 Android APIs individually or running a heavy Android container, we port the Android framework as a self-contained engine that renders to OH surfaces and bridges to OH system services at ~15 HAL boundaries.

**Key insight:** 99% of Android's 57,000 APIs are pure Java that runs unchanged in any JVM. Only ~200 methods (~0.4%) need platform bridges — the same boundaries that Flutter, React Native, and Unity cross.

将 Android 框架作为**可嵌入的运行时引擎**，桥接 Android 应用生态到 OpenHarmony。引擎方案不需要逐个映射 57,000 个 Android API，也不需要运行笨重的 Android 容器，而是将 Android 框架作为自包含引擎移植，渲染到 OH 表面并在约 15 个 HAL 边界处桥接到 OH 系统服务。

**关键洞察：** Android 的 57,000 个 API 中 99% 是纯 Java 代码，无需修改即可在任何 JVM 中运行。只有约 200 个方法（约 0.4%）需要平台桥接——与 Flutter、React Native 和 Unity 跨越的边界相同。

---

## Architecture | 架构

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

---

## Repositories | 仓库

### [westlake](https://github.com/A2OH/westlake) — The Integration Layer | 集成层

The main project. Marries Dalvik VM + OHOS platform to run Android APKs on OpenHarmony.

主项目。将 Dalvik VM 与 OHOS 平台结合，在 OpenHarmony 上运行 Android APK。

- **62,153 lines** of unmodified AOSP code (View, ViewGroup, TextView, LinearLayout, etc.)
- **MiniServer** replaces Android SystemServer: MiniActivityManager, MiniPackageManager, MiniWindowManager, MiniServiceManager, MiniContentResolver, ActivityThread
- **2,470+ tests** across 7 test apps (MockDonalds, Calculator, Notes, SuperApp, etc.)
- **OHBridge** JNI bridge: 106 native methods connecting Java → OHOS APIs
- **Real APK support**: manifest parsing (Facebook, Netflix, Spotify tested), resources.arsc, DEX loading
- **Pixel rendering**: Java2D PNG output for visual validation
- **Architecture docs**: EN + CN with Mermaid diagrams, performance analysis, APK gap analysis

### [dalvik-universal](https://github.com/A2OH/dalvik-universal) — Dalvik VM for Modern Platforms | 面向现代平台的 Dalvik VM

KitKat Dalvik VM ported to run on any modern Linux platform, not just Android.

KitKat Dalvik VM 的移植版本，可在任何现代 Linux 平台上运行，不限于 Android。

- **x86_64 Linux**: Hello World, MockDonalds 14/14, real APK execution
- **ARM32 OHOS**: Hello World on OpenHarmony QEMU
- **46+ fixes** for 64-bit pointer handling
- **libcore_bridge.cpp**: 30+ JNI natives (Math, ICU, regex, I/O)
- **Static binary**: 7.1MB ARM32, no shared library dependencies
- **Future**: ARM64 Ubuntu, RISC-V

### [openharmony-wsl](https://github.com/A2OH/openharmony-wsl) — OHOS on WSL2/QEMU | WSL2/QEMU 上的 OHOS

Build and run OpenHarmony on Windows WSL2 with QEMU ARM32 emulation. No real hardware needed.

在 Windows WSL2 上使用 QEMU ARM32 模拟构建和运行 OpenHarmony。无需真实硬件。

- **98% of OHOS** built from source on WSL2
- **QEMU ARM32** with virtio devices, VNC display
- **Headless ArkUI** (ace_engine standalone)
- **Interactive shell** + Dalvik VM execution on OHOS kernel
- **VNC support** for graphical output

### Other Repos | 其他仓库

| Repo | Description | 描述 |
|------|-------------|------|
| [A2OH-Factory](https://github.com/A2OH/A2OH-Factory) | AI-driven shim generation playbook | AI 驱动的 shim 生成方案 |
| [A2OH-skills](https://github.com/A2OH/A2OH-skills) | Android→OH conversion skill files | Android→OH 转换技能文件 |
| [a2oh-orchestrator](https://github.com/A2OH/a2oh-orchestrator) | Task queue orchestration | 任务队列编排 |

---

## Current Status (March 2026) | 当前状态（2026年3月）

| Milestone | 里程碑 | Status | 状态 |
|-----------|--------|--------|------|
| Real APK runs on Dalvik on OHOS ARM32 QEMU | 真实 APK 在 OHOS ARM32 QEMU 上的 Dalvik 中运行 | **Done** | **已完成** |
| 62K lines unmodified AOSP compiles + runs on Dalvik | 62K 行未修改 AOSP 代码在 Dalvik 上编译运行 | **Done** | **已完成** |
| MockDonalds 14/14 tests on Dalvik x86_64 | MockDonalds 14/14 测试在 Dalvik x86_64 上通过 | **Done** | **已完成** |
| Facebook/Netflix/Spotify manifest parsing | Facebook/Netflix/Spotify manifest 解析 | **Done** | **已完成** |
| ArkUI visual rendering on QEMU | ArkUI 视觉渲染（QEMU） | In progress | 进行中 |
| Run third-party F-Droid APK end-to-end | 端到端运行第三方 F-Droid APK | Next | 下一步 |

---

## Performance: Engine vs Container | 性能对比：引擎方案 vs 容器方案

| Metric | 指标 | Engine (Westlake) | Container (Anbox) |
|--------|------|------------------:|------------------:|
| Memory overhead | 内存开销 | **~15 MB** | 500 MB - 1 GB |
| App startup | 应用启动 | **~2s** | ~5-7s |
| Touch latency | 触摸延迟 | **~20ms** | ~26ms |
| JNI overhead per frame | 每帧 JNI 开销 | **0.08%** | 25% (buffer copy) |
| $50 phone viable | 50 美元手机可用 | **Yes** | No |

---

## Why "A2OH" | 为什么叫 "A2OH"

**A**ndroid **2** (to) **O**pen**H**armony. The engine approach that makes 90-95% of Android apps run on OpenHarmony with zero app modifications.

**A**ndroid **2** (to) **O**pen**H**armony。引擎方案使 90-95% 的 Android 应用无需任何修改即可在 OpenHarmony 上运行。

---

## License | 许可证

Apache 2.0
