cmake_minimum_required(VERSION 3.21)

project(
  xTools
  VERSION 1.0
  LANGUAGES C CXX)

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

set(WITH_GFLAGS OFF)
set(BUILD_TESTING OFF)
set(BUILD_SHARED_LIBS
    OFF
    CACHE BOOL "Reset option" FORCE)
set(WITH_TOOLS
    OFF
    CACHE BOOL "Reset option" FORCE)

# --------------------------------------------------------------------------------------------------
# Qt module
list(APPEND X_TOOLS_QT_COMPONENTS Gui)
list(APPEND X_TOOLS_QT_COMPONENTS Svg)
list(APPEND X_TOOLS_QT_COMPONENTS Core)
list(APPEND X_TOOLS_QT_COMPONENTS Widgets)
list(APPEND X_TOOLS_QT_COMPONENTS Network)
list(APPEND X_TOOLS_QT_COMPONENTS WebSockets)
list(APPEND X_TOOLS_QT_COMPONENTS LinguistTools)

# Qt6 is preferred, but Qt5 is also supported
find_package(QT NAMES Qt6 QUIET COMPONENTS ${X_TOOLS_QT_COMPONENTS})
if(NOT QT_FOUND)
  find_package(QT NAMES Qt5 REQUIRED COMPONENTS ${X_TOOLS_QT_COMPONENTS})
endif()
find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS ${X_TOOLS_QT_COMPONENTS})

get_target_property(QT_TARGET_TYPE Qt${QT_VERSION_MAJOR}::Core TYPE)
message(STATUS "Qt${QT_VERSION_MAJOR}::Core type: ${QT_TARGET_TYPE}")
option(X_TOOLS_AUTO_LINK_MSVC_RUNTIME_LIBRARY "auto link(static or shate) msvc runtime library" ON)
if(WIN32
   AND MSVC
   AND X_TOOLS_AUTO_LINK_MSVC_RUNTIME_LIBRARY)
  if(QT_TARGET_TYPE STREQUAL "STATIC_LIBRARY")
    set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")
  endif()
endif()

# --------------------------------------------------------------------------------------------------
# Qt SerialPort module
option(X_TOOLS_ENABLE_MODULE_SERIALPORT "Enable SerialPort module" ON)
find_package(Qt${QT_VERSION_MAJOR} QUIET COMPONENTS SerialPort)
if(NOT Qt${QT_VERSION_MAJOR}SerialPort_FOUND)
  set(X_TOOLS_ENABLE_MODULE_SERIALPORT OFF)
endif()
if(X_TOOLS_ENABLE_MODULE_SERIALPORT)
  add_compile_definitions(X_TOOLS_ENABLE_MODULE_SERIALPORT)
endif()

# --------------------------------------------------------------------------------------------------
# Qt SerialBus module
option(X_TOOLS_ENABLE_MODULE_SERIALBUS "Enable SerialBus module" ON)
find_package(Qt${QT_VERSION_MAJOR} QUIET COMPONENTS SerialBus)
if(NOT Qt${QT_VERSION_MAJOR}SerialBus_FOUND)
  set(X_TOOLS_ENABLE_MODULE_SERIALBUS OFF)
endif()
if(X_TOOLS_ENABLE_MODULE_SERIALBUS)
  add_compile_definitions(X_TOOLS_ENABLE_MODULE_SERIALBUS)
endif()

# --------------------------------------------------------------------------------------------------
# BLE
option(X_TOOLS_ENABLE_MODULE_BLUETOOTH "Enable Bluetooth module" ON)
find_package(Qt${QT_VERSION_MAJOR} QUIET COMPONENTS Bluetooth)
if((NOT Qt${QT_VERSION_MAJOR}Bluetooth_FOUND) OR (QT_VERSION VERSION_LESS 6.5.0))
  set(X_TOOLS_ENABLE_MODULE_BLUETOOTH OFF)
endif()
if(X_TOOLS_ENABLE_MODULE_BLUETOOTH)
  add_compile_definitions(X_TOOLS_ENABLE_MODULE_BLUETOOTH)
endif()

# --------------------------------------------------------------------------------------------------
# CMake module
include(${CMAKE_CURRENT_LIST_DIR}/CMake/QIFW/QIFW.cmake)
include(${CMAKE_CURRENT_LIST_DIR}/CMake/xToolsCommon.cmake)
include(${CMAKE_CURRENT_LIST_DIR}/CMake/xToolsGitInfo.cmake)
include(${CMAKE_CURRENT_LIST_DIR}/CMake/xToolsDeployQt.cmake)
include(${CMAKE_CURRENT_LIST_DIR}/CMake/xToolsThirdParty.cmake)

# --------------------------------------------------------------------------------------------------
# Git module
x_tools_git_get_latest_tag(${CMAKE_CURRENT_SOURCE_DIR} "X_TOOLS")
x_tools_git_get_latest_commit(${CMAKE_CURRENT_SOURCE_DIR} "X_TOOLS")
x_tools_git_get_latest_commit_time(${CMAKE_CURRENT_SOURCE_DIR} "X_TOOLS")
set(X_TOOLS_VERSION
    ${GIT_TAG}
    CACHE STRING "The version of applicaion")

# --------------------------------------------------------------------------------------------------
# Google log module
x_tools_add_third_party("glog-0.7.0")
add_compile_definitions(X_TOOLS_ENABLE_MODULE_GLOG)

# --------------------------------------------------------------------------------------------------
# SerialBus(Modbus, CAN bus)
option(X_TOOLS_ENABLE_MODULE_SERIALBUS "Enable SerialBus module" ON)
find_package(Qt${QT_VERSION_MAJOR} QUIET COMPONENTS SerialBus)
if(NOT Qt${QT_VERSION_MAJOR}SerialBus_FOUND)
  set(X_TOOLS_ENABLE_MODULE_SERIALBUS OFF)
endif()
if(X_TOOLS_ENABLE_MODULE_SERIALBUS)
  add_compile_definitions(X_TOOLS_ENABLE_MODULE_SERIALBUS)
endif()

option(X_TOOLS_ENABLE_MODULE_MODBUS "Enable Modbus module" ON)
option(X_TOOLS_ENABLE_MODULE_CANBUS "Enable CAN bus module" ON)
if(NOT X_TOOLS_ENABLE_MODULE_SERIALBUS)
  set(X_TOOLS_ENABLE_MODULE_MODBUS OFF)
  set(X_TOOLS_ENABLE_MODULE_CANBUS OFF)
endif()
if(X_TOOLS_ENABLE_MODULE_SERIALBUS AND X_TOOLS_ENABLE_MODULE_MODBUS)
  add_compile_definitions(X_TOOLS_ENABLE_MODULE_MODBUS)
  set(X_TOOLS_MODBUS_DIR ${CMAKE_CURRENT_LIST_DIR}/Source/Modbus/Source)
  file(GLOB MODBUS_SOURCE "${X_TOOLS_MODBUS_DIR}/*.*")
  list(REMOVE_ITEM MODBUS_SOURCE ${X_TOOLS_MODBUS_DIR}/main.cpp)
  list(APPEND X_TOOLS_SOURCE ${MODBUS_SOURCE})
  include_directories(${CMAKE_CURRENT_LIST_DIR}/Source/Modbus/Source)
endif()
if(X_TOOLS_ENABLE_MODULE_SERIALBUS AND X_TOOLS_ENABLE_MODULE_CANBUS)
  add_compile_definitions(X_TOOLS_ENABLE_MODULE_CANBUS)
  set(X_TOOLS_CANBUS_DIR ${CMAKE_CURRENT_LIST_DIR}/Source/CANBus/Source)
  file(GLOB CANBUS_SOURCE "${X_TOOLS_CANBUS_DIR}/*.*")
  list(REMOVE_ITEM CANBUS_SOURCE ${X_TOOLS_CANBUS_DIR}/main.cpp)
  list(APPEND X_TOOLS_SOURCE ${CANBUS_SOURCE})
  include_directories(${CMAKE_CURRENT_LIST_DIR}/Source/CANBus/Source)
endif()

# --------------------------------------------------------------------------------------------------
# Assistant module
set(X_TOOLS_ASSISTANT_DIR "${CMAKE_CURRENT_LIST_DIR}/Source/Assistants")
macro(x_tools_add_assistant dir_name on_off)
  string(TOUPPER ${dir_name} DIR_NAME_UPPER)
  option(X_TOOLS_ENABLE_MODULE_${DIR_NAME_UPPER} "Enable ${dir_name} assistant" ${on_off})
  if(X_TOOLS_ENABLE_MODULE_${DIR_NAME_UPPER})
    include_directories(${X_TOOLS_ASSISTANT_DIR}/${dir_name}/Source)
    add_compile_definitions(X_TOOLS_ENABLE_MODULE_${DIR_NAME_UPPER})
    file(GLOB ASSISTANT_SOURCE "${X_TOOLS_ASSISTANT_DIR}/${dir_name}/Source/*.*")
    list(REMOVE_ITEM ASSISTANT_SOURCE ${X_TOOLS_ASSISTANT_DIR}/${dir_name}/Source/main.cpp)
    list(APPEND X_TOOLS_SOURCE ${ASSISTANT_SOURCE})
  else()
    file(GLOB ASSISTANT_SOURCE "${X_TOOLS_ASSISTANT_DIR}/${dir_name}/Source/*.*")
    list(REMOVE_ITEM ASSISTANT_SOURCE ${X_TOOLS_ASSISTANT_DIR}/${dir_name}/Source/main.cpp)
    foreach(SOURCE_FILE ${ASSISTANT_SOURCE})
      list(REMOVE_ITEM X_TOOLS_SOURCE ${SOURCE_FILE})
    endforeach()
  endif()
endmacro()

option(X_TOOLS_ENABLE_MODULE_ASSISTANTS "Enable assistants module" ON)
if(X_TOOLS_ENABLE_MODULE_ASSISTANTS)
  x_tools_add_assistant("CRC" ON)
  x_tools_add_assistant("mDNS" ON)
  x_tools_add_assistant("Ping" ON)
  x_tools_add_assistant("ASCII" ON)
  x_tools_add_assistant("Base64" ON)
  x_tools_add_assistant("Number" ON)
  x_tools_add_assistant("String" ON)
  x_tools_add_assistant("QRCode" ON)
  x_tools_add_assistant("Broadcast" ON)
  x_tools_add_assistant("FileCheck" ON)
  x_tools_add_assistant("FileMerge" ON)

  if(X_TOOLS_ENABLE_MODULE_QRCODE)
    x_tools_add_third_party("libqrencode-master")
  endif()

  if(X_TOOLS_ENABLE_MODULE_MDNS)
    x_tools_add_third_party("qmdnsengine-master")
  endif()

  set(X_TOOLS_ASSISTANT_DIR ${CMAKE_CURRENT_LIST_DIR}/Source/Assistants)
  include_directories(${X_TOOLS_ASSISTANT_DIR})
  add_compile_definitions(X_TOOLS_ENABLE_MODULE_ASSISTANTS)
  list(APPEND X_TOOLS_SOURCE ${X_TOOLS_ASSISTANT_DIR}/xToolsAssistantFactory.h)
  list(APPEND X_TOOLS_SOURCE ${X_TOOLS_ASSISTANT_DIR}/xToolsAssistantFactory.cpp)
endif()

# --------------------------------------------------------------------------------------------------
# Tools
include_directories(${CMAKE_CURRENT_LIST_DIR}/Source/Common/Common)
include_directories(${CMAKE_CURRENT_LIST_DIR}/Source/Common/CommonUI)
include_directories(${CMAKE_CURRENT_LIST_DIR}/Source/Tools/Tools)
include_directories(${CMAKE_CURRENT_LIST_DIR}/Source/Tools/ToolsUI)
include_directories(${CMAKE_CURRENT_LIST_DIR}/Source/ToolBox/ToolBox)
include_directories(${CMAKE_CURRENT_LIST_DIR}/Source/ToolBox/ToolBoxUI)
include_directories(${CMAKE_CURRENT_LIST_DIR}/Source)

add_subdirectory(${CMAKE_CURRENT_LIST_DIR}/Source/Common)
add_subdirectory(${CMAKE_CURRENT_LIST_DIR}/Source/Tools)

file(GLOB TMP "${CMAKE_CURRENT_SOURCE_DIR}/Source/*.*")
file(GLOB_RECURSE X_TOOLS_TOOLBOX_SOURCE "${CMAKE_CURRENT_SOURCE_DIR}/Source/ToolBox/*.*")
list(APPEND X_TOOLS_SOURCE ${TMP})
list(APPEND X_TOOLS_SOURCE ${CMAKE_CURRENT_SOURCE_DIR}/xTools.qrc)
list(APPEND X_TOOLS_SOURCE ${X_TOOLS_TOOLBOX_SOURCE})
if(WIN32)
  list(APPEND X_TOOLS_SOURCE ${CMAKE_CURRENT_SOURCE_DIR}/xTools.rc)
endif()

# --------------------------------------------------------------------------------------------------
# xTools application Core
list(REMOVE_ITEM X_TOOLS_SOURCE ${CMAKE_CURRENT_SOURCE_DIR}/Source/main.cpp)
add_library(xToolsCore STATIC ${X_TOOLS_SOURCE})
add_dependencies(xToolsCore xToolsTools)
target_link_libraries(xToolsCore PUBLIC xToolsTools)
if(X_TOOLS_ENABLE_MODULE_QRCODE)
  target_link_libraries(xToolsCore PUBLIC qrencode)
endif()
if(X_TOOLS_ENABLE_MODULE_MDNS)
  target_link_libraries(xToolsCore PUBLIC qmdnsengine)
endif()

# --------------------------------------------------------------------------------------------------
# xTools application
option(X_TOOLS_ENABLE_TARGET_XTOOLS "Enable xTools applications" ON)
if(X_TOOLS_ENABLE_TARGET_XTOOLS)
  list(APPEND X_TOOLS_SOURCE ${CMAKE_CURRENT_SOURCE_DIR}/Source/main.cpp)
  x_tools_add_executable(xTools ${X_TOOLS_SOURCE})
  x_tools_copy_style_resources_for_target(xTools)
  x_tools_generate_translations(xTools)
  if(NOT QT_TARGET_TYPE STREQUAL "STATIC_LIBRARY")
    x_tools_deploy_qt(xTools)
  endif()
  x_tools_tar_target(xTools)
  x_tools_generate_installer(xTools ${X_TOOLS_VERSION})

  add_dependencies(xTools xToolsCore)
  target_link_libraries(xTools PRIVATE xToolsCore)
endif()

# -------------------------------------------------------------------------------------------------
# Assistant applications
option(X_TOOLS_ENABLE_TARGET_ASSISTANTS "Enable assistant applications" OFF)
if(X_TOOLS_ENABLE_TARGET_ASSISTANTS)
  add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/Source/Assistants)
endif()
