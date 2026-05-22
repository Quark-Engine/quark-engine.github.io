# UI System

The UI API is a thin wrapper over an immediate-mode editor UI. All UI functions are available as function pointers on `PluginContext` and are only valid during the current callback.

## Windows

Every plugin UI lives inside a window. Use `ui_begin` and `ui_end` to open and close one.

```cpp
void (*on_draw_ui)(PluginContext* ctx) {
    if (ctx->ui_begin("My Plugin")) {
        ctx->ui_text("Hello from my plugin.");
        ctx->ui_button("Click me");
    }
    ctx->ui_end();
}
```

> **Important:** Always call `ui_end()`, even when `ui_begin()` returns `false`. Failing to do so will corrupt the UI layout.

```cpp
bool (*ui_begin)(const char* title);   // Returns true if the window is visible.
void (*ui_end)();                      // Must always be called after ui_begin().
```

## Menus

Plugins can add items to the editor's top menu bar by registering a callback for a menu region (see [UI Regions](#ui-regions)) and using the menu functions inside it.

```cpp
static void on_file_menu(PluginContext* ctx) {
    if (ctx->ui_begin_menu("Export")) {
        if (ctx->ui_menu_item("Export as JSON")) {
            // handle export
        }
        ctx->ui_end_menu();
    }
}

// Register in on_load:
ctx->register_ui_callback(UI_MENU_FILE, on_file_menu);
```

```cpp
bool (*ui_begin_menu)(const char* label);   // Returns true if the menu is open.
void (*ui_end_menu)();
bool (*ui_menu_item)(const char* label);    // Returns true when clicked.
```

## Widgets

These are the available controls for building plugin windows and inspector panels.

### Text

Renders a read-only label.

```cpp
ctx->ui_text("Position");
```

```cpp
void (*ui_text)(const char* text);
```

### Button

Returns `true` on the single frame it is clicked.

```cpp
if (ctx->ui_button("Reset")) {
    ctx->entity_set_position(index, 0, 0, 0);
}
```

```cpp
bool (*ui_button)(const char* label);
```

### Checkbox

Bound directly to a `bool`. Returns `true` when the value changes.

```cpp
static bool show_bounds = false;

ctx->ui_checkbox("Show bounds", &show_bounds);
```

```cpp
bool (*ui_checkbox)(const char* label, bool* value);
```

### Float Slider

Renders a slider clamped to `[min, max]`. Returns `true` while being dragged.

```cpp
static float speed = 1.0f;

ctx->ui_slider_float("Speed", &speed, 0.0f, 10.0f);
```

```cpp
bool (*ui_slider_float)(const char* label, float* value, float min, float max);
```

### Float Input

A direct-entry number field. Returns `true` when the value is committed (Enter or focus loss).

```cpp
static float mass = 1.0f;

ctx->ui_input_float("Mass", &mass);
```

```cpp
bool (*ui_input_float)(const char* label, float* value);
```

### Color Picker

An RGB color picker. `color` is a three-element `float` array in the `[0.0, 1.0]` range. Returns `true` when any component changes.

```cpp
static float tint[3] = { 1.0f, 1.0f, 1.0f };

ctx->ui_color_edit3("Tint", tint);
```

```cpp
bool (*ui_color_edit3)(const char* label, float color[3]);
```

### Layout Helpers

```cpp
void (*ui_separator)();    // Draws a horizontal divider line.
void (*ui_same_line)();    // Places the next widget on the same line as the previous one.
```

Example — two buttons side by side:

```cpp
ctx->ui_button("Accept");
ctx->ui_same_line();
ctx->ui_button("Cancel");
```

## UI Regions

Plugins can inject UI into specific parts of the editor by registering a `PluginUICallback` for a given region. Registered callbacks are called automatically by the host when that region is being drawn.

```cpp
void (*register_ui_callback)(UIRegion region, PluginUICallback callback);
```

Register callbacks during `on_load`:

```cpp
void (*on_load)(PluginContext* ctx) {
    ctx->register_ui_callback(UI_INSPECTOR, on_inspector);
    ctx->register_ui_callback(UI_MENU_FILE, on_file_menu);
}
```

| Region           | Where it draws                              | Typical use                        |
|------------------|---------------------------------------------|------------------------------------|
| `UI_MENU_FILE`   | File menu in the top menu bar               | Export, import, custom file actions |
| `UI_MENU_EDIT`   | Edit menu in the top menu bar               | Undo extensions, selection tools   |
| `UI_MENU_HELP`   | Help menu in the top menu bar               | About dialogs, documentation links |
| `UI_HIERARCHY`   | Scene hierarchy panel                       | Custom tree views, filtering       |
| `UI_INSPECTOR`   | Inspector panel (selected entity)           | Per-component controls, gizmos     |
| `UI_SCENE`       | Scene viewport overlay                      | Debug overlays, in-world widgets   |

## Related Docs

- [Plugin Overview](api/plugins.md)
- [Plugin Lifecycle](api/lifecycle.md)
- [Scene Management](api/scene.md)
- [Entities](api/entities.md)