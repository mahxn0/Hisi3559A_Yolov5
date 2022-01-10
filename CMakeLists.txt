cmake_minimum_required(VERSION 2.8)

project(yolo_nnie)
#-fno-rtti -fno-exceptions
set(OpenCV_DIR /home/mahxn0/nfs/mahxn0/3559_work/yolov5-nnie-master/third_party/opencv3/share/OpenCV)
find_package(OpenCV REQUIRED)

if(NOT DEFINED TARGET_SOC)
    message(FATAL_ERROR "target soc not defined, use -DTARGET_SOC=xxx to specify it!")
endif()

if(${TARGET_SOC} STREQUAL "3559")
    set(lib_3559 "lib_3559")
    set(cc "aarch64-himix100-linux-gcc")
    set(cxx "aarch64-himix100-linux-g++")
elseif(${TARGET_SOC} STREQUAL "3516")
    set(hisi_lib "lib_3516")
    set(cc "arm-himix200-linux-gcc")
    set(cxx "arm-himix200-linux-g++")
else()
    message(FATAL_ERROR "target soc must be 3559 or 3516, use -DTARGET_SOC=xxx to specify it!")
endif()


set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_SYSTEM_NAME Linux)

set(CMAKE_C_COMPILER ${cc})
set(CMAKE_CXX_COMPILER ${cxx})

set(CMAKE_CXX_FLAGS "-O3 -Wextra -fomit-frame-pointer -fstrict-aliasing -ffunction-sections -fdata-sections -ffast-math -fno-exceptions -fpermissive -fexceptions")
set(CMAKE_C_FLAGS "-O3 -Wextra")
# add_definitions("-mfpu=neon-vfpv4 -mfloat-abi=softfp")

set(CMAKE_EXE_LINKER_FLAGS "-pthread -lrt -ldl")


include_directories(./src)
include_directories(./third_party/hisi/include)

#set(mylibs mpi aacdec aacenc detail_ap dnvqe dpu_match dpu_rect dsp hdmi hdr_ap hi_cipher hiavslut hifisheyecalibrate ive md mfnr_ap mpi_photo pciv
#pos_query securec sfnr_ap svpruntime tde upvqe VoiceEngine nnie)

#find_library(COMM_LIB ${mylibs} ./third_party/hisi/lib NO_DEFAULT_PATH)
#message(${COMM_LIB})
set(LIB_DIR ${PROJECT_SOURCE_DIR}/third_party/hisi/lib)
link_directories(${LIB_DIR})

link_libraries(${COMM_LIB})
aux_source_directory(./src/ all_source)

add_executable(yolo_nnie ./sample/yolov5-nnie-sample.cpp ${all_source})

target_link_libraries(yolo_nnie ${OpenCV_LIBS} pthread dl m mpi aacdec aacenc detail_ap dnvqe dpu_match dpu_rect dsp hdmi hdr_ap hi_cipher hiavslut hifisheyecalibrate ive md mfnr_ap mpi_photo pciv
pos_query securec sfnr_ap svpruntime tde upvqe VoiceEngine nnie ) 