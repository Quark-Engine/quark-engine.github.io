# Документация API плагинов

Плагины в Quark Engine представляют собой динамические библиотеки (DLL или `.so`), которые позволяют расширять функционал движка, создавать новые инструменты и модифицировать поведение сцены в реальном времени.

## Быстрый старт

Каждый плагин должен экспортировать функцию `get_plugin()`. Это единственная точка входа, которую ищет движок.

```asm
section .data
    plugin_name    db "Example Plugin", 0
    plugin_version db "1.0.0", 0

    ; Структура Plugin (name, version, on_load, on_unload, on_update, on_draw_ui)
    my_plugin:
        dq plugin_name
        dq plugin_version
        dq 0          ; on_load
        dq 0          ; on_unload
        dq on_update  ; on_update
        dq 0          ; on_draw_ui

section .text
    global get_plugin

on_update:
    ; RCX содержит указатель на PluginContext (Windows x64 ABI)
    ret

get_plugin:
    lea rax, [rel my_plugin]
    ret
```

> **Внимание:** Не сохраняйте указатель `PluginContext` между вызовами функций. Движок может пересоздавать контекст в каждом кадре для обновления данных о выделенных объектах и статистике.

## Следующие шаги

- [Жизненный цикл плагина](lifecycle.md)
- [Функциональные возможности](features.md)
