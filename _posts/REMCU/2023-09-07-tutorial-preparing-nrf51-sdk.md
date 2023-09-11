---
title: Preparing nRF51 Drivers SDK for MCU's Peripheral Forwarding
date: 2023-09-07
categories: [REMCU]
tags: [peripherals, driver, llvm, mcu, debugger, toolchain]     # TAG names should always be lowercase
description: "How to prepare, modify, and build your MCU's SDK for MCU's peripheral forwardin, using the NRF51 SDK as an example."
pin: true
image:
  path: /assets/remcu/preview.png
  alt: MCU's Peripheral Forwarding
---

In my previous article, we explored the intricacies of [MCU's Peripheral Forwarding technology](https://remotemcu.com/chip-peripheral-forwarding.html). In this paper, we will put theory into practice as we provide a hands-on illustration of this concept. Our focus will be on preparing a driver SDK for use in your PC application. We'll turn to the NRF51 SDK as our primary example, accompanied by the REMCU project.

We embark on a journey to construct a shared/dynamic REMCU library for the NRF51422 driver SDK. By doing so, you'll gain the capability to effortlessly link this library into your application and harness the power of driver functions to  control over NRF peripherals from your application

## Preparing nRF51 SDK

Download the nRF5 SDK for your MCU from the official [Nordic Semiconductor website](https://www.nordicsemi.com/-/media/Software-and-other-downloads/SDKs/nRF5/Binaries/nRF5SDK1230.zip).

Extract the nRF5 SDK 12.3.0 archive, you'll find a directory structure that resembles the following:

```bash
$ tree -d -L 2 nRF5_SDK_12.3.0_d7731ad/
nRF5_SDK_12.3.0_d7731ad/
├── components
│   ├── ant
│   ├── ble
│   ├── boards
│   ├── device
│   ├── drivers_ext
│   ├── drivers_nrf
│   ├── libraries
│   ├── nfc
│   ├── proprietary_rf
│   ├── serialization
│   ├── softdevice
│   └── toolchain
├── documentation
├── examples
│   ├── ant
│   ├── ble_central
│   ├── ble_central_and_peripheral
│   ├── ble_peripheral
│   ├── crypto
│   ├── dfu
│   ├── dtm
│   ├── multiprotocol
│   ├── nfc
│   ├── peripheral
│   └── proprietary_rf
├── external
│   ├── cifra_AES128-EAX
│   ├── fatfs
│   ├── freertos
│   ├── micro-ecc
│   ├── nano-pb
│   ├── nfc_adafruit_library
│   ├── nrf_cc310
│   ├── protothreads
│   ├── rtx
│   ├── segger_rtt
│   └── tiny-AES128
└── svd
```

We'll streamline the structure of the nRF5 SDK by removing the RF/BLE stack code and external code(freertos, rtx, fatfs etc). This will help us focus on the drivers and examples.

After making these adjustments, your refined directory structure might look like this:

```bash
$ tree  -L 2 nRF5_SDK_12.3.0_d7731ad/
nRF5_SDK_12.3.0_d7731ad/
├── components
│   ├── boards
│   ├── device
│   ├── drivers_nrf
│   ├── libraries
│   ├── sdk_validation.h
│   └── toolchain
├── examples
│   └── peripheral
└── license.txt
```

## Cmake Build Script

We'll prepare a cmake build script to compile the REMCU shared library using the drivers from the nRF5 SDK for the nRF51422 MCU. For the template of the build script, we'll use a similar script from an [STM32 MCU build script](https://github.com/remotemcu/remcu-chip-sdks/blob/master/stm32/stm32f3/STM32F3-Discovery_FW_V1.1.0/CMakeLists.txt). Сreate a folder named [NRF51422](https://github.com/remotemcu/remcu-chip-sdks/tree/master/nordicsemi/nRF5_SDK_12.3.0_d7731ad/NRF51422) within the SDK directory to place the necessary build script.

```bash
$ tree -L 2 nRF5_SDK_12.3.0_d7731ad/
nRF5_SDK_12.3.0_d7731ad/
├── CMakeLists.txt
├── components
│   ├── boards
│   ├── device
│   ├── drivers_nrf
│   ├── libraries
│   ├── sdk_validation.h
│   └── toolchain
├── examples
│   └── peripheral
├── license.txt
└── NRF51422
    ├── CMakeLists.txt
```

Start by modifying the [CMakeLists.txt](https://github.com/remotemcu/remcu-chip-sdks/blob/master/nordicsemi/nRF5_SDK_12.3.0_d7731ad/NRF51422/CMakeLists.txt) file to define the basic variables such as the library name, major version, and minor version.

```cmake
set(MCU_TYPE nRF51422)

set(MCU_LIB_NAME SDK)

set(MCU_MAJOR_VERSION_LIB V12.3.0)

set(MCU_MINOR_VERSION_LIB 01)
```

Defining the `MCU_SDK_PATH` variable is crucial for specifying the path to the source of the drivers SDK

```cmake
set(MCU_SDK_PATH ${CMAKE_CURRENT_SOURCE_DIR}/..)
```

This below CMake code snippet is used to include a build script and install a header file with defines for NRF51422 build. This code snippet doesn't require modifications  for our purpose.

```cmake
include(${REMCU_VM_PATH}/cmake/mcu_build_target.cmake)

file(INSTALL "${CMAKE_CURRENT_SOURCE_DIR}/defines_${MCU_TYPE}.h" 
      DESTINATION ${ALL_INCLUDE_DIR}
      )

file(RENAME "${ALL_INCLUDE_DIR}/defines_${MCU_TYPE}.h"
            ${ALL_INCLUDE_DIR}/device_defines.h
      )
```

Excluding the installation of the Python script is unnecessary for us, as we lack the Python file containing export functions and defines. 

```cmake
file(INSTALL "${CMAKE_CURRENT_SOURCE_DIR}/${MCU_TYPE}_${MCU_LIB_NAME}.py"
      DESTINATION ${ALL_INCLUDE_DIR}
      )
```

In my upcoming article, I will delve into the process of exporting C functions and defines to a Python file and then demonstrate how to effectively execute them using [ctypes](https://docs.python.org/3/library/ctypes.html). To illustrate this concept, example here:

[Jupyter notebook](https://github.com/remotemcu/remcu_examples/blob/master/stm32f4_discovery/jupyter-notebook/stm32f4_discovery_python.ipynb)

[![](https://raw.githubusercontent.com/remotemcu/remcu_examples/b18752669af243770f67484e8801e5b16bfc8d87/stm32f4_discovery/jupyter-notebook/img/f4_adc.gif)](https://github.com/remotemcu/remcu_examples/blob/master/stm32f4_discovery/jupyter-notebook/stm32f4_discovery_python.ipynb)

## Setup address intervals

Modifying the [conf.cpp](https://github.com/remotemcu/remcu-chip-sdks/blob/master/nordicsemi/nRF5_SDK_12.3.0_d7731ad/NRF51422/conf.cpp) file to set address intervals for MCU peripherals is an essential step in configuring your project. Address intervals define the memory regions that REMCU library can access to communicate with MCU peripherals. Here's how you might approach this:

The `add_to_mem_interval` function is used to define the RAM region. Both STM32 and NRF51422 share a common RAM region, so modifications are not necessary.

The `add_to_adin_interval` function  is used to define  the peripheral address regions. Let's take a look at the NRF51422 datasheet to find appropriate values for setting the peripheral address regions:

![NRF51 memory map](/assets/remcu/nrf51_memory_map.png)

Let's proceed by setting the peripheral address intervals for `AHB peripherals` and `APB peripherals` buses on the **NRF51422**. These buses encompass general peripherals like GPIO, ADC, etc. without considering the RF-related peripheral.

```c
add_to_mem_interval(0x20000000, 0x20000000 + 8*1024); //SRAM  8k
add_to_adin_interval(0x40000000,  0x40008000); //APB peripherals
add_to_adin_interval(0x50000000,  0x50060000); //AHB peripherals
add_to_adin_interval(0xF0000FE0,  0xF0000FE8 + 4); //PAN 26 "System: Manual setup is required to enable the use of peripherals"
```

The last interval that is required for the [systeminit](https://github.com/remotemcu/remcu-chip-sdks/blob/master/nordicsemi/nRF5_SDK_12.3.0_d7731ad/components/toolchain/system_nrf51.c#L62) function on NRF51422. This interval is essential for the `systeminit` function to work properly, as it writes to certain addresses during system initialization.  There's more detailed information in the [code comments of the function](https://github.com/remotemcu/remcu-chip-sdks/blob/master/nordicsemi/nRF5_SDK_12.3.0_d7731ad/components/toolchain/system_nrf51.c#L62).

```c
uint32_t get_RAM_addr_for_test(){
    return 0x20000000;
}
```

The `get_RAM_addr_for_test` function returns the starting address of the RAM region, and this value is utilized for debugger testing. This is a common approach to work around potential issues with certain debuggers that might not work correctly with OpenOCD and GDB server.

```c
/**
 * @brief remcu_debuggerTest
 * Performs test of debugger and debug server while mcu is connected.
 * @return If no error occurs, the function returns NULL
 * else the function returns error message (char array)
 * Don't free the pointer after use!
 * Note:  Invoke the function after establishing a connection with the debugger
 * through the successful utilization of either the
 * remcu_connect2OpenOCD or remcu_connect2GDB functions.
 */
REMCULIB_DLL_API const char* remcu_debuggerTest();
```

 [remcu_debuggerTest](https://github.com/remotemcu/remcu/blob/master/export/remcu.h#L161) function is executed to perform debugger testing. Running such a test can be valuable for verifying that the debugger works correctly in conjunction with REMCU and the specific memory regions you've defined. It also aids in identifying any potential issues or inconsistencies that might arise during debugging sessions. Invoke the function after establishing a connection with the debugger through the successful utilization of either the `remcu_connect2OpenOCD` or `remcu_connect2GDB` functions.


## Defines

The subsequent task involves crafting the `defines_${MCU_TYPE}.h` file, which, in our instance, is named `defines_nRF51422.h`. The file contains essential defines required for the SDK build. These defines must be preserved to ensure they are available when your library is utilized. For example, consider a scenario where an driver SDK's header includes the following code:

```c
#ifdef OPTION1
	void foo();
#endif
```

To access the `foo` function, the `OPTION1` define must be present. Therefore, it's crucial to maintain the necessary defines from `defines_${MCU_TYPE}.h` to guarantee the correct behavior of the SDK functions in your project.

Looking at the [Makefile](https://github.com/remotemcu/remcu-chip-sdks/blob/master/nordicsemi/nRF5_SDK_12.3.0_d7731ad/examples/peripheral/adc/pca10028/blank/armgcc/Makefile) of NRF SDK example can provide valuable insight into the required defines for building for a specific target chip like NRF51422. Let's take a look at an example, such as the [ADC example,](https://github.com/remotemcu/remcu-chip-sdks/blob/master/nordicsemi/nRF5_SDK_12.3.0_d7731ad/examples/peripheral/adc/pca10028/blank/armgcc/Makefile) to identify these required defines.

```makefile
CFLAGS += -DNRF51
CFLAGS += -DNRF51422
CFLAGS += -DBOARD_PCA10028
CFLAGS += -DBSP_DEFINES_ONLY
```

Write the found defines to [defines_nRF51422.h](https://github.com/remotemcu/remcu-chip-sdks/blob/master/nordicsemi/nRF5_SDK_12.3.0_d7731ad/NRF51422/defines_nRF51422.h) file:

```c
#define REMCU_LIB

#define NRF51

#define NRF51422

#define BOARD_PCA10028

#define BSP_DEFINES_ONLY
```

### Makefiles

We can identify the required SDK source and header files by examining the [Makefile](https://github.com/remotemcu/remcu-chip-sdks/blob/master/nordicsemi/nRF5_SDK_12.3.0_d7731ad/examples/peripheral/adc/pca10028/blank/armgcc/Makefile) of an NRF SDK example. The Makefile typically specifies the source and header files that are part of the example. These files are the ones you'll need to include in your REMCU library project to ensure compatibility and proper integration.

```makefile
# Source files common to all targets
SRC_FILES += \
  $(SDK_ROOT)/components/libraries/log/src/nrf_log_backend_serial.c \
  $(SDK_ROOT)/components/libraries/log/src/nrf_log_frontend.c \
  $(SDK_ROOT)/components/libraries/util/app_error.c \
  $(SDK_ROOT)/components/libraries/util/app_error_weak.c \
  $(SDK_ROOT)/components/libraries/util/app_util_platform.c \
  $(SDK_ROOT)/components/libraries/util/nrf_assert.c \
  $(SDK_ROOT)/components/libraries/util/sdk_errors.c \
  $(SDK_ROOT)/components/boards/boards.c \
  $(SDK_ROOT)/components/drivers_nrf/hal/nrf_adc.c \
  $(SDK_ROOT)/components/drivers_nrf/adc/nrf_drv_adc.c \
  $(SDK_ROOT)/components/drivers_nrf/common/nrf_drv_common.c \
  $(SDK_ROOT)/components/drivers_nrf/uart/nrf_drv_uart.c \
  $(PROJ_DIR)/main.c \
  $(SDK_ROOT)/external/segger_rtt/RTT_Syscalls_GCC.c \
  $(SDK_ROOT)/external/segger_rtt/SEGGER_RTT.c \
  $(SDK_ROOT)/external/segger_rtt/SEGGER_RTT_printf.c \
  $(SDK_ROOT)/components/toolchain/gcc/gcc_startup_nrf51.S \
  $(SDK_ROOT)/components/toolchain/system_nrf51.c \

# Include folders common to all targets
INC_FOLDERS += \
  $(SDK_ROOT)/components \
  $(SDK_ROOT)/components/libraries/util \
  $(SDK_ROOT)/components/toolchain/gcc \
  $(SDK_ROOT)/components/drivers_nrf/uart \
  ../config \
  $(SDK_ROOT)/components/drivers_nrf/common \
  $(SDK_ROOT)/components/drivers_nrf/adc \
  $(PROJ_DIR) \
  $(SDK_ROOT)/external/segger_rtt \
  $(SDK_ROOT)/components/libraries/bsp \
  $(SDK_ROOT)/components/drivers_nrf/nrf_soc_nosd \
  $(SDK_ROOT)/components/toolchain \
  $(SDK_ROOT)/components/device \
  $(SDK_ROOT)/components/libraries/log \
  $(SDK_ROOT)/components/boards \
  $(SDK_ROOT)/components/drivers_nrf/delay \
  $(SDK_ROOT)/components/toolchain/cmsis/include \
  $(SDK_ROOT)/components/drivers_nrf/hal \
  $(SDK_ROOT)/components/libraries/log/src \
```

The file list might initially include files related to debugging, logging, and assembler functionalities. However, to streamline the integration process and optimize the build, it's a prudent approach to prune the source list by excluding these components. By removing files associated with debugging, logging, and assembler utilities, you ensure that your REMCU project remains focused on essential drivers and functionalities.

```makefile
SRC_FILES +=  \
  $(SDK_ROOT)/components/boards/boards.c \
  $(SDK_ROOT)/components/drivers_nrf/hal/nrf_adc.c \
  $(SDK_ROOT)/components/drivers_nrf/adc/nrf_drv_adc.c \
  $(SDK_ROOT)/components/drivers_nrf/common/nrf_drv_common.c \
  $(SDK_ROOT)/components/toolchain/system_nrf51.c \

INC_FOLDERS =  \
  $(SDK_ROOT)/components/libraries/util \
  ../config \
  $(SDK_ROOT)/components/drivers_nrf/common \
  $(SDK_ROOT)/components/drivers_nrf/adc \
  $(SDK_ROOT)/components/drivers_nrf/nrf_soc_nosd \
  $(SDK_ROOT)/components/toolchain \
  $(SDK_ROOT)/components/device \
  $(SDK_ROOT)/components/libraries/log \
  $(SDK_ROOT)/components/boards \
  $(SDK_ROOT)/components/toolchain/cmsis/include \
  $(SDK_ROOT)/components/drivers_nrf/hal \
  $(SDK_ROOT)/components/libraries/log/src \
```

Craft a [Makefile](https://github.com/remotemcu/remcu-chip-sdks/blob/master/nordicsemi/nRF5_SDK_12.3.0_d7731ad/NRF51422/Makefile) tailored for the source and header files is a pivotal step in constructing the SDK portion of the REMCU library. These files are first instrumented and compiled to suit the desired functionalities. Subsequently, they seamlessly integrate into the REMCU library build, ensuring a comprehensive incorporation of the nRF5 SDK components.

```makefile
SDK_ROOT := $(MCU_SDK_PATH)

C_SRC +=  \
  $(SDK_ROOT)/components/boards/boards.c \
  $(SDK_ROOT)/components/drivers_nrf/hal/nrf_adc.c \
  $(SDK_ROOT)/components/drivers_nrf/adc/nrf_drv_adc.c \
  $(SDK_ROOT)/components/drivers_nrf/common/nrf_drv_common.c \
  $(SDK_ROOT)/components/toolchain/system_nrf51.c \

INC_PATH =  \
  $(SDK_ROOT)/components/libraries/util \
  $(SDK_ROOT)/examples/peripheral/adc/pca10028/blank/config \
  $(SDK_ROOT)/components/drivers_nrf/common \
  $(SDK_ROOT)/components/drivers_nrf/adc \
  $(SDK_ROOT)/components/drivers_nrf/nrf_soc_nosd \
  $(SDK_ROOT)/components/toolchain \
  $(SDK_ROOT)/components/device \
  $(SDK_ROOT)/components/libraries/log \
  $(SDK_ROOT)/components/boards \
  $(SDK_ROOT)/components/toolchain/cmsis/include \
  $(SDK_ROOT)/components/drivers_nrf/hal \
  $(SDK_ROOT)/components/libraries/log/src \

# TOOLCHAIN OPTIONS
#-------------------------------------------------------------------------------

DEFS += -DNRF51 -DNRF51422 -DBOARD_PCA10028 -DBSP_DEFINES_ONLY

include $(TARGET_MK)
```

The `C_SRC` and `INC_PATH` variables play a role in specifying the list of C source and header files for integration. If your project includes C++ files, it's essential to utilize the `CPP_SRC` variable to accommodate these files appropriately. These variables are utilized within the included [makefile script](https://github.com/remotemcu/remcu/blob/master/mcu_utils/common.mk):


```makefile
include $(TARGET_MK)
```

orchestrating the seamless integration of the designated source and header files. 

The `DEFS` variable serves as a repository for the discovered defines mentioned earlier

In our Makefile setup, the `MCU_SDK_PATH` variable is injected, leveraging its prior definition in the [CMakeLists.txt.](https://github.com/remotemcu/remcu-chip-sdks/blob/nordic/nordicsemi/nRF5_SDK_12.3.0_d7731ad/NRF51422/CMakeLists.txt#L11) This variable, which holds the path to the NRF SDK, obviates the need to manually specify the absolute path.

```makefile
SDK_ROOT := $(MCU_SDK_PATH)
```

## Build

Now, we can attempt to build the library using our custom build script. To simplify the process, we'll utilize a prepared [Docker image](https://hub.docker.com/r/sermkd/remcu_builder) that includes all the necessary tools. This image is designed to streamline the build process for Linux-based system. More info in [README](https://github.com/remotemcu/remcu-chip-sdks#docker-way)

```bash
docker pull sermkd/remcu_builder
```

For users operating on a Windows OS, please follow [this instruction](https://github.com/remotemcu/remcu-chip-sdks#windows-os). Macos [this instruction](https://github.com/remotemcu/remcu-chip-sdks#without-docker). Alternatively, you can utilize [Github Actions](#build-with-github-action) to effortlessly build the library for various platforms, offering a straightforward and hassle-free approach.

I've organized the NRF SDK sources and integrated them into a repository alongside other SDKs that are capable of compiling the REMCU library. Clone it:

```bash
git clone --recurse-submodules https://github.com/remotemcu/remcu-chip-sdks.git
cd remcu-chip-sdks
git checkout 8ee6eb05ed1e584f20f108caf19b08802de23458 
```

Run docker container:

```bash
docker run -it --name remcu-build-docker -v $PWD/remcu-chip-sdks:/remcu-chip-sdks -w /remcu-chip-sdks remcu_builder
```

In the docker container, go to [NRF51422](https://github.com/remotemcu/remcu-chip-sdks/tree/8ee6eb05ed1e584f20f108caf19b08802de23458/nordicsemi/nRF5_SDK_12.3.0_d7731ad/NRF51422) folder and try to build it:

```bash
cd /remcu-mcu-sdks/nordicsemi/nRF5_SDK_12.3.0_d7731ad/NRF51422
mkdir build
cd build
cmake .. -DCMAKE_TOOLCHAIN_FILE=/remcu-mcu-sdks/REMCU/platform/linux_x64.cmake
make
```

The `DCMAKE_TOOLCHAIN_FILE` variable retains the path to the toolchain file that's essential for your particular platform. [A array of toolchain files](https://github.com/remotemcu/remcu/tree/master/platform) is accessible, catering to diverse platforms. Notably, within the Docker container environment, you have the capability to build for both Linux and Raspberry platforms.

After `make` command, we get error:

```bash
/nordicsemi/nRF5_SDK_12.3.0_d7731ad/NRF51422/../components/libraries/util/app_error.h:128:8: error: unknown type name '__INLINE'
static __INLINE void app_error_log(uint32_t id, uint32_t pc, uint32_t info)
```

Upon investigating, we identified that the type name `__INLINE` is defined within the `compiler_abstraction.h` header file, which is included in the `nrf.h` header file. However, when compiling for the PC host, Nordic is unable to include the necessary header files. We can [modify the header file](https://github.com/remotemcu/remcu-chip-sdks/commit/787ebf08bd284dee9ac24136269b461031277def) by incorporating a conditional define called `REMCU_LIB`, which passed during the REMCU project build.

```c
#ifndef REMCU_LIB
    #define NO_REMCU_LIB
#endif //REMCU_LIB

#if defined(_WIN32) && defined(NO_REMCU_LIB)
    /* Do not include nrf specific files when building for PC host */
#elif defined(__unix) && defined(NO_REMCU_LIB)
    /* Do not include nrf specific files when building for PC host */
#elif defined(__APPLE__) && defined(NO_REMCU_LIB)
    /* Do not include nrf specific files when building for PC host */
#else

    /* Device selection for device includes. */
    #if defined (NRF51)
        #include "nrf51.h"
        #include "nrf51_bitfields.h"
        #include "nrf51_deprecated.h"
    #elif defined (NRF52840_XXAA)
        #include "nrf52840.h"
        #include "nrf52840_bitfields.h"
        #include "nrf51_to_nrf52840.h"
        #include "nrf52_to_nrf52840.h"
    #elif defined (NRF52832_XXAA)
        #include "nrf52.h"
        #include "nrf52_bitfields.h"
        #include "nrf51_to_nrf52.h"
        #include "nrf52_name_change.h"
    #else
        #error "Device must be defined. See nrf.h."
    #endif /* NRF51, NRF52832_XXAA, NRF52840_XXAA */

    #include "compiler_abstraction.h"

#endif /* _WIN32 || __unix || __APPLE__ */
```

Attempt the library build once more, and this time, you'll find that the REMCU library is successfully constructed, yielding the [libremcu.so](https://github.com/remotemcu/prebuilt_libraries/blob/master/NordSemi/nRF51422-SDK-V12.3.0-01/Linux_x64/libremcu.so) file.

The next step involves exporting header files for your library. These header files are drawn from the SDK and contain essential defines and driver functions that the library provides. By exporting these header files, enabling users to run driver SDK's functions

Get the header files list from Makefile:

```
  $(SDK_ROOT)/components/libraries/util \
  $(SDK_ROOT)/examples/peripheral/adc/pca10028/blank/config \
  $(SDK_ROOT)/components/drivers_nrf/common \
  $(SDK_ROOT)/components/drivers_nrf/adc \
  $(SDK_ROOT)/components/drivers_nrf/nrf_soc_nosd \
  $(SDK_ROOT)/components/toolchain \
  $(SDK_ROOT)/components/device \
  $(SDK_ROOT)/components/libraries/log \
  $(SDK_ROOT)/components/boards \
  $(SDK_ROOT)/components/toolchain/cmsis/include \
  $(SDK_ROOT)/components/drivers_nrf/hal \
  $(SDK_ROOT)/components/libraries/log/src \
```

 and use it in CmakeLists.txt

```cmake
file(INSTALL 
      "${MCU_SDK_PATH}/components/libraries/util/"
      "${MCU_SDK_PATH}/components/drivers_nrf/common/"
      "${MCU_SDK_PATH}/components/drivers_nrf/adc/"
      "${MCU_SDK_PATH}/components/drivers_nrf/nrf_soc_nosd/"
      "${MCU_SDK_PATH}/components/toolchain/"
      "${MCU_SDK_PATH}/components/device/"
      "${MCU_SDK_PATH}/components/libraries/log/"
      "${MCU_SDK_PATH}/components/boards/"
      "${MCU_SDK_PATH}/components/toolchain/cmsis/include/"
      "${MCU_SDK_PATH}/components/drivers_nrf/hal/"
      "${MCU_SDK_PATH}/components/libraries/log/src/"
      "${MCU_SDK_PATH}/examples/peripheral/adc/pca10028/blank/config/"
      DESTINATION ${ALL_INCLUDE_DIR}
      FILES_MATCHING PATTERN "*.h"
      )
```

All the exported header files derived from this list are situated within the `remcu_include` [folder](https://github.com/remotemcu/prebuilt_libraries/tree/master/NordSemi/nRF51422-SDK-V12.3.0-01/Linux_x64/remcu_include).


## Test. ADC Example


When building a dynamic library, it's possible to inadvertently overlook dependencies for executable binaries. As a means of verification, let's construct a test that also serves as an illustrative example of interacting with the ADC peripheral. Add a sub directory to [CMakeLists.txt](https://github.com/remotemcu/remcu-chip-sdks/blob/master/nordicsemi/nRF5_SDK_12.3.0_d7731ad/NRF51422/CMakeLists.txt#L42)

```cmake
add_subdirectory(test)
```

Create a subdirectory and generate the necessary [CMakeLists.txt](https://github.com/remotemcu/remcu-chip-sdks/blob/master/nordicsemi/nRF5_SDK_12.3.0_d7731ad/NRF51422/test/CMakeLists.txt) file and [main.c](https://github.com/remotemcu/remcu-chip-sdks/blob/nordic/nordicsemi/nRF5_SDK_12.3.0_d7731ad/NRF51422/test/main.c) source code for our test/example. This source code draws inspiration from an [ADC example](https://github.com/remotemcu/remcu-chip-sdks/blob/master/nordicsemi/nRF5_SDK_12.3.0_d7731ad/examples/peripheral/adc/main.c) found within the NRF SDK. Notably, I've omitted any non-driver related header files, swapped out the SDK's logging functions for `printf`, excluded any assembler code, and transformed data reception from an interrupt-driven approach to a polling mechanism. The interrupts are managed by the MCU's processor and are not directly accessible on our PC host.

```shell
$ tree -L 2 nRF5_SDK_12.3.0_d7731ad/NRF51422/
nRF5_SDK_12.3.0_d7731ad/NRF51422/
├── build
│   ├──...
├── conf.cpp
├── defines_nRF51422.h
├── Makefile
└── test
    ├── CMakeLists.txt
    └── main.c

```

After clearing the build directory, proceed to execute the cmake and make commands once again. The entire build process completes without encountering any errors.

```bash
root@55fb5fbdda6d:/remcu-mcu-sdks/nordicsemi/nRF5_SDK_12.3.0_d7731ad/NRF51422/build# rm -rf *
root@55fb5fbdda6d:/remcu-mcu-sdks/nordicsemi/nRF5_SDK_12.3.0_d7731ad/NRF51422/build# cmake .. -DCMAKE_TOOLCHAIN_FILE=/build/AddressInterceptorLib/platform/linux_x64.cmake && make
root@55fb5fbdda6d:/remcu-mcu-sdks/nordicsemi/nRF5_SDK_12.3.0_d7731ad/NRF51422/build# ls
CMakeCache.txt  CMakeFiles  IrTest  Makefile  README.txt  REMCU_LICENSE.txt  build_remcu_object  cmake_install.cmake  libremcu.so  nRF51422-SDK-V12.3.0-01  remcu_include  test
```

Let's delve into the functionality of the example:

```c
    const char * host = argv[1];
    printf("host : %s\n", argv[1]);
    printf("port string : %s\n", argv[2]);
    const uint16_t port = (atoi(argv[2]) & 0xFFFF);
    printf("port : %d\n", port);

  if (port == 6666){
    remcu_connect2OpenOCD(host, 6666, 3);
  } else {
    remcu_connect2GDB(host, port, 3);
  }

  remcu_resetRemoteUnit(__HALT);
  remcu_setVerboseLevel(__ERROR);

  assert(remcu_isConnected());

  SystemInit(); // init function from asm code

  bsp_board_leds_init();
```

Initially, the example parses command line arguments, specifically the host and port values. Depending on the specified port, the example establishes a connection with either [OpenOCD](https://github.com/remotemcu/remcu-chip-sdks/blob/master/nordicsemi/nRF5_SDK_12.3.0_d7731ad/NRF51422/test/main.c#L113) (on port 6666) or a [GDB server](https://github.com/remotemcu/remcu-chip-sdks/blob/master/nordicsemi/nRF5_SDK_12.3.0_d7731ad/NRF51422/test/main.c#L115). Subsequently, it resets the chip, placing it in [HALT mode](https://github.com/remotemcu/remcu-chip-sdks/blob/master/nordicsemi/nRF5_SDK_12.3.0_d7731ad/NRF51422/test/main.c#L118), and verifies the successful establishment of the [connection](https://github.com/remotemcu/remcu-chip-sdks/blob/master/nordicsemi/nRF5_SDK_12.3.0_d7731ad/NRF51422/test/main.c#L121). 

```c
SystemInit();
```

The [SystemInit](https://github.com/remotemcu/remcu-chip-sdks/blob/nordic/nordicsemi/nRF5_SDK_12.3.0_d7731ad/components/toolchain/system_nrf51.c#L62) function is executed whenever the interrupt vector initializes. Given this behavior, it's imperative to explicitly incorporate the [SystemInit](https://github.com/remotemcu/remcu-chip-sdks/blob/nordic/nordicsemi/nRF5_SDK_12.3.0_d7731ad/NRF51422/test/main.c#L123) function within our code.

To initiate the test, follow these steps:

- Establish a connection between the chip and your PC.
- Launch OpenOCD, using version [0.10.0-11-20190118-1134](https://github.com/ilg-archived/openocd/releases/tag/v0.10.0-12-20190422).

```bash
openocd -f interface/stlink.cfg -f target/nrf51.cfg
```

- Execute the test, which includes running the example we've prepared.

```bash
LD_LIBRARY_PATH=$PWD test/test_build_nrf5_adc localhost 6666
```

6666 is a port of OpenOCD TCL server, also you can use GDB server on 3333 port:

```bash
LD_LIBRARY_PATH=$PWD test/test_build_nrf5_adc localhost 3333
```

The `LD_LIBRARY_PATH` environment variable is to ensure that the executable binary of our test can locate and access the REMCU library.

## Windows OS Notes

When working with the Windows OS as your target platform, setting up the environment can be a more intricate process. For detailed guidance on this setup [here](https://github.com/remotemcu/remcu-chip-sdks#windows-os). 

However, if you're looking for a more streamlined and convenient approach, I highly recommend using [GitHub Actions](#build-with-github-action). GitHub Actions offers an automated and remote method to build the REMCU library. By harnessing the power of GitHub Actions, you can easy the build process while circumventing the need for complex environment setup.

It's important to emphasize that specific versions of crucial packages are necessary to ensure a successful build on the Windows platform. You'll need to have the following components installed and configured:

1. **Visual Studio 2017**
2. **Clang Version 8.0.0**
3. **Build System Choice:** Opt for either the `make` or `ninja` build system

In addition to the [modifications](https://github.com/remotemcu/remcu-chip-sdks/commit/787ebf08bd284dee9ac24136269b461031277def) applied for the Linux environment, it's important to underscore that for Windows, additional patching of the NRF SDK is essential. A notable difference lies in the fact that the compiler doesn't define `__GNUC__` on the Windows platform, which can lead to build failures. We can address the issue by adding the necessary define to both the [Makefile](https://github.com/remotemcu/remcu-chip-sdks/commit/c203edb5c4728ff45d24e27523d92f0caaaa15e7#diff-bf7e07b415343d353bad91b79f8bc1b5bda7edd32667dadc4800a2b3f4839a72) and  [defines_nRF51422.h](https://github.com/remotemcu/remcu-chip-sdks/commit/c203edb5c4728ff45d24e27523d92f0caaaa15e7#diff-c6fcf6f8725f31e63ceaf5abbc53e9fa85811df01ab31a518973f1da335f13be) header file.

While the build process might complete successfully, it's crucial to remember that for the Windows OS, you also need to ensure that all functions within the dynamic library are properly exported. This exportation is essential to enable the dynamic library's functions to be accessible and utilized by external code, including executable binaries.

On Windows, you can use the `__declspec(dllexport)` attribute to explicitly mark functions for export from a dynamic library. This attribute tells the compiler to generate the necessary export information for those functions.

We can use the `#pragma clang attribute push` and `#pragma clang attribute pop` directives to conveniently mark functions for export on the Windows platform. This approach streamlines the process of marking functions for export and ensures a cleaner and more organized code structure.

```c
#pragma clang attribute push (__declspec(dllexport), apply_to = function)

// Functions to be exported
void exportedFunction1();
void exportedFunction2();

#pragma clang attribute pop
```

By enclosing the functions you want to export within these directives, you effectively apply the `__declspec(dllexport)` attribute to those functions.

To avoid any confusion and ensure that #pragmas are used exclusively in Windows builds, a practical strategy is to encapsulate them within header files.

```c
#include "remcu_exports_symbol_enter.h"

// Functions to be exported
void exportedFunction1();
void exportedFunction2();

#include "remcu_exports_symbol_exit.h"
```

## Build with GitHub Action

This is the most simple way to build the library and tests. Building the NRF SDK library through GitHub Actions is a simple procedure. Here's a simplified breakdown of the steps:

1. Fork the [REMCU CHIP SDK Collection Repo](https://github.com/remotemcu/remcu-chip-sdks/fork): Begin by forking the target repository where you intend to build the NRF SDK library. This creates a copy of the repository under your account.

2. Commit the [prepared NRF51 SDK](https://github.com/remotemcu/remcu-chip-sdks/tree/master/nordicsemi/nRF5_SDK_12.3.0_d7731ad): Ensure that the prepared NRF51 SDK is available in your forked repository. Commit this SDK to your repository.

3. Change root CMakeLists.txt: Within your forked repository, incorporate the add_subdirectory command into the [root CMakeLists.txt file](https://github.com/remotemcu/remcu-chip-sdks/blob/master/CMakeLists.txt#L18). This command serves as a inclusion to enable the build process for the NRF SDK.
```cmake
add_subdirectory(nordicsemi)
```

4. Intermediate [CMakeLists.txt](https://github.com/remotemcu/remcu-chip-sdks/blob/master/nordicsemi/CMakeLists.txt): create an intermediate CMakeLists.txt file that grants GitHub Actions access to the NRF SDK's source code

5. Include the SDK as an External Project: In the [CMakeLists.txt file](https://github.com/remotemcu/remcu-chip-sdks/blob/master/nordicsemi/nRF5_SDK_12.3.0_d7731ad/CMakeLists.txt) that corresponds to your SDK, define your SDK as an external project. This configuration allows the build system to understand and build the NRF SDK library as part of your project.

```cmake
include(${CMAKE_ROOT}/Modules/ExternalProject.cmake)

set(CMAKE_MODULE_PATH ${REMCU_VM_PATH}/cmake)
include(MultiBuild)

ExternalProject_Add(NRF51422 
      DOWNLOAD_COMMAND ""
      SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/NRF51422/"
      CMAKE_ARGS ${CMAKE_ARGS}
      )
```

- Commit Changes: After making these adjustments, commit the changes to your forked repository. This ensures that your modifications are saved.

- Run GitHub Actions Workflows: Execute the GitHub Actions workflows according to the instructions provided in [the guide](https://github.com/remotemcu/remcu-chip-sdks/tree/master#universal-easy-way-github-actions).
![](https://github.com/remotemcu/remcu-chip-sdks/raw/master/img/finish_workflow.png)

By following these steps, you can integrate your MCU's SDK into the GitHub Actions Build system.


Following the successful build, you'll have a dynamic/shared library at your disposal,

* **remcu.dll remcu.lib** - for Windows

* **libremcu.so** - for Linux

* **libremcu.myLib** - for Macos

Along with the necessary SDK header files residing within the [remcu_include](https://github.com/remotemcu/prebuilt_libraries/tree/master/NordSemi/nRF51422-SDK-V12.3.0-01/Windows_x64/remcu_include) folder. In this [repository](https://github.com/remotemcu/remcu_examples), you'll discover examples that effectively utilize the REMCU library, allowing you to observe practical implementations. These examples serve as valuable references for understanding how to harness the functionalities of your dynamic library.

For further insight, [tutorials](https://remotemcu.com/tutorials) are also available, offering step-by-step guidance on effectively integrating and employing the REMCU library within your projects. By exploring these tutorials, you can unlock a comprehensive understanding of the library's capabilities and become well-versed in its practical application.