cmake_minimum_required(VERSION 3.15)

set(BUILD_EMSCRIPTEN FALSE CACHE BOOL FALSE)

# To build with SDL framwework and VirtualGizmo type:
# cmake -DUSE_SDL:BOOL=TRUE -DUSE_VIRTUALGIZMO:BOOL=TRUE .

set(USE_SDL FALSE CACHE BOOL FALSE)
set(USE_VIRTUALGIZMO FALSE CACHE BOOL FALSE)

set(IMGUI_DIR imgui)

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

# relative source directory
set(SRC "${CMAKE_SOURCE_DIR}/src")

set(SOURCE_FILES
    ${SRC}/glApp.h
    ${SRC}/glApp.cpp
    ${SRC}/glWindow.h
    ${SRC}/glWindow.cpp
    ${SRC}/qJulia.h
    ${SRC}/qJulia.cpp
    ${SRC}/ui/uiMainDlg.h
    ${SRC}/ui/uiMainDlg.cpp
    ${SRC}/ui/uiSettings.cpp
    ${SRC}/tools/glslProgramObject.cpp
    ${SRC}/tools/glslShaderObject.cpp
    ${SRC}/tools/glslProgramObject.h
    ${SRC}/tools/glslShaderObject.h    
    ${SRC}/tools/imGuIZMOquat.cpp
    ${SRC}/tools/imGuIZMOquat.h
    ${SRC}/tools/vGizmo.h
    ${SRC}/tools/vgMath.h
    ${SRC}/tools/vgConfig.h
    ${SRC}/libs/${IMGUI_DIR}/imconfig.h
    ${SRC}/libs/${IMGUI_DIR}/imgui.h
    ${SRC}/libs/${IMGUI_DIR}/imgui.cpp
    ${SRC}/libs/${IMGUI_DIR}/imgui_widgets.cpp
    ${SRC}/libs/${IMGUI_DIR}/imgui_draw.cpp
    ${SRC}/libs/${IMGUI_DIR}/imgui_demo.cpp
    ${SRC}/libs/${IMGUI_DIR}/imgui_impl_opengl3.cpp
    ${SRC}/libs/${IMGUI_DIR}/imgui_impl_opengl3.h
    ${SRC}/libs/${IMGUI_DIR}/imgui_internal.h
    ${SRC}/libs/${IMGUI_DIR}/imstb_rectpack.h
    ${SRC}/libs/${IMGUI_DIR}/imstb_textedit.h
    ${SRC}/libs/${IMGUI_DIR}/imstb_truetype.h)

# additional INCLUDE directories
include_directories(${SRC})
include_directories(${SRC}/libs)
include_directories(${SRC}/tools)
include_directories(${SRC}/tools)

#include_directories($ENV{INCLUDE})
#include_directories("D:/Lang/mingw64/include")

# additional LIB directories
# link_directories($ENV{LIB})


if (${BUILD_EMSCRIPTEN})
    project(wgl_qjSet)

    add_executable(${PROJECT_NAME} ${SOURCE_FILES}
        ${SRC}/libs/${IMGUI_DIR}/imgui_impl_glfw.cpp
        ${SRC}/libs/${IMGUI_DIR}/imgui_impl_glfw.h)

    set(M_GLOBAL_FLAGS "${M_GLOBAL_FLAGS} -DGLAPP_NO_GLSL_PIPELINE -DIMGUI_IMPL_OPENGL_ES3 -DGLFW_INCLUDE_NONE -DGLAPP_NO_OGL_DSA -DGLFW_INCLUDE_ES3")
    #    set(CMAKE_C_COMPILER "emcc")
    #    set(CMAKE_CXX_COMPILER "emcc")

    # -o ${PROJECT_NAME}.html ->  CMAKE_EXECUTABLE_SUFFIX  .html .bc, .js (default), etc
    set(CMAKE_EXECUTABLE_SUFFIX ".html")

    # relative emscripten output files
    set(EMSOUT "emsOut")
    file(MAKE_DIRECTORY ${EMSOUT})
    file(MAKE_DIRECTORY ${EMSOUT}/${CMAKE_BUILD_TYPE})
    set(EMSOUT ${EMSOUT}/${CMAKE_BUILD_TYPE})

    # emcc compiler options
    set(M_EMCC_FLAGS "-DGLAPP_NO_OGL_DSA -DGLFW_INCLUDE_ES3")

    # BUILD TYPE: [Debug|RelWithDebInfo|Release|MinSizeRel]    
    if(${CMAKE_BUILD_TYPE} MATCHES "Debug")
        set(M_EMCC_BINARYEN "-s WASM=0 -s DEMANGLE_SUPPORT=1 ")
        set(M_EMCC_SHELL_FILE "--shell-file \"${CMAKE_SOURCE_DIR}/shell_minimal.html\"")
    else()
        #  -s BINARYEN_METHOD='native-wasm,asmjs' 'interpret-asm2wasm' -s BINARYEN_ASYNC_COMPILATION=0
        if(${CMAKE_BUILD_TYPE} MATCHES "RelWithDebInfo")
            set(M_EMCC_BINARYEN "-s WASM=1")
            set(M_EMCC_SHELL_FILE "--shell-file \"${SRC}/shell_minimal.html\"")
        else()
            if(${CMAKE_BUILD_TYPE} MATCHES "MinSizeRel")
                # one file no memory option: -s SINGLE_FILE=1 --memory-init-file 0 
                set(M_EMCC_BINARYEN "-s WASM=1 -s SINGLE_FILE=1 --memory-init-file 0")
                set(M_EMCC_SHELL_FILE "--shell-file \"${SRC}/mVeryMinimal.html\"")
            else() # Release 
                # -s SINGLE_FILE=1 --memory-init-file 0 -s \"BINARYEN_METHOD='interpret-asm2wasm'\"
                set(M_EMCC_BINARYEN "-s WASM=1 -s \"BINARYEN_METHOD='native-wasm'\" ")
                set(M_EMCC_SHELL_FILE "--shell-file \"${SRC}/mVeryMinimal.html\"")
            endif()    
        endif()    
    endif()

    # grouped EMCC_OPTIONS
    set(M_EMCC_EMBED_FILES "--embed-file \"${CMAKE_SOURCE_DIR}/Shaders/qjFragES2.glsl@Shaders/\" --embed-file \"${CMAKE_SOURCE_DIR}/Shaders/qjVert.glsl@Shaders/\"")
    # set(M_EMCC_EMBED_FILES "--preload-file \"${CMAKE_SOURCE_DIR}/Shaders@/Shaders\"")
    # -s FULL_ES3=1 -s OFFSCREENCANVAS_SUPPORT=1
    set(M_EMCC_OGL_OPTIONS "-s USE_WEBGL2=1 -s USE_GLFW=3 -s LEGACY_GL_EMULATION=0 -s OFFSCREENCANVAS_SUPPORT=1 ")

    # merge all EMCC_OPTIONS
    #    set(M_EMCC_OPTIONS "-s \"BINARYEN_TRAP_MODE='clamp'\" ${M_EMCC_SHELL_FILE} ${M_EMCC_OGL_OPTIONS} ${M_EMCC_BINARYEN} ${M_EMCC_EMBED_FILES}")
    set(M_EMCC_OPTIONS "-s DISABLE_DEPRECATED_FIND_EVENT_TARGET_BEHAVIOR=1 -s ALLOW_MEMORY_GROWTH=1 -s PRECISE_F32=1 ${M_EMCC_SHELL_FILE} ${M_EMCC_OGL_OPTIONS} ${M_EMCC_BINARYEN} ${M_EMCC_EMBED_FILES}")

    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/${EMSOUT}")
    set(CMAKE_BINARY_DIR "${CMAKE_SOURCE_DIR}/${EMSOUT}")

    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${M_GLOBAL_FLAGS} ${M_EMCC_FLAGS} ${M_EMCC_OPTIONS}")


    #    set_target_properties(${PROJECT_NAME}  PROPERTIES LINK_FLAGS "${M_EMCC_OPTIONS}")
else()

    project(qjSet LANGUAGES CXX)



    list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_BINARY_DIR})
    list(APPEND CMAKE_PREFIX_PATH ${CMAKE_BINARY_DIR})

    include(${CMAKE_BINARY_DIR}/conan.cmake)

    conan_cmake_configure(REQUIRES 
        glfw/3.3.5
        GENERATORS cmake_find_package)

    conan_cmake_autodetect(settings)

    conan_cmake_install(PATH_OR_REFERENCE .
        BUILD missing
        REMOTE conancenter
        SETTINGS ${settings})

    find_package(glfw3 REQUIRED)

    set(M_GLOBAL_FLAGS "${M_GLOBAL_FLAGS} -DIMGUI_IMPL_OPENGL_LOADER_GLAD -DGLFW_INCLUDE_NONE -DGLAPP_NO_OGL_DSA")
    if(USE_SDL)
        add_executable(${PROJECT_NAME} ${SOURCE_FILES} 
            ${SRC}/libs/glad/glad.cpp
            ${SRC}/libs/${IMGUI_DIR}/imgui_impl_sdl.cpp
            ${SRC}/libs/${IMGUI_DIR}/imgui_impl_sdl.h)
        set(M_GLOBAL_FLAGS "${M_GLOBAL_FLAGS} -DGLAPP_USE_SDL")
    else()    
        add_executable(${PROJECT_NAME} 
            ${SOURCE_FILES} 
            ${SRC}/libs/glad/glad.cpp
            ${SRC}/libs/${IMGUI_DIR}/imgui_impl_glfw.cpp
            ${SRC}/libs/${IMGUI_DIR}/imgui_impl_glfw.h)
    endif(USE_SDL)    

    set(M_CXX_FLAGS "-DAPP_REQUIRE_OGL45")

    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${M_GLOBAL_FLAGS} ${M_CXX_FLAGS}")

    if(USE_SDL)
        find_package(SDL2 REQUIRED)
        target_link_libraries(${PROJECT_NAME} PUBLIC sdl::sdl)
    else()
        target_link_libraries(${PROJECT_NAME} PUBLIC glfw::glfw)
    endif()


endif ()
