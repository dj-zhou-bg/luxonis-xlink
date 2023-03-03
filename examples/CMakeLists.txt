# Helper 'add_example'
macro(add_example example_name example_src)
    add_executable(${example_name} ${example_src})
    target_link_libraries(${example_name} ${TARGET_NAME})
    set_property(TARGET ${example_name} PROPERTY CXX_STANDARD 11)
    set_property(TARGET ${example_name} PROPERTY CXX_STANDARD_REQUIRED ON)
    set_property(TARGET ${example_name} PROPERTY CXX_EXTENSIONS OFF)
    install(TARGETS ${example_name} RUNTIME DESTINATION bin)
endmacro()

# Examples
add_example(oak-list-devices list_devices.cpp)
add_example(oak-boot-bootloader boot_bootloader.cpp)
add_example(oak-boot-firmware boot_firmware.cpp)
