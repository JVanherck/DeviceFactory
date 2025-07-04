cmake_minimum_required(VERSION 3.13.0)
project(DeviceFactory VERSION 0.1.0)

find_package( OpenCV REQUIRED )
include_directories( ${OpenCV_INCLUDE_DIRS} )


# Realsense
option(WITH_REALSENSE2 "Use RealSense2 SDK" ON)
if(WITH_REALSENSE2)
  find_package(realsense2 REQUIRED )
  add_definitions(-DWITH_REALSENSE2)
endif()

option(WITH_FFMPEG "Include FFMPEG decoder" ON)
if(WITH_FFMPEG)
    find_path(AVCODEC_INCLUDE_DIR libavcodec/avcodec.h)
    find_library(AVCODEC_LIBRARY avcodec)

    find_path(AVFORMAT_INCLUDE_DIR libavformat/avformat.h)
    find_library(AVFORMAT_LIBRARY avformat)

    find_path(AVUTIL_INCLUDE_DIR libavutil/avutil.h)
    find_library(AVUTIL_LIBRARY avutil)

    add_definitions(-DWITH_FFMPEG)

#    find_path(AVDEVICE_INCLUDE_DIR libavdevice/avdevice.h)
#    find_library(AVDEVICE_LIBRARY avdevice)
endif()

add_library(DeviceFactory STATIC
    Device.h
    Device.cpp
    DeviceFactory.cpp
    DeviceFactory.h
    CameraCalibration.h
    CameraCalibration.cpp
    CVVideoCaptureDevice.h
    CVVideoCaptureDevice.cpp
    CVImageCaptureDevice.h
    CVImageCaptureDevice.cpp
    )

if(WITH_REALSENSE2)
    target_sources(DeviceFactory PRIVATE RealSense2Device.h)
    target_sources(DeviceFactory PRIVATE RealSense2Device.cpp)
    target_link_libraries(DeviceFactory ${realsense2_LIBRARY})
endif()

if(WITH_FFMPEG)
   target_sources(DeviceFactory PRIVATE FFMPEGDevice.h)
   target_sources(DeviceFactory PRIVATE FFMPEGDevice.cpp)

   target_include_directories(DeviceFactory PRIVATE ${AVCODEC_INCLUDE_DIR} ${AVFORMAT_INCLUDE_DIR} ${AVUTIL_INCLUDE_DIR} )
   target_link_libraries(DeviceFactory ${AVCODEC_LIBRARY} ${AVFORMAT_LIBRARY} ${AVUTIL_LIBRARY} )
endif()

target_link_libraries(DeviceFactory avcodec)
target_link_libraries(DeviceFactory avformat)
target_link_libraries(DeviceFactory avutil)
target_link_libraries(DeviceFactory swscale)

target_link_libraries(DeviceFactory ${OpenCV_LIBS})


install(TARGETS DeviceFactory
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)


set(CMAKE_INSTALL_INCLUDEDIR ${CMAKE_INSTALL_PREFIX}/include)
install(FILES Device.h
              DeviceFactory.h
              CVVideoCaptureDevice.h
              CameraCalibration.h
              CVImageCaptureDevice.h
              RealSense2Device.h
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/DeviceFactory
)
