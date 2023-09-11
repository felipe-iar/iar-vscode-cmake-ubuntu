# IAR + VSCode + CMake on Ubuntu

This mini-guide provides the essentials for quickly setting up a "hello world" project for an Arm Cortex-M4 target built with the IAR Build Tools (BX) using CMake on Visual Studio Code on an Ubuntu Desktop. In the end we will debug it using a J-Link and the GNU Debugger.

> __Warning__ 
> The information provided in this repository is subject to change without notice and does not represent a commitment on any part of IAR. While the information contained herein might be useful as reference, IAR assumes no responsibility for any errors or omissions.


## Required software
Below you will find the software and their versions used in this guide. Newer versions might work with little or no modification(s).

| Software | Version | Link |
| - | - | - |
| Ubuntu Desktop | 22.04.3 | [link](https://ubuntu.com/download/desktop/thank-you?version=22.04.3&architecture=amd64)
| IAR Build Tools for Arm | 9.40.1 | [link](https://iar.com/bxarm)
| Microsoft Visual Studio Code | 1.82.0 | [link](https://code.visualstudio.com)
| Microsoft CMake Tools Extension | 1.15.31 | [link](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools)
| Microsoft C/C++ Extension Pack | 1.3.0 | [link](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools-extension-pack)
| Cortex-Debug Extension | 1.12.0 | [link](https://marketplace.visualstudio.com/items?itemName=marus25.cortex-debug)
| CMake | 3.22.1 | [link](https://cmake.org)
| Ninja | 1.10.1 | [link](https://ninja-build.org)
| J-Link drivers | 7.92d | [link](https://www.segger.com/downloads/jlink/JLink_Linux_x86_64.deb)
| GNU Debugger | 12.3.rel1 | [Instructions](#debugging-the-project)

## Procedure

### Installing the software
Install the required software:
- Install the IAR Build Tools: `sudo apt install /path/to/bxarm-*.deb` (setup the [license](https://iar.com/siteassets/knowledge/support/tech-notes/ug_lms2_cli.pdf))
- Install CMake: `sudo apt install cmake`
- Install Ninja: `sudo apt install ninja-build`
- Install J-Link drivers: `sudo apt install /path/to/JLink*.deb`
- Install VSCode: `sudo apt install /path/to/vscode-*.deb` and launch it `code`
- Install the VSCode extensions (<kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>X</kbd>):
   - [CMake Tools extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools)
   - [Cortex-Debug](https://marketplace.visualstudio.com/items?itemName=marus25.cortex-debug)

### Creating the source file(s)
- In Visual Studio Code, select: `File` → `Open Folder...` (<kbd>Ctrl</kbd>+<kbd>K</kbd>, <kbd>Ctrl</kbd>+<kbd>O</kbd>).
> __Note__
> VSCode will ask you if you _trust the authors_ of the files in the opened folder.

- Create a new empty folder. (e.g., "`hello`"). This folder will be referred to as `<proj-dir>` from now on.
  
- Create a `<proj-dir>/main.c` source file with the following content:
```c
#include <stdbool.h>
#include <stdio.h>
#include <stdint.h>

__attribute__((used)) uint_fast8_t counter = 0;

void main() {
   while (counter < 10) {
     printf("Hello world! %u\n", counter);
   }
   while (true) {
      ++counter;
   }
}
```

### Creating the CMakeLists
- Create a `<proj-dir>/CMakeLists.txt` file with the following content:
```cmake
cmake_minimum_required(VERSION 3.22)

# Set the project name and its required languages (ASM, C and/or CXX)
project(example LANGUAGES C)

# Add an executable named "hello"
add_executable(hello)

# Add "hello" source file(s)
target_sources(hello PRIVATE main.c)

# Set the compiler options for "hello"
target_compile_options(hello PRIVATE
  --cpu Cortex-M4
  --dlib_config normal
  --debug
  -On
  -e
)

# Set the linker options for "hello"
target_link_options(hello PRIVATE
  --semihosting
  --map .
  --config "${TOOLKIT_DIR}/config/linker/ST/stm32f407xG.icf"
)
```

> __Note__
> For more information on using CMake with the IAR compiler, refer to the [cmake-tutorial](https://github.com/iarsystems/cmake-tutorial).

### Configuring the CMake Tools extension
* Create a `<proj-dir>/.vscode/cmake-kits.json` file with the following content (adjusting the paths as needed):
```json
[
  {
    "name": "IAR BXARM",
    "compilers": {
      "C": "/opt/iarsystems/bxarm/arm/bin/iccarm"
    },
    "cmakeSettings": {
      "CMAKE_BUILD_TYPE": "Debug",
      "TOOLKIT_DIR": "/opt/iarsystems/bxarm/arm"
    }
  }
]
```
> __Note__
> * CMake Tools will prefer [Ninja](https://ninja-build.org/) if it is present, unless configured otherwise.
> * If you change the active kit while a project is configured, the project configuration will be re-generated with the chosen kit.

### Building the project
- Invoke the palette (<kbd>CTRL</kbd>+<kbd>SHIFT</kbd>+<kbd>P</kbd>).
   - Perform __CMake: Configure__.
   - Select __IAR BXARM__ from the drop-down list.
- Invoke the palette 
   - Perform __CMake: Build__.

__Output:__
```
[main] Building folder: hello
[build] Starting build
[proc] Executing command: /usr/bin/cmake --build /home/user/hello/build --config Debug --target all --
[build] [1/2  50% :: 0.070] Building C object CMakeFiles/hello.dir/main.c.o
[build] [2/2 100% :: 0.144] Linking C executable hello.elf
[driver] Build completed: 00:00:00.206
[build] Build finished with exit code 0
```

Happy building!


## Enabling Intellisense for the IAR C/C++ Compiler for Arm in Visual Studio Code 
In order to get accurate results, Intellisense needs: 
- the compiler's internal macros
- the compiler's keywords

For consistency, we will create two header files containing such information.
### `~/.config/Code/IAR/iccarm_predef.h`
This header file can be generated with the following command:
```
mkdir ~/.config/Code/IAR
/opt/iarsystems/bxarm/arm/bin/iccarm --NCG . --c++ --predef_macros ~/.config/Code/IAR/iccarm_predef.h
```

### `~/.config/Code/IAR/iccarm_keywords.h`
Create this header file with the following contents:
```cpp
#define __absolute
#define __arm
#define __big_endian 
#define __cmse_nonsecure_call
#define __cmse_nonsecure_entry
#define __exception
#define __fiq
#define __interwork
#define __intrinsic
#define __irq
#define __little_endian
#define __naked
#define __no_alloc
#define __no_alloc16
#define __no_alloc_str
#define __no_alloc_str16
#define __nested
#define __no_init
#define __noreturn
#define __nounwind
#define __packed
#define __pcrel
#define __ramfunc
#define __root
#define __ro_placement
#define __sbrel
#define __stackless
#define __svc
#define __swi
#define __task
#define __thumb
#define __weak
```
> __Note__ Keywords available as per IAR C/C++ Compiler for Arm 9.40.1

### `~/.config/Code/IAR/settings.json`
The following JSON file configures Microsoft Intellisense to work with the IAR C/C++ Compiler for Arm:
```json
{
    // Intellisense settings
    "C_Cpp.default.compilerPath": "/opt/iarsystems/bxarm/arm/bin/iccarm",
    "C_Cpp.default.cStandard": "c17",
    "C_Cpp.default.cppStandard": "c++17",
    "C_Cpp.default.intelliSenseMode": "clang-arm",
    "C_Cpp.default.configurationProvider": "ms-vscode.cmake-tools",
    "C_Cpp.default.mergeConfigurations": true,
    "C_Cpp.default.includePath": [
        "/opt/iarsystems/bxarm/arm/inc/c",
        "/opt/iarsystems/bxarm/arm/inc/c/aarch32"
    ],
    "C_Cpp.default.forcedInclude": [
        "~/.config/Code/IAR/iccarm_predef.h",
        "~/.config/Code/IAR/iccarm_keywords.h"
    ],
    // CMake settings
    "cmake.enabledOutputParsers": ["cmake", "iar"]
}
```

## Debugging the project
In this final section, it is time to setup the GNU Debugger from Arm. For Ubuntu 22.04, the following dependencies must be satisfied:
* the `libncursesw5` package
* Python 3.8 (available from a 3<sup>rd</sup> party [PPA](https://askubuntu.com/questions/1398568/installing-python-who-is-deadsnakes-and-why-should-i-trust-them)).

### Dependencies
```bash
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt install libncursesw5 python3.8
```

### GNU Debugger (Arm)
```bash
cd ~
wget https://developer.arm.com/-/media/Files/downloads/gnu/12.3.rel1/binrel/arm-gnu-toolchain-12.3.rel1-x86_64-arm-none-eabi.tar.xz
cd /opt
sudo tar xJvf ~/arm-gnu-toolchain-12.3.rel1-x86_64-arm-none-eabi.tar.xz
```

Now we need to setup the launch configuration at `<proj-dir>/.vscode/launch.json`:
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Cortex Debug",
      "type": "cortex-debug",
      "request": "launch",
      "cwd": "${workspaceRoot}",
      "executable": "${command:cmake.launchTargetPath}",
      "device": "stm32f407vg",
      "svdFile": "${workspaceRoot}/stm32-svd/f4/STM32F407.svd",
      "interface": "swd",
      "servertype": "jlink",
      "serverpath": "/opt/SEGGER/JLink/JLinkGDBServerCLExe",
      "armToolchainPath": "/opt/arm-gnu-toolchain-12.3.rel1-x86_64-arm-none-eabi/bin",
      "breakAfterReset": true,
      "runToEntryPoint": "main"
    },
  ]
}
```
> __Note__
> [This](https://github.com/svcguy/stm32-svd) repository provides a collection of `*.svd` files for STM32 devices. SVD files are special XML files used by the `cortex-debug` extension to identify the registers on a microcontroller.

Once the `<proj-dir>/build/hello.elf` executable was built:
- In `main.c`, click on the left curb of the line which increments the counter (`++counter;`) to set a breakpoint.
- Go to __Run__ → __Start Debugging__ (<kbd>F5</kbd>) to start the debugging session.

![VirtualBox_bx-vsc-cmake_11_09_2023_17_50_00](https://github.com/felipe-iar/iar-vscode-cmake-ubuntu/assets/54443595/d1fd583d-5a7c-4e77-92f1-1accb0840a68)

Happy debugging!
