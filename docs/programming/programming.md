---
layout: page
title: Programming
nav_order: 1
---

Probmlem with numerating !!!

```cmake
cmake_minimum_required(VERSION 3.16)
set(CMAKE_SYSTEM_NAME Generic)

# turn off compiler checking (assume that the compiler is working):
set(CMAKE_C_COMPILER_WORKS TRUE)
# set C standard:
set(CMAKE_C_STANDARD 17)
# set path where binary files will be saved:
set(BUILD_DIR ${CMAKE_SOURCE_DIR}/bin)
set(EXECUTABLE_OUTPUT_PATH ${BUILD_DIR})

# create targets variables:
set(TARGET "main")
set(TARGET_ELF "${TARGET}.elf")
set(TARGET_HEX "${TARGET}.hex")
set(TARGET_BIN "${TARGET}.bin")

# create project name and define required languages for building (C and assembler - ASM should be at the end):
project(MY_PROJECT C ASM)

# assign paths into variables:
set(MAIN_SRC_DIR "${CMAKE_SOURCE_DIR}/Src")

set(LINKER_DIR "${CMAKE_SOURCE_DIR}/link")
set(LD_include "-lnosys -L${LINKER_DIR}")
set(linker_script "${LINKER_DIR}/STM32F411RETX_FLASH.ld")

set(MCU_flags   "-mcpu=cortex-m4 -mfpu=fpv4-sp-d16 -mfloat-abi=hard -mthumb ")

# C definitions (additional arguments parse with cmd line):
set(C_DEFS " -DSTM32F411xE ")
# C-specific flags:
set(C_flags     "${MCU_flags} ${C_DEFS} -Wall -fdata-sections -ffunction-sections -fanalyzer ")
# Assembler-specific flags:
set(AS_flags    "${MCU_flags} -Wall -fdata-sections -ffunction-sections ")
# Linker's flags:
set(LD_flags    "${MCU_flags} -specs=nano.specs -specs=nosys.specs -T${linker_script} ${LD_include} -Wl,--print-memory-usage -u _printf_float ")

# CMake variables setup:
set(CMAKE_C_FLAGS "${C_flags}")
set(CMAKE_ASM_FLAGS "${AS_flags}")
set(CMAKE_EXE_LINKER_FLAGS "${LD_flags}")

# add all your executable files:
add_executable(${TARGET_ELF}
    Src/startup/startup_stm32f411retx.s
    Src/startup/system_stm32f4xx.c
    Src/main.c
)

# include all directories where header files occur:
target_include_directories(${TARGET_ELF} PUBLIC
    Src
    Src/startup
    Drivers
    Drivers/Include
)

# link GNU c and m ("math") libraries (more here: https://www.gnu.org/software/libc/manual/pdf/libc.pdf):
target_link_libraries(${TARGET_ELF} PUBLIC c m)

# set shortcut for command:
set(OBJCOPY arm-none-eabi-objcopy)
# make new targets .hex and .bin from .elf file:
add_custom_target(${TARGET_BIN} ALL COMMAND ${OBJCOPY} -O binary -S ${BUILD_DIR}/${TARGET_ELF} ${BUILD_DIR}/${TARGET_BIN})
add_custom_target(${TARGET_HEX} ALL COMMAND ${OBJCOPY} -O ihex -S ${BUILD_DIR}/${TARGET_ELF} ${BUILD_DIR}/${TARGET_HEX})

# define dependencies so that .hex file is created after .elf and .bin as the last one:
add_dependencies( ${TARGET_HEX} ${TARGET_ELF})
add_dependencies(${TARGET_BIN} ${TARGET_ELF} ${TARGET_HEX})
```
TESTS
TEST