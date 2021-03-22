cmake_minimum_required(VERSION 3.7)

project(cutt LANGUAGES C CXX)
enable_language(CUDA)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

option(ENABLE_NVTOOLS "enable compile with -lineinfo, and nvtools" OFF)
option(NO_ALIGN_ALLOC "enable compile with no align allocation" OFF)
option(FINE_TUNE "enable compile CPU optimized with native hardware" OFF)

## 
set(CUTT_SRCS src/CudaMemcpy.cu 
              src/cuttGpuModelKernel.cu 
              src/cuttkernel.cu 
              src/CudaUtils.cu
              src/cuttGpuModel.cpp 
              src/cuttplan.cpp 
              src/cuttTimer.cpp
              src/cutt.cpp
              src/CudaMemcpy.h
              src/cuttGpuModelKernel.h
              src/cuttkernel.h
              src/CudaUtils.h
              src/cuttGpuModel.h
              src/cuttplan.h
              src/cuttTimer.h
              src/LRUCache.h
              src/int_vector.h
              src/cuttTypes.h
              src/calls.h
    )



add_library(cutt STATIC ${CUTT_SRCS})
target_compile_features(cutt PUBLIC cxx_std_11)
target_include_directories(cutt PUBLIC ${PROJECT_SOURCE_DIR}/src)

find_package(CUDA REQUIRED)
if(NOT DEFINED CMAKE_CUDA_STANDARD)
    set(CMAKE_CUDA_STANDARD 11)
    set(CMAKE_CUDA_STANDARD_REQUIRED ON)
endif()
set_target_properties(cutt PROPERTIES
      CUDA_SEPARABLE_COMPILATION ON
                                    )
set_property(TARGET cutt PROPERTY CUDA_ARCHITECTURES 52 53 60 61 62 70 72 75 80 86)

set_target_properties(cutt PROPERTIES CUDA_RESOLVE_DEVICE_SYMBOLS ON)
set(CMAKE_CUDA_FLAGS "${CMAKE_CUDA_FLAGS} -Xcudafe=--display_error_number -Xptxas -dlcm=ca -O3 -D_FORCE_INLINE -Xcompiler -fPIC")

set(CXXFLAGS "${CXXFLAGS} -fPIC -O3")
if(ENABLE_NVTOOLS)
    set(CMAKE_CUDA_FLAGS "${CMAKE_CUDA_FLAGS} -DENABLE_NVTOOLS")
    set(CXXFLAGS ${CXXFLAGS} -DENABLE_NVTOOLS)
endif()

if(NO_ALIGN_ALLOC)
    set(CMAKE_CUDA_FLAGS "${CMAKE_CUDA_FLAGS} -DNO_ALIGNED_ALLOC")
    set(CXXFLAGS ${CXXFLAGS} -DNO_ALIGNED_ALLOC)
endif()

if(FINE_TUNE)
    set(CXXFLAGS "${CXXFLAGS} -march=native")
endif()

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${CXXFLAGS}")
set(CUTT_TMP_FLAGS ${CUTT_TMP_FLAGS} -lpthread)
target_compile_options(cutt PUBLIC ${CUTT_TMP_FLAGS})

set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS}  "-arch=sm_50 \
      -gencode=arch=compute_50,code=sm_50 \
      -gencode=arch=compute_52,code=sm_52 \
      -gencode=arch=compute_60,code=sm_60 \
      -gencode=arch=compute_61,code=sm_61 \
      -gencode=arch=compute_70,code=sm_70 \
      -gencode=arch=compute_75,code=sm_75 ")
target_include_directories(cutt PRIVATE ${CMAKE_CUDA_TOOLKIT_INCLUDE_DIRECTORIES})


## INSTALL
INSTALL(TARGETS cutt 
        LIBRARY
          DESTINATION ${CMAKE_INSTALL_PREFIX}/lib
        ARCHIVE
          DESTINATION ${CMAKE_INSTALL_PREFIX}/lib
      )

set(CUTT_INCLUDES
    src/cutt.h)
install(FILES ${CUTT_INCLUDES} DESTINATION ${CMAKE_INSTALL_PREFIX}/include)

