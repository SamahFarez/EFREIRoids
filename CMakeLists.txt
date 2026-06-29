cmake_minimum_required(VERSION 3.20)
project(asteroid LANGUAGES CXX)

# ==== C++ standard ====
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# ==== Output directories ====
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")

# ==== Options ====
option(USE_SYSTEM_SDL3 "Prefer system SDL3 (if OFF, use bundled libs/SDL)" ON)

# ==== Game sources ====
file(GLOB_RECURSE SRCS "src/*.cpp")

# Exclude SDL bundled source files embedded in src/SDL (not part of the game logic)
list(FILTER SRCS EXCLUDE REGEX ".*/src/SDL/.*")

add_executable(asteroid ${SRCS})

# ==== Include directories ====
target_include_directories(asteroid
    PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/include
)

target_compile_options(asteroid
        PRIVATE
        $<$<CXX_COMPILER_ID:GNU,Clang>:-Wall -Wextra -Wpedantic>
        $<$<CXX_COMPILER_ID:MSVC>:/W4>
)

# =====================================================
# == SDL3 detection (system preferred, fallback to submodule)
# =====================================================

if (USE_SYSTEM_SDL3)
    find_package(SDL3 CONFIG QUIET)
endif()

if (TARGET SDL3::SDL3 OR SDL3_FOUND)
    message(STATUS "Using system SDL3")
elseif (EXISTS "${CMAKE_SOURCE_DIR}/libs/SDL")
    message(STATUS "Using bundled SDL3 at ${CMAKE_SOURCE_DIR}/libs/SDL")
    add_subdirectory("${CMAKE_SOURCE_DIR}/libs/SDL" EXCLUDE_FROM_ALL)
else()
    message(FATAL_ERROR
            "SDL3 not found. Install SDL3 system-wide or provide submodule at libs/SDL.\n"
            "Set -DUSE_SYSTEM_SDL3=OFF to use the bundled version."
    )
endif()

target_link_libraries(asteroid PRIVATE SDL3::SDL3)

# Windows WIN32 subsystem
if (WIN32)
    set_target_properties(asteroid PROPERTIES WIN32_EXECUTABLE TRUE)
endif()

# ==== Copy resources after build ====
add_custom_command(TARGET asteroid POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_directory
        "${CMAKE_CURRENT_SOURCE_DIR}/resources"
        "$<TARGET_FILE_DIR:asteroid>/resources"
)


# ==== Package ====
set(CPACK_PROJECT_NAME ${PROJECT_NAME})
set(CPACK_PROJECT_VERSION ${PROJECT_VERSION})
include(CPack)
