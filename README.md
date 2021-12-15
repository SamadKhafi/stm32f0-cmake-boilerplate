# STM32F0 CMake Boilerplate

## Required tools
- make
- cmake
- arm-none-eabi-gcc
- STM32CubeMX
- A Code Editor (I recommend VS Code + C/C++ Extension)

#

## Project Setup
- Clone this repository
- Create STM32CubeMX project in the same directory
- Use STM32CubeIDE as Toolchain / IDE in STM32CubeMX > Project Manager
- Generate code

#

## Build
### Prepare
*NOTE: Supported CMAKE_BUILD_TYPE value is **Debug** & **Release***
```sh
cmake -DCMAKE_TOOLCHAIN_FILE=arm-none-eabi-gcc.cmake -DCMAKE_BUILD_TYPE=Debug -S . -B ./build
```
### Compile
```sh
cmake --build ./build -- -j4
```
### Output
```
./build/firmware.elf
./build/firmware.hex
./build/firmware.bin
```
