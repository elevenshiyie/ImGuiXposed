# ImGuiXposed

[![Platform](https://img.shields.io/badge/Platform-Android-3DDC84?logo=android&logoColor=white)](#)
[![Framework](https://img.shields.io/badge/Framework-LSPosed%20%7C%20Xposed-blue)](#)
[![Language](https://img.shields.io/badge/Language-Java-orange)](#)
[![Type](https://img.shields.io/badge/Type-Template-lightgrey)](#)

一个基于 Xposed / LSPosed 的 Android 模块模板，用于在目标应用启动阶段加载 ImGui，并为后续自定义 Hook 提供清晰的入口。

> **项目定位**：ImGuiXposed 是模板/框架型项目，不包含任何针对特定应用的修改逻辑、破解功能或业务实现。使用者需要根据实际需求自行编写 Hook 代码。

---

## 目录

- [简介](#简介)
- [设计目标](#设计目标)
- [非目标](#非目标)
- [特性](#特性)
- [环境要求](#环境要求)
- [快速开始](#快速开始)
- [配置说明](#配置说明)
- [工作原理](#工作原理)
- [使用示例](#使用示例)
- [项目结构](#项目结构)
- [常见问题](#常见问题)
- [故障排查](#故障排查)
- [开发指南](#开发指南)
- [贡献](#贡献)
- [安全与免责声明](#安全与免责声明)
- [许可证](#许可证)

---

## 简介

ImGuiXposed 通过 Xposed / LSPosed 框架，在目标应用的 `Activity.onCreate` 执行阶段注入并加载 ImGui。项目定位为一个可复用的 Android 模块模板，适合用于：

- 学习 Xposed / LSPosed 模块的基本开发流程；
- 研究 ImGui 在 Android 目标进程中的加载方式；
- 作为自用调试工具、Mod 菜单或界面注入功能的基础框架；
- 快速验证 Hook 点、作用域配置和 native 库加载逻辑。

项目默认只负责 Hook 框架搭建和 ImGui 加载，不提供现成的游戏修改、应用破解或具体业务功能。

---

## 设计目标

- **简单**：保持模块入口清晰，便于阅读和二次开发。
- **可复用**：可作为多个目标应用 Hook 项目的起始模板。
- **可调试**：包含基础异常处理，便于定位 native 库加载和 Hook 失败问题。
- **作用域明确**：通过目标包名列表控制模块生效范围，避免影响非目标应用。

---

## 非目标

ImGuiXposed 不提供以下内容：

- 针对任何具体应用的修改逻辑；
- 游戏作弊、破解、绕过付费或规避检测功能；
- 开箱即用的成品 Mod 菜单；
- 对目标应用稳定性和兼容性的保证；
- 对第三方应用服务条款或法律法规风险的豁免。

---

## 特性

- 基于 Xposed / LSPosed 的作用域机制运行；
- 在目标 Activity 的 `onCreate` 阶段加载 ImGui；
- 通过 `packageLis` 声明目标应用包名；
- 包含 native 库加载失败时的异常处理逻辑；
- 结构简单，便于继续添加自定义 Hook；
- 适合作为学习 Xposed 模块开发和 ImGui Android 集成的模板。

---

## 环境要求

### 运行环境

- Android 设备已 Root；
- 已安装并启用 LSPosed 或其他兼容的 Xposed 框架；
- 目标应用已安装在设备上；
- 目标应用架构与 ImGui 所需 native 库架构匹配。

### 编译环境

- Android Studio；
- JDK 17 或项目 `gradle.properties` 中指定的 JDK 版本；
- Android SDK 与项目编译配置匹配；
- Gradle 可通过项目自带 Wrapper 运行。

---

## 快速开始

### 1. 配置目标应用

打开 Hook 入口文件，例如：

```text
app/src/main/java/.../MainHook.java
```

找到 `packageLis`，将目标应用包名加入列表：

```java
packageLis.add("com.example.targetapp");
```

`packageLis` 用于声明模块需要处理的目标进程。只有在此列表中声明的包名，才会触发后续加载和 Hook 逻辑。

### 2. 安装并启用模块

1. 将编译生成的 APK 安装到设备；
2. 在 LSPosed 管理器中启用 **ImGuiXposed** 模块；
3. 进入模块的“作用域”设置，勾选目标应用；
4. 强制停止目标应用；
5. 重新启动目标应用。

目标应用启动后，模块会尝试在其 `Activity.onCreate` 阶段加载 ImGui。

---

## 工作原理

Xposed / LSPosed 在目标应用进程启动时加载模块。模块通过 `handleLoadPackage` 判断当前进程是否属于目标包名，然后定位目标 Activity 的 `onCreate` 方法，并在其执行过程中完成 ImGui 的加载与注入。

基本流程如下：

```text
目标应用进程启动
        │
        ▼
Xposed / LSPosed 加载模块
        │
        ▼
handleLoadPackage 判断包名
        │
        ▼
定位目标 Activity.onCreate
        │
        ▼
加载 ImGui / native 库
        │
        ▼
进入自定义 Hook 逻辑
```

`Activity.onCreate` 的标准签名是：

```java
protected void onCreate(Bundle savedInstanceState)
```

使用 `findAndHookMethod` 时，参数类型必须写为 `Bundle.class`，否则可能无法匹配目标方法：

```java
findAndHookMethod(
    "com.example.targetapp.MainActivity",
    lpparam.classLoader,
    "onCreate",
    Bundle.class,
    new XC_MethodHook() {
        @Override
        protected void afterHookedMethod(MethodHookParam param) {
            // 在此加载 ImGui 或添加自定义逻辑
        }
    }
);
```

默认选择 `onCreate` 作为注入点，是因为它处于目标 UI 初始化阶段，适合加载 ImGui 菜单。根据实际需求，也可以选择 `onResume`、`attachBaseContext` 或其他更合适的生命周期方法。

---

## 项目结构

```text
ImGuiXposed/
├── app/
│   └── src/main/
│       ├── java/.../MainHook.java    # Hook 入口、目标包名、ImGui 加载逻辑
│       ├── assets/xposed_init        # Xposed 入口类声明
│       └── AndroidManifest.xml       # 模块元数据与应用配置
├── build.gradle
├── gradle.properties
├── settings.gradle
└── README.md
```


## 安全与免责声明

- 本项目仅供学习、研究与技术交流使用。
- 请勿将本项目用于违反目标应用服务条款、侵犯他人权益或违反法律法规的场景。
- 使用本模块可能导致目标应用崩溃、数据异常、账号风险或被封禁，请自行评估风险。
- 项目作者不对因使用本项目产生的任何直接或间接损失承担责任。
- 在分发或修改前，请确认你拥有相应代码、资源和第三方库的授权。
- 请遵守当地法律法规以及目标应用的服务条款。

---

## 致谢

- 感谢 Xposed / LSPosed 社区提供的框架与文档；
- 感谢 ImGui 及其相关绑定项目的作者；
- 感谢所有提交 Issue、反馈问题和改进建议的贡献者。
