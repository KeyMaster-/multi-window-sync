cmake_minimum_required(VERSION 3.20)

project(vkPresentLatency C CXX)

# Common directories.
set(BIN_DIR  ${CMAKE_CURRENT_BINARY_DIR}/bin)

set(CMAKE_CXX_STANDARD 17)

### Libraries
# Tracy

set(TRACY_ENABLE ON CACHE BOOL "")
set(TRACY_ONLY_IPV4 ON CACHE BOOL "") # Prevents some connection confusion because the server UI uses 127.0.0.1 by default.
set(TRACY_STATIC OFF CACHE BOOL "") # This is required since we are a multi-dll program.
set(TRACY_ON_DEMAND OFF CACHE BOOL "")
add_subdirectory("${CMAKE_CURRENT_SOURCE_DIR}/tracy")

# Build binaries and pdbs into our output bin/pdb directories.
set_target_properties(TracyClient
    PROPERTIES
    LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})

if(WIN32)
  set_target_properties(TracyClient
      PROPERTIES
      RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})
endif()

# Volk
set(VOLK_STATIC_DEFINES VK_USE_PLATFORM_WIN32_KHR)
add_subdirectory("${CMAKE_CURRENT_SOURCE_DIR}/volk")

find_package(Vulkan REQUIRED)
message("Vulkan_LIBRARY=${Vulkan_LIBRARY}")
find_library(Slang_LIBRARY NAMES slang HINTS "$ENV{VULKAN_SDK}/lib" REQUIRED)


### Executable
set(exec_target vkPresentLatency)

add_executable(${exec_target})
target_sources(${exec_target}
  PRIVATE
    src/main.cpp)

target_compile_definitions(${exec_target}
  PRIVATE
  TRACY_IMPORTS
  VK_NO_PROTOTYPES) # We only want to call functions exposed by Volk

  
target_link_libraries(${exec_target}
  PRIVATE
  Tracy::TracyClient
  # We don't use Vulkan::Vulkan because that would link against vulkan-1, which we do not want when using Volk.
  Vulkan::Headers
  ${Slang_LIBRARY}
  volk::volk)