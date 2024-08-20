# Build C++ app with libhttpserver library for RPi

## Follow guide [Mouting RPi img for Cross Compile](../raspberrypi/mouting-rpi-image-img.md)

## Follow guide [Compile libhttpserver using rpi-sysroot](../libraries/libhttpserver-rpi-cross-compile.md)

## Follow guide [Cross compile C++ app for RPi](../raspberrypi/rpi-cpp-app-cross-compile.md)

## Modify project files to include libhttpserver

**`main.cpp`**
```
#include <iostream>
#include <httpserver.hpp>

using namespace httpserver;

class helloworld_resource : public http_resource {
public:
    std::shared_ptr<http_response> render(const http_request&) {
        return std::shared_ptr<http_response>(new string_response("Hello RPi!", 200, "text/html"));
    }
};

int main(int argc, char** argv)
{
    std::cout << "Hello RPi!" << std::endl;

    webserver ws = create_webserver(8081);

    helloworld_resource hwr;
    ws.register_resource("/hello", &hwr);
    ws.start(true);

    return 0;
}

```

**`CMakeLists.txt`**
```
cmake_minimum_required(VERSION 3.22)

# Set the project name and specify the C++ standard
project(cpp_httpserver VERSION 1.0 LANGUAGES CXX)

# Set the C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# Add source files
set(SOURCES
    src/main.cpp
)

# Add the executable target
add_executable(${PROJECT_NAME} ${SOURCES})

# Link against libhttpserver
target_link_libraries(${PROJECT_NAME} ${HTTP_SERVER_LIB})
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

set(RPI_SYSROOT /mnt/rpi-sysroot)

# Specify the location of the target sysroot (optional)
set(CMAKE_SYSROOT ${RPI_SYSROOT})

# Specify the location of the C and C++ standard libraries (optional)
set(CMAKE_FIND_ROOT_PATH ${CMAKE_SYSROOT})

# Specify the include directories and library paths
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} --sysroot=${CMAKE_SYSROOT}")
include_directories(${CMAKE_SYSROOT}/usr/include ${CMAKE_SYSROOT}/usr/local/include)
link_directories(${CMAKE_SYSROOT}/usr/lib ${CMAKE_SYSROOT}/usr/local/lib)

# Set the rpath to ensure the executable can find the libraries at runtime
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
set(CMAKE_BUILD_WITH_INSTALL_RPATH TRUE)
set(CMAKE_INSTALL_RPATH "/usr/lib;/usr/local/lib")

# Add the httpserver library
set(HTTP_SERVER_LIB httpserver)

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
                "${workspaceFolder}/**",
                "/mnt/rpi-sysroot/usr/include",
                "/mnt/rpi-sysroot/usr/local/include",
                "/mnt/rpi-sysroot/usr/local/include/httpserver"
            ],
            "defines": [],
            "compilerPath": "/usr/bin/aarch64-linux-gnu-g++",
            "cStandard": "c11",
            "cppStandard": "c++17",
            "intelliSenseMode": "linux-gcc-arm64"
        }
    ],
    "version": 4
}
```

## Build project

```
cmake .. -DCMAKE_TOOLCHAIN_FILE=../toolchain.cmake
make
```

## Copy executable to RPi via scp

```
scp cpp_httpserver pi@raspberrypi.local:/home/pi
```

Run app and access browser on `raspberrypi.local:8081/hello`