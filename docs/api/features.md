# Функциональные возможности

## Интерфейс (UI)

Движок предоставляет Immediate-mode UI систему. Все вызовы должны находиться внутри колбэка `on_draw_ui`.

| Функция | Описание |
|---|---|
| `ui_begin(title)` / `ui_end()` | Открыть / закрыть окно |
| `ui_button(label)` | Кнопка, возвращает `true` при нажатии |
| `ui_slider_float(label, &val, min, max)` | Слайдер для вещественных чисел |
| `ui_color_edit3(label, color[3])` | Выбор цвета в формате RGB |

```cpp
void on_draw_ui(PluginContext* ctx) {
    ui_begin("Настройки");

    static float speed = 1.0f;
    ui_slider_float("Скорость", &speed, 0.0f, 10.0f);

    static float color[3] = {1, 0, 0};
    ui_color_edit3("Цвет", color);

    ui_end();
}
```

## Работа с сущностями

Полный контроль над объектами в сцене через `PluginContext`.

### Чтение

```cpp
Vec3 pos = entity_get_position(ctx, entity_id);
const char* name = entity_get_name(ctx, entity_id);
```

### Мутация

```cpp
entity_set_scale(ctx, entity_id, {2.0f, 2.0f, 2.0f});
entity_set_rotation(ctx, entity_id, {0, 90, 0});
```

### Управление сценой

```cpp
EntityId new_entity = scene_spawn(ctx, "cube");
scene_delete(ctx, entity_id);
```

> **Внимание:** Не сохраняйте указатель `PluginContext` между вызовами функций. Движок может пересоздавать контекст в каждом кадре.
