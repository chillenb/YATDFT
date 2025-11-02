cmake_minimum_required(VERSION 3.10)

project(YATDFT LANGUAGES C)

option(YATDFT_ENABLE_TESTS "Enable compilation of test codes" ON)
option(YATDFT_ENABLE_LIBXC "Enable Libxc for exchange-correlation functionals" OFF)

find_package(OpenMP REQUIRED)

find_package(BLAS REQUIRED)

find_package(simint REQUIRED)

if(YATDFT_ENABLE_LIBXC)
    find_package(Libxc REQUIRED)
endif()

add_library(yatdft STATIC
    src/acc_JKmat.c
    src/build_Dmat.c
    src/build_HF_mat.c
    src/build_JKmat_DF.c
    src/build_XCmat.c
    src/CDIIS.c
    src/CMS_basis.c
    src/CMS_simint.c
    src/eval_XC_func.c
    src/gen_int_grid.c
    src/gen_Lebedev_grid.c
    src/ket_sp_list.c
    src/setup_DF.c
    src/TinyDFT_typedef.c
    src/utils.c
)

target_compile_definitions(yatdft PUBLIC -DUSE_MKL -DYATDFT_ALIGNLEN=64)
target_include_directories(yatdft INTERFACE src)

target_link_libraries(yatdft PUBLIC OpenMP::OpenMP_C BLAS::BLAS simint::simint)
if(YATDFT_ENABLE_LIBXC)
    target_link_libraries(yatdft PUBLIC Libxc::xc)
endif()


option(YATDFT_FORCE_AVX512 "Force AVX-512 with pragma omp simd" ON)


if(YATDFT_FORCE_AVX512)
    if(CMAKE_C_COMPILER_ID MATCHES "Intel")
        target_compile_options(yatdft PRIVATE -mavx512f -mavx512cd -mavx512bw -mavx512dq -mavx512vl -O3 -fimf-use-svml=true)
    elseif(CMAKE_C_COMPILER_ID STREQUAL "GNU" OR
           CMAKE_C_COMPILER_ID MATCHES "Clang")
        target_compile_options(yatdft PRIVATE -mavx512f -mavx512cd -mavx512bw -mavx512dq -mavx512vl -O3 -ffast-math)
    endif()
endif()


if (YATDFT_ENABLE_TESTS)
   add_executable(AnySCF tests/AnySCF.c)
   target_link_libraries(AnySCF yatdft)
endif()