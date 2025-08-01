project("MarathonRecomp")

if (WIN32)
    option(MARATHON_RECOMP_D3D12 "Add D3D12 support for rendering" ON)
endif()

if (APPLE)
    option(MARATHON_RECOMP_METAL "Add Metal support for rendering" ON)
endif()

if (CMAKE_SYSTEM_NAME MATCHES "Linux")
    option(MARATHON_RECOMP_FLATPAK "Configure the build for Flatpak compatibility." OFF)
endif()

function(BIN2C)
    cmake_parse_arguments(BIN2C_ARGS "" "TARGET_OBJ;SOURCE_FILE;DEST_FILE;ARRAY_NAME;COMPRESSION_TYPE" "" ${ARGN})

    if(NOT BIN2C_ARGS_TARGET_OBJ)
        message(FATAL_ERROR "TARGET_OBJ not specified.")
    endif()

    if(NOT BIN2C_ARGS_SOURCE_FILE)
        message(FATAL_ERROR "SOURCE_FILE not specified.")
    endif()

    if(NOT BIN2C_ARGS_DEST_FILE)
        set(BIN2C_ARGS_DEST_FILE "${BIN2C_ARGS_SOURCE_FILE}")
    endif()

    if(NOT BIN2C_ARGS_COMPRESSION_TYPE)
        set(BIN2C_ARGS_COMPRESSION_TYPE "none")
    endif()

    add_custom_command(OUTPUT "${BIN2C_ARGS_DEST_FILE}.c"
        COMMAND $<TARGET_FILE:file_to_c> "${BIN2C_ARGS_SOURCE_FILE}" "${BIN2C_ARGS_ARRAY_NAME}" "${BIN2C_ARGS_COMPRESSION_TYPE}" "${BIN2C_ARGS_DEST_FILE}.c" "${BIN2C_ARGS_DEST_FILE}.h"
        DEPENDS "${BIN2C_ARGS_SOURCE_FILE}"
        BYPRODUCTS "${BIN2C_ARGS_DEST_FILE}.h"
        COMMENT "Generating binary header for ${BIN2C_ARGS_SOURCE_FILE}..."
    )

    set_source_files_properties(${BIN2C_ARGS_DEST_FILE}.c PROPERTIES SKIP_PRECOMPILE_HEADERS ON)
    target_sources(${BIN2C_ARGS_TARGET_OBJ} PRIVATE ${BIN2C_ARGS_DEST_FILE}.c)
endfunction()

add_compile_options(
    -fno-strict-aliasing
    -Wno-switch
    -Wno-unused-function
    -Wno-unused-variable
    -Wno-unused-but-set-variable
    -Wno-void-pointer-to-int-cast
    -Wno-int-to-void-pointer-cast
    -Wno-invalid-offsetof
    -Wno-null-arithmetic
    -Wno-null-conversion
    -Wno-tautological-undefined-compare
)

if (WIN32)
    add_compile_options(/fp:strict)
else()
    add_compile_options(-ffp-model=strict)
endif()

add_compile_definitions(
    SDL_MAIN_HANDLED
    _DISABLE_CONSTEXPR_MUTEX_CONSTRUCTOR # Microsoft wtf?
    _CRT_SECURE_NO_WARNINGS)

set(MARATHON_RECOMP_PRECOMPILED_HEADERS
    "stdafx.h"
)

set(MARATHON_RECOMP_KERNEL_CXX_SOURCES
    "kernel/imports.cpp"
    "kernel/xdm.cpp"
    "kernel/heap.cpp"
    "kernel/memory.cpp"
    "kernel/xam.cpp"
    "kernel/io/file_system.cpp"
)

set(MARATHON_RECOMP_LOCALE_CXX_SOURCES
    "locale/config_locale.cpp"
    "locale/locale.cpp"
)

if (WIN32)
    set(MARATHON_RECOMP_OS_CXX_SOURCES
        "os/win32/logger_win32.cpp"
        "os/win32/media_win32.cpp"
        "os/win32/process_win32.cpp"
        "os/win32/user_win32.cpp"
        "os/win32/version_win32.cpp"
    )
elseif (CMAKE_SYSTEM_NAME MATCHES "Linux")
    set(MARATHON_RECOMP_OS_CXX_SOURCES
        "os/linux/logger_linux.cpp"
        "os/linux/media_linux.cpp"
        "os/linux/process_linux.cpp"
        "os/linux/user_linux.cpp"
        "os/linux/version_linux.cpp"
    )
elseif (APPLE)
    set(MARATHON_RECOMP_OS_CXX_SOURCES
        "os/macos/logger_macos.cpp"
        "os/macos/media_macos.cpp"
        "os/macos/process_macos.cpp"
        "os/macos/user_macos.cpp"
        "os/macos/version_macos.cpp"
    )
endif()

set(MARATHON_RECOMP_CPU_CXX_SOURCES
    "cpu/guest_thread.cpp"
)

set(MARATHON_RECOMP_GPU_CXX_SOURCES
    "gpu/video.cpp"
    "gpu/imgui/imgui_common.cpp"
    "gpu/imgui/imgui_font_builder.cpp"
    "gpu/imgui/imgui_snapshot.cpp"
)

set(MARATHON_RECOMP_APU_CXX_SOURCES
    "apu/audio.cpp"
    "apu/xma_decoder.cpp"
    "apu/embedded_player.cpp"
    "apu/driver/sdl2_driver.cpp"
)

set(MARATHON_RECOMP_HID_CXX_SOURCES
    "hid/hid.cpp"
    "hid/driver/sdl_hid.cpp"
)

set(MARATHON_RECOMP_PATCHES_CXX_SOURCES
    "patches/aspect_ratio_patches.cpp"
    "patches/audio_patches.cpp"
    "patches/camera_patches.cpp"
    "patches/frontend_listener.cpp"
    "patches/misc_patches.cpp"
    "patches/TitleTask_patches.cpp"
    "patches/video_patches.cpp"
)

set(MARATHON_RECOMP_UI_CXX_SOURCES
    "ui/achievement_menu.cpp"
    "ui/achievement_overlay.cpp"
    "ui/black_bar.cpp"
    "ui/button_guide.cpp"
    "ui/fader.cpp"
    "ui/game_window.cpp"
    "ui/imgui_utils.cpp"
    "ui/installer_wizard.cpp"
    "ui/message_window.cpp"
    "ui/options_menu.cpp"
    "ui/options_menu_thumbnails.cpp"
    "ui/tv_static.cpp"
)

set(MARATHON_RECOMP_INSTALL_CXX_SOURCES
    "install/installer.cpp"
    "install/iso_file_system.cpp"
    "install/update_checker.cpp"
    "install/xcontent_file_system.cpp"
    "install/hashes/game.cpp"
    "install/hashes/episode_sonic.cpp"
    "install/hashes/episode_shadow.cpp"
    "install/hashes/episode_silver.cpp"
    "install/hashes/episode_amigo.cpp"
    "install/hashes/mission_sonic.cpp"
    "install/hashes/mission_shadow.cpp"
    "install/hashes/mission_silver.cpp"
)

set(MARATHON_RECOMP_USER_CXX_SOURCES
    "user/achievement_data.cpp"
    "user/achievement_manager.cpp"
    "user/config.cpp"
    "user/registry.cpp"
    "user/paths.cpp"
)

set(MARATHON_RECOMP_MOD_CXX_SOURCES
    "mod/mod_loader.cpp"
)

set(MARATHON_RECOMP_UTILS_CXX_SOURCES
    "utils/bit_stream.cpp"
    "utils/ring_buffer.cpp"
)

set(MARATHON_RECOMP_THIRDPARTY_SOURCES
    "${MARATHON_RECOMP_THIRDPARTY_ROOT}/imgui/backends/imgui_impl_sdl2.cpp"
    "${MARATHON_RECOMP_THIRDPARTY_ROOT}/imgui/imgui.cpp"
    "${MARATHON_RECOMP_THIRDPARTY_ROOT}/imgui/imgui_demo.cpp"
    "${MARATHON_RECOMP_THIRDPARTY_ROOT}/imgui/imgui_draw.cpp"
    "${MARATHON_RECOMP_THIRDPARTY_ROOT}/imgui/imgui_tables.cpp"
    "${MARATHON_RECOMP_THIRDPARTY_ROOT}/imgui/imgui_widgets.cpp"
    "${MARATHON_RECOMP_THIRDPARTY_ROOT}/implot/implot.cpp"
    "${MARATHON_RECOMP_THIRDPARTY_ROOT}/implot/implot_demo.cpp"
    "${MARATHON_RECOMP_THIRDPARTY_ROOT}/implot/implot_items.cpp"
    "${MARATHON_RECOMP_TOOLS_ROOT}/XenosRecomp/thirdparty/smol-v/source/smolv.cpp"
)

set(MARATHON_RECOMP_THIRDPARTY_INCLUDES
    "${MARATHON_RECOMP_THIRDPARTY_ROOT}/concurrentqueue"
    "${MARATHON_RECOMP_THIRDPARTY_ROOT}/ddspp"
    "${MARATHON_RECOMP_THIRDPARTY_ROOT}/imgui"
    "${MARATHON_RECOMP_THIRDPARTY_ROOT}/implot"
    "${MARATHON_RECOMP_THIRDPARTY_ROOT}/json/include"
    "${MARATHON_RECOMP_THIRDPARTY_ROOT}/magic_enum/include"
    "${MARATHON_RECOMP_THIRDPARTY_ROOT}/stb"
    "${MARATHON_RECOMP_THIRDPARTY_ROOT}/unordered_dense/include"
    "${MARATHON_RECOMP_TOOLS_ROOT}/bc_diff"
    "${MARATHON_RECOMP_TOOLS_ROOT}/XenosRecomp/thirdparty/smol-v/source"
)

set_source_files_properties(${MARATHON_RECOMP_THIRDPARTY_SOURCES} PROPERTIES SKIP_PRECOMPILE_HEADERS ON)

set(MARATHON_RECOMP_CXX_SOURCES
    "app.cpp"
    "exports.cpp"
    "main.cpp"
    "misc_impl.cpp"
    "sdl_listener.cpp"
    "stdafx.cpp"
    "version.cpp"
    
    ${MARATHON_RECOMP_KERNEL_CXX_SOURCES}
    ${MARATHON_RECOMP_LOCALE_CXX_SOURCES}
    ${MARATHON_RECOMP_OS_CXX_SOURCES}
    ${MARATHON_RECOMP_CPU_CXX_SOURCES}
    ${MARATHON_RECOMP_GPU_CXX_SOURCES}
    ${MARATHON_RECOMP_APU_CXX_SOURCES}
    ${MARATHON_RECOMP_HID_CXX_SOURCES}
    ${MARATHON_RECOMP_PATCHES_CXX_SOURCES}
    ${MARATHON_RECOMP_UI_CXX_SOURCES}
    ${MARATHON_RECOMP_INSTALL_CXX_SOURCES}
    ${MARATHON_RECOMP_USER_CXX_SOURCES}
    ${MARATHON_RECOMP_UTILS_CXX_SOURCES}
    ${MARATHON_RECOMP_MOD_CXX_SOURCES}
    ${MARATHON_RECOMP_THIRDPARTY_SOURCES}
)

include("version.cmake")

set(VERSION_TXT "${PROJECT_SOURCE_DIR}/res/version.txt")

# Only show Git info and build type if not Release.
set(SHOW_GIT_INFO_AND_BUILD_TYPE 0)
if (NOT ${CMAKE_BUILD_TYPE} MATCHES "Release")
    set(SHOW_GIT_INFO_AND_BUILD_TYPE 1)
endif()

if (MARATHON_RECOMP_METAL)
    set(XCRUN_TOOL "/usr/bin/xcrun")
endif()

GenerateVersionSources(
    OUTPUT_DIR ${PROJECT_SOURCE_DIR}
    VERSION_TXT ${VERSION_TXT}
    H_TEMPLATE "${PROJECT_SOURCE_DIR}/res/version.h.template"
    CXX_TEMPLATE "${PROJECT_SOURCE_DIR}/res/version.cpp.template"
    BUILD_TYPE ${CMAKE_BUILD_TYPE}
    SHOW_GIT_INFO ${SHOW_GIT_INFO_AND_BUILD_TYPE}
    SHOW_BUILD_TYPE ${SHOW_GIT_INFO_AND_BUILD_TYPE}
)

if (WIN32)
    # Create binary version number for Win32 integer attributes.
    CreateVersionString(
        VERSION_TXT ${VERSION_TXT}
        OUTPUT_CSV 1
        OUTPUT_VAR WIN32_VERSION_BINARY
    )

    # Create string version number for Win32 detailed attributes.
    CreateVersionString(
        VERSION_TXT ${VERSION_TXT}
        BUILD_TYPE ${CMAKE_BUILD_TYPE}
        SHOW_GIT_INFO ${SHOW_GIT_INFO_AND_BUILD_TYPE}
        SHOW_BUILD_TYPE ${SHOW_GIT_INFO_AND_BUILD_TYPE}
        OUTPUT_VAR WIN32_VERSION_STRING
    )

    # Set Win32 icon path.
    set(WIN32_ICON_PATH "${PROJECT_SOURCE_DIR}/../MarathonRecompResources/images/game_icon.ico")

    configure_file("res/win32/res.rc.template" "${CMAKE_BINARY_DIR}/res.rc" @ONLY)
    add_executable(MarathonRecomp ${MARATHON_RECOMP_CXX_SOURCES} "${CMAKE_BINARY_DIR}/res.rc")

    # Hide console for release configurations.
    if (${CMAKE_BUILD_TYPE} MATCHES "Release")
        target_link_options(MarathonRecomp PRIVATE "/SUBSYSTEM:WINDOWS" "/ENTRY:mainCRTStartup")
    endif()
elseif (APPLE)
    # Create version number for app bundle.
    CreateVersionString(
            VERSION_TXT ${VERSION_TXT}
            OUTPUT_VAR MACOS_BUNDLE_VERSION
    )

    add_executable(MarathonRecomp MACOSX_BUNDLE
            ${MARATHON_RECOMP_CXX_SOURCES}
            res/macos/game_icon.icns
    )
    set_source_files_properties(res/macos/game_icon.icns PROPERTIES
            MACOSX_PACKAGE_LOCATION Resources)
    set_target_properties(MarathonRecomp PROPERTIES
            OUTPUT_NAME "Marathon Recompiled"
            MACOSX_BUNDLE_INFO_PLIST ${CMAKE_CURRENT_SOURCE_DIR}/res/macos/MacOSXBundleInfo.plist.in
            MACOSX_BUNDLE_GUI_IDENTIFIER sonicnext-dev.MarathonRecomp
            MACOSX_BUNDLE_BUNDLE_NAME "Marathon Recompiled"
            MACOSX_BUNDLE_BUNDLE_VERSION ${MACOS_BUNDLE_VERSION}
            MACOSX_BUNDLE_SHORT_VERSION_STRING ${MACOS_BUNDLE_VERSION}
            MACOSX_BUNDLE_ICON_FILE "game_icon.icns"
    )

    # Linking with MoltenVK directly would prevent using the system Vulkan loader to load with debug layers.
    # Instead, copy the MoltenVK dylib to the app bundle along with an ICD file for the loader to find it.
    # In the event the loader is not installed, the MoltenVK dylib can still be picked up directly in the app bundle.
    set(MVK_BUNDLED_PATH "Resources/vulkan/icd.d")
    set(MVK_DST "${CMAKE_CURRENT_BINARY_DIR}/Marathon Recompiled.app/Contents/${MVK_BUNDLED_PATH}")
    set_property(TARGET MarathonRecomp APPEND PROPERTY BUILD_RPATH "@executable_path/../${MVK_BUNDLED_PATH}")

    set(MVK_ICD_SRC "${MARATHON_RECOMP_THIRDPARTY_ROOT}/MoltenVK/MoltenVK/MoltenVK/icd/MoltenVK_icd.json")
    set(MVK_ICD_DST "${MVK_DST}/MoltenVK_icd.json")
    set(MVK_DYLIB_SRC "${CMAKE_BINARY_DIR}/thirdparty/MoltenVK/libMoltenVK.dylib")
    set(MVK_DYLIB_DST "${MVK_DST}/libMoltenVK.dylib")

    add_custom_command(
            OUTPUT ${MVK_DST}
            COMMAND ${CMAKE_COMMAND} -E make_directory ${MVK_DST})
    add_custom_command(
            OUTPUT ${MVK_ICD_DST}
            DEPENDS ${MVK_ICD_SRC} ${MVK_DST}
            COMMAND ${CMAKE_COMMAND} -E copy ${MVK_ICD_SRC} ${MVK_ICD_DST})
    add_custom_command(
            OUTPUT ${MVK_DYLIB_DST}
            DEPENDS ${MVK_DYLIB_SRC} ${MVK_DST}
            COMMAND ${CMAKE_COMMAND} -E copy ${MVK_DYLIB_SRC} ${MVK_DYLIB_DST})
    add_custom_target(CopyMoltenVK DEPENDS ${MVK_ICD_DST} ${MVK_DYLIB_DST})
    add_dependencies(CopyMoltenVK MoltenVK)
    add_dependencies(MarathonRecomp CopyMoltenVK)
else()
    add_executable(MarathonRecomp ${MARATHON_RECOMP_CXX_SOURCES})
endif()

if (MARATHON_RECOMP_FLATPAK)
    target_compile_definitions(MarathonRecomp PRIVATE 
        "MARATHON_RECOMP_FLATPAK"
        "GAME_INSTALL_DIRECTORY=\"/var/data\""
    )
endif()

if (MARATHON_RECOMP_D3D12)
    find_package(directx-headers CONFIG REQUIRED)
    find_package(directx12-agility CONFIG REQUIRED)
    target_compile_definitions(MarathonRecomp PRIVATE MARATHON_RECOMP_D3D12)
endif()

if (MARATHON_RECOMP_METAL)
    target_compile_definitions(MarathonRecomp PRIVATE MARATHON_RECOMP_METAL)
endif()

if (SDL_VULKAN_ENABLED)
    target_compile_definitions(MarathonRecomp PRIVATE SDL_VULKAN_ENABLED)
endif()

find_package(CURL REQUIRED)

if (MARATHON_RECOMP_D3D12)
    file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/D3D12)
    add_custom_command(TARGET MarathonRecomp POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_PROPERTY:Microsoft::DirectX12-Core,IMPORTED_LOCATION_RELEASE> $<TARGET_FILE_DIR:MarathonRecomp>/D3D12
            COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_PROPERTY:Microsoft::DirectX12-Layers,IMPORTED_LOCATION_DEBUG> $<TARGET_FILE_DIR:MarathonRecomp>/D3D12
            COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_PROPERTY:Microsoft::DirectXShaderCompiler,IMPORTED_LOCATION> $<TARGET_FILE_DIR:MarathonRecomp>
            COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_PROPERTY:Microsoft::DXIL,IMPORTED_LOCATION> $<TARGET_FILE_DIR:MarathonRecomp>
            COMMAND_EXPAND_LISTS
    )

    target_link_libraries(MarathonRecomp PRIVATE
            Microsoft::DirectX-Headers
            Microsoft::DirectX-Guids
            Microsoft::DirectX12-Agility
            Microsoft::DirectXShaderCompiler
            Microsoft::DXIL
            dxgi
    )
endif()

if (WIN32)
    target_link_libraries(MarathonRecomp PRIVATE
        comctl32
        dwmapi
        ntdll
        Shcore
        Synchronization
        winmm
        windowsapp
    )
endif()

target_link_libraries(MarathonRecomp PRIVATE
    fmt::fmt
    libzstd_static
    msdf-atlas-gen::msdf-atlas-gen
    nfd::nfd
    o1heap
    XenonUtils
    SDL2::SDL2-static
    SDL2_mixer
    tomlplusplus::tomlplusplus
    MarathonRecompLib
    xxHash::xxhash
    CURL::libcurl
    plume
)

target_link_libraries(MarathonRecomp PRIVATE ffmpeg)

target_include_directories(MarathonRecomp PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}
    "${CMAKE_CURRENT_SOURCE_DIR}/api"
    ${MARATHON_RECOMP_THIRDPARTY_INCLUDES}
)

if (CMAKE_SYSTEM_NAME MATCHES "Linux")
    find_package(X11 REQUIRED)
    target_include_directories(MarathonRecomp PRIVATE ${X11_INCLUDE_DIR})
    target_link_libraries(MarathonRecomp PRIVATE ${X11_LIBRARIES})
endif()

target_precompile_headers(MarathonRecomp PUBLIC ${MARATHON_RECOMP_PRECOMPILED_HEADERS})

function(compile_shader FILE_PATH TARGET_NAME)
    set(HLSL_FILE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/gpu/shader/hlsl/${FILE_PATH}.hlsl)
    set(MSL_FILE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/gpu/shader/msl/${FILE_PATH}.metal)
    cmake_path(GET HLSL_FILE_PATH STEM HLSL_NAME)
    cmake_path(GET MSL_FILE_PATH STEM MSL_NAME)
    if (MARATHON_RECOMP_METAL)
        add_custom_command(
                OUTPUT ${MSL_FILE_PATH}.ir
                COMMAND ${XCRUN_TOOL} -sdk macosx metal -o ${MSL_FILE_PATH}.ir -c ${MSL_FILE_PATH} -D__air__ -frecord-sources -gline-tables-only
                DEPENDS ${MSL_FILE_PATH}
        )
        add_custom_command(
                OUTPUT ${MSL_FILE_PATH}.metallib
                COMMAND ${XCRUN_TOOL} -sdk macosx metallib -o ${MSL_FILE_PATH}.metallib ${MSL_FILE_PATH}.ir
                DEPENDS ${MSL_FILE_PATH}.ir
        )
        BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${MSL_FILE_PATH}.metallib" DEST_FILE "${MSL_FILE_PATH}.metallib" ARRAY_NAME "g_${MSL_NAME}_air")
    endif()
    if (MARATHON_RECOMP_D3D12)
        add_custom_command(
            OUTPUT ${HLSL_FILE_PATH}.dxil.h
            COMMAND ${DIRECTX_DXC_TOOL} -T ${TARGET_NAME} -HV 2021 -all-resources-bound -Wno-ignored-attributes -E shaderMain -Fh ${HLSL_FILE_PATH}.dxil.h ${HLSL_FILE_PATH} -Vn g_${HLSL_NAME}_dxil
            DEPENDS ${HLSL_FILE_PATH}
        )
        target_sources(MarathonRecomp PRIVATE ${HLSL_FILE_PATH}.dxil.h)
    endif()
    add_custom_command(
        OUTPUT ${HLSL_FILE_PATH}.spirv.h
        COMMAND ${DIRECTX_DXC_TOOL} -T ${TARGET_NAME} -HV 2021 -all-resources-bound -spirv -fvk-use-dx-layout ${ARGN} -E shaderMain -Fh ${HLSL_FILE_PATH}.spirv.h ${HLSL_FILE_PATH} -Vn g_${HLSL_NAME}_spirv
        DEPENDS ${HLSL_FILE_PATH}
    )
    target_sources(MarathonRecomp PRIVATE ${HLSL_FILE_PATH}.spirv.h)
endfunction()

function(compile_vertex_shader FILE_PATH)
    compile_shader(${FILE_PATH} vs_6_0 -fvk-invert-y)
endfunction()

function(compile_pixel_shader FILE_PATH)
    compile_shader(${FILE_PATH} ps_6_0)
endfunction()

compile_pixel_shader(blend_color_alpha_ps)
compile_vertex_shader(copy_vs)
compile_pixel_shader(copy_color_ps)
compile_pixel_shader(copy_depth_ps)
compile_pixel_shader(csd_filter_ps)
compile_vertex_shader(csd_no_tex_vs)
compile_vertex_shader(csd_vs)
compile_pixel_shader(enhanced_motion_blur_ps)
compile_pixel_shader(gaussian_blur_3x3)
compile_pixel_shader(gaussian_blur_5x5)
compile_pixel_shader(gaussian_blur_7x7)
compile_pixel_shader(gaussian_blur_9x9)
compile_pixel_shader(gamma_correction_ps)
compile_pixel_shader(imgui_ps)
compile_vertex_shader(imgui_vs)
compile_pixel_shader(movie_ps)
compile_vertex_shader(movie_vs)
compile_pixel_shader(resolve_msaa_color_2x)
compile_pixel_shader(resolve_msaa_color_4x)
compile_pixel_shader(resolve_msaa_color_8x)
compile_pixel_shader(resolve_msaa_depth_2x)
compile_pixel_shader(resolve_msaa_depth_4x)
compile_pixel_shader(resolve_msaa_depth_8x)

function(generate_aggregate_header INPUT_DIRECTORY OUTPUT_FILE)
    get_filename_component(ABS_OUTPUT_FILE "${OUTPUT_FILE}" ABSOLUTE)
    file(GLOB_RECURSE HEADER_FILES "${INPUT_DIRECTORY}/*.h")
    set(HEADER_CONTENT "#pragma once\n\n")

    foreach(HEADER_FILE IN LISTS HEADER_FILES)
        get_filename_component(ABS_HEADER_FILE "${HEADER_FILE}" ABSOLUTE)
        if (ABS_HEADER_FILE STREQUAL ABS_OUTPUT_FILE)
            continue()
        endif()
        file(RELATIVE_PATH RELATIVE_HEADER_FILE "${INPUT_DIRECTORY}" "${HEADER_FILE}")
        string(APPEND HEADER_CONTENT "#include \"${RELATIVE_HEADER_FILE}\"\n")
    endforeach()

    if (EXISTS "${OUTPUT_FILE}")
        file(READ "${OUTPUT_FILE}" EXISTING_CONTENT)
        if (EXISTING_CONTENT STREQUAL HEADER_CONTENT)
            return()
        endif()
    endif()

    file(WRITE "${OUTPUT_FILE}" "${HEADER_CONTENT}")
endfunction()

generate_aggregate_header(
    "${CMAKE_CURRENT_SOURCE_DIR}/api"
    "${CMAKE_CURRENT_SOURCE_DIR}/api/Marathon.h"
)

set(RESOURCES_SOURCE_PATH "${PROJECT_SOURCE_DIR}/../MarathonRecompResources")
set(RESOURCES_OUTPUT_PATH "${PROJECT_SOURCE_DIR}/res")

## Miscellaneous ##
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/bc_diff/button_bc_diff.bin" DEST_FILE "${RESOURCES_OUTPUT_PATH}/bc_diff/button_bc_diff.bin" ARRAY_NAME "g_button_bc_diff" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/font/im_font_atlas.bin" DEST_FILE "${RESOURCES_OUTPUT_PATH}/font/im_font_atlas.bin" ARRAY_NAME "g_im_font_atlas" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/font/im_font_atlas.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/font/im_font_atlas.dds" ARRAY_NAME "g_im_font_atlas_texture" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/achievements_menu/trophy.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/achievements_menu/trophy.dds" ARRAY_NAME "g_trophy" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/common/controller.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/common/controller.dds" ARRAY_NAME "g_controller" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/common/general_window.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/common/general_window.dds" ARRAY_NAME "g_general_window" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/common/hedge-dev.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/common/hedge-dev.dds" ARRAY_NAME "g_hedgedev" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/common/kbm.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/common/kbm.dds" ARRAY_NAME "g_kbm" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/common/select.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/common/select.dds" ARRAY_NAME "g_select" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/common/light.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/common/light.dds" ARRAY_NAME "g_light" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/common/main_menu1.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/common/main_menu1.dds" ARRAY_NAME "g_main_menu1" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/common/arrow.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/common/arrow.dds" ARRAY_NAME "arrow" COMPRESSION_TYPE "zstd")

## Installer ##
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/installer/install_001.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/installer/install_001.dds" ARRAY_NAME "g_install_001" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/installer/install_002.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/installer/install_002.dds" ARRAY_NAME "g_install_002" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/installer/install_003.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/installer/install_003.dds" ARRAY_NAME "g_install_003" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/installer/install_004.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/installer/install_004.dds" ARRAY_NAME "g_install_004" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/installer/install_005.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/installer/install_005.dds" ARRAY_NAME "g_install_005" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/installer/install_006.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/installer/install_006.dds" ARRAY_NAME "g_install_006" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/installer/install_007.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/installer/install_007.dds" ARRAY_NAME "g_install_007" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/installer/install_008.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/installer/install_008.dds" ARRAY_NAME "g_install_008" COMPRESSION_TYPE "zstd")

## Options Menu ##
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/achievement_notifications.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/achievement_notifications.dds" ARRAY_NAME "g_achievement_notifications" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/allow_background_input_ps.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/allow_background_input_ps.dds" ARRAY_NAME "g_allow_background_input_ps" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/allow_background_input_xb.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/allow_background_input_xb.dds" ARRAY_NAME "g_allow_background_input_xb" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/antialiasing_none.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/antialiasing_none.dds" ARRAY_NAME "g_antialiasing_none" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/antialiasing_2x.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/antialiasing_2x.dds" ARRAY_NAME "g_antialiasing_2x" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/antialiasing_4x.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/antialiasing_4x.dds" ARRAY_NAME "g_antialiasing_4x" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/antialiasing_8x.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/antialiasing_8x.dds" ARRAY_NAME "g_antialiasing_8x" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/aspect_ratio.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/aspect_ratio.dds" ARRAY_NAME "g_aspect_ratio" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/battle_theme.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/battle_theme.dds" ARRAY_NAME "g_battle_theme" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/brightness.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/brightness.dds" ARRAY_NAME "g_brightness" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/channel_stereo.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/channel_stereo.dds" ARRAY_NAME "g_channel_stereo" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/channel_surround.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/channel_surround.dds" ARRAY_NAME "g_channel_surround" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/control_tutorial_ps.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/control_tutorial_ps.dds" ARRAY_NAME "g_control_tutorial_ps" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/control_tutorial_xb.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/control_tutorial_xb.dds" ARRAY_NAME "g_control_tutorial_xb" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/controller_icons.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/controller_icons.dds" ARRAY_NAME "g_controller_icons" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/default.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/default.dds" ARRAY_NAME "g_default" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/effects_volume.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/effects_volume.dds" ARRAY_NAME "g_effects_volume" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/fps.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/fps.dds" ARRAY_NAME "g_fps" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/fullscreen.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/fullscreen.dds" ARRAY_NAME "g_fullscreen" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/gi_texture_filtering_bilinear.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/gi_texture_filtering_bilinear.dds" ARRAY_NAME "g_gi_texture_filtering_bilinear" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/gi_texture_filtering_bicubic.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/gi_texture_filtering_bicubic.dds" ARRAY_NAME "g_gi_texture_filtering_bicubic" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/hints.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/hints.dds" ARRAY_NAME "g_hints" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/horizontal_camera.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/horizontal_camera.dds" ARRAY_NAME "g_horizontal_camera" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/language.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/language.dds" ARRAY_NAME "g_language" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/master_volume.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/master_volume.dds" ARRAY_NAME "g_master_volume" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/monitor.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/monitor.dds" ARRAY_NAME "g_monitor" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/motion_blur_off.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/motion_blur_off.dds" ARRAY_NAME "g_motion_blur_off" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/motion_blur_original.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/motion_blur_original.dds" ARRAY_NAME "g_motion_blur_original" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/motion_blur_enhanced.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/motion_blur_enhanced.dds" ARRAY_NAME "g_motion_blur_enhanced" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/movie_scale_fit.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/movie_scale_fit.dds" ARRAY_NAME "g_movie_scale_fit" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/movie_scale_fill.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/movie_scale_fill.dds" ARRAY_NAME "g_movie_scale_fill" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/music_attenuation.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/music_attenuation.dds" ARRAY_NAME "g_music_attenuation" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/music_volume.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/music_volume.dds" ARRAY_NAME "g_music_volume" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/shadow_resolution_x512.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/shadow_resolution_x512.dds" ARRAY_NAME "g_shadow_resolution_x512" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/shadow_resolution_x1024.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/shadow_resolution_x1024.dds" ARRAY_NAME "g_shadow_resolution_x1024" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/shadow_resolution_x2048.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/shadow_resolution_x2048.dds" ARRAY_NAME "g_shadow_resolution_x2048" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/shadow_resolution_x4096.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/shadow_resolution_x4096.dds" ARRAY_NAME "g_shadow_resolution_x4096" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/shadow_resolution_x8192.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/shadow_resolution_x8192.dds" ARRAY_NAME "g_shadow_resolution_x8192" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/time_transition_ps.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/time_transition_ps.dds" ARRAY_NAME "g_time_of_day_transition_playstation" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/time_transition_xb.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/time_transition_xb.dds" ARRAY_NAME "g_time_of_day_transition_xbox" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/transparency_antialiasing_false.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/transparency_antialiasing_false.dds" ARRAY_NAME "g_transparency_antialiasing_false" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/transparency_antialiasing_true.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/transparency_antialiasing_true.dds" ARRAY_NAME "g_transparency_antialiasing_true" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/ui_alignment_centre.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/ui_alignment_centre.dds" ARRAY_NAME "g_ui_alignment_centre" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/ui_alignment_edge.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/ui_alignment_edge.dds" ARRAY_NAME "g_ui_alignment_edge" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/vertical_camera.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/vertical_camera.dds" ARRAY_NAME "g_vertical_camera" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/voice_language.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/voice_language.dds" ARRAY_NAME "g_voice_language" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/vibration_ps.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/vibration_ps.dds" ARRAY_NAME "g_vibration_ps" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/vibration_xb.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/vibration_xb.dds" ARRAY_NAME "g_vibration_xb" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/vsync_on.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/vsync_on.dds" ARRAY_NAME "g_vsync_on" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/vsync_off.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/vsync_off.dds" ARRAY_NAME "g_vsync_off" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/window_size.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/window_size.dds" ARRAY_NAME "g_window_size" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/thumbnails/xbox_color_correction.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/thumbnails/xbox_color_correction.dds" ARRAY_NAME "g_xbox_color_correction" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/miles_electric.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/miles_electric.dds" ARRAY_NAME "g_miles_electric" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/options_static.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/options_static.dds" ARRAY_NAME "g_options_static" COMPRESSION_TYPE "zstd")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/options_menu/options_static_flash.dds" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/options_menu/options_static_flash.dds" ARRAY_NAME "g_options_static_flash" COMPRESSION_TYPE "zstd")

## Game Icon ##
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/images/game_icon.bmp" DEST_FILE "${RESOURCES_OUTPUT_PATH}/images/game_icon.bmp" ARRAY_NAME "g_game_icon")

## Audio ##
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/music/installer.ogg" DEST_FILE "${RESOURCES_OUTPUT_PATH}/music/installer.ogg" ARRAY_NAME "g_installer_music")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/sounds/window_open.ogg" DEST_FILE "${RESOURCES_OUTPUT_PATH}/sounds/window_open.ogg" ARRAY_NAME "g_window_open")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/sounds/window_close.ogg" DEST_FILE "${RESOURCES_OUTPUT_PATH}/sounds/window_close.ogg" ARRAY_NAME "g_window_close")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/sounds/cursor2.ogg" DEST_FILE "${RESOURCES_OUTPUT_PATH}/sounds/cursor2.ogg" ARRAY_NAME "g_cursor2")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/sounds/deside.ogg" DEST_FILE "${RESOURCES_OUTPUT_PATH}/sounds/deside.ogg" ARRAY_NAME "g_deside")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/sounds/move.ogg" DEST_FILE "${RESOURCES_OUTPUT_PATH}/sounds/move.ogg" ARRAY_NAME "g_move")
BIN2C(TARGET_OBJ MarathonRecomp SOURCE_FILE "${RESOURCES_SOURCE_PATH}/sounds/main_deside.ogg" DEST_FILE "${RESOURCES_OUTPUT_PATH}/sounds/main_deside.ogg" ARRAY_NAME "g_main_deside")
