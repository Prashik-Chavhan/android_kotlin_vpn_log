
# For more information about using CMake with Android Studio, read the
# documentation: https://d.android.com/studio/projects/add-native-code.html.
# For more examples on how to use CMake, see https://github.com/android/ndk-samples.

# Sets the minimum CMake version required for this project.
cmake_minimum_required(VERSION 3.22.1)

# Declares the project name. The project name can be accessed via ${ PROJECT_NAME},
# Since this is the top level CMakeLists.txt, the project name is also accessible
# with ${CMAKE_PROJECT_NAME} (both CMake variables are in-sync within the top level
# build script scope).
project("firewallapp")

# Set C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Creates and names a library, sets it as either STATIC
# or SHARED, and provides the relative paths to its source code.
# You can define multiple libraries, and CMake builds them for you.
# Gradle automatically packages shared libraries with your APK.
#
# In this top level CMakeLists.txt, ${CMAKE_PROJECT_NAME} is used to define
# the target library name; in the sub-module's CMakeLists.txt, ${PROJECT_NAME}
# is preferred for the same purpose.
#
# In order to load a library into your app from Java/Kotlin, you must call
# System.loadLibrary() and pass the name of the library defined here;
# for GameActivity/NativeActivity derived applications, the same library name must be
# used in the AndroidManifest.xml file.
add_library(${CMAKE_PROJECT_NAME} SHARED
    # List C/C++ source files with relative paths to this CMakeLists.txt.
    firewallapp.cpp)

# Include PcapPlusPlus and libpcap headers
include_directories(
        ${CMAKE_SOURCE_DIR}/src/main/cpp/include/pcapplusplus
        ${CMAKE_SOURCE_DIR}/src/main/cpp/include/pcap
)

# Import PcapPlusPlus static libraries
add_library(Common++ STATIC IMPORTED)
set_target_properties(Common++ PROPERTIES IMPORTED_LOCATION
        ${CMAKE_SOURCE_DIR}/libs/arm64-v8a/libCommon++.a)

add_library(Packet++ STATIC IMPORTED)
set_target_properties(Packet++ PROPERTIES IMPORTED_LOCATION
        ${CMAKE_SOURCE_DIR}/libs/arm64-v8a/libPacket++.a)

add_library(Pcap++ STATIC IMPORTED)
set_target_properties(Pcap++ PROPERTIES IMPORTED_LOCATION
        ${CMAKE_SOURCE_DIR}/libs/arm64-v8a/libPcap++.a)

# Link required system libraries
find_library(log-lib log)
find_library(android-lib android)
find_library(z-lib z)

# Specifies libraries CMake should link to your target library. You
# can link libraries from various origins, such as libraries defined in this
# build script, prebuilt third-party libraries, or Android system libraries.
# Link libraries to your native lib
target_link_libraries(${CMAKE_PROJECT_NAME}
        Common++
        Packet++
        Pcap++
        ${android-lib}
        ${log-lib}
        ${z-lib}
)
