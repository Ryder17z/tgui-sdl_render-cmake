# You only need one for the whole project.
cmake_minimum_required (VERSION 3.21)

# Don't set() the project name first and then pass it to project()
project("tgui_Test" VERSION 0.1)

#FetchContent is better than ExternalContent if the external lib uses CMake
include(FetchContent)

option(NO_EXTERNAL_LIBS "Disable fetching external libraries before building" OFF)

if(NO_EXTERNAL_LIBS)
    find_package(SDL2 REQUIRED)
    set(TGUI_STATIC_LIBRARIES TRUE)
    find_package(TGUI 1 REQUIRED)
endif()

add_subdirectory (src)
