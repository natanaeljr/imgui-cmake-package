cmake_minimum_required(VERSION 3.5)
project(imgui VERSION 1.79)

#########################################################################################
# ImGui
#########################################################################################
# Find all ImGui files
file(GLOB IMGUI_HEADERS "imgui/*.h")
file(GLOB IMGUI_SOURCES "imgui/*.cpp")

# ImGui target
add_library(imgui STATIC ${IMGUI_SOURCES})
target_include_directories(imgui INTERFACE
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>"
    "$<INSTALL_INTERFACE:include>")

# Install ImGui
install(TARGETS imgui EXPORT imguiTargets)
install(FILES ${IMGUI_HEADERS} DESTINATION include/imgui)

#########################################################################################
# Package Configuration
#########################################################################################
# Helpers functions for creating config files that can be included by other projects
include(CMakePackageConfigHelpers)

# Create config package
configure_package_config_file(
    "Config.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/imguiConfig.cmake"
    INSTALL_DESTINATION "lib/cmake/imgui")
write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/imguiConfigVersion.cmake"
    COMPATIBILITY SameMajorVersion)

# Install config package
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/imguiConfig.cmake"
              "${CMAKE_CURRENT_BINARY_DIR}/imguiConfigVersion.cmake"
        DESTINATION "lib/cmake/imgui")
install(EXPORT "imguiTargets"
        NAMESPACE "imgui::"
        DESTINATION "lib/cmake/imgui")
