cmake_minimum_required(VERSION 3.28)

project("imgui_md" LANGUAGES C CXX)

set(CMAKE_C_STANDARD 17)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS OFF)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# --- Dependency configuration ---
#
# Two orthogonal decisions are exposed per dependency:
#   BUILD_*   : should imgui_md build the dependency internally (via CPM)?
#   *_TARGET  : what is the name of the target imgui_md links against?
#
# Defaults build both dependencies internally under their canonical target
# names, so a standalone build works out of the box. A consumer that already
# provides ImGui and/or md4c sets the matching BUILD_* to OFF and points
# *_TARGET at its own target (e.g. -DBUILD_IMGUI=OFF -DIMGUI_TARGET=imgui::imgui).

option(BUILD_IMGUI "Build the ImGui static library internally." ON)
set(IMGUI_TARGET "ImGui" CACHE STRING "Name of the ImGui target to link against.")

option(BUILD_MD4C "Build the md4c static library internally." ON)
set(MD4C_TARGET "md4c" CACHE STRING "Name of the md4c target to link against.")

# --- #

if(NOT COMMAND CPMAddPackage)
    include(cmake/get_cpm.cmake)
endif()


if(BUILD_IMGUI)
    # ----- Dear ImGui (docking branch) -----
    message(STATUS "[imgui] building ImGui internally via CPM")
    CPMAddPackage(
        NAME ImGui
        VERSION 1.92.6-docking
        GIT_SHALLOW TRUE
        GITHUB_REPOSITORY ocornut/imgui
        DOWNLOAD_ONLY ON
    )

    # ----- ImGui core static library -----

    add_library(ImGui STATIC)
    target_sources(ImGui
        PRIVATE
            ${ImGui_SOURCE_DIR}/imgui.cpp
            ${ImGui_SOURCE_DIR}/imgui.h
            ${ImGui_SOURCE_DIR}/imgui_demo.cpp
            ${ImGui_SOURCE_DIR}/imgui_draw.cpp
            ${ImGui_SOURCE_DIR}/imgui_internal.h
            ${ImGui_SOURCE_DIR}/imgui_tables.cpp
            ${ImGui_SOURCE_DIR}/imgui_widgets.cpp
            ${ImGui_SOURCE_DIR}/imstb_rectpack.h
            ${ImGui_SOURCE_DIR}/imstb_textedit.h
            ${ImGui_SOURCE_DIR}/imstb_truetype.h
    )
    target_include_directories(ImGui
        PUBLIC
            ${ImGui_SOURCE_DIR}
    )
else()
    message(STATUS "[imgui] using consumer-provided target '${IMGUI_TARGET}'")
endif()

if(BUILD_MD4C)
    # ----- MD4C (CommonMark parser, C library) -----
    #
    # Pulled via CPM. MD4C is the parser imgui_md is built on.
    # We disable md2html and md4c-html since this library only needs the parser core.
    message(STATUS "[md4c] building md4c internally via CPM")
    CPMAddPackage(
        NAME md4c
        VERSION 0.5.3
        GIT_SHALLOW TRUE
        GITHUB_REPOSITORY mity/md4c
        GIT_TAG release-0.5.3
        OPTIONS
            "BUILD_MD2HTML_EXECUTABLE OFF"
            "BUILD_SHARED_LIBS OFF"
    )
else()
    message(STATUS "[md4c] using consumer-provided target '${MD4C_TARGET}'")
endif()

add_library(imgui_md STATIC)

target_sources(imgui_md
    PRIVATE
        imgui_md.h
        imgui_md.cpp
)
target_include_directories(imgui_md
    PUBLIC
        ${CMAKE_CURRENT_SOURCE_DIR}
)
target_link_libraries(imgui_md PUBLIC ${IMGUI_TARGET} ${MD4C_TARGET})
