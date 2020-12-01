cmake_minimum_required(VERSION 3.10)
project(aws-iot-device-client)

list(APPEND CMAKE_MODULE_PATH "./sdk-cpp-workspace/lib/cmake")

file(GLOB DC_SRC "source/*.cpp" /
        "source/*.c" /
        "source/external/cJSON/*.c" /
        "source/**/*.cpp" /
        "source/**/*.c")


set(DC_PROJECT_NAME aws-iot-device-client)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_C_FLAGS_DEBUGOPT "")
set( CMAKE_BUILD_TYPE Debug) # Switch to Release for the "Release" build
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -pthread")

#########################################
# OpenSSL dependency                    #
#########################################
find_package(OpenSSL REQUIRED)

#########################################
# AWS IoT v2 SDK C++ dependency         #
#########################################

include_directories(/include)
link_directories(/lib)

### These next two lines are just a test to verify the Jobs SDK lib is available for sanity check
find_library(jobs_lib IotJobs-cpp)
message("-- *STATUS* Found jobs_lib: ${jobs_lib}")


### Now build the DC executable
add_executable(${DC_PROJECT_NAME} ${DC_SRC})
set_target_properties(${DC_PROJECT_NAME} PROPERTIES LINKER_LANGUAGE CXX)
target_compile_definitions(${DC_PROJECT_NAME} PRIVATE "-DDEBUG_BUILD")

if (MSVC)
    target_compile_options(${DC_PROJECT_NAME} PRIVATE /W4 /WX)
else ()
    target_compile_options(${DC_PROJECT_NAME} PRIVATE -Wall -Wno-long-long -pedantic -Werror)
endif ()

target_link_libraries(${DC_PROJECT_NAME} IotJobs-cpp IotDeviceCommon-cpp IotDeviceDefender-cpp IotSecureTunneling-cpp aws-crt-cpp aws-c-mqtt aws-c-auth aws-c-io s2n aws-c-cal aws-c-http aws-c-compression aws-c-common aws-c-iot)
target_link_libraries(${DC_PROJECT_NAME} OpenSSL::SSL)
target_link_libraries(${DC_PROJECT_NAME} OpenSSL::Crypto)

add_subdirectory(test)