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
