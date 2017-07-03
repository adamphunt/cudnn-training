cmake_minimum_required (VERSION 2.6)
project (cudnn-training)

SET(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake ${CMAKE_MODULE_PATH})

find_package(CUDA 6.5 REQUIRED)
FIND_PACKAGE(CUDNN 5 REQUIRED)

MESSAGE(STATUS "CUDNN_FOUND: ${CUDNN_FOUND}")
MESSAGE(STATUS "CUDNN_INCLUDE_DIRS: ${CUDNN_INCLUDE_DIRS}")
MESSAGE(STATUS "CUDNN_LIBRARIES: ${CUDNN_LIBRARIES}")
MESSAGE(STATUS "CUDNN_LIBRARY_DIRS: ${CUDNN_LIBRARY_DIRS}")


# Uncomment the following line to use gflags
set(USE_GFLAGS 1)

if (CMAKE_BUILD_TYPE STREQUAL "Debug")
  message("Debug mode")
  set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS};-std=c++11;-g;-lineinfo;-Xcompiler;-ggdb)
else()
  set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS};-std=c++11;-O2;-DNDEBUG;-Xcompiler;-DNDEBUG;-Wno-deprecated-gpu-targets)
endif()

set(CUDA_PROPAGATE_HOST_FLAGS OFF)
#set(CUDA_USE_STATIC_CUDA_RUNTIME OFF)

CUDA_SELECT_NVCC_ARCH_FLAGS(ARCH_FLAGS Auto)
LIST(APPEND CUDA_NVCC_FLAGS ${ARCH_FLAGS})

# Addresses a bug where code is not compiled as C++11 in non-CUDA code and older g++ versions
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

if(USE_GFLAGS)
  add_definitions(-DUSE_GFLAGS)
endif()

include_directories(${CUDNN_INCLUDE_DIRS})
link_directories(${CUDNN_LIBRARY_DIRS})

cuda_add_executable(trainlenet lenet.cu readubyte.cpp)
cuda_add_cublas_to_target(trainlenet)

if(USE_GFLAGS)
  target_link_libraries(trainlenet gflags cudnn)
else()
  target_link_libraries(trainlenet cudnn)
endif()


