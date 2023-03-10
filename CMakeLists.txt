cmake_minimum_required(VERSION 3.24)

set(CMAKE_TOOLCHAIN_FILE "${CMAKE_CURRENT_SOURCE_DIR}/vcpkg/scripts/buildsystems/vcpkg.cmake" CACHE STRING "Vcpkg toolchain file")

project(hello_aciton_cpp)

set(CMAKE_CXX_STANDARD 17)

if(MSVC)
    # Without a byte order mark (BOM), Visual Studio assumes that the source file
    # is encoded using the current user code page, so we specify UTF-8.
    add_compile_options(/utf-8)
    add_compile_definitions(NOMINMAX)
endif(MSVC)

add_library(common_lib INTERFACE)

find_package(Protobuf REQUIRED)
PROTOBUF_GENERATE_CPP(PROTO_SRCS PROTO_HDRS test.proto)
message("PROTO_SRCS = ${PROTO_SRCS}")
message("PROTO_HDRS = ${PROTO_HDRS}")
add_library(protos STATIC ${PROTO_SRCS} ${PROTO_HDRS})
target_include_directories(protos PUBLIC ${CMAKE_CURRENT_BINARY_DIR})
target_link_libraries(protos PUBLIC protobuf::libprotobuf)

find_package(Eigen3 REQUIRED)
find_package(argparse REQUIRED)
find_package(fmt REQUIRED)
find_package(spdlog REQUIRED)
find_package(nlohmann_json REQUIRED)
find_package(asio REQUIRED)
find_package(PahoMqttCpp REQUIRED)

target_link_libraries(common_lib INTERFACE protos Eigen3::Eigen argparse::argparse fmt::fmt spdlog::spdlog nlohmann_json::nlohmann_json asio::asio)

if(WIN32)
    target_link_libraries(common_lib INTERFACE PahoMqttCpp::paho-mqttpp3)
else()
    target_link_libraries(common_lib INTERFACE PahoMqttCpp::paho-mqttpp3-static)
endif()

add_executable(main main.cpp)
target_link_libraries(main PRIVATE common_lib)

add_executable(test_main test_main.cpp)
find_package(Catch2 REQUIRED)
target_link_libraries(test_main PRIVATE common_lib Catch2::Catch2WithMain)

include(CTest)
enable_testing()
add_test(NAME test_main COMMAND test_main)
