cmake_minimum_required(VERSION 3.16)

project(general_developer_whisper_shared_library)

set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -s")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -s")

set(WHISPER_LIBRARY_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/lib/whisper.cpp")

# Platform-specific configurations
if(ANDROID)
  set(GGML_OPENMP OFF)
endif()

# ✅ iOS Specific Configurations
if(IOS)
  set(CMAKE_SYSTEM_NAME iOS)
  set(CMAKE_OSX_SYSROOT iphoneos)  
  set(CMAKE_OSX_DEPLOYMENT_TARGET 13.0)  # ✅ Updated to iOS 13.0

  # Ensure we only build for arm64 (iOS devices)
  set(CMAKE_OSX_ARCHITECTURES "arm64")

  # Set iOS-specific compiler flags
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -arch arm64 -mios-version-min=13.0")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -arch arm64 -mios-version-min=13.0")

  # Enable dSYM (debug symbols) for App Store uploads
  set(CMAKE_XCODE_ATTRIBUTE_DEBUG_INFORMATION_FORMAT "dwarf-with-dsym")
  set(CMAKE_XCODE_ATTRIBUTE_DEAD_CODE_STRIPPING "YES")

  # Disable OpenMP for iOS
  set(GGML_OPENMP OFF)
endif()

set(GGML_CCACHE OFF)

if(UNIX AND NOT APPLE)
  set(CMAKE_BUILD_RPATH "$ORIGIN")
endif()

# ✅ Output path configuration - Only for iOS Devices
if(IOS)
  set(LIBRARY_OUTPUT_PATH ${CMAKE_CURRENT_SOURCE_DIR}/build_ios/Release-iphoneos)
endif()

# Add Whisper library
add_subdirectory(${WHISPER_LIBRARY_DIRECTORY} "whisper.cpp")

# ✅ For iOS, create a shared framework with the correct properties
if(IOS)
  set_target_properties(whisper PROPERTIES
    FRAMEWORK TRUE
    FRAMEWORK_VERSION A
    OUTPUT_NAME "whisper"
    MACOSX_FRAMEWORK_IDENTIFIER com.example.whisper
    MACOSX_FRAMEWORK_BUNDLE_VERSION 1.0.0
    MACOSX_FRAMEWORK_SHORT_VERSION_STRING 1.0.0
    INSTALL_RPATH "@executable_path/Frameworks"
    INSTALL_NAME_DIR "@executable_path/Frameworks"
    BUILD_WITH_INSTALL_RPATH TRUE
    MACOSX_RPATH TRUE
  )

  # Ensure proper linking
  target_link_libraries(whisper PRIVATE "-framework Accelerate")

  # Proper dynamic linking for iOS/macOS
  target_link_options(whisper PRIVATE
    "-Wl,-install_name,@rpath/whisper.framework/whisper"
  )
endif()
