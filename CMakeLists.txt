cmake_minimum_required(VERSION 3.12)

project(nanovg VERSION 1.0)

option(NANOVG_BUILD_EXAMPLES "Build nanovg example programs" ON)
option(NANOVG_NO_STB_IMAGE "Remove stb_image.h for loading common image formats" OFF)

if(NOT NANOVG_NO_STB_IMAGE)

    set(STB_IMAGE_URL https://raw.githubusercontent.com/nothings/stb/master/stb_image.h)
    set(STB_IMAGE_DEST ${CMAKE_CURRENT_SOURCE_DIR}/src/stb_image.h)

    message("Remove stb_image.h: OFF")
    message("Downloading stb_image.h")

    if(NOT EXISTS ${STB_IMAGE_DEST})
        file(DOWNLOAD ${STB_IMAGE_URL} ${STB_IMAGE_DEST}
             SHOW_PROGRESS
        )
    endif()

    message("stb_image.h downloaded")

endif()

if(NANOVG_NO_STB_IMAGE)
     message("Remove stb_image.h: ON")
endif()

if(NANOVG_NO_STB_IMAGE AND EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/src/stb_image.h)
    file(REMOVE ${CMAKE_CURRENT_SOURCE_DIR}/src/stb_image.h)
endif()

set(NANOVG_SOURCES 
	"src/nanovg.c"
)

set(NANOVG_INCLUDES
	"src/fontstash.h"
	"src/nanovg.h"
	"src/nanovg_gl.h"
	"src/nanovg_gl_utils"
	"src/stb_truetype.h"
)

if(NOT NANOVG_NO_STB_IMAGE)
    list(APPEND NANOVG_INCLUDES "src/stb_image.h")
endif()

add_library(nanovg STATIC ${NANOVG_SOURCES})

if(NANOVG_NO_STB_IMAGE)
    target_compile_definitions(nanovg PUBLIC NVG_NO_STB)
endif()

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