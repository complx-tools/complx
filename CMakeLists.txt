cmake_minimum_required(VERSION 3.5)
# Ignore warnings about rpath behavior on OS X
cmake_policy(SET CMP0042 NEW)
project(complx LANGUAGES CXX VERSION 4.20.2)
include(GNUInstallDirs)

# Add code coverage stuff.
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake)
set(CMAKE_SKIP_RPATH TRUE)

set(wxWidgets_USE_LIBS stc xrc xml html adv net core base)

find_package(wxWidgets REQUIRED)
find_package(liblc3 REQUIRED)
find_package(liblc3_replay REQUIRED)
if(CMAKE_HOST_UNIX)
    include(FindPkgConfig)
    pkg_check_modules(GLIB2 REQUIRED glib-2.0)
endif(CMAKE_HOST_UNIX)

message(${LIBLC3_REPLAY_INCLUDE_DIR})

include(${wxWidgets_USE_FILE})
include_directories(${LIBLC3_INCLUDE_DIR})
include_directories(${LIBLC3_REPLAY_INCLUDE_DIR})
include_directories(${GLIB2_INCLUDE_DIRS})
include_directories(include)
include_directories(source/gen)

# Require C++11 build
if(NOT CMAKE_CXX_FLAGS MATCHES "-std=(c|gnu)\\+\\+11")
    message(STATUS "This project requires C++11. Adding -std=c++11 to CXXFLAGS.")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11" CACHE STRING "Flags used by the compiler during all build types." FORCE)
endif()

# Additional debug flags
set(CMAKE_CXX_FLAGS_DEBUG "-g -pg")
set(CMAKE_DEBUG_POSTFIX d)
# Install directories
set(CMAKE_INSTALL_LIBDIR lib CACHE STRING "where to install libraries/plugins")
set(CMAKE_INSTALL_BINDIR bin CACHE STRING "where to install programs")

# Set default warning flags
set(PROJECT_WARNING_FLAGS "-Wall -Wno-deprecated -Wno-deprecated-declarations" CACHE STRING "Compiler warning flags to include")
mark_as_advanced(PROJECT_WARNING_FLAGS)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${PROJECT_WARNING_FLAGS}")

# Default to release build if not specified
if(NOT CMAKE_BUILD_TYPE)
    message(STATUS "Setting build type to 'Release' as none was specified.")
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build." FORCE)
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release"
    "MinSizeRel" "RelWithDebInfo")
endif(NOT CMAKE_BUILD_TYPE)

if(CMAKE_BUILD_TYPE MATCHES "Debug")
    set(CMAKE_EXE_LINKER_FLAGS "-fprofile-arcs")
endif(CMAKE_BUILD_TYPE MATCHES "Debug")

add_definitions(-DPREFIX=${CMAKE_INSTALL_PREFIX})

set(SRC_COMPLX
    source/gen/AddressDialogDecl.cpp
    source/gen/AdvancedLoadDialogDecl.cpp
    source/gen/BreakpointDialogDecl.cpp
    source/gen/CallStackDialogDecl.cpp
    source/gen/CallSubroutineDialogDecl.cpp
    source/gen/ComplxFrameDecl.cpp
    source/gen/DebugInfoDialogDecl.cpp
    source/gen/LC3ConsoleDecl.cpp
    source/gen/MemoryViewFrameDecl.cpp
    source/gen/RunForDialogDecl.cpp
    source/gen/WatchpointDialogDecl.cpp

    source/AddressDialog.cpp
    source/AdvancedLoadDialog.cpp
    source/BreakpointDialog.cpp
    source/CallStackDialog.cpp
    source/CallSubroutineDialog.cpp
    source/ComplxApp.cpp
    source/ComplxFrame.cpp
    source/DebugInfoDialog.cpp
    source/DebugInfoList.cpp
    source/GridCellBinaryRenderer.cpp
    source/GridCellInfoRenderer.cpp
    source/LC3Console.cpp
    source/LC3RunThread.cpp
    source/MemoryGrid.cpp
    source/MemoryView.cpp
    source/MemoryViewContainer.cpp
    source/MemoryViewFrame.cpp
    source/RunForDialog.cpp
    source/WatchpointDialog.cpp
    source/assets/icon32.xpm
)

add_executable(
    complx
    ${SRC_COMPLX}
)

target_link_libraries(
    complx
    ${LIBLC3_LIBRARIES}
    ${LIBLC3_REPLAY_LIBRARIES}
    ${wxWidgets_LIBRARIES}
    ${GLIB2_LIBRARIES}
)

install(PROGRAMS ${CMAKE_CURRENT_BINARY_DIR}/complx DESTINATION ${CMAKE_INSTALL_BINDIR})
install(FILES doc/complx-tools.pdf DESTINATION share/doc/complx-tools)
install(FILES doc/complx-tips.txt DESTINATION share/doc/complx-tools)
install(FILES assets/complx.png DESTINATION share/icons)
install(FILES assets/complx.desktop DESTINATION share/applications)

# uninstall target
configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/cmake_uninstall.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
    IMMEDIATE @ONLY)

add_custom_target(uninstall
    COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)