cmake_minimum_required (VERSION 3.15)

project (csofa LANGUAGES C)

include(GNUInstallDirs)

# set SOURCE dir
set(SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/src")

# set .c files except for test_sofa.c
file(GLOB_RECURSE SOURCES "${SOURCE_DIR}/*.c")
list(REMOVE_ITEM SOURCES "${SOURCE_DIR}/test_sofa.c")

# set .h files
file(GLOB_RECURSE HEADERS "${SOURCE_DIR}/*.h")

# Add your library
add_library(csofa ${SOURCES})

if (BUILD_SHARED_LIBS AND MSVC)
    target_compile_definitions(csofa PRIVATE MYLIB_EXPORTS)
endif()

# Add include directories
target_include_directories(csofa PUBLIC
    $<BUILD_INTERFACE:${SOURCE_DIR}>   # for headers when building
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>  # for client in install mode
)
# Install the library and its headers
install(TARGETS csofa
        EXPORT csofa_targets
        ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
        LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
        RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}")

install(FILES ${HEADERS} DESTINATION include)


# Generate and install *-targets.cmake 
install(EXPORT csofa_targets
        FILE csofa-targets.cmake
        NAMESPACE csofa::
        DESTINATION share/csofa)

# Generate the config file in the current binary dir (this ensures it's not placed directly in source)
file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/csofa-config.cmake"
"include(CMakeFindDependencyMacro)\n"
"include(\"\${CMAKE_CURRENT_LIST_DIR}/csofa-targets.cmake\")\n"
)

# Install the generated config file
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/csofa-config.cmake"
        DESTINATION share/csofa)

