# Build System

基于CMake的构建系统被组织为一组高级逻辑目标。每个目标对应于一个可执行文件或库，或者是包含自定义命令的自定义目标。目标之间的依赖关系在构建系统中表达，以确定构建顺序和响应变化的再生规则。

可执行文件和库是使用add_executable()和add_library()命令定义的。生成的二进制文件具有适合目标平台的前缀、后缀和扩展名。二进制目标之间的依赖关系使用target_link_libraries()命令表示:

```cmake
add_library(archive archive.cpp zip.cpp lzma.cpp)
add_executable(zipapp zipapp.cpp)
target_link_libraries(zipapp archive)
```

add_library 可以构建静态库

add_executable 可以构建可执行文件

target_link_libraries 构建静态库与可执行文件之间的关系。