cmake_minimum_required(VERSION 3.10)

# Languages (C & Assembly)
enable_language(C ASM)

# Language Requirements
set(CMAKE_C_STANDARD            99)
set(CMAKE_C_STANDARD_REQUIRED   ON)
set(CMAKE_C_EXTENSIONS          OFF)

# Project Name & Version
project("ProjectName" VERSION "1.0.0")

# CONFIGURATION
# These settings depends on your microcontroller
set(MCU_DEFINE      "STM32F030x6")
set(STARTUP_FILE    "${CMAKE_SOURCE_DIR}/Core/Startup/startup_stm32f030f4px.s")
set(LINKER_FILE     "${CMAKE_SOURCE_DIR}/STM32F030F4PX_FLASH.ld")

set(SOURCES_DIR     "${CMAKE_SOURCE_DIR}/Core/Src")
set(DRIVERS_DIR     "${CMAKE_SOURCE_DIR}/Drivers")
set(CMSIS_DIR       "${DRIVERS_DIR}/CMSIS")
set(DEVICE_DIR      "${DRIVERS_DIR}/CMSIS/Device/ST/STM32F0xx")
set(HAL_DIR         "${DRIVERS_DIR}/STM32F0xx_HAL_Driver")

set(OUTPUT_NAME     "firmware")
set(TARGET_NAME     "${OUTPUT_NAME}.elf")

file(GLOB_RECURSE   SRC_FILES       LIST_DIRECTORIES false "${SOURCES_DIR}/*.c")
file(GLOB_RECURSE   HAL_FILES       LIST_DIRECTORIES false "${HAL_DIR}/Src/*.c")

add_executable(${TARGET_NAME}
    ${STARTUP_FILE}
    ${SRC_FILES}
    ${HAL_FILES}
)

target_compile_definitions(${TARGET_NAME} PRIVATE
    "-DUSE_HAL_DRIVER"
    "-D${MCU_DEFINE}"
)

target_include_directories(${TARGET_NAME} PRIVATE
    "${CMAKE_SOURCE_DIR}/Core/Inc"
    "${CMSIS_DIR}/Include"
    "${DEVICE_DIR}/Include"
    "${HAL_DIR}/Inc"
)

target_compile_options(${TARGET_NAME} PRIVATE
    "-mcpu=cortex-m0"
    "-mthumb"

    "-fdata-sections"
    "-ffunction-sections"
    "-fno-common"

    "-Wall"
    "-Wshadow"
    "-Wformat-overflow"
    "-Wformat-truncation"

    $<$<CONFIG:Debug>:-O0>
    $<$<CONFIG:Release>:-Ofast>
)

target_link_options(${TARGET_NAME} PRIVATE
    "-T${LINKER_FILE}"
    "-mcpu=cortex-m0"
    "-mthumb"
    "-specs=nosys.specs"
    "-specs=nano.specs"
    "-lc"
    "-lm"
    "-Wl,-Map=${OUTPUT_NAME}.map,--cref"
    "-Wl,--gc-sections"         # garbage collect unused code
    "-Wl,--print-memory-usage"
    # "-Wl,--print-gc-sections" # prints garbage collected sections
)

add_custom_command(TARGET ${TARGET_NAME}
    POST_BUILD
    COMMAND arm-none-eabi-size ${TARGET_NAME}
)

add_custom_command(TARGET ${TARGET_NAME}
    POST_BUILD
    COMMAND arm-none-eabi-objcopy -O ihex ${TARGET_NAME} "${OUTPUT_NAME}.hex"   # generate .hex file
    COMMAND echo "[100%] Built target ${OUTPUT_NAME}.hex"
    COMMAND arm-none-eabi-objcopy -O binary ${TARGET_NAME} "${OUTPUT_NAME}.bin" # generate .bin file
    COMMAND echo "[100%] Built target ${OUTPUT_NAME}.bin"
)