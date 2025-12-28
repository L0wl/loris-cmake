cmake_minimum_required(VERSION 3.14)
project(Loris LANGUAGES C CXX)

option(BUILD_SHARED_LIBS "Build shared libraries" ON)
option(LORIS_BUILD_PYTHON "Build Python bindings" OFF) # По умолчанию выкл, чтобы упростить сборку
file(GLOB ${PROJECT_NAME}_SOURCES "src/*.C" "src/*.c")

if(NOT LORIS_BUILD_PYTHON)
    list(FILTER LORIS_SOURCES EXCLUDE REGEX ".*_pi\\.C$")
endif()

# --- Создание библиотеки ---
add_library(${PROJECT_NAME} SHARED ${${PROJECT_NAME}_SOURCES})

# --- Настройка путей к заголовкам ---
target_include_directories(${PROJECT_NAME} PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
    # В папке win лежат готовые config.h и loris.h для Windows
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/win> 
    $<INSTALL_INTERFACE:include>
)

# --- Настройки компилятора для MinGW/Windows ---
if(WIN32)
    target_compile_definitions(${PROJECT_NAME} PRIVATE 
        _CRT_SECURE_NO_WARNINGS 
        WIN32
        _WIN32
    )
    # Если в коде есть проверки на M_PI и прочее
    target_compile_definitions(${PROJECT_NAME} PRIVATE _USE_MATH_DEFINES)
endif()

# --- (Опционально) Установка ---
install(TARGETS ${PROJECT_NAME} 
    EXPORT LorisTargets
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
    RUNTIME DESTINATION bin
)
install(DIRECTORY src/ DESTINATION include/${PROJECT_NAME} FILES_MATCHING PATTERN "*.h")