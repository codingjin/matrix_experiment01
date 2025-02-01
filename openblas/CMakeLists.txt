# cmake min version
cmake_minimum_required(VERSION 3.5)

# project's name
PROJECT(TEST C)

#nk_libraries setting
#set(CMAKE_CXX_STANDARD 11)

# set var
SET(OPENBLAS_DIR /home/jin/OpenBLAS-develop)
SET(MKL_DIR /opt/intel/oneapi/mkl/2025.0)

# add definition
OPTION(DEFINED_DOUBLE "test DOUBLE" ON) # Disabled by default
OPTION(TEST_PACK "test gemm_pack and gemm_compute" ON) # Disabled by default
OPTION(DEBUG "debug mode for alpha and beta" OFF) # Disabled by default
OPTION(TEST_MKL "whether if testing mkl" OFF) # Disabled by default
OPTION(MULTI_THREAD "test multi-thread" OFF) # Disabled by default

IF(DEBUG)
    add_definitions(-DDEBUG)
ENDIF(DEBUG)

IF(DEFINED_DOUBLE)
    add_definitions(-DDOUBLE)
ENDIF(DEFINED_DOUBLE)

IF(TEST_PACK)
    add_definitions(-DPACK)
ENDIF(TEST_PACK)

IF(TEST_MKL)
# mkl
    include_directories(${MKL_DIR}/include/ ${MKL_DIR}/lib/intel64)
    link_directories(${MKL_DIR}/include/ ${MKL_DIR}/lib/intel64)
ELSE()
# openblas
	link_directories(${OPENBLAS_DIR}/install/include/)
    include_directories(${OPENBLAS_DIR})
    include_directories(include)
ENDIF(TEST_MKL)

# find all source code *.c
IF(TEST_MKL)
    add_executable(compute "src/mkl_gemm_compute.c")
ELSE()
    #file(GLOB SOURCES "src/gemm_compute.c")
    add_executable(mm_openblas "src/mm_openblas.c")
ENDIF(TEST_MKL)

# add link library directory
IF(TEST_MKL)
    IF(MULTI_THREAD)
        target_link_libraries(compute -Wl,--start-group ${MKL_DIR}/lib/intel64/libmkl_intel_ilp64.a ${MKL_DIR}/lib/intel64/libmkl_gnu_thread.a ${MKL_DIR}/lib/intel64/libmkl_core.a -Wl,--end-group -lgomp -lpthread -lm -ldl)
    ELSE()
        target_link_libraries(compute -L${MKL_DIR}/lib/intel64 -I${MKL_DIR}/include -lmkl_intel_ilp64 -lmkl_sequential -lmkl_core -lm -ldl -std=c++11 -isystem include -lpthread)
    ENDIF(MULTI_THREAD)
ELSE()
    IF(MULTI_THREAD)
        target_link_libraries(mm_openblas -L${OPENBLAS_DIR}/install/include/ -lm ${OPENBLAS_DIR}/libopenblas_haswellp-r0.3.28.dev.a -lpthread)
    ELSE()
        target_link_libraries(mm_openblas -L${OPENBLAS_DIR}/install/include/ -lm ${OPENBLAS_DIR}/libopenblas_haswellp-r0.3.28.dev.a)
    ENDIF(MULTI_THREAD)
ENDIF(TEST_MKL)


add_custom_target (
    clear
    @echo "[make] cleaning..." &
    rm -f mm_openblas
    )
