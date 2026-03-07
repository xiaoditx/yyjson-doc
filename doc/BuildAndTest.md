构建与测试
==============

@note 翻译最后更新时间：2026/2/27 13:01<br>翻译质量反馈：[yyjson文档中文翻译](https://xiaoditx.github.io/moments/2026-3/07-2115/)

将本库集成到项目中有多种方式：通过源代码、包管理器以及 CMake。

# 源代码
本库旨在提供一个跨平台的 JSON 库，因此使用 ANSI C（实际上是 C99，但兼容严格的 C89）编写。你可以将 `yyjson.h` 和 `yyjson.c` 文件复制到你的项目中，无需任何配置即可开始使用。

该库已在 [Github CI](https://github.com/ibireme/yyjson/actions) 中，使用 `gcc`、`clang`、`msvc`、`tcc` 编译器以及 `x86`、`arm`、`ppc`、`riscv`、`s390x` 架构上进行了测试。如果遇到任何编译问题，请[报告 bug](https://github.com/ibireme/yyjson/issues/new?template=bug_report.md)。

默认情况下，库的所有功能都是启用的，但你可以通过添加编译时选项来裁剪部分功能。例如，当不需要序列化功能时，可以禁用 JSON writer 以减小二进制体积；或者禁用注释支持以提高解析性能。详情请参阅 `编译时选项` 部分。


# 包管理器

你可以使用一些流行的包管理器，如 `vcpkg`、`conan` 和 `xmake` 来下载和安装 yyjson。这些包管理器中的 yyjson 包由社区贡献者保持更新。如果版本过旧，请在其仓库中创建 issue 或 pull request。

## 使用 vcpkg

你可以使用 [vcpkg](https://github.com/Microsoft/vcpkg/) 依赖项管理器构建和安装 yyjson：

```shell
git clone https://github.com/Microsoft/vcpkg.git
cd vcpkg
./bootstrap-vcpkg.sh  # 对于 Powershell 使用 ./bootstrap-vcpkg.bat
./vcpkg integrate install
./vcpkg install yyjson
```

如果版本过旧，请在 vcpkg 仓库上[创建 issue 或 pull request](https://github.com/Microsoft/vcpkg)。

# CMake

## 使用 CMake 构建库

克隆仓库并创建构建目录：
```shell
git clone https://github.com/ibireme/yyjson.git
cmake -E make_directory build; cd build
```

构建静态库：
```shell
cmake .. 
cmake --build .
```

构建共享库：
```shell
cmake .. -DBUILD_SHARED_LIBS=ON
cmake --build .
```

支持的 CMake 选项（默认为 OFF）：

- `-DYYJSON_BUILD_TESTS=ON` 构建所有测试。
- `-DYYJSON_BUILD_FUZZER=ON` 使用 LibFuzzing 构建模糊测试器。
- `-DYYJSON_BUILD_MISC=ON` 构建杂项。
- `-DYYJSON_BUILD_DOC=ON` 使用 doxygen 构建文档。
- `-DYYJSON_ENABLE_COVERAGE=ON` 为测试启用代码覆盖率。
- `-DYYJSON_ENABLE_VALGRIND=ON` 为测试启用 valgrind 内存检查器。
- `-DYYJSON_ENABLE_SANITIZE=ON` 为测试启用 sanitizer。
- `-DYYJSON_ENABLE_FASTMATH=ON` 为测试启用 fast-math。
- `-DYYJSON_FORCE_32_BIT=ON` 强制为测试使用 32 位模式（gcc/clang/icc）。

- `-DYYJSON_DISABLE_READER=ON` 如果不需要 JSON 读取器，禁用它。
- `-DYYJSON_DISABLE_WRITER=ON` 如果不需要 JSON 写入器，禁用它。
- `-DYYJSON_DISABLE_INCR_READER=ON` 如果不需要增量读取器，禁用它。
- `-DYYJSON_DISABLE_UTILS=ON` 禁用 JSON Pointer、JSON Patch 和 JSON Merge Patch。
- `-DYYJSON_DISABLE_FAST_FP_CONV=ON` 禁用内置的快速浮点数转换。
- `-DYYJSON_DISABLE_NON_STANDARD=ON` 在编译时禁用对非标准 JSON 的支持。
- `-DYYJSON_DISABLE_UTF8_VALIDATION=ON` 在编译时禁用 UTF-8 验证。
- `-DYYJSON_DISABLE_UNALIGNED_MEMORY_ACCESS=ON` 在编译时禁用对非对齐内存访问的支持。


## 使用 CMake 作为依赖项

你可以将 yyjson 下载并解压到你的项目目录中，并在你的 `CMakeLists.txt` 文件中链接它：
```cmake
# 添加一些选项（可选）
set(YYJSON_DISABLE_NON_STANDARD ON CACHE INTERNAL "")

# 添加 `yyjson` 子目录
add_subdirectory(vendor/yyjson)

# 将 yyjson 链接到你的目标
target_link_libraries(your_target PRIVATE yyjson)
```

如果你的 CMake 版本高于 3.11，你可以使用以下代码让 CMake 自动下载它：
```cmake
include(FetchContent)

# 让 CMake 下载 yyjson
FetchContent_Declare(
    yyjson
    GIT_REPOSITORY https://github.com/ibireme/yyjson.git
    GIT_TAG master # master 或版本号，例如 0.6.0
)
FetchContent_GetProperties(yyjson)
if(NOT yyjson_POPULATED)
  FetchContent_Populate(yyjson)
  add_subdirectory(${yyjson_SOURCE_DIR} ${yyjson_BINARY_DIR} EXCLUDE_FROM_ALL)
endif()

# 将 yyjson 链接到你的目标
target_link_libraries(your_target PRIVATE yyjson)
```


## 使用 CMake 生成项目
如果你想用其他编译器或 IDE 构建或调试 yyjson，请尝试以下命令：
```shell
cmake -E make_directory build; cd build

# 用于 Linux/Unix 的 Clang：
cmake .. -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++

# 用于 Linux/Unix 的 Intel ICC：
cmake .. -DCMAKE_C_COMPILER=icc -DCMAKE_CXX_COMPILER=icpc

# 其他版本的 GCC：
cmake .. -DCMAKE_C_COMPILER=/usr/local/gcc-8.2/bin/gcc -DCMAKE_CXX_COMPILER=/usr/local/gcc-8.2/bin/g++

# 用于 Windows 的 Microsoft Visual Studio：
cmake .. -G "Visual Studio 16 2019" -A x64
cmake .. -G "Visual Studio 16 2019" -A Win32
cmake .. -G "Visual Studio 15 2017 Win64"

# 用于 macOS 的 Xcode：
cmake .. -G Xcode

# 用于 iOS 的 Xcode：
cmake .. -G Xcode -DCMAKE_SYSTEM_NAME=iOS

# 使用 XCTest 的 Xcode
cmake .. -G Xcode -DYYJSON_BUILD_TESTS=ON
```

## 使用 CMake 生成文档

本项目使用 [doxygen](https://www.doxygen.nl/) 生成文档。
在继续之前，请确保你的系统上安装了 `doxygen`，
最好使用 `doc/Doxyfile.in` 中指定的版本。


构建文档：
```shell
cmake -E make_directory build; cd build
cmake .. -DYYJSON_BUILD_DOC=ON
cmake --build .
```

生成的 HTML 文件将位于 `build/doxygen/html`。

你也可以在线浏览预生成的文档：
https://ibireme.github.io/yyjson/doc/doxygen/html/


## 使用 CMake 和 CTest 进行测试

构建并运行所有测试：
```shell
cmake -E make_directory build; cd build
cmake .. -DYYJSON_BUILD_TESTS=ON
cmake --build .
ctest --output-on-failure
```

使用 [valgrind](https://valgrind.org/) 内存检查器构建并运行测试（继续之前请确保已安装 `valgrind`）：
```shell
cmake -E make_directory build; cd build
cmake .. -DYYJSON_BUILD_TESTS=ON -DYYJSON_ENABLE_VALGRIND=ON
cmake --build .
ctest --output-on-failure
```

使用 sanitizer 构建并运行测试（编译器应为 `gcc` 或 `clang`）：
```shell
cmake -E make_directory build; cd build
cmake .. -DYYJSON_BUILD_TESTS=ON -DYYJSON_ENABLE_SANITIZE=ON
cmake --build .
ctest --output-on-failure
```

使用 `gcc` 构建并运行代码覆盖率：
```shell
cmake -E make_directory build; cd build
cmake .. -DCMAKE_BUILD_TYPE=Debug -DYYJSON_BUILD_TESTS=ON -DYYJSON_ENABLE_COVERAGE=ON
cmake --build . --config Debug
ctest --output-on-failure

lcov -c -d ./CMakeFiles --include "*/yyjson.*" -o cov.info
genhtml cov.info -o ./cov_report
```

使用 `clang` 构建并运行代码覆盖率：
```shell
cmake -E make_directory build; cd build
cmake .. -DCMAKE_BUILD_TYPE=Debug -DYYJSON_BUILD_TESTS=ON -DYYJSON_ENABLE_COVERAGE=ON -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++
cmake --build . --config Debug

export LLVM_PROFILE_FILE=cov/profile-%p.profraw
ctest --output-on-failure

ctest_files=$(grep -o "test_\w\+" CTestTestfile.cmake | uniq | tr '\n' ' ')
ctest_files=$(echo $ctest_files | sed 's/  $//' | sed "s/ / -object /g")
llvm-profdata merge -sparse cov/profile-*.profraw -o coverage.profdata
llvm-cov show $ctest_files -instr-profile=coverage.profdata -format=html > coverage.html
```

使用 [LibFuzzer](https://llvm.org/docs/LibFuzzer.html) 构建并运行模糊测试（编译器应为 `LLVM Clang`，不支持 `Apple Clang` 或 `gcc`）：
```shell
cmake -E make_directory build; cd build
cmake .. -DYYJSON_BUILD_FUZZER=ON -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++
cmake --build .
./fuzzer -dict=fuzzer.dict ./corpus
```


# 编译时选项
本库提供了一些编译时选项，可以在编译期间定义为 1 来禁用特定功能。
例如，要禁用 JSON writer：
```shell
cmake -E make_directory build; cd build
cmake .. -DYYJSON_DISABLE_WRITER=ON
gcc -DYYJSON_DISABLE_WRITER=1 ...
```

## YYJSON_DISABLE_READER
在编译时定义为 1 以禁用 JSON 读取器。<br/>
这将禁用所有名称中包含 `read` 的函数。<br/>
二进制体积减少约 60%。<br/>
当不需要 JSON 解析时推荐使用。<br/>

## YYJSON_DISABLE_WRITER
在编译时定义为 1 以禁用 JSON 写入器。<br/>
这将禁用所有名称中包含 `write` 的函数。<br/>
二进制体积减少约 30%。<br/>
当不需要 JSON 序列化时推荐使用。<br/>

## YYJSON_DISABLE_INCR_READER
在编译时定义为 1 以禁用 JSON 增量读取器。<br/>
这将禁用所有名称中包含 `incr` 的函数。<br/>
当不需要 JSON 增量读取器时推荐使用。<br/>

## YYJSON_DISABLE_UTILS
在编译时定义为 1 以禁用 JSON Pointer、JSON Patch 和 JSON Merge Patch 支持。<br/>
这将禁用所有名称中包含 `ptr` 或 `patch` 的函数。<br/>
当不需要这些函数时推荐使用。<br/>

## YYJSON_DISABLE_FAST_FP_CONV
在编译时定义为 1 以禁用 yyjson 中的快速浮点数转换。<br/>
将使用 libc 的 `strtod/snprintf` 代替。<br/>
这会使二进制体积减少约 30%，但会显著降低浮点数的读写速度。<br/>
当处理的 JSON 中浮点数较少时推荐使用。<br/>

## YYJSON_DISABLE_NON_STANDARD
在编译时定义为 1 以禁用对非标准 JSON 功能的支持：
- YYJSON_READ_ALLOW_INF_AND_NAN
- YYJSON_READ_ALLOW_COMMENTS
- YYJSON_READ_ALLOW_TRAILING_COMMAS
- YYJSON_READ_ALLOW_INVALID_UNICODE
- YYJSON_READ_ALLOW_BOM
- YYJSON_READ_ALLOW_EXT_NUMBER
- YYJSON_READ_ALLOW_EXT_ESCAPE
- YYJSON_READ_ALLOW_EXT_WHITESPACE
- YYJSON_READ_ALLOW_SINGLE_QUOTED_STR
- YYJSON_READ_ALLOW_UNQUOTED_KEY
- YYJSON_READ_JSON5
- YYJSON_WRITE_ALLOW_INF_AND_NAN
- YYJSON_WRITE_ALLOW_INVALID_UNICODE

这会使二进制体积减少约 10%，并略微提高性能。<br/>
当不需要处理非标准 JSON 时推荐使用。

## YYJSON_DISABLE_UTF8_VALIDATION
在编译时定义为 1 以禁用 UTF-8 验证。

如果所有输入字符串都保证是有效的 UTF-8（例如，语言级别的字符串类型已经过验证），可以使用此选项。

禁用 UTF-8 验证可将非 ASCII 字符串的性能提高约 3% 到 7%。

注意：如果启用此标志但传入非法的 UTF-8 字符串，可能会出现以下错误：
- 解析 JSON 字符串时可能会忽略转义字符。
- 解析 JSON 字符串时可能会忽略结束引号，导致该字符串与下一个值合并。
- 使用 `yyjson_mut_val` 序列化时，可能越界访问字符串末尾，可能导致段错误。

## YYJSON_EXPORTS
在将库构建为 Windows DLL 时，定义为 1 以导出符号。

## YYJSON_IMPORTS
在使用库作为 Windows DLL 时，定义为 1 以导入符号。
