# Helper 'add_test'
macro(add_test test_name test_src)
    add_executable(${test_name} ${test_src})
    target_link_libraries(${test_name} ${TARGET_NAME})
    set_property(TARGET ${test_name} PROPERTY CXX_STANDARD 11)
    set_property(TARGET ${test_name} PROPERTY CXX_STANDARD_REQUIRED ON)
    set_property(TARGET ${test_name} PROPERTY CXX_EXTENSIONS OFF)
    install(TARGETS ${test_name} RUNTIME DESTINATION bin)
endmacro()

# Tests
add_test(oak-multiple-open-stream multiple_open_stream.cpp)
add_test(oak-multithreading-search-test multithreading_search_test.cpp)
