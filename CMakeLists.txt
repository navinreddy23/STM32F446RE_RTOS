cmake_minimum_required(VERSION 3.14)

project(LED_RTOS_STM32F4)

enable_language(C ASM)
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS OFF)

#Output directory
set (CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/Debug)

#Set the files specific to architecture, board etc.
set(STARTUP startup_stm32f446xx.s)
set(LINKER_FILE STM32F446RETx_FLASH.ld)
set(CPU_ARCH cortex-m4)
set(FREERTOS_ARMCORE_DIR ARM_CM4F)
set(FREERTOS_HEAP heap_4.c)
set(ST_DEVICE_FAMILY STM32F4xx)
set(ST_DEVICE STM32F446xx)
set(ST_BOARD nucleo)

set(DRIVERS_DIR ${CMAKE_SOURCE_DIR}/Drivers/${ST_DEVICE_FAMILY}_HAL_Driver)
set(FREERTOS_DIR ${CMAKE_SOURCE_DIR}/Middlewares/Third_Party/FreeRTOS)
set(CMSIS_DIR Drivers/CMSIS)
set(BSP_DIR ${CMAKE_SOURCE_DIR}/BSP/${ST_DEVICE_FAMILY}-${ST_BOARD})


set(BSP_SRC ${BSP_DIR}/stm32f4xx_nucleo.c)
set(CMSIS_OS_SRC ${FREERTOS_DIR}/Source/CMSIS_RTOS_V2/cmsis_os2.c)
set(CMSIS_OS_HEADER ${FREERTOS_DIR}/Source/CMSIS_RTOS_V2/cmsis_os2.h)

file(GLOB STM32Fx_DRIVER_SRC ${DRIVERS_DIR}/Src/*.c)
list(FILTER STM32Fx_DRIVER_SRC EXCLUDE REGEX ".*template.c$")

file(GLOB SOURCE Src/*.c)

file(GLOB FREERTOS_SOURCE ${FREERTOS_DIR}/Source/*.c)


set(FREERTOS_PORT ${FREERTOS_DIR}/Source/portable/GCC/${FREERTOS_ARMCORE_DIR}/port.c)
set(FREERTOS_HEAP ${FREERTOS_DIR}/Source/portable/MemMang/${FREERTOS_HEAP})

set(EXECUTABLE ${PROJECT_NAME}.out)

add_executable(${EXECUTABLE} ${STARTUP} ${STM32Fx_DRIVER_SRC} ${SOURCE} ${FREERTOS_PORT} ${FREERTOS_SOURCE} ${FREERTOS_HEAP} ${CMSIS_OS_SRC} ${CMSIS_OS_HEADER})

target_compile_definitions(${EXECUTABLE} PRIVATE
        -DUSE_HAL_DRIVER
        -D${ST_DEVICE}
        )

target_include_directories(${EXECUTABLE} PRIVATE
        Inc
	${BSP_DIR}
        ${DRIVERS_DIR}/Inc
        ${CMSIS_DIR}/Device/ST/${ST_DEVICE_FAMILY}/Include
        ${CMSIS_DIR}/Include
	${FREERTOS_DIR}/Source/include
	${FREERTOS_DIR}/Source/portable/GCC/${FREERTOS_ARMCORE_DIR}
	${FREERTOS_DIR}/Source/CMSIS_RTOS_V2/
        )

target_compile_options(${EXECUTABLE} PRIVATE
        -mcpu=${CPU_ARCH}
        -mthumb
        -mfpu=fpv4-sp-d16
        -mfloat-abi=hard

        -fdata-sections
        -ffunction-sections

        -Wall

        $<$<CONFIG:Debug>:-Og>
        )

target_link_options(${EXECUTABLE} PRIVATE
        -T${CMAKE_SOURCE_DIR}/${LINKER_FILE}
        -mcpu=${CPU_ARCH}
        -mthumb
        -mfpu=fpv4-sp-d16
        -mfloat-abi=hard
        -specs=nano.specs
        -lc
        -lm
        -lnosys
        -Wl,-Map=${PROJECT_NAME}.map,--cref
        -Wl,--gc-sections
        )

# Print executable size
add_custom_command(TARGET ${EXECUTABLE}
        POST_BUILD
        COMMAND ${CMAKE_SIZE_UTIL} ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${EXECUTABLE})

# Create hex file
add_custom_command(TARGET ${EXECUTABLE}
        POST_BUILD
        COMMAND ${CMAKE_OBJCOPY} -O ihex ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${EXECUTABLE} ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${PROJECT_NAME}.hex
        COMMAND ${CMAKE_OBJCOPY} -O binary ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${EXECUTABLE} ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${PROJECT_NAME}.bin
	)
