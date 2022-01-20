# cairoxp
This repository provides a minimal example of rendering 2D panel graphics (Cairo) on a background thread inside an X-Plane plugin. A bit of an explanation is provided below, as well as a guide to getting the plugin built and running inside the simulator. This is not meant for absolute beginners--there are many things here that are not explained and/or error prone.

## Screenshot inside the default C172

![Preview](https://github.com/aeroc7/cairoxp/blob/main/screenshots/panel.png)

## Screenshot of the performance monitor
Due to the rendering occuring on a background thread, separate from X-Plane, there is minimal impact on the simulator performance itself, leaving a smooth experience for the end user. The main drawback of this approach is the lack of flexibility in interacting with X-Plane. X-Plane SDK functions can **only** be called from the main X-Plane thread, AKA the thread that `XPluginStart` and `XPluginStop` get called on. Failing to adhere to this could lead to undefined behavior, and possibly even X-Plane crashing. This means that in order to get data from the simulator, you'd have to setup your own structure of variables protected with some sort of synchronization between your reads and writes, so that you're not calling X-Plane SDK functions from anywhere but the main thread.

![Performance](https://github.com/aeroc7/cairoxp/blob/main/screenshots/performance.png)

## How it works
[cairo_mt.cpp](https://github.com/aeroc7/cairoxp/blob/main/src/cairo_mt.cpp) provides a wrapper around all of the operations needed in the background. This includes:
- Managing the render thread that calls the callbacks you provide (start, draw loop, stop)
- Managing the X-Plane draw callback for the gauges phase
- Managing the Cairo surface
- Managing the [PBO](https://www.khronos.org/opengl/wiki/Pixel_Buffer_Object) for asynchronous transfers of the Cairo pixel data, to the OpenGL texture in the X-Plane draw callback

The rest is very straightforward. Once you've initialized GLEW, you can create an instance of the CairoMt class, where you provide `std::function<void(cairo_t *cr)>` callbacks for start, loop, and stop. There is also another parameter where you provide a structure `PanelDims`, which contain the dimensions of your desired texture, and the x, y offset of the texture on X-Plane's panel.png. An example of this is provided in [draw.cpp](https://github.com/aeroc7/cairoxp/blob/main/src/draw.cpp)

# Building
Firstly, you need to clone the repository: `git clone https://github.com/aeroc7/cairoxp.git`

## Windows (using MinGW on [MSYS2](https://www.msys2.org/))
You require the following dependencies:
```
pacman -S mingw-w64-x86_64-toolchain
pacman -S mingw64/mingw-w64-x86_64-cmake
pacman -S mingw64/mingw-w64-x86_64-make
pacman -S mingw64/mingw-w64-x86_64-meson
pacman -S mingw64/mingw-w64-x86_64-python3
pacman -S mingw64/mingw-w64-x86_64-python3-setuptools
pacman -S mingw64/mingw-w64-x86_64-python3-pip
pacman -S msys/make
pacman -S msys/patch
pacman -S msys/pkg-config
```
- `cd` into the cloned folder (ex. `cd cairoxp`)
- Create a build directory `mkdir build && cd build`
- Generate a release build with CMake `cmake -G "Unix Makefiles" -DCMAKE_BUILD_TYPE=Release ..`
  - You may also want to set an install location with `-DCMAKE_INSTALL_PREFIX=your/install/directory` (The proper X-Plane plugin folder structure will be generated automatically as well)
- Start the build process with `make -j[num_cpus + 1]`
  - Replace the `[num_cpus + 1]` with the number of cpus your processor has + 1
- Install with `make install`

## Linux (tested on Ubuntu 21.10)
You require the following dependencies:
`sudo apt-get install build-essential cmake meson libglu1-mesa-dev mesa-common-dev python3 python3-pip python3-setuptools`

- `cd` into the cloned folder (ex. `cd cairoxp`)
- Create a build directory `mkdir build && cd build`
- Generate a release build with CMake `cmake -DCMAKE_BUILD_TYPE=Release ..`
  - You may also want to set an install location with `-DCMAKE_INSTALL_PREFIX=your/install/directory` (The proper X-Plane plugin folder structure will be generated automatically as well)
- Start the build process with `make -j[num_cpus + 1]`
  - Replace the `[num_cpus + 1]` with the number of cpus your processor has + 1 (`cat /proc/cpuinfo | grep "siblings"`)
- Install with `make install`

