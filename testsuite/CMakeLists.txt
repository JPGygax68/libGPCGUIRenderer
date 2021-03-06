cmake_minimum_required(VERSION 3.0)

add_library(libGPCGUICanvasTestsuite STATIC dummy.cpp include/gpc/gui/canvas_testsuite.hpp include/gpc/gui/fonts.hpp)

target_link_libraries(libGPCGUICanvasTestsuite PRIVATE libGPCGUICanvas)

target_include_directories(libGPCGUICanvasTestsuite PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)

# Embed a font

if (NOT GPCFontRasterizer)
  find_package(GPCFontRasterizer REQUIRED)
endif()

add_custom_command(
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/LiberationSans-Regular-20.rft
  COMMAND GPCFontRasterizer input="${CMAKE_CURRENT_SOURCE_DIR}/LiberationSans-Regular.ttf" size=20 output="${CMAKE_CURRENT_BINARY_DIR}/LiberationSans-Regular-20.rft"
)

if (NOT GPCBin2C)
  find_package(GPCBin2C REQUIRED)
endif()

add_custom_command(
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/LiberationSans-Regular-20.rft.h
  DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/LiberationSans-Regular-20.rft
  COMMAND GPCBin2C --input=${CMAKE_CURRENT_BINARY_DIR}/LiberationSans-Regular-20.rft --output=${CMAKE_CURRENT_BINARY_DIR}/LiberationSans-Regular-20.rft.h
)

target_sources(libGPCGUICanvasTestsuite PUBLIC ${CMAKE_CURRENT_BINARY_DIR}/LiberationSans-Regular-20.rft.h)

target_include_directories(libGPCGUICanvasTestsuite PUBLIC ${CMAKE_CURRENT_BINARY_DIR})

# We need SDL2

find_library(SDL2_LIB SDL2)
if (NOT SDL2_LIB) 
    message(ERROR "Couldn't find SDL2 library")
endif()

add_library(SDL2 STATIC IMPORTED)
set_target_properties(SDL2 PROPERTIES IMPORTED_LOCATION ${SDL2_LIB})
find_path(SDL2_INCLUDE_DIR SDL2/SDL.h)
if (NOT SDL2_INCLUDE_DIR) 
    message(ERROR "Couldn't find SDL2 include directory")
endif()
set_target_properties(SDL2 PROPERTIES INTERFACE_INCLUDE_DIRECTORIES ${SDL2_INCLUDE_DIR})

find_library(SDL2_MAIN_LIB SDL2main)
if (NOT SDL2_MAIN_LIB) 
    message(ERROR "Couldn't find SDL2 \"main\" object file")
endif()

add_library(SDL2_MAIN STATIC IMPORTED)
set_target_properties(SDL2_MAIN PROPERTIES IMPORTED_LOCATION ${SDL2_MAIN_LIB})

target_link_libraries(libGPCGUICanvasTestsuite PRIVATE SDL2 SDL2_MAIN)

find_library(SDL2_IMAGE_LIB SDL2_image)
if (NOT SDL2_IMAGE_LIB) 
    message(ERROR "Couldn't find SDL2_image library")
endif()

add_library(SDL2_IMAGE STATIC IMPORTED)
set_target_properties(SDL2_IMAGE PROPERTIES IMPORTED_LOCATION ${SDL2_IMAGE_LIB})

target_link_libraries(libGPCGUICanvasTestsuite PRIVATE SDL2_IMAGE)

# Boost Concept Checking Library

set(Boost_USE_STATIC_LIBS        ON) # only find static libs
set(Boost_USE_MULTITHREADED      ON)
set(Boost_USE_STATIC_RUNTIME     ON)
find_package(Boost REQUIRED)
target_include_directories(libGPCGUICanvasTestsuite PUBLIC ${Boost_INCLUDE_DIRS})

# GPC Fonts

find_package(libGPCFonts REQUIRED)
#target_include_directories(TestApp PRIVATE libGPCFonts)
target_link_libraries(libGPCGUICanvasTestsuite PRIVATE libGPCFonts)
