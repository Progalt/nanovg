cmake_minimum_required(VERSION 3.12)

project(nanovg VERSION 1.0)

option(NANOVG_BUILD_EXAMPLES "Build nanovg example programs" ON)

set(NANOVG_SOURCES 
	"src/nanovg.c"
)

set(NANOVG_INCLUDES
	"src/fontstash.h"
	"src/nanovg.h"
	"src/nanovg_gl.h"
	"src/nanovg_gl_utils"
	"src/stb_image.h"
	"src/stb_truetype.h"
)

add_library(nanovg STATIC ${NANOVG_SOURCES})

target_include_directories(nanovg PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
    $<INSTALL_INTERFACE:include>
)

target_compile_features(nanovg PUBLIC c_std_11)

if(NANOVG_BUILD_EXAMPLES)

	include(FetchContent)
    FetchContent_Declare(
        glfw
        GIT_REPOSITORY https://github.com/glfw/glfw.git
        GIT_TAG        3.4
    )
    FetchContent_MakeAvailable(glfw)

    add_executable(nanovg_demo "example/glad/glad.c" "example/example_gl3.c" "example/perf.c" "example/demo.c")

    target_link_libraries(nanovg_demo PRIVATE nanovg glfw)
    target_include_directories(nanovg_demo PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/src)

     add_custom_command(
        TARGET nanovg_demo POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_directory
            ${CMAKE_CURRENT_SOURCE_DIR}/example/data
            $<TARGET_FILE_DIR:nanovg_demo>/example/data
        COMMENT "Copying example/data to build directory."
    )

    # Create a testbed 

    add_executable(nanovg_test "example/glad/glad.c" "example/testbed.c")
    target_link_libraries(nanovg_test PRIVATE nanovg glfw)
endif()