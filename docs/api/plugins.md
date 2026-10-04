# Plugin API

Quark Engine plugins are dynamically loaded libraries: `.dll` on Windows,
`.so` on Linux, and `.dylib` on macOS. A plugin exports one entry point,
`GetPlugin()`, which returns a descriptor containing its metadata and lifecycle
callbacks. The public declarations are in `include/plugins/plugin.h`.

## Minimal plugin

```cpp
#include "plugins/plugin.h"

static void OnLoad(SPluginContext* pCtx)
{
    (void)pCtx;
}

static void OnUnload()
{
}

static void OnUpdate(SPluginContext* pCtx)
{
    (void)pCtx;
}

static void OnDrawUI(SPluginContext* pCtx)
{
    if (pCtx->pfnUiBegin("Example Plugin"))
    {
        pCtx->pfnUiText("Hello from the plugin.");
    }
    pCtx->pfnUiEnd();
}

static SPlugin s_Plugin =
{
    "Example Plugin",
    "1.0.0",
    OnLoad,
    OnUnload,
    OnUpdate,
    OnDrawUI
};

PLUGIN_EXPORT SPlugin* GetPlugin()
{
    return &s_Plugin;
}
```

The descriptor and its strings must remain valid for the entire lifetime of the
loaded library, so use static storage or another lifetime that outlasts the
plugin. `GetPlugin()` must use the exact exported name and return a non-null
descriptor. The descriptor's lifecycle callbacks are expected to be non-null.

## Plugin descriptor

```cpp
struct SPlugin
{
    const char* pName;
    const char* pVersion;
    void (*pfnOnLoad)(SPluginContext* pCtx);
    void (*pfnOnUnload)();
    void (*pfnOnUpdate)(SPluginContext* pCtx);
    void (*pfnOnDrawUI)(SPluginContext* pCtx);
};
```

- `pName` and `pVersion` are static, null-terminated strings.
- `pfnOnLoad` runs after the library is loaded and receives the host context.
- `pfnOnUnload` runs before the library is unloaded; it does not receive a context.
- `pfnOnUpdate` runs during the host's simulation update.
- `pfnOnDrawUI` runs during the host's UI pass.

See [Plugin Lifecycle](lifecycle.md) for callback timing and cleanup details.

## Host context

`SPluginContext` supplies frame state and host functions for UI, entities,
components, assets, scene operations, events, and editor integration. Pass the
provided `pScene` and `pAssets` handles to API functions that require them.

The context is host-owned and must not be retained after a callback returns.
Use its current value on each callback. `deltaTime` is elapsed time in seconds;
`entityCount` is the current entity count; `pSelected` may be null and should be
checked before dereferencing.

```cpp
void OnUpdate(SPluginContext* pCtx)
{
    if (pCtx->pSelected != nullptr && *pCtx->pSelected >= 0)
    {
        const int selected = *pCtx->pSelected;
        const char* pName = pCtx->pfnEntityGetName(pCtx->pScene, selected);
        (void)pName;
    }
}
```

## Extending the editor

Plugins can register callbacks for editor UI regions with
`pfnRegisterUICallback`, subscribe to scene/editor events with
`pfnRegisterEventCallback`, and register custom component factories with
`pfnRegisterComponentFactory`. These registrations should generally happen in
`pfnOnLoad`. Registered UI and event callbacks are removed by the host when the
plugin is unloaded. Note that
`pfnOnUnload` does not receive a context, while
`pfnUnregisterComponentFactory` requires one; the current API has no direct way
to unregister a component factory from that callback. Avoid unloading a plugin
that has registered component factories until this lifecycle limitation is
addressed.

The API also exposes scene commands (`pfnSceneBeginCommand` and
`pfnSceneEndCommand`) so grouped scene edits can participate in undo/redo.
Use the feature-specific pages for signatures and examples:

- [Plugin Lifecycle and Events](lifecycle.md)
- [UI System](ui.md)
- [Scene Management](scene.md)
- [Entities and Components](entities.md)

## ABI compatibility

New fields added to `SPluginContext` are appended to its end to preserve offsets
for plugins compiled against older headers. Build plugins against the headers
for the engine version they target, and null-check newly added fields when
supporting older host versions.
