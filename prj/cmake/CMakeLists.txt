cmake_minimum_required(VERSION 2.8)

option(AVX512 "AVX512 enable" ON)
option(SIMD_TEST "Test framework enable" ON)

set(TRUNK_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../..)

if((NOT DEFINED CMAKE_BUILD_TYPE) OR (CMAKE_BUILD_TYPE STREQUAL ""))
    set(CMAKE_BUILD_TYPE "Release")
endif()
message("Build type: '${CMAKE_BUILD_TYPE}'")

if(NOT ${TARGET} STREQUAL "")
    set(CMAKE_SYSTEM_PROCESSOR ${TARGET})
endif()
message("Target: ${CMAKE_SYSTEM_PROCESSOR}")

if(NOT ${TEST_FLAGS} STREQUAL "")
    message("Test flags: ${TEST_FLAGS}")
endif()

if((NOT LIBRARY STREQUAL "STATIC") AND (NOT LIBRARY STREQUAL "SHARED"))
    set(LIBRARY "STATIC")
endif()

if(NOT TOOLCHAIN STREQUAL "")
    set(CMAKE_C_COMPILER ${TOOLCHAIN})
    set(CMAKE_CXX_COMPILER ${TOOLCHAIN})
    execute_process(COMMAND ${CMAKE_CXX_COMPILER} -dumpversion OUTPUT_VARIABLE CMAKE_CXX_COMPILER_VERSION)
endif()
message("Library: ${LIBRARY}")
message("Toolchain: ${CMAKE_CXX_COMPILER}")
message("ID: ${CMAKE_CXX_COMPILER_ID}")
message("Version: ${CMAKE_CXX_COMPILER_VERSION}")

include_directories(${TRUNK_DIR}/src)

if((CHECK_VERSION STREQUAL "") OR (NOT (CHECK_VERSION STREQUAL "0")))
	if (WIN32)
		execute_process(COMMAND ${TRUNK_DIR}/prj/cmd/GetVersion.cmd ${TRUNK_DIR})
	else()
		execute_process(COMMAND bash ${TRUNK_DIR}/prj/sh/GetVersion.sh ${TRUNK_DIR})
	endif()
else()
    message("Skip version checking.")
	add_definitions(-DSIMD_VERSION="unknown")
endif()

if(CMAKE_BUILD_TYPE STREQUAL "Release")
    set(COMMON_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC -std=c++11 -O3")
elseif(CMAKE_BUILD_TYPE STREQUAL "Debug")
    set(COMMON_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC -std=c++11 -O0")
else()
    message(FATAL_ERROR "Unknown value of CMAKE_BUILD_TYPE!")
endif()

if((CMAKE_SYSTEM_PROCESSOR STREQUAL "i686") OR (CMAKE_SYSTEM_PROCESSOR STREQUAL "x86_64"))

if(CMAKE_SYSTEM_PROCESSOR STREQUAL "i686")
    set(COMMON_CXX_FLAGS "${COMMON_CXX_FLAGS} -m32")
elseif(CMAKE_SYSTEM_PROCESSOR STREQUAL "x86_64")
    set(COMMON_CXX_FLAGS "${COMMON_CXX_FLAGS} -m64")
endif()

    file(GLOB_RECURSE SIMD_BASE_SRC ${TRUNK_DIR}/src/Simd/SimdBase*.cpp)
    set_source_files_properties(${SIMD_BASE_SRC} PROPERTIES COMPILE_FLAGS "${COMMON_CXX_FLAGS}")

    file(GLOB_RECURSE SIMD_SSE1_SRC ${TRUNK_DIR}/src/Simd/SimdSse1*.cpp)
    set_source_files_properties(${SIMD_SSE1_SRC} PROPERTIES COMPILE_FLAGS "${COMMON_CXX_FLAGS} -msse")

    file(GLOB_RECURSE SIMD_SSE2_SRC ${TRUNK_DIR}/src/Simd/SimdSse2*.cpp)
    set_source_files_properties(${SIMD_SSE2_SRC} PROPERTIES COMPILE_FLAGS "${COMMON_CXX_FLAGS} -msse2")

    file(GLOB_RECURSE SIMD_SSE3_SRC ${TRUNK_DIR}/src/Simd/SimdSse3*.cpp)
    set_source_files_properties(${SIMD_SSE3_SRC} PROPERTIES COMPILE_FLAGS "${COMMON_CXX_FLAGS} -msse3")

    file(GLOB_RECURSE SIMD_SSSE3_SRC ${TRUNK_DIR}/src/Simd/SimdSsse3*.cpp)
    set_source_files_properties(${SIMD_SSSE3_SRC} PROPERTIES COMPILE_FLAGS "${COMMON_CXX_FLAGS} -mssse3")

    file(GLOB_RECURSE SIMD_SSE41_SRC ${TRUNK_DIR}/src/Simd/SimdSse41*.cpp)
    set_source_files_properties(${SIMD_SSE41_SRC} PROPERTIES COMPILE_FLAGS "${COMMON_CXX_FLAGS} -msse4.1")

    file(GLOB_RECURSE SIMD_SSE42_SRC ${TRUNK_DIR}/src/Simd/SimdSse42*.cpp)
    set_source_files_properties(${SIMD_SSE42_SRC} PROPERTIES COMPILE_FLAGS "${COMMON_CXX_FLAGS} -msse4.2")

    file(GLOB_RECURSE SIMD_AVX1_SRC ${TRUNK_DIR}/src/Simd/SimdAvx1*.cpp)
    set_source_files_properties(${SIMD_AVX1_SRC} PROPERTIES COMPILE_FLAGS "${COMMON_CXX_FLAGS} -mavx")

    file(GLOB_RECURSE SIMD_AVX2_SRC ${TRUNK_DIR}/src/Simd/SimdAvx2*.cpp)
    if ((CMAKE_CXX_COMPILER MATCHES "clang") OR (CMAKE_CXX_COMPILER_ID MATCHES "Clang"))
        set_source_files_properties(${SIMD_AVX2_SRC} PROPERTIES COMPILE_FLAGS "${COMMON_CXX_FLAGS} -mavx2 -mfma -mf16c -mbmi -mlzcnt")
    else()
        set_source_files_properties(${SIMD_AVX2_SRC} PROPERTIES COMPILE_FLAGS "${COMMON_CXX_FLAGS} -mavx2 -mfma -mf16c -mbmi -mlzcnt -fabi-version=4")
    endif()

    set(SIMD_LIB_FLAGS "${COMMON_CXX_FLAGS} -mavx2")
    set(SIMD_ALG_SRC ${SIMD_BASE_SRC} ${SIMD_SSE1_SRC} ${SIMD_SSE2_SRC} ${SIMD_SSE3_SRC} ${SIMD_SSSE3_SRC} ${SIMD_SSE41_SRC} ${SIMD_SSE42_SRC} ${SIMD_AVX1_SRC} ${SIMD_AVX2_SRC})

    if((((CMAKE_CXX_COMPILER_ID MATCHES "GNU") OR (CMAKE_CXX_COMPILER MATCHES "gnu")) AND (NOT(CMAKE_CXX_COMPILER_VERSION VERSION_LESS "4.9.0"))) OR (CMAKE_CXX_COMPILER MATCHES "clang") OR (CMAKE_CXX_COMPILER_ID MATCHES "Clang"))    
        file(GLOB_RECURSE SIMD_AVX512F_SRC ${TRUNK_DIR}/src/Simd/SimdAvx512f*.cpp)
        set_source_files_properties(${SIMD_AVX512F_SRC} PROPERTIES COMPILE_FLAGS "${COMMON_CXX_FLAGS} -mavx512f -mavx512dq -mfma")

        file(GLOB_RECURSE SIMD_AVX512BW_SRC ${TRUNK_DIR}/src/Simd/SimdAvx512bw*.cpp)
        set_source_files_properties(${SIMD_AVX512BW_SRC} PROPERTIES COMPILE_FLAGS "${COMMON_CXX_FLAGS} -mavx512f -mavx512bw -mavx512vl -mavx512dq -mbmi -mlzcnt -mfma")

        if(UNIX AND AVX512)
            set(SIMD_LIB_FLAGS "${SIMD_LIB_FLAGS} -mavx512bw")
            set(SIMD_ALG_SRC ${SIMD_ALG_SRC} ${SIMD_AVX512F_SRC} ${SIMD_AVX512BW_SRC})
            message("Use AVX-512F and AVX-512BW")
        endif()
    endif()

    file(GLOB_RECURSE SIMD_LIB_SRC ${TRUNK_DIR}/src/Simd/SimdLib.cpp)
    set_source_files_properties(${SIMD_LIB_SRC} PROPERTIES COMPILE_FLAGS "${SIMD_LIB_FLAGS}")
    add_library(Simd ${LIBRARY} ${SIMD_LIB_SRC} ${SIMD_ALG_SRC})

    if(SIMD_TEST)
        file(GLOB_RECURSE TEST_SRC_C ${TRUNK_DIR}/src/Test/*.c)
        if((CMAKE_CXX_COMPILER MATCHES "clang") OR (CMAKE_CXX_COMPILER_ID MATCHES "Clang")) 
            set_source_files_properties(${TEST_SRC_C} PROPERTIES COMPILE_FLAGS "${CMAKE_C_FLAGS} -x c")
        endif()
        file(GLOB_RECURSE TEST_SRC_CPP ${TRUNK_DIR}/src/Test/*.cpp)
        if(NOT ${TARGET} STREQUAL "")
            set_source_files_properties(${TEST_SRC_CPP} PROPERTIES COMPILE_FLAGS "${SIMD_LIB_FLAGS}")
        else()
            set_source_files_properties(${TEST_SRC_CPP} PROPERTIES COMPILE_FLAGS "${COMMON_CXX_FLAGS} ${TEST_FLAGS} -mtune=native")
        endif()
        add_executable(Test ${TEST_SRC_C} ${TEST_SRC_CPP})
        target_link_libraries(Test Simd -lpthread)
    endif()

elseif((CMAKE_SYSTEM_PROCESSOR STREQUAL "ppc") OR (CMAKE_SYSTEM_PROCESSOR STREQUAL "ppc64"))

    file(GLOB_RECURSE SIMD_BASE_SRC ${TRUNK_DIR}/src/Simd/SimdBase*.cpp)
    set_source_files_properties(${SIMD_BASE_SRC} PROPERTIES COMPILE_FLAGS "${COMMON_CXX_FLAGS}")

    file(GLOB_RECURSE SIMD_VMX_SRC ${TRUNK_DIR}/src/Simd/SimdVmx*.cpp)
     set_source_files_properties(${SIMD_VMX_SRC} PROPERTIES COMPILE_FLAGS "${COMMON_CXX_FLAGS} -maltivec")

    file(GLOB_RECURSE SIMD_VSX_SRC ${TRUNK_DIR}/src/Simd/SimdVsx*.cpp)
    set_source_files_properties(${SIMD_VSX_SRC} PROPERTIES COMPILE_FLAGS "${COMMON_CXX_FLAGS} -mvsx")

    file(GLOB_RECURSE SIMD_LIB_SRC ${TRUNK_DIR}/src/Simd/SimdLib.cpp)
    set_source_files_properties(${SIMD_LIB_SRC} PROPERTIES COMPILE_FLAGS "${COMMON_CXX_FLAGS} -mvsx")
    add_library(Simd ${LIBRARY} ${SIMD_LIB_SRC} ${SIMD_BASE_SRC} ${SIMD_VMX_SRC} ${SIMD_VSX_SRC})

    if(SIMD_TEST)
        file(GLOB_RECURSE TEST_SRC_C ${TRUNK_DIR}/src/Test/*.c)
        file(GLOB_RECURSE TEST_SRC_CPP ${TRUNK_DIR}/src/Test/*.cpp)
        if(NOT ${TARGET} STREQUAL "")
            set_source_files_properties(${TEST_SRC_CPP} PROPERTIES COMPILE_FLAGS "${COMMON_CXX_FLAGS} -mvsx")
        else()
            set_source_files_properties(${TEST_SRC_CPP} PROPERTIES COMPILE_FLAGS "${COMMON_CXX_FLAGS} ${TEST_FLAGS} -mtune=native")
        endif()
        add_executable(Test ${TEST_SRC_C} ${TEST_SRC_CPP})
        target_link_libraries(Test Simd -lpthread)
    endif()

elseif((CMAKE_SYSTEM_PROCESSOR MATCHES "arm") OR (CMAKE_SYSTEM_PROCESSOR STREQUAL "aarch64"))

    if(CMAKE_SYSTEM_PROCESSOR MATCHES "arm")
        set(CXX_NEON_FLAG "-mfpu=neon -mfpu=neon-fp16 -mfp16-format=ieee")
    else()
        set(CXX_NEON_FLAG "")
    endif()

    if((CMAKE_CXX_COMPILER_ID STREQUAL "Clang") OR (CMAKE_CXX_COMPILER MATCHES "clang"))    
        add_definitions(-DSIMD_NEON_FP16_DISABLE)
    endif()

    file(GLOB_RECURSE SIMD_BASE_SRC ${TRUNK_DIR}/src/Simd/SimdBase*.cpp)
    set_source_files_properties(${SIMD_BASE_SRC} PROPERTIES COMPILE_FLAGS "${COMMON_CXX_FLAGS}")

    file(GLOB_RECURSE SIMD_NEON_SRC ${TRUNK_DIR}/src/Simd/SimdNeon*.cpp)
    set_source_files_properties(${SIMD_NEON_SRC} PROPERTIES COMPILE_FLAGS "${COMMON_CXX_FLAGS} ${CXX_NEON_FLAG}")

    file(GLOB_RECURSE SIMD_LIB_SRC ${TRUNK_DIR}/src/Simd/SimdLib.cpp)
    set_source_files_properties(${SIMD_LIB_SRC} PROPERTIES COMPILE_FLAGS "${COMMON_CXX_FLAGS} ${CXX_NEON_FLAG}")
    add_library(Simd ${LIBRARY} ${SIMD_LIB_SRC} ${SIMD_BASE_SRC} ${SIMD_NEON_SRC})

    if(SIMD_TEST)
        file(GLOB_RECURSE TEST_SRC_C ${TRUNK_DIR}/src/Test/*.c)
        file(GLOB_RECURSE TEST_SRC_CPP ${TRUNK_DIR}/src/Test/*.cpp)
        if(NOT ${TARGET} STREQUAL "")
            set_source_files_properties(${TEST_SRC_CPP} PROPERTIES COMPILE_FLAGS "${COMMON_CXX_FLAGS} ${CXX_NEON_FLAG} -D_GLIBCXX_USE_NANOSLEEP")
        else()
            set_source_files_properties(${TEST_SRC_CPP} PROPERTIES COMPILE_FLAGS "${COMMON_CXX_FLAGS} ${TEST_FLAGS} -mtune=native -D_GLIBCXX_USE_NANOSLEEP")
        endif()
        add_executable(Test ${TEST_SRC_C} ${TEST_SRC_CPP})
        target_link_libraries(Test Simd -lpthread)
    endif()

elseif((CMAKE_SYSTEM_PROCESSOR STREQUAL "mips"))

    set(COMMON_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC -Ofast -EL")

    set(CXX_MSA_FLAG "-mmsa")#"-mmsa -mhard-float -mfp64")

    file(GLOB_RECURSE SIMD_BASE_SRC ${TRUNK_DIR}/src/Simd/SimdBase*.cpp)
    set_source_files_properties(${SIMD_BASE_SRC} PROPERTIES COMPILE_FLAGS "${COMMON_CXX_FLAGS}")

    file(GLOB_RECURSE SIMD_MSA_SRC ${TRUNK_DIR}/src/Simd/SimdMsa*.cpp)
    set_source_files_properties(${SIMD_MSA_SRC} PROPERTIES COMPILE_FLAGS "${COMMON_CXX_FLAGS} ${CXX_MSA_FLAG}")

    file(GLOB_RECURSE SIMD_LIB_SRC ${TRUNK_DIR}/src/Simd/SimdLib.cpp)
    set_source_files_properties(${SIMD_LIB_SRC} PROPERTIES COMPILE_FLAGS "${COMMON_CXX_FLAGS} ${CXX_MSA_FLAG}")
    add_library(Simd ${LIBRARY} ${SIMD_LIB_SRC} ${SIMD_BASE_SRC} ${SIMD_MSA_SRC})

    if(SIMD_TEST)
        file(GLOB_RECURSE TEST_SRC_C ${TRUNK_DIR}/src/Test/*.c)
        set_source_files_properties(${TEST_SRC_C} PROPERTIES COMPILE_FLAGS "${COMMON_C_FLAGS} -EL")
        file(GLOB_RECURSE TEST_SRC_CPP ${TRUNK_DIR}/src/Test/*.cpp)
        if(NOT ${TARGET} STREQUAL "")
            set_source_files_properties(${TEST_SRC_CPP} PROPERTIES COMPILE_FLAGS "${COMMON_CXX_FLAGS} ${CXX_MSA_FLAG}")# -D_GLIBCXX_USE_NANOSLEEP")
        else()
            set_source_files_properties(${TEST_SRC_CPP} PROPERTIES COMPILE_FLAGS "${COMMON_CXX_FLAGS} ${TEST_FLAGS} -mtune=native")# -D_GLIBCXX_USE_NANOSLEEP")
        endif()
        add_executable(Test ${TEST_SRC_C} ${TEST_SRC_CPP})
        target_link_libraries(Test Simd -lpthread "${CMAKE_MODULE_LINKER_FLAGS} -EL")
    endif()

else()
    message(FATAL_ERROR "Unknown value of CMAKE_SYSTEM_PROCESSOR!")
endif()
