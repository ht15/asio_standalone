# Add source files
file(GLOB SOURCES *.cpp)

file(GLOB_RECURSE EXAMPLES examples/cpp11/*.cpp)

file(GLOB TESTS tests/**.cpp)

# Main library
if(ASIO_SEPARATE_COMPILATION)
    add_library(asio STATIC ${SOURCES})
    target_link_libraries(asio ${ASIO_LIBS})
    install(TARGETS asio
            ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})
endif()

# Examples
foreach(example ${EXAMPLES})
    # 或方法2：子目录前缀
    get_filename_component(subdir ${example} DIRECTORY)
    get_filename_component(subdir_name ${subdir} NAME)
    get_filename_component(file_name ${example} NAME_WE)
    set(unique_name "${subdir_name}_${file_name}")
    add_executable(${unique_name} ${example})
    target_link_libraries(${unique_name} ${ASIO_LIBS})
endforeach()

# Tests
if(BUILD_TESTING)
    enable_testing()
    foreach(test ${TESTS})
        get_filename_component(test_name ${test} NAME_WE)
        add_executable(${test_name} ${test})
        target_link_libraries(${test_name} ${ASIO_LIBS})
        add_test(NAME ${test_name} COMMAND ${test_name})
    endforeach()
endif()