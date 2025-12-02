# The following lines of boilerplate have to be in your project's
# CMakeLists in this exact order for cmake to work correctly
cmake_minimum_required(VERSION 3.5)

# Add the root of this git repo to the component search path.
set(EXTRA_COMPONENT_DIRS "../")

set(_idf_version_major 4)
if(DEFINED ENV{IDF_PATH})
    set(_idf_version_file "$ENV{IDF_PATH}/version.txt")
    if(EXISTS "${_idf_version_file}")
        file(STRINGS "${_idf_version_file}" _idf_version_line LIMIT_COUNT 1)
        string(REGEX REPLACE "^v([0-9]+)\\..*" "\\1" _idf_version_guess "${_idf_version_line}")
        if(_idf_version_guess MATCHES "^[0-9]+$")
            set(_idf_version_major ${_idf_version_guess})
        endif()
    endif()
endif()

set(_littlefs_partition_subtype "littlefs")
if(_idf_version_major LESS 5)
    set(_littlefs_partition_subtype "spiffs")
endif()

set(_littlefs_partition_template "${CMAKE_CURRENT_LIST_DIR}/partitions_demo_esp_littlefs.csv.in")
set(_littlefs_partition_output "${CMAKE_BINARY_DIR}/partitions_demo_esp_littlefs.csv")
set(LFS_PARTITION_SUBTYPE "${_littlefs_partition_subtype}")
configure_file("${_littlefs_partition_template}" "${_littlefs_partition_output}" @ONLY)

set(_sdkconfig_partition_defaults "${CMAKE_BINARY_DIR}/sdkconfig.partitions.defaults")
file(WRITE "${_sdkconfig_partition_defaults}"
"CONFIG_PARTITION_TABLE_CUSTOM=y
CONFIG_PARTITION_TABLE_CUSTOM_FILENAME=\"${_littlefs_partition_output}\"
")

set(SDKCONFIG_DEFAULTS "${CMAKE_CURRENT_LIST_DIR}/sdkconfig.defaults;${_sdkconfig_partition_defaults}")

include($ENV{IDF_PATH}/tools/cmake/project.cmake)
project(demo_esp_littlefs)
