# Usage
## Ninja build
`cmake -GNinja  -DCMAKE_TOOLCHAIN_FILE=../arm-none-eabi-gcc.cmake -DCMAKE_BUILD_TYPE=Debug ..`

`ninja`

## Make build
`cmake  -DCMAKE_TOOLCHAIN_FILE=../arm-none-eabi-gcc.cmake -DCMAKE_BUILD_TYPE=Debug ..`

`make`

## Generate Eclipse project

Note: Generate from the root directory (This will include the source files)

`cmake -G "Eclipse CDT4 - Ninja" -DCMAKE_ECLIPSE_VERSION=4.11.0 -DCMAKE_TOOLCHAIN_FILE=./arm-none-eabi-gcc.cmake -DCMAKE_BUILD_TYPE=Debug ./`

See the folder ScreenShots as a reference for adding a Debugger. This projected was tested with ST-LINKv2, arm-none-eabi-gdb, and openOCD.


Application Source: https://deepbluembedded.com/stm32-pwm-example-timer-pwm-mode-tutorial/

