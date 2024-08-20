# Cross compile C++ app for RPi

## Follow guide [Mouting RPi img for Cross Compile](../raspberrypi/mouting-rpi-image-img.md)

## Create vscode project with simple C++ HelloWorld app

Project structure:

```
cpp_helloworld/
├── .vscode
│   └── c_cpp_properties.json
├── build
├── CMakeLists.txt
├── src
│   └── main.cpp
└── toolchain.cmake
```

**`main.cpp`**
```
#include <iostream>

int main(int argc, char const *argv[])
{
    std::cout << "Hello RPi!" << std::endl;
    return 0;
}
```

**`CMakeLists.txt`**
```
cmake_minimum_required(VERSION 3.22)

# Set the project name and specify the C++ standard
project(helloworld VERSION 1.0 LANGUAGES CXX)

# Set the C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# Add source files
set(SOURCES
    src/main.cpp
)

# Add the executable target
add_executable(${PROJECT_NAME} ${SOURCES})
```
**`toolchain.cmake`**
```
# Define the system and toolchain
set(CMAKE_SYSTEM_NAME Linux)
set(CMAKE_SYSTEM_PROCESSOR arm)

set(RPI_GCC_TRIPLE "aarch64-linux-gnu")

# Specify the cross compiler
set(CMAKE_C_COMPILER ${RPI_GCC_TRIPLE}-gcc)
set(CMAKE_CXX_COMPILER ${RPI_GCC_TRIPLE}-g++)

set(RPI_SYSROOT $ENV{HOME}/raspberry/rpi-sysroot)

# Specify the location of the target sysroot (optional)
set(CMAKE_SYSROOT ${RPI_SYSROOT})

# Specify the location of the C and C++ standard libraries (optional)
set(CMAKE_FIND_ROOT_PATH ${RPI_SYSROOT})

# Ensure that only the files inside the sysroot are used
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_PACKAGE ONLY)
```

**`c_cpp_properties.json`**
```
{
    "configurations": [
        {
            "name": "Linux",
            "includePath": [
                "${workspaceFolder}/**"
            ],
            "defines": [],
            "compilerPath": "/usr/bin/aarch64-linux-gnu-gcc",
            "cStandard": "c11",
            "cppStandard": "c++17",
            "intelliSenseMode": "linux-gcc-arm64"
        }
    ],
    "version": 4
}
```

## Run cmake and then make in the build folder

```
cd build
cmake .. -DCMAKE_TOOLCHAIN_FILE=../toolchain.cmake
make
```

## Copy helloworld executable to RPi using scp

```
scp helloworld pi@raspberrypi.local:/home/pi
```
