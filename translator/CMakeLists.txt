set(SRC
    reader/prim_reader.cpp
    reader/read_arnold_type.cpp
    reader/read_camera.cpp
    reader/read_geometry.cpp
    reader/read_light.cpp
    reader/read_shader.cpp
    reader/read_options.cpp
    reader/reader.cpp
    reader/registry.cpp
    reader/utils.cpp

    writer/prim_writer.cpp
    writer/registry.cpp
    writer/write_arnold_type.cpp
    writer/write_camera.cpp
    writer/write_geometry.cpp
    writer/write_light.cpp
    writer/write_shader.cpp
    writer/writer.cpp)

set(HDR
    reader/prim_reader.h
    reader/read_arnold_type.h
    reader/read_camera.h
    reader/read_geometry.h
    reader/read_light.h
    reader/read_shader.h
    reader/read_options.h
    reader/reader.h
    reader/registry.h
    reader/utils.h

    utils/utils.h

    writer/prim_writer.h
    writer/registry.h
    writer/write_arnold_type.h
    writer/write_camera.h
    writer/write_geometry.h
    writer/write_light.h
    writer/write_shader.h
    writer/writer.h)

add_library(translator STATIC ${COMMON_SRC} ${SRC})
if (BUILD_HEADERS_AS_SOURCES)
    target_sources(translator PRIVATE ${COMMON_HDR} ${HDR})
endif ()
target_include_directories(translator SYSTEM PUBLIC "${USD_INCLUDE_DIR}")
target_include_directories(translator SYSTEM PUBLIC "${ARNOLD_INCLUDE_DIR}")
target_include_directories(translator SYSTEM PUBLIC "${Boost_INCLUDE_DIRS}")
target_include_directories(translator SYSTEM PUBLIC "${TBB_INCLUDE_DIRS}")
if (BUILD_USE_PYTHON3)
    target_include_directories(translator SYSTEM PUBLIC "${Python3_INCLUDE_DIRS}")
else ()
    target_include_directories(translator SYSTEM PUBLIC "${Python2_INCLUDE_DIRS}")
endif ()
target_include_directories(translator PUBLIC "${CMAKE_CURRENT_BINARY_DIR}")

target_link_libraries(translator INTERFACE "${ARNOLD_LIBRARY}")
if (USD_HAS_PYTHON)
    target_link_libraries(translator INTERFACE "${Boost_LIBRARIES}")
    if (BUILD_USE_PYTHON3)
        target_link_libraries(translator INTERFACE Python3::Python)
    else ()
        target_link_libraries(translator INTERFACE Python2::Python)
    endif ()
endif ()

if (USD_STATIC_BUILD)
    if (WIN32)
        target_link_options(translator INTERFACE "-WHOLEARCHIVE:${USD_usd_m_LIBRARY}")
        target_link_libraries(translator INTERFACE Ws2_32 Dbghelp Shlwapi advapi32)
        target_link_libraries(translator INTERFACE "${TBB_LIBRARIES}")
    else ()
        set(_staticlibs "${USD_usd_m_LIBRARY}")
        if (TBB_STATIC_BUILD)
            set(_staticlibs "${_staticlibs},${TBB_LIBRARIES}")
        else ()
            target_link_libraries(translator INTERFACE "${TBB_LIBRARIES}")
        endif ()
        if (LINUX)
            target_link_libraries(translator INTERFACE dl pthread)
            target_link_options(translator INTERFACE "-Wl,--whole-archive,${_staticlibs},--no-whole-archive")
        else () # APPLE
            target_link_options(translator INTERFACE "-Wl,-all_load,${_staticlibs},-noall_load")
        endif ()
    endif ()
else ()
    target_link_libraries(translator INTERFACE "${TBB_LIBRARIES}")
    if (USD_MONOLITHIC_BUILD)
        target_link_libraries(translator INTERFACE usd_ms)
    else ()
        if (LINUX)
            target_link_libraries(translator INTERFACE dl pthread)
        endif ()
        target_link_libraries(translator INTERFACE gf sdf tf usd ar vt work)
        target_link_libraries(translator INTERFACE usdGeom usdShade usdUtils usdLux usdVol usdSkel usdRender)
    endif ()
endif ()

if (NOT WIN32)
    set_target_properties(translator PROPERTIES POSITION_INDEPENDENT_CODE ON)
endif ()
