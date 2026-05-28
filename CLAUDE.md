# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

Unofficial Lockzhiner RK2206 development board repository based on OpenHarmony 3.0 LTS (LiteOS-M kernel). Target hardware: Rockchip RK2206, Cortex-M4 200MHz, 256KB RAM, 8MB PSRAM, 8MB FLASH.

## Build Commands

```bash
# Activate the Python virtual environment first (from repo root)
source venv/bin/activate
hb set -root .

# Full build
hb build -f
# Or equivalently:
python build.py -f

# Flash to device
python flash.py
```

The build system is **hb** (HarmonyOS Build), which wraps GN (generate `.ninja` files) → Ninja (compile `.o` files) → Make (link into `.elf/.bin/.hex`). The Python virtual environment at `venv/` contains the `hb` CLI tool.

Toolchain: `gcc-arm-none-eabi-10.3` (included in the repo root as `.tar.bz2`). The device build configuration is at `device/rockchip/rk2206/sdk_liteos/config.gni`.

## Adding or Removing Samples

Only one sample can be compiled at a time. Edit `vendor/lockzhiner/rk2206/samples/BUILD.gn` to enable exactly one feature target in the `features` list.

If the sample introduces new library dependencies, add them to the `LIBS` variable in `device/rockchip/rk2206/sdk_liteos/Makefile` (the `-l<name>` flags are linked via `--whole-archive` for boot/hardware/hdf/app libs, and via `--start-group/--end-group` for common libs).

## Product Configuration

`vendor/lockzhiner/rk2206/config.json` defines which OpenHarmony subsystems and components are included in the build. The top-level `build/lite/BUILD.gn` reads this file and resolves each component against its definition in `build/lite/components/<subsystem>.json`, validating kernel compatibility before adding the component's targets to the build DAG.

## Directory Architecture

| Directory | Purpose |
|---|---|
| `vendor/lockzhiner/rk2206/` | Board-specific: product config, samples, HAL adapters, HDF config/drivers |
| `device/rockchip/rk2206/sdk_liteos/` | Device SDK: linker script (`board.ld`), Makefile, partition layout (`include/link.h`), flash image tooling |
| `device/rockchip/rk2206/adapter/` | Adapter layer bridging OHOS components to the RK2206 SDK |
| `build/lite/` | Build system (hb tool, GN config, component definitions) |
| `kernel/liteos_m/` | LiteOS-M kernel source |
| `base/` | Base subsystems (startup, security/HUKS, hiviewdfx/logging) |
| `foundation/` | Foundation subsystems (communication: WiFi, SoftBus, IPC) |
| `drivers/` | HDF (Hardware Driver Foundation) framework |
| `third_party/` | Third-party libraries (lwIP, mbedTLS, cJSON, FatFs, CMSIS, paho MQTT, etc.) |
| `prebuilts/` | Prebuilt toolchains |
| `out/` | Build output |

Samples live in `vendor/lockzhiner/rk2206/samples/` organized by category:
- `a0~a9`: Kernel primitives (threads, semaphores, timers, mutexes, queues, events, file/PWM HALs)
- `b1~b13`: Basic peripherals (ADC, NFC, EEPROM, LCD, OLED, UART, WiFi TCP/UDP, GPIO, I2C, watchdog)
- `c1~c8`: E53 sensor modules (agriculture, smart covers, street lamp, vehicle, body induction, gesture, smoke, temperature)
- `d1~d7`: IoT cloud applications (MQTT + Huawei Cloud IoTDA)
- `x0~x4`: Smart Home series (doorbell → smart door → sensor → voice → MQTT net control, with dependency chain)
- `v0_screen_bot`: Smart pet companion robot (multi-threaded: LCD animations, servo laser, ADC key menu, voice, UDP, MQTT, auto environment control)

## Key Constraints

- **LiteOS-M** is a lightweight RTOS, not Linux — no MMU, no processes, flat memory model. All code runs in a single address space as threads.
- The linker script at `device/rockchip/rk2206/sdk_liteos/board.ld` defines the memory layout, including NOR FLASH partitions (loader, liteos, rootfs, userfs). The flash image generation tooling is at `device/rockchip/tools/package/mkimage.sh`.
- The `ohos_config.json` at repo root stores absolute paths from the original development environment (`/home/openharmony`). The `hb set -root .` command regenerates this.
