cmake_minimum_required(VERSION 3.5)
project(imgui VERSION 1.79)

#########################################################################################
# ImGui Options
#########################################################################################
# Library options
option(BUILD_SHARED_LIBS "Build shared libraries or set OFF for static"            ON)
option(IMGUI_DEMO        "Include the ImGui demo window implementation in library" ON)

# Backend implementation options
set(IMGUI_PLATF_BACKEND  "" CACHE STRING "Platform backend (only glfw for now)")
set(IMGUI_RENDER_BACKEND "" CACHE STRING "Graphics Library backend (only opengl for now")
set(IMGUI_OPENGL_LOADER  "" CACHE STRING "OpenGL loader if selected (glad or glbinding3)")

# Option validations
if(IMGUI_PLATF_BACKEND STREQUAL "")
message(FATAL_ERROR "No ImGui Platform backend selected")
endif()
if(IMGUI_RENDER_BACKEND STREQUAL "")
message(FATAL_ERROR "No ImGui Graphics Library backend selected")
endif()
if(IMGUI_OPENGL_LOADER STREQUAL "" AND IMGUI_RENDER_BACKEND STREQUAL "opengl")
message(WARNING "No OpenGL Loader selected, ImGui will use the first found in the system")
endif()

#########################################################################################
# ImGui Target
#########################################################################################
if(BUILD_SHARED_LIBS)
add_library(imgui SHARED)
else()
add_library(imgui STATIC)
endif()
target_sources(imgui PRIVATE
    "imgui/imgui.cpp"
    "imgui/imgui_draw.cpp"
    "imgui/imgui_widgets.cpp"
    "$<$<BOOL:IMGUI_DEMO>:imgui/imgui_demo.cpp>"
)
target_include_directories(imgui PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/imgui>
    $<INSTALL_INTERFACE:include>)

install(TARGETS imgui EXPORT imguiTargets)
install(FILES "imgui/imgui.h" "imgui/imconfig.h" DESTINATION include/imgui)

#########################################################################################
# Platform Backend GLFW
#########################################################################################
if(IMGUI_PLATF_BACKEND STREQUAL "glfw")
message(STATUS "ImGui selected Platform Backend: GLFW")

if(BUILD_SHARED_LIBS)
add_library(imgui_glfw SHARED)
else()
add_library(imgui_glfw STATIC)
endif()
target_sources(imgui_glfw PRIVATE "imgui/examples/imgui_impl_glfw.cpp")
target_include_directories(imgui_glfw PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/imgui>
    $<INSTALL_INTERFACE:include>)
find_package(glfw3 REQUIRED)
target_link_libraries(imgui_glfw PUBLIC glfw)

install(TARGETS imgui_glfw EXPORT imguiTargets)
install(FILES "imgui/examples/imgui_impl_glfw.h" DESTINATION include/imgui)

endif()

#########################################################################################
# Render Backend OpenGL
#########################################################################################
if(IMGUI_RENDER_BACKEND STREQUAL "opengl")
message(STATUS "ImGui selected Rederer Backend: OpenGL")

if(BUILD_SHARED_LIBS)
add_library(imgui_opengl SHARED)
else()
add_library(imgui_opengl STATIC)
endif()
target_sources(imgui_opengl PRIVATE "imgui/examples/imgui_impl_opengl3.cpp")
target_include_directories(imgui_opengl PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/imgui>
    $<INSTALL_INTERFACE:include>)

# Opengl Loader: GLAD
if(IMGUI_OPENGL_LOADER STREQUAL "glad")
message(STATUS "ImGui selected OpenGL Loader: GLAD")
find_package(glad REQUIRED)
target_compile_definitions(imgui_opengl PUBLIC IMGUI_IMPL_OPENGL_LOADER_GLAD)
target_link_libraries(imgui_opengl PUBLIC glad::glad)
endif()

# Opengl Loader: GLBinding3
if(IMGUI_OPENGL_LOADER STREQUAL "glbinding3")
message(STATUS "ImGui selected OpenGL Loader: GLBinding3")
find_package(glbinding REQUIRED)
target_compile_definitions(imgui_opengl PUBLIC IMGUI_IMPL_OPENGL_LOADER_GLBINDING3)
target_link_libraries(imgui_opengl PUBLIC glbinding::glbinding)
endif()

install(TARGETS imgui_opengl EXPORT imguiTargets)
install(FILES "imgui/examples/imgui_impl_opengl3.h" DESTINATION include/imgui)

endif()


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
