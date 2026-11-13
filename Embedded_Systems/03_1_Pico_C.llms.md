# Pico with C/C++

Pico with C/C++

Author

Benedict Thekkel

## 🧰 1. Prerequisites

| Tool            | Purpose                                            |
|-----------------|----------------------------------------------------|
| VS Code         | Editor                                             |
| CMake + Ninja   | Build system                                       |
| `pico-sdk`      | Raspberry Pi official C/C++ SDK                    |
| `arm-none-eabi` | GCC toolchain for ARM Cortex-M0+                   |
| `OpenOCD`       | Debugging (optional for SWD)                       |
| UF2 uploader    | Flashing firmware via USB (drag-and-drop or tools) |

------------------------------------------------------------------------

## 🧭 2. Pico SDK Workflow Overview

    [ C Code ] → [ CMake ] → [ Ninja ] → [ ELF/BIN/UF2 ] → [ Raspberry Pi Pico ]

------------------------------------------------------------------------

## 🖥️ 3. Install Toolchain on Linux (Ubuntu)

``` bash
sudo apt update
sudo apt install cmake gcc-arm-none-eabi build-essential ninja-build libnewlib-arm-none-eabi git
```

On Windows/macOS: use [Raspberry Pi Pico official getting started guide](https://datasheets.raspberrypi.com/pico/getting-started-with-pico.pdf)

------------------------------------------------------------------------

## 📂 4. Project Structure

``` bash
pico-project/
├── CMakeLists.txt          # Top-level
├── blink/
│   ├── CMakeLists.txt
│   └── blink.c
└── pico-sdk/               # Submodule or symlink to SDK
```

------------------------------------------------------------------------

## 📄 5. `blink.c` Example

``` c
#include "pico/stdlib.h"

int main() {
    const uint LED_PIN = 25;
    gpio_init(LED_PIN);
    gpio_set_dir(LED_PIN, GPIO_OUT);

    while (true) {
        gpio_put(LED_PIN, 1);
        sleep_ms(500);
        gpio_put(LED_PIN, 0);
        sleep_ms(500);
    }
}
```

------------------------------------------------------------------------

## ⚙️ 6. `blink/CMakeLists.txt`

``` cmake
add_executable(blink
    blink.c
)

target_link_libraries(blink pico_stdlib)

pico_add_extra_outputs(blink)
```

------------------------------------------------------------------------

## 🧱 7. Top-level `CMakeLists.txt`

``` cmake
cmake_minimum_required(VERSION 3.13)
include(pico-sdk/pico_sdk_init.cmake)

project(pico_project C CXX ASM)
pico_sdk_init()

add_subdirectory(blink)
```

------------------------------------------------------------------------

## 🔄 8. Building the Project

``` bash
# First-time setup
git clone -b master https://github.com/raspberrypi/pico-sdk.git
cd pico-sdk
git submodule update --init

# Back to project
cd ..
mkdir build && cd build
cmake .. -DPICO_SDK_PATH=../pico-sdk -G Ninja
ninja
```

You should see:

    [100%] Built target blink

------------------------------------------------------------------------

## 🚀 9. Flash to Pico (UF2)

- Hold **BOOTSEL**, plug in Pico → it shows up as a USB drive
- Copy `blink.uf2` from `build/` to that drive

``` bash
cp blink.uf2 /media/YOUR_USER/RPI-RP2/
```

Or use `picotool`:

``` bash
sudo apt install picotool
picotool load blink.uf2 -f
```

------------------------------------------------------------------------

## 🛠 10. Set Up VS Code

### Extensions to install:

- **C/C++** (by Microsoft)
- **CMake Tools**
- **Cortex-Debug** *(if using SWD debugging)*
- **CMake Tools Helper** (optional)
- **CodeLLDB** (optional for debugging with `picoprobe`)

### `.vscode/settings.json`

``` json
{
  "cmake.generator": "Ninja",
  "cmake.configureSettings": {
    "PICO_SDK_PATH": "${workspaceFolder}/pico-sdk"
  },
  "C_Cpp.default.configurationProvider": "ms-vscode.cmake-tools"
}
```

### `.vscode/launch.json` *(for debugging via SWD)*

``` json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Cortex Debug",
      "type": "cortex-debug",
      "request": "launch",
      "servertype": "openocd",
      "executable": "build/blink.elf",
      "cwd": "${workspaceRoot}",
      "device": "RP2040",
      "configFiles": [
        "interface/picoprobe.cfg",
        "target/rp2040.cfg"
      ]
    }
  ]
}
```

------------------------------------------------------------------------

## 🧪 11. Testing with Serial Output

Use USB UART (CDC) via USB connection.

``` c
stdio_init_all();  // Enables printf over USB

printf("Hello from Pico!\n");
```

Use:

``` bash
screen /dev/ttyACM0 115200
```

or

``` bash
picocom -b 115200 /dev/ttyACM0
```

------------------------------------------------------------------------

## 🧠 12. Tips & Best Practices

| Tip | Description |
|----|----|
| Use `pico_add_extra_outputs()` | Builds `.uf2`, `.elf`, `.bin` for flashing |
| Don’t forget `pico_sdk_init()` | Required in top-level `CMakeLists.txt` |
| Use Git submodule for SDK | Keeps SDK in sync with your project |
| Debug with SWD or USB CDC logging | Debug via UART or SWD using `picoprobe` |
| Use `build/` for all CMake outputs | Keep source tree clean |

------------------------------------------------------------------------

## 📚 Further Reading

- [Raspberry Pi Pico SDK Docs](https://raspberrypi.github.io/pico-sdk-doxygen/)
- [Getting Started with Raspberry Pi Pico (PDF)](https://datasheets.raspberrypi.com/pico/getting-started-with-pico.pdf)
- [pico-examples GitHub](https://github.com/raspberrypi/pico-examples)

------------------------------------------------------------------------

## ✅ Summary

| Task                   | Command / Tool                     |
|------------------------|------------------------------------|
| Build                  | `cmake .. && ninja`                |
| Flash (USB)            | Drag `.uf2` to Pico drive          |
| Debug (optional)       | `picoprobe` + OpenOCD              |
| Serial Print Debugging | `stdio_init_all()`, then `picocom` |

------------------------------------------------------------------------

Back to top
