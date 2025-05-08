cmake_minimum_required(VERSION 3.16)
project(ImGuiDemo)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

set(GLAD_DIR "${CMAKE_CURRENT_SOURCE_DIR}/thirdparty/glad")
set(GLAD_INCLUDE_DIR "${GLAD_DIR}/include")
set(GLAD_SOURCE_FILE "${GLAD_DIR}/src/glad.c")

if (EXISTS "${GLAD_SOURCE_FILE}")
    add_library(glad_static STATIC ${GLAD_SOURCE_FILE})
    target_include_directories(glad_static PUBLIC ${GLAD_INCLUDE_DIR})
else()
    message(WARNING "\n==== GLAD NOT FOUND ====\n"
        "You must generate a GLAD loader from https://gen.glad.sh/\n\n"
        "Options:\n"
        " - Language: C\n"
        " - Specification: OpenGL\n"
        " - API: gl (version 3.3 core)\n"
        " - Profile: Core\n"
        " - Generate loader: Yes\n\n"
        "Then extract it to: thirdparty/glad/\n"
        "Expected file: thirdparty/glad/src/glad.c\n"
        "============================\n"
    )
endif()

include(FetchContent)

FetchContent_Declare(
  glfw
  GIT_REPOSITORY https://github.com/glfw/glfw.git
  GIT_TAG 3.3.8
)
set(GLFW_BUILD_DOCS OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_TESTS OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
set(GLFW_INSTALL OFF CACHE BOOL "" FORCE)
set(BUILD_SHARED_LIBS OFF CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(glfw)

FetchContent_Declare(
  imgui
  GIT_REPOSITORY https://github.com/ocornut/imgui.git
  GIT_TAG v1.91.1
)
FetchContent_MakeAvailable(imgui)
set(IMGUI_SRC
    ${imgui_SOURCE_DIR}/imgui.cpp
    ${imgui_SOURCE_DIR}/imgui_demo.cpp
    ${imgui_SOURCE_DIR}/imgui_draw.cpp
    ${imgui_SOURCE_DIR}/imgui_tables.cpp
    ${imgui_SOURCE_DIR}/imgui_widgets.cpp
    ${imgui_SOURCE_DIR}/backends/imgui_impl_glfw.cpp
    ${imgui_SOURCE_DIR}/backends/imgui_impl_opengl3.cpp
)

add_executable(${PROJECT_NAME} main.cpp ${IMGUI_SRC})

target_include_directories(${PROJECT_NAME} PRIVATE
    ${imgui_SOURCE_DIR}
    ${imgui_SOURCE_DIR}/backends
    ${glfw_SOURCE_DIR}/include
    ${GLAD_INCLUDE_DIR}
)

if (TARGET glad_static)
    target_link_libraries(${PROJECT_NAME} PRIVATE glfw glad_static opengl32)
else()
    target_link_libraries(${PROJECT_NAME} PRIVATE glfw opengl32)
endif()

if (NOT CMAKE_SYSTEM_NAME STREQUAL "Linux")
    set(CMAKE_INTERPROCEDURAL_OPTIMIZATION TRUE)
else()
    set(CMAKE_INTERPROCEDURAL_OPTIMIZATION FALSE)
endif()

if (MSVC AND CMAKE_BUILD_TYPE STREQUAL "Release")
	add_compile_options(/arch:AVX2)
    target_compile_definitions(${PROJECT_NAME} PUBLIC _CRT_SECURE_NO_WARNINGS)
    set_target_properties(${PROJECT_NAME} PROPERTIES LINK_FLAGS "/SUBSYSTEM:WINDOWS /ENTRY:mainCRTStartup")
endif()

if (MINGW AND CMAKE_BUILD_TYPE STREQUAL "Release")
    target_compile_options(${PROJECT_NAME} PRIVATE -O3 -march=native -ffast-math -fno-exceptions -fno-rtti)
    target_link_options(${PROJECT_NAME} PRIVATE -s)
endif()

if (MSVC)
    target_compile_options(${PROJECT_NAME} PRIVATE /UUNICODE /U_UNICODE)
elseif(MINGW)
    target_compile_options(${PROJECT_NAME} PRIVATE -UUNICODE -U_UNICODE)
endif()

target_compile_definitions(${PROJECT_NAME} PUBLIC GLFW_INCLUDE_NONE=1)

# Startup project for Visual Studio
set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT ${PROJECT_NAME})
