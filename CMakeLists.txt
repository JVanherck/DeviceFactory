cmake_minimum_required(VERSION 3.13)
project(DeviceFactory VERSION 0.1.0 LANGUAGES CXX)

# Find dependencies
find_package(OpenCV REQUIRED)

option(WITH_REALSENSE2 "Use RealSense2 SDK" ON)
if(WITH_REALSENSE2)
    find_package(realsense2 REQUIRED)
endif()

option(WITH_FFMPEG "Include FFMPEG decoder" ON)
if(WITH_FFMPEG)
    find_path(AVCODEC_INCLUDE_DIR libavcodec/avcodec.h REQUIRED)
    find_library(AVCODEC_LIBRARY avcodec REQUIRED)

    find_path(AVFORMAT_INCLUDE_DIR libavformat/avformat.h REQUIRED)
    find_library(AVFORMAT_LIBRARY avformat REQUIRED)

    find_path(AVUTIL_INCLUDE_DIR libavutil/avutil.h REQUIRED)
    find_library(AVUTIL_LIBRARY avutil REQUIRED)

    find_library(SWSCALE_LIBRARY swscale REQUIRED)
endif()

# Collect source files
set(DEVICE_FACTORY_SOURCES
    Device.cpp
    DeviceFactory.cpp
    CameraCalibration.cpp
    CVVideoCaptureDevice.cpp
    CVImageCaptureDevice.cpp
)

set(DEVICE_FACTORY_HEADERS
    Device.h
    DeviceFactory.h
    CameraCalibration.h
    CVVideoCaptureDevice.h
    CVImageCaptureDevice.h
)

if(WITH_REALSENSE2)
    list(APPEND DEVICE_FACTORY_SOURCES RealSense2Device.cpp)
    list(APPEND DEVICE_FACTORY_HEADERS RealSense2Device.h)
endif()

if(WITH_FFMPEG)
    list(APPEND DEVICE_FACTORY_SOURCES FFMPEGDevice.cpp)
    list(APPEND DEVICE_FACTORY_HEADERS FFMPEGDevice.h)
endif()

# Create shared library
add_library(DeviceFactory SHARED ${DEVICE_FACTORY_SOURCES} ${DEVICE_FACTORY_HEADERS})

# Include directories
target_include_directories(DeviceFactory
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
        $<INSTALL_INTERFACE:include/DeviceFactory>
        ${OpenCV_INCLUDE_DIRS}
)

# Compile definitions
if(WITH_REALSENSE2)
    target_compile_definitions(DeviceFactory PUBLIC WITH_REALSENSE2)
endif()

if(WITH_FFMPEG)
    target_compile_definitions(DeviceFactory PUBLIC WITH_FFMPEG)
endif()

# Link dependencies
target_link_libraries(DeviceFactory
    PUBLIC
        ${OpenCV_LIBS}
)

if(WITH_REALSENSE2)
    target_link_libraries(DeviceFactory PUBLIC realsense2::realsense2)
endif()

if(WITH_FFMPEG)
    target_include_directories(DeviceFactory PRIVATE
        ${AVCODEC_INCLUDE_DIR}
        ${AVFORMAT_INCLUDE_DIR}
        ${AVUTIL_INCLUDE_DIR}
    )
    target_link_libraries(DeviceFactory
        PRIVATE
            ${AVCODEC_LIBRARY}
            ${AVFORMAT_LIBRARY}
            ${AVUTIL_LIBRARY}
            ${SWSCALE_LIBRARY}
    )
endif()

include(GNUInstallDirs)

set(CMAKE_INSTALL_INCLUDEDIR ${CMAKE_INSTALL_PREFIX}/include)

install(FILES ${DEVICE_FACTORY_HEADERS}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/DeviceFactory
)

install(TARGETS DeviceFactory
    EXPORT DeviceFactoryTargets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/DeviceFactory
)

install(EXPORT DeviceFactoryTargets
    FILE DeviceFactoryTargets.cmake
    NAMESPACE DeviceFactory::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/DeviceFactory
)


include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/DeviceFactoryConfigVersion.cmake
  VERSION 1.0.0
  COMPATIBILITY AnyNewerVersion
)

configure_package_config_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/cmake/DeviceFactoryConfig.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/DeviceFactoryConfig.cmake
  INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/DeviceFactory
)

install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/DeviceFactoryConfig.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/DeviceFactoryConfigVersion.cmake
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/DeviceFactory
)


