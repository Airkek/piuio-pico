cmake_minimum_required(VERSION 3.13)

project(piuio_pico)

if(NOT WIN32) # colors for cmake
    string(ASCII 27 Esc)
    set(ColourReset "${Esc}[m")
    set(ColourBold  "${Esc}[1m")
    set(Red         "${Esc}[31m")
    set(Green       "${Esc}[32m")
    set(Yellow      "${Esc}[33m")
    set(Blue        "${Esc}[34m")
    set(Magenta     "${Esc}[35m")
    set(Cyan        "${Esc}[36m")
    set(White       "${Esc}[37m")
    set(BoldRed     "${Esc}[1;31m")
    set(BoldGreen   "${Esc}[1;32m")
    set(BoldYellow  "${Esc}[1;33m")
    set(BoldBlue    "${Esc}[1;34m")
    set(BoldMagenta "${Esc}[1;35m")
    set(BoldCyan    "${Esc}[1;36m")
    set(BoldWhite   "${Esc}[1;37m")
endif()

if (NOT DEFINED BUTTON_BOARD)
    set(BUTTON_BOARD 0)
    message("${Yellow}BUTTON_BOARD is not defined. Processing with BUTTON_BOARD=0${ColourReset}")
elseif(NOT DEFINED CAB_LIGHTS_WS2812)
    set(CAB_LIGHTS_WS2812 0)
    message("${Yellow}To enable cabinet WS2812 lights, add ${BoldMagenta}-DCAB_LIGHTS_WS2812=1 ${Yellow}to cmake parameters${ColourReset}")
endif()

if (BUTTON_BOARD)
    add_compile_definitions(ENABLE_BUTTON_BOARD)
    message("${Blue}Firmware type is ${BoldCyan}ButtonBoard${ColourReset}")
    message("${Blue}Cabinet lights ${BoldRed}are not available${ColourReset}")
else()
    message("${Blue}Firmware type is ${BoldCyan}PIUIO${ColourReset}")
    if (CAB_LIGHTS_WS2812)
        add_compile_definitions(ENABLE_WS2812_SUPPORT)
        message("${Blue}Cabinet lights ${BoldGreen}enabled ${Cyan}(WS2812)${ColourReset}")
    else()
        message("${Blue}Cabinet lights ${BoldRed}disabled${ColourReset}")
    endif()

endif()

set(PICO_SDK_FETCH_FROM_GIT ON)
include(pico_sdk_import.cmake)
pico_sdk_init()

add_executable(piuio_pico
    main.c)

target_sources(piuio_pico PUBLIC
        ${CMAKE_CURRENT_LIST_DIR}/main.c
        ${CMAKE_CURRENT_LIST_DIR}/piuio_ws2812.c
        ${CMAKE_CURRENT_LIST_DIR}/usb_descriptors.c
        )

# Make sure TinyUSB can find tusb_config.h
target_include_directories(piuio_pico PUBLIC
        ${CMAKE_CURRENT_LIST_DIR})

pico_generate_pio_header(piuio_pico ${CMAKE_CURRENT_LIST_DIR}/ws2812.pio)

# In addition to pico_stdlib required for common PicoSDK functionality, add dependency on tinyusb_device
# for TinyUSB device support and tinyusb_board for the additional board support library used by the example
target_link_libraries(piuio_pico
                      PUBLIC
                      pico_stdlib
                      pico_multicore
                      tinyusb_board
                      tinyusb_device
                      hardware_pio
                      hardware_dma
                      hardware_irq)

# Uncomment this line to enable fix for Errata RP2040-E5 (the fix requires use of GPIO 15)
#target_compile_definitions(piuio_pico PUBLIC PICO_RP2040_USB_DEVICE_ENUMERATION_FIX=1)

pico_add_extra_outputs(piuio_pico)
pico_enable_stdio_usb(piuio_pico 0)
pico_enable_stdio_uart(piuio_pico 0)
pico_add_extra_outputs(piuio_pico)
