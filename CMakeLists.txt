cmake_minimum_required(VERSION 3.10)

project(cpp-start
    VERSION 1.0.0
    DESCRIPTION "Template repo for C++ projects."
    LANGUAGES CXX
)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

file(GLOB_RECURSE SOURCES src/*.cpp CONFIGURE_DEPENDS) # All source file in src/ will be added
file(GLOB_RECURSE HEADERS include/*.h include/*.hpp CONFIGURE_DEPENDS) # All header files in include/ will be added
file(GLOB TESTS tests/*.cpp) # Each source file in tests/ will be treated as a separate test. Any sources in subdirectories will be treated as test-common sources
file(GLOB BENCHMARKS benchmarks/*.cpp) # Each source file in benchmarks/ will be treated as a separate benchmark. Any sources in subdirectories will be treated as benchmark-common sources

add_library(${PROJECT_NAME} STATIC ${SOURCES})

target_include_directories(${PROJECT_NAME}
    PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    PRIVATE $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/${PROJECT_NAME}>
    PRIVATE $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
)

set(CMAKE_CXX_WARN_DEPRECATED YES)
set(CMAKE_CXX_WARN_UNINITIALIZED YES)
set(CMAKE_CXX_WARN_UNUSED_PARAMETER YES)
set(CMAKE_CXX_WARN_UNUSED_VARIABLE YES)
set(CMAKE_CXX_WARN_UNUSED_BUT_SET_VARIABLE YES)
set(CMAKE_CXX_WARN_UNUSED_FUNCTION YES)
set(CMAKE_CXX_WARN_UNUSED_RESULT YES)

install(TARGETS ${PROJECT_NAME}
    EXPORT ${PROJECT_NAME}-config
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib/static
)

export(EXPORT ${PROJECT_NAME}-config
    FILE "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config.cmake"
    NAMESPACE ${PROJECT_NAME}::
)

enable_testing()
file(GLOB_RECURSE TEST_SOURCES tests/*.cpp CONFIGURE_DEPENDS)
foreach(TEST_FILE ${TESTS})
    list(TEST_SOURCES REMOVE_ITEM ${TEST_FILE})
endforeach()
foreach(TEST_FILE ${TESTS})
    get_filename_component(TEST_NAME_RAW ${TEST_FILE} NAME_WE)
    set(TEST_NAME ${PROJECT_NAME}_${TEST_NAME_RAW}_test)
    add_executable(${TEST_NAME} ${TEST_FILE} ${TEST_SOURCES})
    target_link_libraries(${TEST_NAME} PRIVATE
        ${PROJECT_NAME}
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

file(GLOB_RECURSE BENCHMARK_SOURCES tests/*.cpp CONFIGURE_DEPENDS)
foreach(BENCHMARK_FILE ${BENCHMARKS})
    list(BENCHMARK_SOURCES REMOVE_ITEM ${BENCHMARK_FILE})
endforeach()
foreach(BENCHMARK_FILE ${BENCHMARKS})
    get_filename_component(BENCHMARK_NAME_RAW ${BENCHMARK_FILE} NAME_WE)
    set(BENCHMARK_NAME ${PROJECT_NAME}_${BENCHMARK_NAME_RAW}_benchmark)
    add_executable(${BENCHMARK_NAME} ${BENCHMARK_FILE} ${BENCHMARK_SOURCES})
    target_link_libraries(${BENCHMARK_NAME} PRIVATE
        ${PROJECT_NAME}
    )
    set_target_properties(${BENCHMARK_NAME} PROPERTIES
        CXX_VISIBILITY_PRESET "default"
        BUILD_WITHOUT_DEPENDS ON
    )
endforeach()
