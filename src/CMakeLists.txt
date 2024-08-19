set(BUILD_SHARED_LIBS FALSE) # force use of static libs

# GLOB can cause issues in the CMake cache and IDEs
# See https://stackoverflow.com/questions/1027247/is-it-better-to-specify-source-files-with-glob-or-each-file-individually-in-cmak/1060061#1060061
file(GLOB_RECURSE project_src
        "*.h"
        "*.hpp"
        "*.cpp"
        "*.c"
)
# Only used to set up "filters" in (not vscode)
foreach(_source IN ITEMS ${project_src})
    get_filename_component(_source_path "${_source}" PATH)
    file(RELATIVE_PATH _source_path_rel ${CMAKE_CURRENT_LIST_DIR} "${_source_path}")
    string(REPLACE "/" "\\" _group_path "${_source_path_rel}")
    source_group("${_group_path}" FILES "${_source}")
endforeach()

# .exe
add_executable (${PROJECT_NAME})

# Use target_sources() to add source dependencies
# You can repeat it as many times as you want in any subdir too!
target_sources(${PROJECT_NAME}
    PRIVATE
        #${project_src}
# [[[cog
#    import cog, os
#    file_paths = []
#    for root, dirs, files in os.walk('.'):
#        for file in files:
#            if file.endswith('.cpp') or file.endswith('.hpp') or file.endswith('.c') or file.endswith('.h'):
#                if "skeleton" in file: continue
#                file_path = os.path.relpath(os.path.join(root, file), '.')
#                file_paths.append(file_path)
#    file_paths.sort(key=lambda x: (x != 'main.cpp', x))
#    for file_path in file_paths:
#               cog.outl("\"%s\"" % file_path)
# ]]]
"main.cpp"
# [[[end]]]
# [[[
)

# Keep target_link_libraries() in the same place where you create targets
# (Not a requirement)
if(NO_EXTERNAL_LIBS)
    target_link_libraries(${PROJECT_NAME}
        PRIVATE
        Freetype::Freetype
        SDL2::SDL2
        SDL2_ttf::SDL2_ttf
        TGUI::TGUI
    )
else()
    message(STATUS "Fetching Dependencies: SDL2, SDL2_TTF, Freetype, TGUI")

    # This downloads Freetype directly to the binary dir
    FetchContent_Declare(Freetype
        GIT_REPOSITORY https://gitlab.freedesktop.org/freetype/freetype.git
    )
    target_link_libraries(${PROJECT_NAME}
        PRIVATE
        Freetype::Freetype
    )

    # This downloads SDL2 directly to the binary dir
    FetchContent_Declare(SDL2-static
        GIT_REPOSITORY https://github.com/libsdl-org/SDL.git
        GIT_TAG 132b88749c284e4d51d93f1e08f5bb0ff9d705dc
        # Make sure we're building SDL2 statically!
        CMAKE_ARGS
            -DSDL_STATIC=ON
            -DSDL_SHARED=OFF # Force only static building/linking
    )
    # Expose SDL to the project like a find_package()
    # Only possible because SDL uses CMake!
    FetchContent_MakeAvailable(SDL2-static)
    target_link_libraries(${PROJECT_NAME}
        PRIVATE
        SDL2-static
    )

    # Set the options required for a static build before fetching content
    set(FETCHCONTENT_QUIET FALSE)
    set(SDL2_TTF_STATIC ON CACHE BOOL "" FORCE)
    set(SDL2TTF_VENDORED ON CACHE BOOL "" FORCE)
    # This downloads SDL_TTF directly to the binary dir
    FetchContent_Declare(SDL2_TTF
        GIT_REPOSITORY https://github.com/libsdl-org/SDL_ttf.git
        GIT_TAG 4a318f8dfaa1bb6f10e0c5e54052e25d3c7f3440 # 2.22.0
        # Make sure we're building SDL2_TTF statically!
        CMAKE_ARGS
            -DSDL2_TTF_STATIC=ON
            -DSDL_SHARED=OFF # Force only static building/linking
            -DSDL2TTF_VENDORED=ON
            #-DSDL2_TTF_TARGET_NAME=SDL2_TTF-static # doesn't do anything?
    )
    # Fetch the content and make it available
    FetchContent_MakeAvailable(SDL2_TTF)
    target_link_libraries(${PROJECT_NAME}
        PRIVATE
        SDL2_ttf
    )

    # set up TGUI parameters before downloading
    set(TGUI_STATIC_LIBRARIES TRUE)
    set(TGUI_CXX_STANDARD 17)
    set(TGUI_BACKEND SDL_RENDERER)
    # This downloads TGUI directly to the binary dir
    FetchContent_Declare(TGUI
        GIT_REPOSITORY https://github.com/texus/TGUI.git
        GIT_TAG v1.4.0
        
        #CMAKE_ARGS -DTGUI_CXX_STANDARD=17 -DTGUI_SHARED_LIBS=FALSE
    )
    # Expose TGUI to the project like a find_package()
    # Only possible because TGUI uses CMake!
    FetchContent_MakeAvailable(TGUI)
    target_link_libraries(${PROJECT_NAME}
        PRIVATE
        TGUI::TGUI
    )
endif()


# Set the correct debugger working directory in case Visual Studio is to be used
set_property(TARGET ${PROJECT_NAME} PROPERTY VS_DEBUGGER_WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}/bin")


# Without the trailing / after assets, it would copy the directory instead of its contents
install(TARGETS ${PROJECT_NAME} DESTINATION ${CMAKE_SOURCE_DIR}/bin) #exectuable

install(DIRECTORY "$<TARGET_FILE:tgui>" DESTINATION ${CMAKE_SOURCE_DIR}/bin) # libtgui.a

install(DIRECTORY ${PROJECT_SOURCE_DIR}/src/assets/ DESTINATION ${CMAKE_SOURCE_DIR}/bin) # bucket.png and other assets

install(DIRECTORY ../build/_deps/tgui-src/themes/ DESTINATION ${CMAKE_SOURCE_DIR}/bin/themes) # black.txt and etc
