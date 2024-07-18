### tgui-sdl_render-cmake
TGUI running on SDL_RENDER (cmake project)
----
Fetches and builds TGUI and SDL2, linking them statically.

Intended for Linux and Windows platforms.
*Untested but would probably work on MacOS as well

* Note: Consider this as a WIP template.

### Project structure
| Folder/File Path | Description           |
|-----------------|-----------------------|
| /CMakeLists.txt | Initial setup of project |
| /src/*           | base folder           |
| /src/main.cpp   | the example code      |
| /src/CMakeLists.txt | builds the project |
| /src/assets/*    | icons and etc         |
| /build/*         | used when building    |
| /bin/*           | executable directory  |
| /run_cmake.bat     | calls cmake on Windows |
| /run_cmake.sh     | calls cmake on Linux |

### Configuration
```/CMakeLists.txt``` sets the name of the executable "tgui_Test" and can be changed as desired. 

See Optional dependencies if you do not want to update ```/src/CMakeLists.txt``` when a source file has been added, renamed or removed.

At the bottom of ```/src/CMakeLists.txt``` is the commands to "install" (copy) files into the the ```/bin/``` folder so that the application will run, like the theme file "black.txt".

### Optional dependencies:
```/src/update_sources.py``` can be used to update ```/src/CMakeLists.txt``` *target_sources* section.

Only necessary when files have been added or removed inside ```/src/``` folder.
| Target filetypes | .c | .cpp | .h | .hpp |
|------------------|----|------|----|------|

Python 3.x and ![cog](https://github.com/nedbat/cog) required
