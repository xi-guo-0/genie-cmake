cmake_minimum_required(VERSION 3.20)
project(Genie)

if(NOT DEFINED QNN_SDK_ROOT)
  message(FATAL_ERROR "QNN_SDK_ROOT not set")
elseif(NOT EXISTS ${QNN_SDK_ROOT})
  message(FATAL_ERROR "QNN_SDK_ROOT path '${QNN_SDK_ROOT}' does not exist")
endif()

add_library(tokenizers_capi STATIC IMPORTED)
set_target_properties(tokenizers_capi PROPERTIES IMPORTED_LOCATION
                                                 ${tokenizers_capi_PATH})

set(Genie_SRC_DIR ${QNN_SDK_ROOT}/examples/Genie/Genie/src)
set(qualla_DIR ${QNN_SDK_ROOT}/examples/Genie/Genie/src/qualla)

file(
  GLOB
  Genie_SRC
  ${Genie_SRC_DIR}/*.cpp
  ${qualla_DIR}/*.cpp
  ${qualla_DIR}/dialogs/*.cpp
  ${qualla_DIR}/engines/lib.cpp
  ${qualla_DIR}/engines/qnn-htp.cpp
  ${qualla_DIR}/loggers/*.cpp
  ${qualla_DIR}/samplers/*.cpp
  ${qualla_DIR}/tokenizers/*.cpp
  ${qualla_DIR}/utils/*.cpp
  ${qualla_DIR}/engines/qnn-htp/*.cpp
  ${qualla_DIR}/engines/qnn-api/*.cpp)

add_library(${PROJECT_NAME} SHARED ${Genie_SRC})
target_compile_features(${PROJECT_NAME} PRIVATE cxx_std_20)
target_include_directories(
  ${PROJECT_NAME}
  PUBLIC ${QNN_SDK_ROOT}/include/Genie
         ${QNN_SDK_ROOT}/include/QNN
         ${QNN_SDK_ROOT}/include/QNN/HTP
         ${Genie_SRC_DIR}
         ${qualla_DIR}/include
         ${qualla_DIR}/engines/qnn-htp
         ${qualla_DIR}/engines/qnn-api
         ${qualla_DIR}/tokenizers)
target_compile_definitions(
  ${PROJECT_NAME}
  PRIVATE GENIE_E2T_FEATURE
          GENIE_LADE_FEATURE
          GENIE_LORA_FEATURE
          GENIE_MULTISTREAM_FEATURE
          GENIE_SPD_FEATURE
          GENIE_SSD_FEATURE
          SPILLFILL
          FMT_HEADER_ONLY)
target_link_libraries(${PROJECT_NAME} PRIVATE tokenizers_capi)

add_executable(genie-t2t-run
               ${QNN_SDK_ROOT}/examples/Genie/genie-t2t-run/main.cpp)
target_link_libraries(genie-t2t-run PUBLIC ${PROJECT_NAME})
