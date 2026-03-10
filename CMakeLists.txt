cmake_minimum_required(VERSION 3.25)

project(ECS-Engine
    VERSION 0.1.0
    LANGUAGES C CXX
)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

find_package(glfw3 CONFIG REQUIRED)
find_package(glm CONFIG REQUIRED)
find_package(OpenGL REQUIRED)

add_library(glad STATIC
    external/glad/src/glad.c
)

target_include_directories(glad
    PUBLIC
        ${CMAKE_CURRENT_SOURCE_DIR}/external/glad/include
)

add_executable(EcsEngine
    src/main.cpp
)

target_include_directories(EcsEngine
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/include
)

target_compile_features(EcsEngine PRIVATE cxx_std_20)

if (MSVC)
    target_compile_options(EcsEngine PRIVATE /W4 /permissive-)
else()
    target_compile_options(EcsEngine PRIVATE -Wall -Wextra -Wpedantic)
endif()

target_link_libraries(EcsEngine
    PRIVATE
        glad
        glfw
        glm::glm
        OpenGL::GL
)

set_property(TARGET EcsEngine PROPERTY VS_DEBUGGER_WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}")