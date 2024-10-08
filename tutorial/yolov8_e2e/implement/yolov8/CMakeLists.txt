cmake_minimum_required(VERSION 3.5)
project(
  yolov8
  VERSION 1.0.0
  LANGUAGES C CXX)
set(CMAKE_CXX_STANDARD 20)

string(APPEND CMAKE_CXX_FLAGS " /Zc:__cplusplus")
#
# unreferenced formal parameter
string(APPEND CMAKE_CXX_FLAGS " /wd4100")
# std::codecvt_utf8<wchar_t,1114111,(std::codecvt_mode)0>': warning STL4017:
# std::wbuffer_convert, std::wstring_convert, and the <codecvt> header
# (containing std::codecvt_mode, std::codecvt_utf8, std::codecvt_utf16, and
# std::codecvt_utf8_utf16) are deprecated in C++17. (The std::codecvt class
# template is NOT deprecated.) The C++ Standard doesn't provide equivalent
# non-deprecated functionality; consider using MultiByteToWideChar() and
# WideCharToMultiByte() from <Windows.h> instead. You can define
# _SILENCE_CXX17_CODECVT_HEADER_DEPRECATION_WARNING or
# _SILENCE_ALL_CXX17_DEPRECATION_WARNINGS to acknowledge that you have received
# this warning.
#
string(APPEND CMAKE_CXX_FLAGS " /wd4996")
string(APPEND CMAKE_CXX_FLAGS " /WX-")
set(CMAKE_COMPILE_WARNING_AS_ERROR OFF)

if(NOT ONNXRUNTIME_ROOTDIR)
  if(WIN32)
    set(ONNXRUNTIME_ROOTDIR "C:/Program Files/onnxruntime")
  else()
    set(ONNXRUNTIME_ROOTDIR "/usr/local")
  endif()
endif()

# The ORT package has a different include directory structure to a local install
# via cmake. We added the path for the pre-built package above. Add the path for
# a local install to support either usage. TODO: If we want to support
# additional EPs being loadable from a local install we also need to add EP
# specific directories under /include/onnxruntime/core/providers
include_directories(
  "${ONNXRUNTIME_ROOTDIR}/include" # Pre-built package
  "${ONNXRUNTIME_ROOTDIR}/include/onnxruntime" # Linux local install to
                                               # /usr/local
  "${ONNXRUNTIME_ROOTDIR}/include/onnxruntime/core/session") # Windows local
                                                             # install
link_directories("${ONNXRUNTIME_ROOTDIR}/lib")
link_directories("${CMAKE_INSTALL_PREFIX}/lib")
# find_package(Eigen3)
find_package(glog REQUIRED)
find_package(
  OpenCV
  COMPONENTS core highgui imgproc
  REQUIRED)

add_executable(camera_yolov8 demo_yolov8_onnx.cpp util/getopt.c
)
target_link_libraries(camera_yolov8 ${ORT_LIBRARY} ${OpenCV_LIBS} onnxruntime glog::glog)
install(TARGETS camera_yolov8 RUNTIME DESTINATION bin)

add_executable(camera_yolov8_nx1x4 demo_yolov8_onnx_n.cpp util/getopt.c
)
target_link_libraries(camera_yolov8_nx1x4 ${ORT_LIBRARY} ${OpenCV_LIBS} onnxruntime glog::glog)
install(TARGETS camera_yolov8_nx1x4 RUNTIME DESTINATION bin)

add_executable(test_jpeg_yolov8 test_yolov8_onnx.cpp util/getopt.c
)
target_link_libraries(test_jpeg_yolov8 ${ORT_LIBRARY} ${OpenCV_LIBS} onnxruntime glog::glog)
install(TARGETS test_jpeg_yolov8 RUNTIME DESTINATION bin)

