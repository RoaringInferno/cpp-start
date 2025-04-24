cmake_minimum_required(VERSION 3.10)

project(cpp-start
    VERSION 1.0.0
    DESCRIPTION "Template repo for C++ projects."
    LANGUAGES CXX
)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

file(GLOB_RECURSE SOURCES src/*.cpp)
list(REMOVE_ITEM SOURCES ${CMAKE_SOURCE_DIR}/src/main.cpp)
file(GLOB_RECURSE HEADERS include/*.h include/*.hpp)
file(GLOB_RECURSE TESTS tests/*.cpp)

# Compile to a static library first
set(LIBNAME lib_${PROJECT_NAME})
add_library(${LIBNAME} STATIC ${SOURCES})
target_include_directories(${LIBNAME}
    PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
)

# Add main executable
add_executable(${PROJECT_NAME} ${CMAKE_SOURCE_DIR}/src/main.cpp)
target_link_libraries(${PROJECT_NAME} PUBLIC ${LIBNAME})

set(CMAKE_CXX_WARN_DEPRECATED YES)
set(CMAKE_CXX_WARN_UNINITIALIZED YES)
set(CMAKE_CXX_WARN_UNUSED_PARAMETER YES)
set(CMAKE_CXX_WARN_UNUSED_VARIABLE YES)
set(CMAKE_CXX_WARN_UNUSED_BUT_SET_VARIABLE YES)
set(CMAKE_CXX_WARN_UNUSED_FUNCTION YES)
set(CMAKE_CXX_WARN_UNUSED_RESULT YES)

enable_testing()
foreach(TEST_FILE ${TESTS})
    get_filename_component(TEST_NAME_RAW ${TEST_FILE} NAME_WE)
    set(TEST_NAME ${PROJECT_NAME}_${TEST_NAME_RAW}_test)
    add_executable(${TEST_NAME} ${TEST_FILE})
    target_link_libraries(${TEST_NAME} PRIVATE
        ${LIBNAME}
    )
    add_test(
        NAME ${TEST_NAME}
        COMMAND ${TEST_NAME}
    )
    set_target_properties(${TEST_NAME} PROPERTIES
        CXX_VISIBILITY_PRESET "default"
        BUILD_WITHOUT_DEPENDS ON
    )
endforeach()

set(CTEST_SOURCE_DIRECTORY ${CMAKE_SOURCE_DIR})
set(CTEST_BINARY_DIRECTORY ${CMAKE_BINARY_DIR})
set(CTEST_COMMAND ${CMAKE_CTEST_COMMAND})

set(CTEST_TEST_TYPE "unit")
set(CTEST_TEST_TIMEOUT 300)

foreach(BENCHMARK_FILE ${BENCHMARKS})
    get_filename_component(BENCHMARK_NAME_RAW ${BENCHMARK_FILE} NAME_WE)
    set(BENCHMARK_NAME ${PROJECT_NAME}_${BENCHMARK_NAME_RAW}_benchmark)
    add_executable(${BENCHMARK_NAME} ${BENCHMARK_FILE})
    target_link_libraries(${BENCHMARK_NAME} PRIVATE
        ${LIBNAME}
    )
    set_target_properties(${BENCHMARK_NAME} PROPERTIES
        CXX_VISIBILITY_PRESET "default"
        BUILD_WITHOUT_DEPENDS ON
    )
endforeach()
