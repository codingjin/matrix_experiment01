# cmake min version
cmake_minimum_required(VERSION 3.5)

# project's name
PROJECT(TEST C)

#nk_libraries setting
#set(CMAKE_CXX_STANDARD 11)

# set var
SET(MKL_DIR /opt/intel/oneapi/mkl/2025.0)

# add definition
OPTION(DEBUG "debug mode for alpha and beta" OFF) # Disabled by default
OPTION(TEST_MKL "whether if testing mkl" ON) # Disabled by default
OPTION(MULTI_THREAD "test multi-thread" ON) # Disabled by default

IF(TEST_MKL)
    add_definitions(-DTEST_MKL)
ENDIF(TEST_MKL)

IF(DEBUG)
    add_definitions(-DDEBUG)
ENDIF(DEBUG)

IF(TEST_MKL)
# mkl
    include_directories(include)
    include_directories(${MKL_DIR}/include/ ${MKL_DIR}/lib/intel64)
    link_directories(${MKL_DIR}/include/ ${MKL_DIR}/lib/intel64)
ENDIF(TEST_MKL)

# find all source code *.c
IF(TEST_MKL)
    add_executable(mm_onemkl "src/mm_onemkl.c")
ENDIF(TEST_MKL)

IF(TEST_MKL)
    IF(MULTI_THREAD)
        target_link_libraries(mm_onemkl -Wl,--start-group ${MKL_DIR}/lib/intel64/libmkl_intel_ilp64.a ${MKL_DIR}/lib/intel64/libmkl_gnu_thread.a ${MKL_DIR}/lib/intel64/libmkl_core.a -Wl,--end-group -lgomp -lpthread -lm -ldl)
    ELSE()
        target_link_libraries(mm_onemkl -L${MKL_DIR}/lib/intel64 -I${MKL_DIR}/include -lmkl_intel_ilp64 -lmkl_sequential -lmkl_core -lm -ldl -std=c++11 -isystem include -lpthread)
    ENDIF(MULTI_THREAD)
ENDIF(TEST_MKL)



#add_custom_target (
#    gdb
#    @echo "[make] testing..." &
#    gdb ./onemkl
#    )

#add_custom_target (
#    run
#    @echo "[make] testing..." &
#    ./onemkl
#    )

add_custom_target (
    clear
    @echo "[make] cleaning..." &
    rm -rf *
    )
