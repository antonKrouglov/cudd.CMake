# CMake config for CUDD library

CMake build config for [CUDD library](https://github.com/ivmai/cudd). 

This config is compatible with Visual Studio [Code] and Windows. 

## Command line

    md build
    cd build

    cmake ..
        ---OR with options set---
    cmake .. -DCUDD_BUILD_TESTS=ON -DCUDD_ENABLE_DDDMP=ON -DCUDD_ENABLE_OBJ=ON -DCUDD_ENABLE_SHARED=OFF
        ---OR building for VS 2022 ---
    cmake .. -G "Visual Studio 17 2022"
        ---OR release build---
    cmake .. -G "MinGW Makefiles" -DCMAKE_BUILD_TYPE:STRING=Release

    cmake --build .
    ctest --verbose
    cmake --install . --prefix install

## VSCode setup

Following [this tutorial](https://code.visualstudio.com/docs/cpp/config-mingw) I have used:
- VSCode Version: 1.82.0 (system setup)
- CMake Tools extension v1.15.31

You may edit `.vscode\settings.json` to change install prefix and other options.

My MSYS2 setup (some commands below may be redundant):

    pacman -Syu --noconfirm  #update all packages on the system # https://stackoverflow.com/a/64458777/2746150
    pacman -S base-devel
    pacman -S mingw-w64-x86_64-doxygen
    pacman -S automake-wrapper
    pacman -S --needed base-devel mingw-w64-x86_64-toolchain #https://code.visualstudio.com/docs/cpp/config-mingw
    pacman -S --noconfirm git wget tar gzip autoconf automake make libtool patch unzip xz bison flex pkg-config
    pacman -S --noconfirm mingw-w64-x86_64-gcc

## Visual Studio setup
Following [this tutorial](https://learn.microsoft.com/en-us/cpp/build/cmake-projects-in-visual-studio) I have used:
- Visual Studio 2022 (17.7.3), compilers v17.7.3 (x64), Windows SDK 10.0.22621.0

To adjust CMake settings in Visual Studio edit [CMakeSettings.json](CMakeSettings.json). You may want to change generator back to Ninja (default). `"cmakeCommandArgs": "--debug-trycompile",` is used to troubleshoot CMake's  macros(utility-modules) as per this [issue](https://github.com/conan-io/conan/issues/12012). 

Tests do not work properly then building from Visual Studio.

## Build options
These options inherit config options from  [Autotools-based build](README):

- **CUDD_ENABLE_DDDMP** - Build includes [libdddmp](dddmp/doc/dddmp-2.0-A4.ps). ON by default.
- **CUDD_ENABLE_OBJ** - Build includes C++ wrapper classes. ON by default.
- **CUDD_ENABLE_SHARED** - Build shared library (i.e. DLL for Windows). OFF by default.
- **CUDD_BUILD_TESTS** - Build and run tests. ON by default.

Tests and shared libarary options are mutually exclusive.

Either edit these options in [CMakeLists.txt](CMakeLists.txt) or specify at the [command line](#Command-line).


## Implementation details
Single [CMakeLists.txt](CMakeLists.txt) is used for the sake of simplicity.

Tested on Windows 10 Enterprise N LTSC 21H2 with compilers/toolchains:
- gcc 13.2.0 x86_64-w64-mingw32 (mingw64)
- Visual Studio 2022 (17.7.3), compilers v17.7.3 (x64), SDK 10.0.22621.0

CMake executables tested:
- cmake version 3.27.4 (mingw64-shipped)
- cmake version 3.26.4-msvc4 (VS2022-shipped)

This config has not been tested on native Linux, but should work there nonetheless. Anyhow you can fallback to [original autotools-based build config](README).

The following options were fully implemented in this build config: 
- option(CUDD_ENABLE_DDDMP "Include libdddmp in - libcudd" ON/OFF)
- option(CUDD_ENABLE_OBJ "Include C++ libobj in libcudd" ON/OFF)
- option(CUDD_ENABLE_SHARED "Build shared libraries" ON/OFF)

The following options were implemented partially: 
- option(CUDD_BUILD_TESTS "Build the tests" ON)

## Testing
Tests require static library option(CUDD_ENABLE_SHARED OFF).

Only one test have been adapted from autotools-based build config:
- `cplusplus/test_obj.test.in`

This test script may fail while displaying a good exit code (0). So please pay attention to the test text output.

TODO: `test_obj.test.in` sh script needs to be fixed to eliminate `sed` errors and to return correct exit codes.

Testing is set up as a postbuild action so it will run automatically after each build.

Testing on Windows requires `sh.exe` to be available at somewhere along system PATH or at C:\Program Files\Git\bin\ (git Bash).

## Installing
CMake `install` option has been implemented as well. Target install path can be specified in settings or by command line switch `--prefix <install_path>`. If not specified explicitly <install_path> defaults to `Program Files\CUDD\` on Windows or `/usr/` on Linux. Shared library goes to `bin` subfolder. Static library goes to `lib` subfolder. Headers go to `include` subfolder.


## Further reading
- [Binary decision diagram](https://en.wikipedia.org/wiki/Binary_decision_diagram) | Wikipedia

- CUDD [tutorial](https://davidkebo.com/cudd) by David Kebo Houngninou.

- CUDD [manual](https://www.cs.rice.edu/~lm30/RSynth/CUDD/cudd/doc/cuddIntro.html) | html. PDF version of this manual can be build from this repository by `make`. You need pdflatex and makeindex installed.

- Donald Knuth | 7.1.4. Binary Decision Diagrams | [Art of Computer Programming, Volume 4A](https://books.google.com/books?id=IkuEBAAAQBAJ). Introduction [video](https://youtu.be/SQE21efsf7Y).

- [Binary Decision Diagrams](https://www.ecs.umass.edu/ece/labs/vlsicad/ece667/reading/somenzi99bdd.pdf) | April 2000 | Fabio Somenzi

- [Graph-Based Algorithms for Boolean Function Manipulation](https://ieeexplore.ieee.org/document/1676819) | Bryant 1986[.](https://www.cs.cmu.edu/~bryant/pubdir/ieeetc86.pdf)

- Complete Binary Decision Diagrams tools reference is available at [johnyf's github](https://github.com/johnyf/tool_lists/blob/main/bdd.md).

## Credits
CUDD library author is Fabio Somenzi, who wrote it more than 20 years ago. Still it is the best library available.

DDDMP library is created by Giampiero Cabodi and Stefano Quer.

This repository has been forked from the [Ivan Maidanski's github](https://github.com/ivmai/cudd) who kindly provides space for CUDD library.

CMake configuration draft has been taken from [Olzhas Rakhimov's github](https://github.com/rakhimov/cudd/tree/cmake).





