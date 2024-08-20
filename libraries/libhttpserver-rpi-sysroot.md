# Compile libhttpserver using rpi-sysroot

## Follow guide [Mouting RPi img for Cross Compile](../raspberrypi/mouting-rpi-image-img.md)

## Inside rpi-sysroot install git and libmicrohttpd-dev

```
apt install git libmicrohttpd-dev
```
## Clone libhttpserver library

```
cd /home/pi

git clone --branch 0.19.0 --single-branch https://github.com/etr/libhttpserver.git
```

## Follow instructions for libhttpserver compilation

```
./bootstrap
mkdir build
cd build
../configure
make
make install # (optionally to install on the system)
```

Now you have a rpi-sysroot with libhttpserver library for cross development