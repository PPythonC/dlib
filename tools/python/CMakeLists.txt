
CMAKE_MINIMUM_REQUIRED(VERSION 3.10.0)

if (WIN32 AND NOT "${CMAKE_GENERATOR}" MATCHES "Visual Studio") 
   message(FATAL_ERROR "\n"
      "!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!\n"
      "You must use Visual Studio to build a python extension on windows. If you "
      "are getting this error it means you have not installed Visual C++.  Note that "
      "there are many flavors of Visual Studio, like Visual Studio for C\# development. " 
      "You need to install Visual Studio for C++. \n"
      "!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!\n")
endif()

project(dlib_python_bindings)

# Pybind11's cmake scripts enable link time optimization by default.  However,
# it makes linking take a really long time and doesn't seem to substantively
# improve runtime performance.  So we disable LTO here to make building dlib
# faster.
set(PYBIND11_LTO_CXX_FLAGS "")


# Avoid cmake warnings about changes in behavior of some Mac OS X path 
# variable we don't care about.
if (POLICY CMP0042)
    cmake_policy(SET CMP0042 NEW)
endif()


# To avoid dll hell, always link everything statically when compiling in
# visual studio.  This way, the resulting library won't depend on a bunch
# of other dll files and can be safely copied to someone else's computer
# and expected to run.
if (MSVC)
    include(${CMAKE_CURRENT_LIST_DIR}/../../dlib/cmake_utils/tell_visual_studio_to_use_static_runtime.cmake)
endif()

add_subdirectory(../../dlib/external/pybind11 pybind11_build)
add_subdirectory(../../dlib dlib_build)

add_definitions(-DDLIB_VERSION=${DLIB_VERSION})

# Tell cmake to compile all these cpp files into a dlib python module.
set(python_srcs
   src/dlib.cpp
   src/matrix.cpp
   src/vector.cpp
   src/svm_c_trainer.cpp
   src/svm_rank_trainer.cpp
   src/decision_functions.cpp
   src/other.cpp
   src/basic.cpp
   src/cca.cpp
   src/sequence_segmenter.cpp
   src/svm_struct.cpp
   src/image.cpp
   src/image2.cpp
   src/image3.cpp
   src/image4.cpp
   src/rectangles.cpp
   src/object_detection.cpp
   src/shape_predictor.cpp
   src/correlation_tracker.cpp
   src/face_recognition.cpp
   src/cnn_face_detector.cpp
   src/global_optimization.cpp
   src/image_dataset_metadata.cpp
   src/numpy_returns.cpp
   src/line.cpp
)

# Only add the GUI module if requested
if(NOT ${DLIB_NO_GUI_SUPPORT})
   list(APPEND python_srcs src/gui.cpp)
endif()

pybind11_add_module(_dlib_pybind11 ${python_srcs})
target_link_libraries(_dlib_pybind11 PRIVATE dlib::dlib)

# When invoked from setup.py CMAKE_LIBRARY_OUTPUT_DIRECTORY will be set.  But when you are just building via say
# `cd dlib/tools/python; mkdir build; cd build; cmake ..` it won't be set.  It only matters for building the actual
# distribution.  So skip this for people building directly with cmake (which they might do when testing stuff).
if (CMAKE_LIBRARY_OUTPUT_DIRECTORY) 
   configure_file(${PROJECT_SOURCE_DIR}/dlib/__init__.py.in ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/dlib/__init__.py)
endif()
