cmake_minimum_required(VERSION 3.20)
project(myserver CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

option(ENABLE_SANITIZERS "Enable ASan/UBSan" OFF)
option(USE_TSAN "Use ThreadSanitizer (mutually exclusive with ASan)" OFF)

if(NOT MSVC)
  add_compile_options(-Wall -Wextra -Werror -fno-omit-frame-pointer)
  if(USE_TSAN)
    add_compile_options(-fsanitize=thread)
    add_link_options(-fsanitize=thread)
  elseif(ENABLE_SANITIZERS)
    add_compile_options(-fsanitize=address,undefined)
    add_link_options(-fsanitize=address,undefined)
  endif()
else()
  add_compile_options(/W4 /WX /EHsc)
endif()

find_package(asio CONFIG REQUIRED)
find_package(spdlog CONFIG REQUIRED)
find_package(protobuf CONFIG REQUIRED)

# 阶段1再加 src/ tests/ 子目录
