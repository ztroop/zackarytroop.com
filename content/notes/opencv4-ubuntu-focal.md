+++
title = "OpenCV4 in Ubuntu Focal 20.04"
slug = "opencv4-ubuntu-focal"
date = 2021-03-10
+++

The `opencv` version available in Ubuntu 20.04 is too old! We need a newer version!

We'll install from source. First, we need to install some dependencies:

```sh
sudo apt install build-essential cmake git pkg-config libgtk-3-dev \
    libavcodec-dev libavformat-dev libswscale-dev libv4l-dev \
    libxvidcore-dev libx264-dev libjpeg-dev libpng-dev libtiff-dev \
    gfortran openexr libatlas-base-dev python3-dev python3-numpy \
    libtbb2 libtbb-dev libdc1394-22-dev
```

Clone the OpenCV’s and OpenCV contrib repositories:

```sh
mkdir ~/opencv_build && cd ~/opencv_build
git clone https://github.com/opencv/opencv.git
git clone https://github.com/opencv/opencv_contrib.git
```

At the time of writing, the current version in the repositories is version 4.5.2.

If you want to install an older version of OpenCV, `cd` to both `opencv` and `opencv_contrib` directories and run `git checkout <opencv-version>`.

```sh
cd ~/opencv_build/opencv
mkdir build && cd build
```

Set up the OpenCV build with CMake:

```sh
cmake -D CMAKE_BUILD_TYPE=RELEASE \
    -D CMAKE_INSTALL_PREFIX=/usr/local \
    -D INSTALL_C_EXAMPLES=ON \
    -D INSTALL_PYTHON_EXAMPLES=ON \
    -D OPENCV_GENERATE_PKGCONFIG=ON \
    -D OPENCV_EXTRA_MODULES_PATH=~/opencv_build/opencv_contrib/modules \
    -D BUILD_EXAMPLES=ON ..
```

Start the compilation process:

```sh
make -j8
```

Modify the `-j` flag according to the amount of CPU cores in your system.

To verify whether OpenCV has been installed successfully, run `pkg-config`:

```sh
pkg-config --modversion opencv4
```

It's fortunate that this process isn't much different from [18.04](https://linuxize.com/post/how-to-install-opencv-on-ubuntu-18-04/) instructions. 😉
