+++
draft = false
title = "Clion Libraries"
date = 2017-11-14T15:56:05+01:00
summary = "How to link a static library project in CLion to a unittest executable."
tags = ["tech"]
showLogo = false
logo = ""
hasMath = false
+++

[CLion](https://www.jetbrains.com/clion/) is my preferred C++ IDE. For my Master's thesis, today I had to implement the calculation of [Shapley's value](https://en.wikipedia.org/wiki/Shapley_value) - a concept from game theory.   
I thought, okay, let's do this properly. I'll be using the calculation inside a simulator, but I want to implement it outside the scope of this huge code framework, so that I can run easily unit-test it. So I'll use CLion and let it compile a library. Later in the simulator I can just link the library in. My unit testing binary also just links the library. So if later I fix some bugs or add some functionality, I can easily run my tests to see if I broke anything, and then let it run on my simulator, without the hassle of copying over the code.

CLion uses CMake to compile things. CMake is a wrapper for Makefiles. It has its own language and generates the Makefiles for you - it's a bit like SASS for CSS.   
Let's see how to set up this whole thing in CLion.

Create a new project which I'll call `shapley`. `src/shapley.{cpp,h}` will describe the library functionality. `src/tests/unittests.cpp` will describe a binary that runs the unit tests. So the initial folder structure is this:

```
src/
  lib/
    CMakeLists.txt
    shapley.cpp
    shapley.h    
  tests/
    CMakeLists.txt
    unittests.cpp  
CMakeLists.txt
```

Note the three `CMakeList.txt`'s. The one inside `src/tests/` is for compiling just the unit tests. `src/lib/CMakeLists.txt` handles the library compilation. The top one includes both others and handles global configuration rules.

First the **`src/tests/CMakeLists.txt`:**

```
cmake_minimum_required(VERSION 3.8)
set(CMAKE_CXX_STANDARD 11)
project(shapley-unittests)

include_directories(..)

find_library(LIBRARY-PATH :libcppunit.so)
link_directories(${LIBRARY-PATH})

set(SOURCE_FILES_UNITTESTS
        unittests.cpp
)

set(CMAKE_BUILD_TYPE Debug)
add_executable(unittests ${SOURCE_FILES_UNITTESTS})
target_link_libraries(unittests shapley :libcppunit.so)
```

`include_directories(..)` includes the `src/` directory so that we can include the library header as `#include "shapley.h"`.   
`set(SOURCE_FILES_UNITTESTS ...)` sets this project's source files. Note that this is only the `unittests.cpp`.   
`target_link_libraries(unittests shapley :libcppunit.so)` links both the `libshapley.a` and the `libcppunit.so`.

Secondly **`src/lib/CMakeLists.txt`:**

```
cmake_minimum_required(VERSION 3.8)
set(CMAKE_CXX_STANDARD 11)
project(shapley-lib)

include_directories(..)

set(SOURCE_FILES_SHAPLEY-LIB
        shapley.cpp
        shapley.h)

add_library(shapley ${SOURCE_FILES_SHAPLEY-LIB})
```

Finally the **top `CMakeLists.txt`:**

```
cmake_minimum_required(VERSION 3.8)
project(shapley)

set(CMAKE_CXX_STANDARD 11)

add_subdirectory(src/tests)
add_subdirectory(src/lib)

set_target_properties(unittests PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/out/")
set_target_properties(shapley PROPERTIES ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/out/")
```

`add_subdirectory()` includes the other `CMakeLists.txt`s.   
The `set_target_properties` simply set the output directory.

This covers the CMake setup. Now your `src/tests/unittests.cpp` can simply do this:

```
#include <iostream>
#include "shapley.h"

int main(int argc, const char *argv[]) {
	hello();
	std::cout << "Hello World" << std::endl;
	return 0;
}
```

and define `hello()` in your library, and it'll work.
