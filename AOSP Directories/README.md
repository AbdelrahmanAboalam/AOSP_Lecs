# Directories in AOSP

This README provides an overview of the primary directories within AOSP, detailing each directory’s purpose and contents. Understanding these directories is essential for developers working on the Android source code and device-specific customizations.

---

## Agenda

1. [Art](#1-art)
2. [Bionic](#2-bionic)
3. [Build](#3-build)
4. [Cts (Compatibility Test Suite)](#4-cts-compatibility-test-suite)
5. [Dalvik](#5-dalvik)
6. [Device](#6-device)
7. [External](#7-external)
8. [Frameworks](#8-frameworks)
9. [Hardware](#9-hardware)
10. [Packages](#10-packages)
11. [Prebuilts](#11-prebuilts)
12. [System](#12-system)
13. [Toolchain](#13-toolchain)
14. [Out](#14-out)
15. [Summary](#15-summary)

---

## 1. **Art**
- **Role**: Contains the code for the Android Runtime (ART).
- **Details**: 
  - ART is the runtime environment in Android, replacing the original Dalvik virtual machine.
  - It uses Ahead-of-Time (AOT) compilation to improve performance and reduce memory usage, providing a more efficient runtime for Android applications.

## 2. **Bionic**
- **Role**: Holds the code for Bionic, Android’s C-like standard library.
- **Details**: 
  - Bionic is optimized for Android’s mobile and embedded environments, focusing on low memory and CPU usage.
  - Replaces the traditional `glibc` library found in Linux with a leaner and faster implementation suitable for resource-constrained devices.

## 3. **Build**
- **Role**: Contains scripts and configurations for setting up the Android build environment.
- **Details**:
  - Houses both legacy and current build systems used by Android.
  - Key script: `envsetup.sh`, which sets environment variables and introduces helper commands like `lunch` to configure and execute builds.

## 4. **Cts (Compatibility Test Suite)**
- **Role**: Contains the Compatibility Test Suite (CTS) for Android.
- **Details**:
  - CTS tests ensure that a device meets Android’s compatibility standards.
  - Passing CTS is required for a device to be certified as compatible with Android, ensuring reliability and interoperability across devices.

## 5. **Dalvik**
- **Role**: Contains the source code for Dalvik, Android’s original virtual machine.
- **Details**:
  - Dalvik was replaced by ART but is maintained in AOSP for backward compatibility.
  - It was originally designed to support efficient memory usage for mobile applications by using a Just-in-Time (JIT) compiler.

## 6. **Device**
- **Role**: Stores device-specific code, configurations, and customizations.
- **Details**:
  - Includes board-specific configurations, toolchain settings, kernel selections, and device-specific applications.
  - This directory is crucial for developers customizing Android to specific hardware, allowing for tailored OS builds per device.

## 7. **External**
- **Role**: Holds buildable packages that are external to AOSP but necessary for the system.
- **Details**:
  - Contains packages like cryptographic libraries, XML parsers, and other third-party dependencies.
  - This directory helps ensure that external dependencies are explicitly managed, reducing version conflicts and enhancing modularity.

## 8. **Frameworks**
- **Role**: Core of the Android system, hosting the Android Framework and public API.
- **Details**:
  - Key services, such as `ActivityManagerService` and `PackageManagerService`, reside here.
  - Provides APIs for applications and manages essential services like user input, sensor data, and activity management.
  - Components are organized by type, including Java code (`core/java`) and native implementations (`base/native`).

## 9. **Hardware**
- **Role**: Contains the Hardware Abstraction Layer (HAL) code.
- **Details**:
  - Provides an interface for Android to interact with hardware components (e.g., Bluetooth, Wi-Fi, cameras).
  - While common hardware functions are covered here, device-specific customizations typically go into the `device` directory.
  - This directory allows Android to support a wide variety of hardware by abstracting the hardware-specific implementations.

## 10. **Packages**
- **Role**: Contains default applications and packages bundled with Android.
- **Details**:
  - Applications included by default in the Android system (e.g., DeskClock, Phone, Camera) are located here.
  - These applications are part of the system image and cannot typically be uninstalled by the end user.

## 11. **Prebuilts**
- **Role**: Holds prebuilt binaries and dependencies needed for Android builds.
- **Details**:
  - Unlike `external`, which contains source code, `prebuilts` includes compiled binaries, such as the `clang` and `gcc` compilers, and the QEMU-based Android emulator.
  - Provides essential dependencies for the build process without requiring recompilation.

## 12. **System**
- **Role**: Houses essential system components, libraries, and daemons.
- **Details**:
  - Contains native daemons, SELinux policies, and other core system data required for Android’s operation.
  - Examples include `vold` (volume daemon), trusted certificates, and security policies.
  - Ensures system stability and reliability by centralizing core components.

## 13. **Toolchain**
- **Role**: Hosts automated testing and benchmarking tools.
- **Details**:
  - This directory includes tools that developers can use to test and gather performance data on the Android OS.
  - Required for system-level benchmarking and performance optimization.

## 14. **Out**
- **Role**: Serves as a scratchpad for build outputs.
- **Details**:
  - Not part of the core AOSP directory structure but used by the build process as a temporary storage area for intermediate and final build artifacts.
  - Helps keep the source directories clean, making it easy to reset the build environment by deleting this directory.

---

## Summary

Each directory within AOSP has a specific purpose, supporting the modular and scalable nature of the Android system. Understanding the role of each directory is key for developers working on AOSP, whether for system customization, application development, or device compatibility. This organized structure allows for efficient development, integration, and deployment across diverse hardware configurations.

--- 

This README includes an agenda for easy navigation. Let me know if you need any additional customization!
