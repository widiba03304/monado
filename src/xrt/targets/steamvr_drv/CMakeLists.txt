# Copyright 2020-2021, Collabora, Ltd.
# SPDX-License-Identifier: BSL-1.0

get_property(AUX_BINDINGS_DIR GLOBAL PROPERTY AUX_BINDINGS_DIR_PROP)
set(INPUT_PROFILES_INPUT_DIR ${AUX_BINDINGS_DIR})
set(INPUT_PROFILES_OUTPUT_DIR "${PROJECT_BINARY_DIR}/steamvr-monado/resources/input/")

add_custom_command(
	OUTPUT "${INPUT_PROFILES_OUTPUT_DIR}"
	COMMAND
		${PYTHON_EXECUTABLE} ${INPUT_PROFILES_INPUT_DIR}/steamvr_profiles.py
		${INPUT_PROFILES_INPUT_DIR}/bindings.json "${INPUT_PROFILES_OUTPUT_DIR}" "-s"
	DEPENDS ${INPUT_PROFILES_INPUT_DIR}/bindings.py ${INPUT_PROFILES_INPUT_DIR}/bindings.json
	COMMENT "Generating SteamVR input profiles to ${INPUT_PROFILES_OUTPUT_DIR}"
	)
add_custom_target(
	steamvr_generated_input_profiles
	DEPENDS "${INPUT_PROFILES_OUTPUT_DIR}"
	COMMENT "Generating SteamVR input profiles"
	)

add_library(driver_monado MODULE main.c)
add_dependencies(driver_monado steamvr_generated_input_profiles)

target_link_libraries(
	driver_monado
	PRIVATE
		xrt-external-openvr
		aux_util
		st_ovrd
		st_prober
		target_lists
		target_instance_no_comp
	)

# meta data that the steamvr plugin needs in the base directory of the steamvr plugin
file(COPY driver.vrdrivermanifest DESTINATION ${PROJECT_BINARY_DIR}/steamvr-monado)
file(COPY resources DESTINATION ${PROJECT_BINARY_DIR}/steamvr-monado)

# determine the output directory for the steamvr plugin
if(WIN32)
	# FIXME need to account for different architectures
	if("${CMAKE_SIZEOF_VOID_P}" EQUAL "8")
		set(PLUGINDIR "${PROJECT_BINARY_DIR}/steamvr-monado/bin/win64")
	else()
		set(PLUGINDIR "${PROJECT_BINARY_DIR}/steamvr-monado/bin/win32")
	endif()
elseif(APPLE)
	if("${CMAKE_SIZEOF_VOID_P}" EQUAL "8")
		set(PLUGINDIR "${PROJECT_BINARY_DIR}/steamvr-monado/bin/osx64")
	else()
		set(PLUGINDIR "${PROJECT_BINARY_DIR}/steamvr-monado/bin/osx32")
	endif()
elseif(NOT ANDROID)
	if("${CMAKE_SIZEOF_VOID_P}" EQUAL "8")
		set(PLUGINDIR "${PROJECT_BINARY_DIR}/steamvr-monado/bin/linux64")
	else()
		set(PLUGINDIR "${PROJECT_BINARY_DIR}/steamvr-monado/bin/linux32")
	endif()
endif()

# message("SteamVR plugin path: ${PLUGINDIR}")
set_target_properties(driver_monado PROPERTIES LIBRARY_OUTPUT_DIRECTORY "${PLUGINDIR}")

# don't add lib prefix to driver_monado.so
set_target_properties(driver_monado PROPERTIES PREFIX "")

install(
	DIRECTORY "${PROJECT_BINARY_DIR}/steamvr-monado"
	DESTINATION "${CMAKE_INSTALL_PREFIX}/share"
	)
