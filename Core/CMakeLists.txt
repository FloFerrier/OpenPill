set(BUILD_TARGET "${SELECTED_BOARD}_${CMAKE_BUILD_TYPE}" CACHE INTERNAL "")

add_executable(${BUILD_TARGET}
    Src/main.c
    Src/stm32_it.c
    Src/led.c
    Log/log.c
    Log/Driver/log_driver.c
)

target_include_directories(${BUILD_TARGET} PRIVATE
    ${CMAKE_SOURCE_DIR}/HAL/Inc
    ${CMAKE_SOURCE_DIR}/HAL/STM32_HAL_Driver/Inc
    ${CMAKE_SOURCE_DIR}/HAL/STM32_HAL_Driver/Inc/Legacy
    ${CMAKE_SOURCE_DIR}/CMSIS/include
    ${CMAKE_SOURCE_DIR}/CMSIS/Device/Include
    ${CMAKE_SOURCE_DIR}/OS
    ${CMAKE_SOURCE_DIR}/OS/include
    Inc
    Log
    Log/Driver
)

target_compile_definitions(${BUILD_TARGET} PRIVATE
    -DSTM32L152xE
    -DUSE_HAL_DRIVER
    -D${SELECTED_BOARD}
    PROJECT_ROOT="${CMAKE_SOURCE_DIR}/"
    $<$<CONFIG:Debug>:DEBUG>
)

target_link_libraries(${BUILD_TARGET} PRIVATE stm32_hal_driver freertos_kernel)

target_compile_options(${BUILD_TARGET} PRIVATE
    -mcpu=${CPU}
    -mthumb
    -mfloat-abi=${FLOAT_ABI}
    -fdata-sections
    -ffunction-sections
    -Wall
    -Wextra
    $<$<CONFIG:Debug>:-Og -g -gdwarf-2>
    $<$<CONFIG:Release>:-O2>
)

target_link_options(${BUILD_TARGET} PRIVATE
    -T${LINKER_FILE}
    -mcpu=${CPU}
    -mthumb
    -mfloat-abi=${FLOAT_ABI}
    -specs=nano.specs
    -specs=nosys.specs
    -Wl,-Map=output.map,--cref
    -Wl,--print-memory-usage
    -Wl,--gc-sections
)

set(TARGET_BIN "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${BUILD_TARGET}.elf" CACHE INTERNAL "")

add_custom_target(flash
    COMMAND openocd -f ${CMAKE_SOURCE_DIR}/config/openocd.cfg -c "setup stm32l1" -c "program_release ${TARGET_BIN}"
    DEPENDS /usr/bin/openocd ${TARGET_BIN}
    VERBATIM
)

add_custom_target(debug
    COMMAND openocd -f ${CMAKE_SOURCE_DIR}/config/openocd.cfg -c "setup stm32l1"
    COMMAND gdb-multiarch --tui --eval-command "target remote :3333" --eval-command "monitor reset halt" --eval-command "load" ${TARGET_BIN}
    DEPENDS /usr/bin/openocd /usr/bin/gdb-multiarch ${TARGET_BIN}
    VERBATIM
)