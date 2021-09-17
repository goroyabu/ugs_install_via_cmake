# ugs_install_via_cmake

This is a unofficial installer of SLAC Unified Graphics System (UGS).
Details of UGS are described at http://ftp.riken.jp/pub/iris/ugs.

This installer can work for Mac with macOS Big Sur and Apple M1.

UGS installed by this installer can be found by CMake command `find_package`. 

Here, an example to use UGS in other project,

```
find_package(ugs REQUIRED)
find_package(X11 REQUIRED)

# ~~~~~~~~~~ #

target_link_libraries(
    ${SOME_TARGET} PRIVATE
    ugs ${X11_LIBRARIES} ${X11_Xaw_LIB} ${X11_Xmu_LIB} ${X11_Xt_LIB} )
```

--------------------

## 1. Requirement

- CMake >= 3.15
- Fortran Compiler
- X11 and its Athena Widget (Xaw)

## 2. Installation

```
mkdir /path/to/build/ && cd /path/to/build/
cmake /path/to/source && make -j && make install
```

### 2.1 xwtest 

A command `xwtest` to check the UGS is also installed to `/path/to/install/bin/xwtest`.

### 2.2 Uninstallation

```
cd /path/to/build/
make uninstall
```