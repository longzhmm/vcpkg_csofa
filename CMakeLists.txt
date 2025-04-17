cmake_minimum_required (VERSION 3.15)

project (iausofa LANGUAGES C)

include(GNUInstallDirs)

# set SOURCE dir
set(SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/src")

# set .c files except for test_sofa.c
file(GLOB_RECURSE SOURCES "${SOURCE_DIR}/*.c")
list(REMOVE_ITEM SOURCES "${SOURCE_DIR}/test_sofa.c")

# set .h files
file(GLOB_RECURSE HEADERS "${SOURCE_DIR}/*.h")

# Add your library
add_library(iausofa ${SOURCES})

if (BUILD_SHARED_LIBS AND MSVC)
    set_target_properties(iausofa PROPERTIES WINDOWS_EXPORT_ALL_SYMBOLS ON)
endif()

# Add include directories
target_include_directories(iausofa PUBLIC
    $<BUILD_INTERFACE:${SOURCE_DIR}>   # for headers when building
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>  # for client in install mode
)
# Install the library and its headers
install(TARGETS iausofa
        EXPORT iausofa_targets
        ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
        LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
        RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}")

install(FILES ${HEADERS} DESTINATION include)


# Generate and install *-targets.cmake 
install(EXPORT iausofa_targets
        FILE iausofa-targets.cmake
        NAMESPACE iausofa::
        DESTINATION share/iausofa)

# Generate the config file in the current binary dir (this ensures it's not placed directly in source)
file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/iausofa-config.cmake"
"include(CMakeFindDependencyMacro)\n"
"include(\"\${CMAKE_CURRENT_LIST_DIR}/iausofa-targets.cmake\")\n"
)

# Install the generated config file
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/iausofa-config.cmake"
        DESTINATION share/iausofa)

