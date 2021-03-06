# Copyright (C) 2018 Istituto Italiano di Tecnologia (IIT). All rights reserved.
# This software may be modified and distributed under the terms of the
# GNU Lesser General Public License v2.1 or any later version.

# ===================
# CORE SHARED LIBRARY
# ===================

set(TOOLBOX_CORE_SRC
    src/Block.cpp
    src/BlockInformation.cpp
    src/Log.cpp
    src/Parameter.cpp
    src/Parameters.cpp
    src/ConvertStdVector.cpp
    src/Signal.cpp)

set(TOOLBOX_CORE_HDR
    include/Core/Block.h
    include/Core/BlockInformation.h
    include/Core/Log.h
    include/Core/Parameter.h
    include/Core/Parameters.h
    include/Core/ConvertStdVector.h
    include/Core/Signal.h)

add_library(Core ${TOOLBOX_CORE_SRC} ${TOOLBOX_CORE_HDR})
add_library(WBToolbox::Core ALIAS Core)

target_include_directories(Core PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/WBToolbox>)

set_target_properties(Core PROPERTIES
    # VERSION ${PROJECT_VERSION}
    PUBLIC_HEADER "${TOOLBOX_CORE_HDR}"
    OUTPUT_NAME "ToolboxCore")

install(
    TARGETS Core
    EXPORT ToolboxCoreExport
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/WBToolbox/Core)

install_basic_package_files(ToolboxCore
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion
    EXPORT ToolboxCoreExport
    FIRST_TARGET Core
    NAMESPACE WBToolbox::
    NO_CHECK_REQUIRED_COMPONENTS_MACRO)

# =====================
# CORE SIMULINK LIBRARY
# =====================

if(WBT_USES_MATLAB)

    # Install TLC file
    install(
        FILES ${CMAKE_CURRENT_SOURCE_DIR}/WBToolbox.tlc
        DESTINATION ${CMAKE_INSTALL_PREFIX}/mex)

    # Compile S-Function
    matlab_add_mex(
        NAME Mex
        OUTPUT_NAME WBToolbox
        SRC src/SimulinkBlockInformation.cpp
            include/Core/SimulinkBlockInformation.h
            src/WBToolbox.cpp
        LINK_TO WBToolbox::Core MxAnyType shlibpp::shlibpp)
    add_library(WBToolbox::Mex ALIAS Mex)

    target_compile_definitions(Mex PUBLIC "MATLAB_MEX_FILE")

    target_include_directories(Mex PUBLIC
        ${Matlab_INCLUDE_DIRS}
        ${Matlab_ROOT_DIR}/simulink/include)

    # Install S-Function
    install(
        TARGETS Mex
        EXPORT ToolboxMexExport
        DESTINATION mex)

    install_basic_package_files(ToolboxMex
        VERSION ${PROJECT_VERSION}
        COMPATIBILITY AnyNewerVersion
        EXPORT ToolboxMexExport
        FIRST_TARGET Mex
        DEPENDENCIES ToolboxCore MxAnyType shlibpp
        NAMESPACE WBToolbox::
        NO_CHECK_REQUIRED_COMPONENTS_MACRO)
endif()

# ==================
# CORE CODER LIBRARY
# ==================

set(CODER_HDR
    include/Core/CoderBlockInformation.h
    include/Core/GeneratedCodeWrapper.h)

set(CODER_SRC
    src/CoderBlockInformation.cpp)

add_library(Coder ${CODER_HDR} ${CODER_SRC})
add_library(WBToolbox::Coder ALIAS Coder)

set_target_properties(Coder PROPERTIES
    # VERSION ${PROJECT_VERSION}
    PUBLIC_HEADER "${CODER_HDR}"
    OUTPUT_NAME "ToolboxCoder"
)

target_link_libraries(Coder PUBLIC WBToolbox::Core)
target_include_directories(Coder INTERFACE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/WBToolbox>)

# CMake macro and functions
install(
    FILES "${CMAKE_SOURCE_DIR}/cmake/AddGeneratedCodeLib.cmake"
    DESTINATION "${CMAKE_INSTALL_DATADIR}/WBToolbox/cmake"
)

install(
    TARGETS Coder
    EXPORT ToolboxCoderExport
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/WBToolbox/Core
)

set(EXTRA_CONTENT
    "list(APPEND CMAKE_MODULE_PATH @CMAKE_INSTALL_PREFIX@/share/WBToolbox/cmake)")

install_basic_package_files(ToolboxCoder
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion
    EXPORT ToolboxCoderExport
    FIRST_TARGET Coder
    DEPENDENCIES ToolboxCore
    NAMESPACE WBToolbox::
    NO_CHECK_REQUIRED_COMPONENTS_MACRO
    INCLUDE_CONTENT ${EXTRA_CONTENT})
