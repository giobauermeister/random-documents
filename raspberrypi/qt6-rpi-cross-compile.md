## Qt6 Cross Compile for RPi

Ubuntu and GCC versions

```
Linux DESKDEV-ITG 6.5.0-44-generic #44~22.04.1-Ubuntu SMP PREEMPT_DYNAMIC Tue Jun 18 14:36:16 UTC 2 x86_64 x86_64 x86_64 GNU/Linux
```

```
ldd --version

ldd (Ubuntu GLIBC 2.35-0ubuntu3.8) 2.35
Copyright (C) 2022 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
Written by Roland McGrath and Ulrich Drepper.
```

```
aarch64-linux-gnu-gcc-11 --version

aarch64-linux-gnu-gcc-11 (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0
Copyright (C) 2021 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

```
aarch64-linux-gnu-ld --version

GNU ld (GNU Binutils for Ubuntu) 2.38
Copyright (C) 2022 Free Software Foundation, Inc.
This program is free software; you may redistribute it under the terms of
the GNU General Public License version 3 or (at your option) a later version.
This program has absolutely no warranty.
```

## Build Qt6 on local host machine

```
mkdir $HOME/raspberry/qt
cd $HOME/raspberry/qt
git clone "https://codereview.qt-project.org/qt/qt5"
cd qt5/
git checkout 6.5.3
perl init-repository -f
cd ..
mkdir $HOME/raspberry/qt/qt-hostbuild
cd $HOME/raspberry/qt/qt-hostbuild/
cmake ../qt5/ -GNinja -DCMAKE_BUILD_TYPE=Release -DQT_BUILD_EXAMPLES=OFF -DQT_BUILD_TESTS=OFF -DCMAKE_INSTALL_PREFIX=$HOME/raspberry/qt/qt-host
cmake --build . --parallel 8
cmake --install .
```
## Configure command when building for RPi

```
cd qt-pibuild

../qt5/configure \
-release \
-opengl es2 \
-nomake examples \
-nomake tests \
-skip qtwayland \
-skip qtwebengine \
-qt-host-path $HOME/raspberry/qt/qt-host \
-extprefix $HOME/raspberry/qt/qt-raspi \
-prefix /usr/local/qt6 \
-device linux-rasp-pi4-aarch64 \
-device-option CROSS_COMPILE=aarch64-linux-gnu- \
-- \
-DCMAKE_TOOLCHAIN_FILE=$HOME/raspberry/qt/mytoolchain.cmake \
-DQT_FEATURE_kms=ON \
-DQT_FEATURE_opengles2=ON \
-DQT_FEATURE_opengles3=ON \
-DQT_FEATURE_vulkan=ON \
-DQT_FEATURE_xcb=OFF \
-DFEATURE_xcb_xlib=OFF \
-DQT_FEATURE_xlib=OFF \
-DFEATURE_dbus=OFF
```

## Copy Qt6 to rasp after build

```
rsync -avz --rsync-path="sudo rsync" qt-raspi/* pi@rpigio.local:/usr/local/qt6
```

## Change RPi config file

```
sudo vi /boot/firmware/config.txt
```
```
# Automatically load overlays for detected DSI displays
#display_auto_detect=1

# Enable DRM VC4 V3D driver
#dtoverlay=vc4-kms-v3d
dtoverlay=vc4-fkms-v3d
max_framebuffers=2
```

## Install packages on RPi
sudo apt install install xinput-calibrator evtest libts-bin

## Add touchscreen rules if needs rotation

```
sudo vi /etc/udev/rules.d/99-touchscreen.rules

ENV{DEVNAME}=="*/input/event0", ENV{LIBINPUT_CALIBRATION_MATRIX}="1 0 0 0 1 0 0 0 1"
```
|           Rotation        |                  Matrix        |
|---------------------------|--------------------------------|
|0°	                                   | 1 0 0 0 1 0 0 0 1   |
|90° Clockwise	                       |  0 -1 1 1 0 0 0 0 1 |
|90° Counter-Clockwise                 |  0 1 0 -1 0 1 0 0 1 |
|180° (Inverts X and Y)                | -1 0 1 0 -1 1 0 0 1 |
|invert Y                              | -1 0 1 1 1 0 0 0 1  |
|invert X                              | -1 0 1 0 1 0 0 0 1  |
|expand to twice the size horizontally | 0.5 0 0 0 1 0 0 0 1 |

## Calibrate screen

```
sudo TSLIB_FBDEVICE=/dev/fb0 TSLIB_TSDEVICE=/dev/input/event0 ts_calibrate
```

## Run Qt app

```
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/qt6/lib/
./calculator -platform eglfs
```