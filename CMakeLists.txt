cmake_minimum_required(VERSION 3.21)

# Установка toolchain file для vcpkg
set(CMAKE_TOOLCHAIN_FILE "$ENV{VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake" CACHE STRING "")

project(
    DiscordRichPresence
    VERSION 1.2.0
    LANGUAGES CXX
)

# Установка стандарта C++
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Добавление пути к модулям CMake
list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake")
include(XSEPlugin)

# Добавление путей включения для vcpkg
include_directories(${CMAKE_BINARY_DIR}/vcpkg_installed/${VCPKG_TARGET_TRIPLET}/include)

# Создание библиотеки (перемещено сюда и изменено)
if(NOT TARGET ${PROJECT_NAME})
    add_library(${PROJECT_NAME} SHARED)
endif()

# Поиск SimpleIni
find_path(SIMPLEINI_INCLUDE_DIRS "ConvertUTF.c")

# Добавление путей включения для SimpleIni
target_include_directories(
    ${PROJECT_NAME}
    PRIVATE
        ${SIMPLEINI_INCLUDE_DIRS}
)

# Поиск зависимостей
find_package(magic_enum CONFIG REQUIRED)
find_package(RapidJSON CONFIG REQUIRED)

# Связывание библиотек
target_link_libraries(
    ${PROJECT_NAME} 
    PRIVATE
        magic_enum::magic_enum
)

# Добавление RapidJSON как системного включения
target_include_directories(${PROJECT_NAME} SYSTEM PRIVATE ${RAPIDJSON_INCLUDE_DIRS})

# Добавление исходных файлов
target_sources(${PROJECT_NAME}
    PRIVATE
        src/RichPresence.cpp
        src/XSEPlugin.cpp
        src/connection_win.cpp
        src/discord_rpc.cpp
        src/dllmain.cpp
        src/rpc_connection.cpp
        src/serialization.cpp
        # Добавьте здесь другие .cpp файлы, если они есть
)

# Дополнительные настройки компилятора (если необходимо)
if(MSVC)
    target_compile_options(${PROJECT_NAME} PRIVATE /W4 /WX)
else()
    target_compile_options(${PROJECT_NAME} PRIVATE -Wall -Wextra -Werror)
endif()

# Настройка выходного каталога (если необходимо)
set_target_properties(${PROJECT_NAME} PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin"
)

# Копирование зависимостей (если необходимо)
add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
        $<TARGET_FILE:magic_enum::magic_enum>
        $<TARGET_FILE_DIR:${PROJECT_NAME}>
)

# Установка (если необходимо)
install(TARGETS ${PROJECT_NAME}
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
)