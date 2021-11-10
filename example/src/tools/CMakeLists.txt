add_library(tools STATIC glslProgramObject.cpp glslShaderObject.cpp imGuIZMOquat.cpp
    ../libs/glad/glad.c)
target_include_directories(tools PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/../libs)
