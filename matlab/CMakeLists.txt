# Copyright (C) 2018 Istituto Italiano di Tecnologia (IIT). All rights reserved.
# This software may be modified and distributed under the terms of the
# GNU Lesser General Public License v2.1 or any later version.

# =========================================
# MATLAB CLASSES AND SIMULINK CONFIGURATION
# =========================================

# The logic about library development is the following:
#
# * Developers should edit the library (e.g. changing block's masks) stored in the
#   matlab/library/WBToolboxLibrary_repository.mdl file.
# * Developers should export the changes using the export_library custom target.
# * Users will use the files installed from the exported/ folder.


# Custom script to generate the library to be committed on the repository.
# This target is excluded from the normal build and must be called explicitly by the
# developer who modifies the library.
add_custom_target(export_libraries
    ${Matlab_MAIN_PROGRAM} -nosplash -nodesktop -r export_library
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})
set_target_properties(export_libraries PROPERTIES EXCLUDE_FROM_DEFAULT_BUILD 1)

set(WBTOOLBOX_SHARE_DIR "${CMAKE_INSTALL_DATADIR}/WBToolbox")

# The following line is to properly configure the installation script of the toolbox
configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/startup_wbitoolbox.m.in
    ${CMAKE_BINARY_DIR}/startup_wbitoolbox.m)

# Install configuration files
install(
    FILES ${CMAKE_BINARY_DIR}/startup_wbitoolbox.m
    DESTINATION ${WBTOOLBOX_SHARE_DIR})
install(
    FILES ${CMAKE_CURRENT_SOURCE_DIR}/slblocks.m
    DESTINATION ${WBTOOLBOX_SHARE_DIR})

# Install the Matlab package folder
install(
    DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/+WBToolbox
    DESTINATION ${WBTOOLBOX_SHARE_DIR})

# Install images for the Simulink Library
install(
    DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/library/images
    DESTINATION ${WBTOOLBOX_SHARE_DIR})

# Install script for enabling debugging capabilities
# (update the mex and avoid restarting matlab)
configure_file(
    setupForMatlabDebug.m.in
    ${CMAKE_BINARY_DIR}/toolbox/setupForMatlabDebug.m
    @ONLY)

#if MAJOR >= 2014 && MINOR >= b
# Note: We had issues with Matlab 2014b and .mdl models.
# But this issue seems to have been disappeared in 2015b. We have to check if we need to enable this if again
# TODO: check if the mdl support is still required
# if (${Matlab_VERSION_STRING} GREATER "8.3")
# install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/WBToolboxLibrary.slx DESTINATION ${WB-TOOLBOX_SHARE_DIR})
# else()
# install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/WBToolboxLibrary.mdl DESTINATION ${WB-TOOLBOX_SHARE_DIR})
# endif()
install(
    FILES ${CMAKE_CURRENT_SOURCE_DIR}/library/exported/WBToolboxLibrary.slx
    DESTINATION ${WBTOOLBOX_SHARE_DIR})
