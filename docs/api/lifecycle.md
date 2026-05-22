# Plugin Lifecycle

This page explains when each plugin callback is called and what each callback should be used for.

## Load Order

```text
Load library
→ call get_plugin()
→ call on_load()
→ call on_update() every frame
→ call on_draw_ui() every UI frame
→ call on_unload()
→ unload library
```

If `get_plugin()` returns `nullptr`, or if required fields in `Plugin` are missing, the engine should treat the load as failed.

## on_load

`on_load(PluginContext* ctx)` is called once after the library is loaded. Use it for initialization, resource setup, and registering callbacks.

```cpp
static void on_load(PluginContext* ctx) {
    ctx->register_ui_callback(UI_INSPECTOR, my_ui);
}
```

## on_update

`on_update(PluginContext* ctx)` is called every simulation frame. Use it for time-based logic, animation, and non-UI plugin behavior.

```cpp
static void on_update(PluginContext* ctx) {
    float dt = ctx->delta_time;
    (void)dt;
}
```

## on_draw_ui

`on_draw_ui(PluginContext* ctx)` is called every UI pass. Use the `ui_*` functions on `ctx` to draw controls.

```cpp
static void on_draw_ui(PluginContext* ctx) {
    if (ctx->ui_begin("My Plugin")) {
        ctx->ui_text("Hello from plugin");
        ctx->ui_end();
    }
}
```

## on_unload

`on_unload()` is called once before the library is unloaded. Free heap allocations and release external resources here.

```cpp
static void on_unload() {
    // cleanup
}
```

## Rules

- Keep callbacks fast.
- Do not block the main thread.
- Do not keep `PluginContext*` after the call ends.
- Make sure every `ui_begin()` has a matching `ui_end()`.

## Related Docs

- [Plugins](api/plugins.md)
- [UI System](api/ui.md)
- [Scene Management](api/scene.md)
- [Entities](api/entities.md)