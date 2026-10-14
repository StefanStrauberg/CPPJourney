---
tags: [cpp/справочник, cpp/cmake]
---

# Шпаргалка — CMake

Подробности в уроках [[00.4 CMake с нуля]], [[00.5 Проект курса в VS Code]] и [[02.6 CMake — библиотеки и структура проекта]].

## Команды в CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.30)        # всегда первой строкой
project(CppJourney LANGUAGES CXX)           # имя проекта, язык C++

set(CMAKE_CXX_STANDARD 23)                  # стандарт C++
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)   # куда класть .exe

add_subdirectory(ember)                     # подключить папку со своим CMakeLists.txt

add_executable(game main.cpp map.cpp)       # программа
add_library(ember STATIC src/math.cpp)      # статическая библиотека (.lib)
add_library(utils INTERFACE)                # библиотека только из заголовков

target_include_directories(ember PUBLIC include)    # где искать заголовки
target_link_libraries(game PRIVATE ember console)   # подключить библиотеки
target_compile_options(game PRIVATE /W4)            # флаги компилятора
target_compile_definitions(game PRIVATE DEBUG_DRAW=1)   # макросы препроцессора

set(NAME "value")                           # переменная
message(STATUS "Значение: ${NAME}")         # вывод при конфигурации
if(MSVC) ... endif()                        # условие
```

## PUBLIC / PRIVATE / INTERFACE

| Слово | Нужно самой цели | Передаётся зависящим целям |
|---|---|---|
| `PRIVATE` | да | нет |
| `PUBLIC` | да | да |
| `INTERFACE` | нет | да |

- Публичные заголовки библиотеки → `PUBLIC`.
- Библиотека используется только в `.cpp` → `target_link_libraries(... PRIVATE ...)`.
- Заголовки библиотеки A подключают заголовки библиотеки B → `target_link_libraries(A PUBLIC B)`.

## Флаги MSVC курса

| Флаг | Смысл |
|---|---|
| `/W4` | Максимум полезных предупреждений |
| `/WX` | Предупреждения считаются ошибками |
| `/permissive-` | Строгое соответствие стандарту |
| `/utf-8` | Исходники и строки в UTF-8 |
| `/Zc:__cplusplus` | Честное значение `__cplusplus` |

## Конфигурации

| Конфигурация | Флаги MSVC | Для чего |
|---|---|---|
| `Debug` | `/Od /Ob0 /RTC1` + отладочная информация | Отладка |
| `Release` | `/O2 /Ob2 /DNDEBUG` | Скорость |
| `RelWithDebInfo` | `/O2 /Ob1 /DNDEBUG` + отладочная информация | Профилирование, отладка оптимизированного кода |
| `MinSizeRel` | `/O1 /Ob1 /DNDEBUG` | Размер |

## Командная строка

```powershell
cmake --preset msvc                  # конфигурация по пресету
cmake --build --preset debug         # сборка Debug
cmake --build --preset release       # сборка Release
cmake --build --preset debug --target m02_rogue   # только одна цель

# без пресетов
cmake -S . -B build -G "Visual Studio 18 2026" -A x64
cmake --build build --config Release
```

## Пресеты (CMakePresets.json)

```json
{
  "version": 6,
  "configurePresets": [
    {
      "name": "msvc",
      "displayName": "MSVC x64 (Visual Studio 2026)",
      "generator": "Visual Studio 18 2026",
      "architecture": "x64",
      "binaryDir": "${sourceDir}/build"
    }
  ],
  "buildPresets": [
    { "name": "debug",   "displayName": "Debug",   "configurePreset": "msvc", "configuration": "Debug" },
    { "name": "release", "displayName": "Release", "configurePreset": "msvc", "configuration": "Release" }
  ]
}
```

## Когда что-то сломалось

| Проблема | Решение |
|---|---|
| CMake ведёт себя странно | `Ctrl+Shift+P` → **CMake: Delete Cache and Reconfigure** (или удалить папку `build`) |
| Новый `.cpp` «не виден», `LNK2019` | Добавить файл в `add_executable`/`add_library` |
| Не находится `<ember/...>` | Цель не подключила библиотеку через `target_link_libraries`, или путь объявлен `PRIVATE` |
| `Could not create named generator Visual Studio 18 2026` | CMake старше 4.2 |
| `CMAKE_BUILD_TYPE` не действует | Генератор Visual Studio мульти-конфигурационный: выбирай конфигурацию build-пресетом |

## Правила хорошего тона

- Настраивай цели через `target_*`, а не глобальными командами.
- Перечисляй исходники явно, без `file(GLOB)`.
- Имена целей уникальны во всём проекте: в курсе они начинаются с номера модуля (`m02_rogue`).
- Публичные заголовки библиотеки — в `include/<имя_библиотеки>/`.

---
[[🏠 Главная|🏠 На главную]]
