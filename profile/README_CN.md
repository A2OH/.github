> [🇺🇸 English Version](https://github.com/A2OH/.github/blob/main/profile/README.md)

# A2OH — Android到OpenHarmony

在OpenHarmony上运行未修改的Android APK——不使用容器，不做API适配——采用受Flutter启发的引擎方案。

## 使命

通过将Android框架作为**可嵌入的运行时引擎**，将Android应用生态桥接到OpenHarmony。不需要逐一映射57,000个Android API，也不需要运行重量级的Android容器。我们将Android框架作为自包含引擎移植，渲染到OH表面，并在约15个HAL级边界处桥接OH系统服务。

**核心洞察：** Android的57,000个API中，99%是纯Java代码，在任何JVM中都能原样运行。只有约200个方法（约0.4%）需要平台桥接——与Flutter、React Native和Unity跨越的边界相同。

## 架构

```
┌─────────────────────────────────┐
│     Android APK（未修改）        │
├─────────────────────────────────┤
│  AOSP框架（62K行代码）           │  ← 真实AOSP View、TextView等
│  + MiniServer（替代              │  ← 6个轻量级管理器
│    Android SystemServer）        │
├─────────────────────────────────┤
│  Dalvik Universal VM            │  ← KitKat可移植解释器
│  (x86_64, ARM32, ARM64, RISC-V) │     64位移植
├─────────────────────────────────┤
│  liboh_bridge.so（约15个桥接）   │  ← Canvas→OH_Drawing等
├─────────────────────────────────┤
│  OpenHarmony OS                 │  ← XComponent, NativeWindow
└─────────────────────────────────┘
```

## 代码仓库

### [westlake](https://github.com/A2OH/westlake) — 集成层

主项目。将Dalvik VM + OHOS平台结合，在OpenHarmony上运行Android APK。

- **62,153行**未修改的AOSP代码（View、ViewGroup、TextView、LinearLayout等）
- **MiniServer**替代Android SystemServer
- **2,470+测试**覆盖7个测试应用
- **OHBridge** JNI桥接：106个原生方法
- **真实APK支持**：已测试Facebook、Netflix、Spotify清单解析

### [dalvik-universal](https://github.com/A2OH/dalvik-universal) — 通用Dalvik VM

KitKat Dalvik VM移植到任何现代Linux平台。

- **x86_64 Linux**：Hello World、MockDonalds 14/14通过
- **ARM32 OHOS**：在OpenHarmony QEMU上运行Hello World
- **46+修复**用于64位指针处理
- **静态二进制**：7.1MB ARM32，无共享库依赖

### [openharmony-wsl](https://github.com/A2OH/openharmony-wsl) — WSL2上的OHOS

在Windows WSL2上使用QEMU ARM32模拟构建和运行OpenHarmony。

- **98%的OHOS**从源码构建
- **QEMU ARM32**配合virtio设备、VNC显示
- **交互式Shell** + Dalvik VM执行

## 当前状态（2026年3月）

| 里程碑 | 状态 |
|--------|------|
| 真实APK在Dalvik上运行于OHOS ARM32 QEMU | **完成** |
| 62K行未修改AOSP代码编译+运行于Dalvik | **完成** |
| MockDonalds 14/14测试通过 | **完成** |
| ArkUI可视化渲染（VNC） | 进行中 |

## 性能：引擎 vs 容器

| 指标 | 引擎（Westlake） | 容器（Anbox） |
|------|------------------:|------------------:|
| 内存开销 | **约15 MB** | 500 MB - 1 GB |
| 应用启动 | **约2秒** | 约5-7秒 |
| JNI每帧开销 | **0.08%** | 25%（缓冲区拷贝） |
| 50美元手机可用 | **是** | 否 |

## 为什么叫"A2OH"

**A**ndroid **2**（to）**O**pen**H**armony。引擎方案使90-95%的Android应用无需修改即可在OpenHarmony上运行。

## 许可证

Apache 2.0
