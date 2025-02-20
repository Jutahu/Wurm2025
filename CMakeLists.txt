cmake_minimum_required(VERSION 3.11) # FetchContent is available in 3.11+
project(Matopeli2025)

# Generate compile_commands.json
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

include_directories(
        ${PROJECT_SOURCE_DIR}/include
        ${PROJECT_SOURCE_DIR}/src
)

file(GLOB all_SRCS
        "${PROJECT_SOURCE_DIR}/include/*.h"
        "${PROJECT_SOURCE_DIR}/include/*.hpp"
        "${PROJECT_SOURCE_DIR}/src/*.cpp"
        "${PROJECT_SOURCE_DIR}/src/*.c"
        )

# Dependencies
set(RAYLIB_VERSION 5.5)
find_package(raylib ${RAYLIB_VERSION} QUIET) # QUIET or REQUIRED
if (NOT raylib_FOUND) # If there's none, fetch and build raylib
  include(FetchContent)
  FetchContent_Declare(
    raylib
    DOWNLOAD_EXTRACT_TIMESTAMP OFF
    URL https://github.com/raysan5/raylib/archive/refs/tags/${RAYLIB_VERSION}.tar.gz
  )
  FetchContent_GetProperties(raylib)
  if (NOT raylib_POPULATED) # Have we downloaded raylib yet?
    set(FETCHCONTENT_QUIET NO)
    FetchContent_MakeAvailable(raylib)
    set(BUILD_EXAMPLES OFF CACHE BOOL "" FORCE) # don't build the supplied examples
  endif()
endif()

# Our Project

add_executable(${PROJECT_NAME} ${all_SRCS})
#set(raylib_VERBOSE 1)
target_link_libraries(${PROJECT_NAME} raylib)

file(GLOB_RECURSE ASSETS "${CMAKE_SOURCE_DIR}/include/*")
foreach(ASSET ${ASSETS})
    file(RELATIVE_PATH ASSET_RELATIVE_PATH "${CMAKE_SOURCE_DIR}/" ${ASSET})

    ##build directory
    set(DESTINATION_PATH "${CMAKE_BINARY_DIR}/${ASSET_RELATIVE_PATH}")

    ##copy to target path
    add_custom_command(
        TARGET Matopeli2025 POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy ${ASSET} ${PROJECT_SOURCE_DIR}/BUILD
        COMMENT "Copying: ${ASSET_RELATIVE_PATH}"
    )
endforeach()

# Web Configurations

# Checks if OSX and links appropriate frameworks (Only required on MacOS)
if (APPLE)
    target_link_libraries(${PROJECT_NAME} "-framework IOKit")
    target_link_libraries(${PROJECT_NAME} "-framework Cocoa")
    target_link_libraries(${PROJECT_NAME} "-framework OpenGL")
endif()
