# Create factors from categorical variables

![Unit tests](https://github.com/libscran/factorize/actions/workflows/run-tests.yaml/badge.svg)
![Documentation](https://github.com/libscran/factorize/actions/workflows/doxygenate.yaml/badge.svg)
[![Codecov](https://codecov.io/gh/libscran/factorize/graph/badge.svg?token=JWV0I4WJX2)](https://codecov.io/gh/libscran/factorize)

## Overview

This repository contains functions to create [R-style factors](https://r4ds.had.co.nz/factors.html) from categorical variables.
Each factor is represented by (i) an array of integer codes in the interval $[0, N)$ and (ii) an array of length $N$ containing sorted and unique levels.
For any given observation, its value in the categorical variable can be retrieved by indexing the array of levels by its code.
Factors are useful as they map arbitrary variables onto integer codes that can be easily processed by other functions.

## Quick start

We can create a factor from any categorical variable:

```cpp
#include "factorize/factorize.hpp"

std::vector<std::string> group { "A", "B", "C", "A", "B", "C" };

std::vector<int> codes(group.size());
auto levels = factorize::create_factor(group.size(), group.data(), codes.data());

group[0] == levels[codes[0]]; // true
```

We can also easily create a factor from multiple variables, where the "levels" will be sorted and unique combinations of the variables.

```cpp
std::vector<char> grouping1 { 'c', 'a', 'b', 'a', 'b', 'c' };
std::vector<char> grouping2 { 'A', 'B', 'C', 'C', 'B', 'A' };

std::vector<int> combined_codes(grouping1.size()); 
auto combined_levels = factorize::combine_to_factor(
    grouping1.size(), 
    std::vector<const int*>{ grouping1.data(), grouping2.data() },
    combined_codes.data()
);

grouping1[0] == combined_levels[0][combined_codes[0]]; // true
grouping2[0] == combined_levels[1][combined_codes[0]]; // true
```

Check out the [reference documentation](https://libscran.github.io/factorize) for more details.

## Building projects

### CMake with `FetchContent`

If you're using CMake, you just need to add something like this to your `CMakeLists.txt`:

```cmake
include(FetchContent)

FetchContent_Declare(
  factorize
  GIT_REPOSITORY https://github.com/libscran/factorize
  GIT_TAG master # or any version of interest
)

FetchContent_MakeAvailable(factorize)
```

Then you can link to **factorize** to make the headers available during compilation:

```cmake
# For executables:
target_link_libraries(myexe ltla::factorize)

# For libaries
target_link_libraries(mylib INTERFACE ltla::factorize)
```

### CMake with `find_package()`

```cmake
find_package(ltla_factorize CONFIG REQUIRED)
target_link_libraries(mylib INTERFACE ltla::factorize)
```

To install the library, use:

```sh
mkdir build && cd build
cmake .. -DFACTORIZE_TESTS=OFF
cmake --build . --target install
```

By default, this will use `FetchContent` to fetch all external dependencies.
If you want to install them manually, use `-DFACTORIZE_FETCH_EXTERN=OFF`.
See the tags in [`extern/CMakeLists.txt`](extern/CMakeLists.txt) to find compatible versions of each dependency.

### Manual

If you're not using CMake, the simple approach is to just copy the files in `include/` - either directly or with Git submodules - and include their path during compilation with, e.g., GCC's `-I`.
This also requires the external dependencies listed in [`extern/CMakeLists.txt`](extern/CMakeLists.txt). 
