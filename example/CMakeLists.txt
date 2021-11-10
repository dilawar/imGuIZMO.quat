cmake_minimum_required(VERSION 3.15)

project(testgui LANGUAGES CXX)

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release)
endif()

set(CMAKE_CXX_STANDARD 17)

set(M_GLOBAL_FLAGS "-DIMGUI_DIR -DGLM_ENABLE_EXPERIMENTAL")

if(USE_VIRTUALGIZMO)
    set(M_GLOBAL_FLAGS "${M_GLOBAL_FLAGS} -DGLAPP_USE_VIRTUALGIZMO")
endif()

if(NOT EXISTS "${CMAKE_BINARY_DIR}/conan.cmake")
    message(STATUS "Downloading conan.cmake from https://github.com/conan-io/cmake-conan")
    file(DOWNLOAD "https://raw.githubusercontent.com/conan-io/cmake-conan/v0.16.1/conan.cmake"
        "${CMAKE_BINARY_DIR}/conan.cmake"
        EXPECTED_HASH SHA256=396e16d0f5eabdc6a14afddbcfff62a54a7ee75c6da23f32f7a31bc85db23484
        TLS_VERIFY ON)
endif()


list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_BINARY_DIR})
list(APPEND CMAKE_PREFIX_PATH ${CMAKE_BINARY_DIR})

include(${CMAKE_BINARY_DIR}/conan.cmake)

conan_cmake_configure(REQUIRES
    glfw/3.3.5
    imgui/1.85
    GENERATORS cmake_find_package)

conan_cmake_autodetect(settings)

conan_cmake_install(PATH_OR_REFERENCE .
    BUILD missing
    REMOTE conancenter
    SETTINGS ${settings})

find_package(glfw3 REQUIRED)
find_package(imgui REQUIRED)

#
# Global includes
#
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/src/tools
    ${CMAKE_CURRENT_SOURCE_DIR}/src/libs)

#
# relative source directory
#
add_subdirectory(src/libs)
add_subdirectory(src/ui)
add_subdirectory(src/tools)

set(SOURCE_FILES
    src/glApp.cpp
    src/glWindow.cpp
    src/qJulia.cpp)

#
# additional INCLUDE directories
#
include_directories(src)
include_directories(src/libs)
include_directories(src/tools)
include_directories(src/tools)

add_executable(testgui
    ${SOURCE_FILES}
    ${imgui_RES_DIRS}/bindings/imgui_impl_glfw.cpp
    ${imgui_RES_DIRS}/bindings/imgui_impl_opengl3.cpp)

#set(M_CXX_FLAGS "-DAPP_REQUIRE_OGL45")
# set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${M_GLOBAL_FLAGS} ${M_CXX_FLAGS}")

target_link_libraries(testgui PRIVATE ui tools _glad)
target_link_libraries(testgui PRIVATE imgui::imgui glfw::glfw)
