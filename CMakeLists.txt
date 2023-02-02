cmake_minimum_required(VERSION 3.13)

# set the project name
project(test C ASM)

set(CMAKE_VERBOSE_MAKEFILE ON)

include(CMakeForceCompiler)
set(CMAKE_SYSTEM_NAME               Generic)
set(CMAKE_SYSTEM_PROCESSOR          aarch64)
set(BAREMETAL_ARM_TOOLCHAIN_PATH	"/opt/arm-gnu-toolchain/bin/")

set(BINARY_PREFIX   "aarch64-linux-gnu")

# path to toolchain binaries
set(CMAKE_AR                    ${BINARY_PREFIX}-ar) #${BAREMETAL_ARM_TOOLCHAIN_PATH}
set(CMAKE_ASM_COMPILER          ${BINARY_PREFIX}-gcc)
set(CMAKE_C_COMPILER            ${BINARY_PREFIX}-gcc)
#set(CMAKE_CXX_COMPILER          <path_to_cpp_compiler)
set(CMAKE_LINKER                ${BINARY_PREFIX}-ld)
#set(CMAKE_OBJCOPY               <path_to_objcopy>)
#set(CMAKE_RANLIB                <path_to_ranlib>)
#set(CMAKE_SIZE                  <path_to_size>)
#set(CMAKE_STRIP                 <path_to_strip>)


add_definitions(-DGUEST)
set ( CMAKE_C_FLAGS "-c -mcpu=cortex-a57 -ffreestanding -Wall -Wextra -g")
# set ( STACK_SIZE 0xf00 )
set ( TARGET test )

# .S ASM files
set_property(SOURCE startup.S APPEND PROPERTY LANGUAGE C)
set_property(SOURCE FreeRTOS_asm_vectors.S APPEND PROPERTY LANGUAGE C)


include_directories(Source/include)
include_directories(include)
include_directories(Source/portable/GCC/ARM_CA57_64_BIT)


set(DEMO_SOURCES main.c 
                 startup.S
                 FreeRTOS_asm_vectors.S
                 FreeRTOS_tick_config.c
                 vectors.c
                 exception.c
                 sysctrl.c
                 pstate.c
                 gic_v3.c
                 uart.c
                 printf-stdarg.c
                 example.c
                 nostdlib.c
                )


file(GLOB FREERTOS_SOURCES "Source/queue.c" "Source/list.c" "Source/tasks.c" "Source/timers.c")
file(GLOB FREERTOS_PORTABLE_SOURCES "Source/portable/GCC/ARM_CA57_64_BIT/*.c" "Source/portable/GCC/ARM_CA57_64_BIT/*.S")
file(GLOB FREERTOS_MEM_SOURCES "Source/portable/MemMang/heap_1.c")

#add_subdirectory(include) # TODO: rename to a more suitable ex-config
#add_subdirectory(Source)

add_executable(${TARGET}.elf ${FREERTOS_SOURCES} ${FREERTOS_MEM_SOURCES} ${FREERTOS_PORTABLE_SOURCES} ${DEMO_SOURCES})

SET(LINKER_SCRIPT "${CMAKE_CURRENT_SOURCE_DIR}/linker.ld")
target_link_options(${TARGET}.elf PUBLIC "LINKER:-T${LINKER_SCRIPT}")
target_link_libraries(${TARGET}.elf)



#set_target_properties(${TARGET}.elf PROPERTIES LINK_DEPENDS qemu_aarch64_virt_linkerscript)