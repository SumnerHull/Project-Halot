
# Halot One 3D Printer - Reverse Engineering & Control Summary

## 🧠 System Overview

### 🖥️ OS & Architecture
- **OS**: TinaLinux (OpenWrt-style embedded Linux)
- **CPU**: ARM (likely ARMv7), working with `arm-linux-gnueabihf` toolchain
- **UI Framework**: Qt 5.12.9 (confirmed pre-installed, using OpenGL ES2)
- **Graphics**: Linux framebuffer via `/dev/fb0`, OpenGL ES2, EGL
- **Framebuffer test**: `fbpad` shows white fuzz ✅

## 📟 Display & UI

- **Framebuffer**: 800×480, 32bpp (confirmed with framebuffer info dump)
- **Qt Apps**: Tested minimal app with `linuxfb` backend
- **Touchscreen**: Detected via evdev (`/dev/input/event2`, device: `cxsw_ctp`)
- **Qt Touch Input**: Confirmed via EVDEV params

## 🧩 System Setup & Build Env

- **Cross Toolchain**: `arm-linux-gnueabihf-g++`
- **Sysroot**: Extracted from printer to `~/sysroot-arm`
- **Qt Build**:
  - Built from `qt-everywhere-src-5.12.9`
  - Patched `qmake.conf` for OpenGL ES2 support
  - Added `khrplatform.h` manually
  - Linked against ARM `libGLESv2.so` from `libgles2-mesa-dev:armhf`
  - Qt `configure` and build now succeed ✅

## ⚙️ Physical Hardware

### Verified I/O Headers

| Header     | Pins                             | Function               | Status        |
|------------|----------------------------------|------------------------|---------------|
| **MT3**    | `gnd, dir, step1, en1`           | Stepper signal lines   | Not effective |
| **Random** | `oa1, oa2, ob1, ob2`             | Stepper output lines   | ✅ Works       |
| **Limit SW** | `sw6, gnd, vcc`                | Endstop                | Present       |
| **UV Light** | `led+, led+, led-, led-`, `led_en5v` | Resin UV lighting | Present       |
| **FAN1/2** | 2-pin headers                    | Fans (TBD)             | Unconfirmed   |

## 🦾 Motor Control

### Discovery
- **Only "Random Header" moves motor** when connected
- Movement happens **at boot**
- MT3 header has no effect

### GPIO Observations

| GPIO   | Direction | Boot Change | Role Candidate |
|--------|-----------|-------------|----------------|
| 3      | out       | lo → hi     | STEP or EN     |
| 210    | out       | lo → hi     | DIR or EN      |
| 1      | out       | hi          | Possibly power |
| 257    | out       | hi          | Reset/enable   |

### Test Tools
- Custom C++ app cycles GPIO combos for EN, DIR, STEP
- Awaiting feedback on which combination worked for full integration

## ❌ What Didn’t Work

- `/usr/bin/sys_attr`: Init-time tool, doesn’t control motor
- `/etc/init.d/play`: UI/animation launcher, unrelated to motion

## ✅ Next Steps

1. Confirm which GPIO combo moves motor via `stepper_test_refined`
2. Build motor control module into Qt UI
3. Add UV light toggle and limit switch reading
4. Finalize touchscreen-based control panel

---

Built with 💡 by reverse engineering the Halot One's system from root.
