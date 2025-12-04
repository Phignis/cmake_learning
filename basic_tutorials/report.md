# Report

This report provides details about CMake tutorials in step by step. Please find all source files of each step in their subdirectories.

## Step 0

### A bit of context

CMake is a configuration program. It is therefore not in itself a build system such as Make or Ninja, rather a configurator describing dependancies and how to build, delegating the proper build steps to the selected build system.

Therefore, CMake is sometimes called a "meta build system", as it is a unique interface abstracting the choosen build system. But CMake core responsibility and principle is to build a proper build system for the project with a given environment and eventual user configuration details, and not actually building the project.

CMake is mainly used to manage builds for C and C++ projects, but may be used for other languages. This tutorial setting is different C++ projects.

### Installing CMake

Packages from the distribution default mirrors may not have the latest release. Therefore, rather than using `apt` on Debian, I built the CMake system from the source repository. Please refer to the following command to see the installation process.

```sh
$ wget https://github.com/Kitware/CMake/releases/download/v4.2.0/cmake-4.2.0.tar.gz && tar -xzf cmake-4.2.0.tar.gz
$ cd cmake-4.2.0.tar.gz && ./bootstrap && make && sudo make install && hash -r && cd .. && rm cmake-4.2.0.tar.gz
```

`hash -r` was necessary, as I had installed cmake from apt before. When invoking `cmake`, it was seeking in the latest known location `/usr/bin/cmake` while the new installation is in `/usr/local/bin/cmake`.

### Getting the tutorials

To get the tutorials, an archive is provided in the zip format. `unzip` was used to extract the tutorials.

```sh
$ wget https://cmake.org/cmake/help/v4.2/_downloads/a286de3ce1de92f369d0043181e214ac/cmake-4.2.0-tutorial-source.zip && unzip cmake-4.2.0-tutorial-source.zip && rm cmake-4.2.0-tutorial-source.zip
```

### First building test

Let's try building for the first time. I only have Make as an available build system (generator for CMake). Moving to the Step0 subfolder, you notice a C++ source file (.cxx) and a `CMakeLists.txt`. The source file is quite simple (hello word) ; the second file describes how to create the build system. 

`CMAKE\_GENERATOR` is not defined. Therefore the generator (build system) will be automatically chose between all available (so Make for now, as it is the only available). The file defines a minimal CMake version, as well as an project and an executable being part of this project. The executable "hello" is the name of the resulting executable once the build system created by CMake will be ran. Source files to compile when creating the executable are done with `target_sources(executable_name [visibility] source_file)`.

By running `cmake -B build`, we define a specific build directory, the build system will therefore be located in `./Step0/build/`. Once builded, we have in this build directory a Makefile, containing all rules to compile the project. By running `make` in the build folder, we obtain the executable to be run.

As evoked above in section `A bit of context`, the CMake command only generate the build system, not the executable in itself. The point of CMake is not to be another build system, but to generate this build system. The generated MakeFile is a very detailed and complete version, even for a simple CMake file. CMake allows the developer to focus on the project, describe it directly, rather than creating a lot of rules for each functionnalities in Make.

### Specify a generator

By default, Make was used as the build system to generate (generator). However, you can specify other generators either by adding "CMAKE\_GENERATOR=Ninja" in the CMakeLists.txt, or `cmake -G Ninja -B build`.
Because we reuse the same build directory, you need first to remove it. Indeed, CMake builds are **incremental**, therefore there is a conflict with the already existing make build system.

Building again with Ninja, you now obtain at the root of the build directory a build.ninja instead of the MakeFile previously with the default generator.

**We only used a mono-configuration generator**. The specific type of release (Debug, Release...) can be set in the CMakeLists.txt via the `CMAKE_BUILD_TYPE` variable. However, multi-configuration generators also exists, to output multiple level of release from one CMake based on configuration at build time. If a multi-configuration generator is in use, you need to add `--config <release_level>` to the previous cmake commands.

Running in `Step0` the following commands create both build system in proper subfolders :
```sh
$ cmake -G 'Unix Makefiles' -B build/make ; cmake -G Ninja -B build/ninja
```

## Step 1

One or more `CMakeLists.txt` can be used to describe and create the build system. Those files can also be refered as `CML` or `lists files`.

A root CML is oftenly used, used as the entry point for other CML. It usually contains the guaranteed behaviour with `cmake_minimum_required(VERSION 4.2)`. It ensures CMake will behave as the wanted version. The command `project()` inform CMake you are building a complete software, with a proper name, and eventual versions, languages...

Official CMake guidance recommand using lower-case for commands, opposite of SQL prefering upper-case.

### Exercise 1 - Building an executable

As said in step 0, `add_executable()` allows to create a target the created build system will have to compile into. This target may have dependancies, header and source files, but also compiler and linker flags. `target_sources()` allows to include source files to the executable. The visibility seen in step 0 (private) is the scope of the files qualified (in this case). PRIVATE means those files (and more generally any qualified property) is only used for building the target ; but not available when using it. INTERFACE means it is not used to build the target, but is available when using it. PUBLIC are both for building and available when using.

When declaring the project, it is possible to declare the language, being either C, C++, C#, Obj-C, Obj-C++, Fortran, Swift, CUDA and various ASM for the most notable ones.

When building the build system with `cmake -B build`, rather than invoking the specific build command (make in my case), you can also invoke `cmake --build build`. This will invoke the proper build command, effectively and fully abstracting out the build system.

### Exercise 2 - Building a library

Libraries are a core component of middle and large scale project. Therefore, it is needed to understand how it differs from executable. Unlike executable, never meant to be used by another part of the project, explaining why all properties of a project should be in PRIVATE, libraries are meant to be used. When using libraries such as the std in C++, you include the header file, but the source file is not needed in itself, as it is pre-compiled in object files.

Therefore, the library headers must be PUBLIC, as they are needed when library is used, and not when compiled.
The `BASE_DIRS` property of the `FILE_SET` allow to include in code headers directly from this base dir, without having to precise the whole relative path.
Once the project fully built, the file `libMathFunctions.a` can be found, which is the static library created from the object files compiled from source.

### Exercise 3 - Linking libraries and executables

`target_link_libraries()` is used to describe relations between all target. In our case, it can be used to link a library when compiling an executable.
All properties of target are set to named values. Interface headers will for instance be added to the value `INTERFACE_HEADER_SETS`, private ones to `HEADER_SETS` and public ones to both.
All non-interface properties are used to build the target in itself, along with the interfaces of any used targets.

Once the library added to the executable, we can directly `#include <MathFunctions.h>`, as the base dir is set to the MathFunctions. This adds to the compiler all base dirs as sources to search for libraries, alike `/usr/lib` and `/lib`.

Something noticeable is that linking a depencency that is yet to be created isn't issuing with an error. Similarly to OOP and unlike C functions, the evaluation is not a top-down one.

### Exercise 4 - Subdirectories

As said in the beginning of this step, you may have multiple CMLs, allowing to seperate the build system structure. To do so, you can use `add_subdirectory()`, which will indicate a second CML is present.

Once build again with the argument `--clean-first` because we changed the structure (same as when changing generator), the build directory will contain the new subdirectories, with the executable in the Tutorial one.

