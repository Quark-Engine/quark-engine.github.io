# Plugin API

Plugins in Quark Engine are dynamic libraries (`.dll` or `.so`) that extend the engine, add editor tools, and can change scene behavior at runtime. The API is built around a single exported entry point and a host-provided context for UI, entity access, and scene control.

## Quick Start

Every plugin must export `get_plugin()`. This is the only symbol the engine looks for when loading the library.

```cpp
#include "plugin.h"

static void on_update(PluginContext* ctx) {
    (void)ctx;
}

static Plugin plugin = {
    "Example Plugin",
    "1.0.0",
    nullptr,
    nullptr,
    on_update,
    nullptr
};

PLUGIN_EXPORT Plugin* get_plugin() {
    return &plugin;
}
```

> **Important:** Do not store `PluginContext*` between calls. It is only valid during the current callback and may be recreated every frame.

## What This File Covers

- What a plugin is.
- How the host loads plugins.
- What `Plugin` and `PluginContext` contain.
- The minimal plugin example.
- Links to lifecycle and features docs.

## Plugin Structure

```cpp
struct Plugin {
    const char* name;
    const char* version;
    void (*on_load)(PluginContext* ctx);
    void (*on_unload)();
    void (*on_update)(PluginContext* ctx);
    void (*on_draw_ui)(PluginContext* ctx);
};
```

- `name` is the display name shown in the plugin manager.
- `version` is a string like `"1.0.0"`.
- `on_load` is called once after the plugin is loaded.
- `on_unload` is called before the library is unloaded.
- `on_update` is called every simulation frame.
- `on_draw_ui` is called every UI pass.

## PluginContext

```cpp
struct PluginContext {
    float delta_time;
    int entity_count;
    int* selected;
    ...
};
```

`PluginContext` contains current-frame data and host functions for UI, entity access, and scene management. Treat it as temporary and never keep it after the callback returns.

## Related Docs

- [Plugin Lifecycle](api/lifecycle.md)
- [UI System](api/ui.md)
- [Scene Management](api/scene.md)
- [Entities](api/entities.md)