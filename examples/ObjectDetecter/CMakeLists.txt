cmake_minimum_required(VERSION 3.5.1)

project(ObjectDetecter)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

set(CUDA_TOOLKIT_ROOT_DIR /usr/local/cuda)

# find packages
find_package(Eigen3 REQUIRED)
find_package(CUDA QUIET REQUIRED)

set(PROTOBUF_INCLUDE_DIRS /usr/local/include/google)

set(PROTOBUF_LIBRARIES /usr/local/lib/libprotobuf.so)

include_directories(/usr/local/include/google/tensorflow
        ${PROTOBUF_INCLUDE_DIRS}
        ${EIGEN3_INCLUDE_DIRS}
        /home/hjzh/Library_Sources/3rd_library/tensorflow/bazel-tensorflow/external/nsync/public
        /usr/local/include
        /usr/local/cuda/include)

link_directories(/usr/local/lib)

add_executable(ObjectDetecter ObjectDetecter.cc main.cc ObjectDetecter.h)

target_link_libraries(ObjectDetecter tensorflow_cc
        tensorflow_framework ${PROTOBUF_LIBRARIES}
        avcodec avutil avdevice avfilter avformat swresample swscale postproc
        pthread PocoNet PocoUtil PocoFoundation /usr/local/lib/libopencv_core.so /usr/local/lib/libopencv_cudev.so
        /usr/local/lib/libopencv_photo.so /usr/local/lib/libopencv_shape.so /usr/local/lib/libopencv_video.so
        /usr/local/lib/libopencv_highgui.so /usr/local/lib/libopencv_imgproc.so
        /usr/local/lib/libopencv_videoio.so /usr/local/lib/libopencv_core.so /usr/local/lib/libopencv_cudacodec.so
        /usr/local/lib/libopencv_cudev.so /usr/local/lib/libopencv_imgcodecs.so)