cmake_minimum_required(VERSION 3.13)
project(pdc_mini_aevol)


# add source files then put them into the source variable
file(GLOB SOURCE 
    ${PROJECT_SOURCE_DIR}/src/*.cpp
    ${PROJECT_SOURCE_DIR}/src/*/*.cpp
)

# add include/
#target_include_directories(${PROJECT_NAME} PUBLIC "./include")
include_directories(
    ${PROJECT_SOURCE_DIR}/inc/
)

set(CMAKE_CXX_STANDARD 14)

set(DO_TRACES OFF CACHE BOOL "Whether to enable the Traces library")
set(USE_OMP OFF CACHE BOOL "Whether to enable OpenMP parallelization")
set(USE_CUDA OFF CACHE BOOL "Whether to enable CUDA parallelization")

if ( DO_TRACES )
    add_definitions(-DTRACES)
    message( STATUS "Traces are activated" )
endif ( DO_TRACES )

if ( USE_CUDA )
    # https://cliutils.gitlab.io/modern-cmake/chapters/packages/CUDA.html
    find_package(CUDA REQUIRED)
    include(CheckLanguage)
    check_language(CUDA)
    if(CMAKE_CUDA_COMPILER)
        enable_language(CUDA)
        set(CMAKE_CUDA_STANDARD 14)
        set(CMAKE_CUDA_STANDARD_REQUIRED ON)
        # https://arnon.dk/tag/nvcc-flags/
        set(CMAKE_CUDA_FLAGS "-arch=sm_60")
        set(CMAKE_CUDA_FLAGS_DEBUG "-Xcompiler -rdynamic -lineinfo")
    else()
        message(FATAL_ERROR "No CUDA support found")
    endif()
endif ( USE_CUDA )

if ( USE_OMP )
    find_package(OpenMP REQUIRED)
endif ( USE_OMP )

find_package(ZLIB REQUIRED)

add_library(
    ${PROJECT_SOURCE_DIR}/micro_aevol
    ${PROJECT_SOURCE_DIR}/Abstract_ExpManager.cpp
    ${PROJECT_SOURCE_DIR}/ExpManager.cpp
    ${PROJECT_SOURCE_DIR}/AeTime.cpp
    ${PROJECT_SOURCE_DIR}/DnaMutator.cpp
    ${PROJECT_SOURCE_DIR}/MutationEvent.cpp
    ${PROJECT_SOURCE_DIR}/Organism.cpp
    ${PROJECT_SOURCE_DIR}/Stats.cpp
    ${PROJECT_SOURCE_DIR}/Threefry.cpp
    ${PROJECT_SOURCE_DIR}/Dna.cpp)

target_link_libraries(micro_aevol PUBLIC ZLIB::ZLIB)

if ( OPENMP_FOUND )
    target_link_libraries(micro_aevol PUBLIC OpenMP::OpenMP_CXX)
    target_compile_definitions(micro_aevol PUBLIC USE_OMP)
endif ()

add_executable(micro_aevol_cpu src/main.cpp)
target_link_libraries(micro_aevol_cpu micro_aevol)

if ( USE_CUDA )
    add_subdirectory(cuda)
    add_executable(micro_aevol_gpu src/main.cpp)
    # nvToolsExt for enhanced profiling (ad-hoc chunks)
    target_link_libraries(micro_aevol_gpu PUBLIC cuda_micro_aevol micro_aevol)
endif ()
