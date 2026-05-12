cmake_minimum_required(VERSION 3.28)

project("imgui_md" LANGUAGES C CXX)

set(CMAKE_C_STANDARD 17)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS OFF)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# --- #

if(NOT COMMAND CPMAddPackage)
    include(cmake/get_cpm.cmake)
endif()


if(NOT TARGET ImGui)
    # ----- Dear ImGui (docking branch) -----
    message(STATUS "[imgui] ImGui not present, fetching via CPM")
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
    message(STATUS "[imgui] ImGui already provided by consumer, skipping")
endif()

if(NOT TARGET md4c)
    # ----- MD4C (CommonMark parser, C library) -----
    #
    # Pulled via CPM. MD4C is the parser imgui_md is built on.
    # We disable md2html and md4c-html since this library only needs the parser core.
    message(STATUS "[md4c] md4c not present, fetching via CPM")
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
    message(STATUS "[md4c] md4c already provided by consumer, skipping")
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
target_link_libraries(imgui_md PUBLIC ImGui md4c)
