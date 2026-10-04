# Plugin Lifecycle

This page describes the lifecycle callbacks and event subscriptions exposed by
`include/plugins/plugin.h`.

## Load and unload

The host loads the dynamic library, resolves `GetPlugin()`, obtains its
`SPlugin` descriptor, and calls `pfnOnLoad` when a context is available.
`pfnOnUpdate` and `pfnOnDrawUI` are then called during their respective host
passes. When a plugin is disabled or unloaded, the host calls `pfnOnUnload`
before unloading the library.

```text
Load library
→ GetPlugin()
→ pfnOnLoad(SPluginContext*)
→ pfnOnUpdate(SPluginContext*) during simulation
→ pfnOnDrawUI(SPluginContext*) during UI
→ pfnOnUnload()
→ unload library
```

The editor's Plugins preferences toggle enables or disables a plugin. A
disabled plugin is unloaded immediately and its disabled state is persisted by
the editor; enabling it loads the library again and runs `pfnOnLoad`.

## `pfnOnLoad`

Use `pfnOnLoad(SPluginContext* pCtx)` for initialization and registrations:
UI callbacks, event callbacks, and component factories. The context is
borrowed from the host and is valid only for the duration of the call.

```cpp
static void OnLoad(SPluginContext* pCtx)
{
    pCtx->pfnRegisterUICallback(pCtx, UI_INSPECTOR, DrawInspector);
    pCtx->pfnRegisterEventCallback(pCtx, PLUGIN_EVENT_SCENE_LOADED, OnSceneEvent);
}
```

## `pfnOnUpdate`

`pfnOnUpdate(SPluginContext* pCtx)` runs during the simulation update. Use
`pCtx->deltaTime` for frame-rate-independent work and avoid blocking or doing
expensive work every frame.

```cpp
static void OnUpdate(SPluginContext* pCtx)
{
    const float frameSeconds = pCtx->deltaTime;
    (void)frameSeconds;
}
```

## `pfnOnDrawUI`

`pfnOnDrawUI(SPluginContext* pCtx)` runs during the UI pass. Pair every
`pfnUiBegin()` with `pfnUiEnd()`, even if the begin function returns `false`.
Registered region callbacks are invoked by the host when their region is drawn.

```cpp
static void OnDrawUI(SPluginContext* pCtx)
{
    if (pCtx->pfnUiBegin("Plugin"))
    {
        pCtx->pfnUiText("Plugin controls");
    }
    pCtx->pfnUiEnd();
}
```

## `pfnOnUnload`

`pfnOnUnload()` runs immediately before the library is unloaded and does not
receive `SPluginContext`. Release plugin-owned resources here. The host removes
that plugin's UI and event callback registrations as part of unloading.

There is currently an API limitation for plugin-defined component factories:
`pfnUnregisterComponentFactory` requires an `SPluginContext*`, but
`pfnOnUnload()` has no context parameter. Do not retain a context pointer to
work around this, because its lifetime is limited to the host callback. Until
the host provides a context-aware teardown hook, avoid unloading a plugin while
it has a registered component factory.

## Events

Register an `FPluginEventCallback` with `pfnRegisterEventCallback`. The callback
receives the current context, event kind, and a related entity index. The index
is `-1` for scene-wide notifications; entity-related event indices follow the
event's documented timing, so query current state rather than caching indices.

```cpp
static void OnSceneEvent(SPluginContext* pCtx, EPluginEvent event, int entityIndex)
{
    if (event == PLUGIN_EVENT_SCENE_LOADED)
    {
        (void)pCtx;
        (void)entityIndex;
    }
}

static void OnLoad(SPluginContext* pCtx)
{
    pCtx->pfnRegisterEventCallback(
        pCtx, PLUGIN_EVENT_SCENE_LOADED, OnSceneEvent);
}
```

Available event values:

| Event | Meaning |
|---|---|
| `PLUGIN_EVENT_ENTITY_CREATED` | An entity was appended to the scene. |
| `PLUGIN_EVENT_ENTITY_DELETED` | An entity is about to be removed. |
| `PLUGIN_EVENT_ENTITY_SELECTED` | Active or multi-selection changed. |
| `PLUGIN_EVENT_TRANSFORM_CHANGED` | An entity transform changed. |
| `PLUGIN_EVENT_SCENE_LOADED` | The current scene finished loading. |
| `PLUGIN_EVENT_SCENE_SAVED` | The current scene finished saving. |

Use `pfnUnregisterEventCallback(pCtx, event, callback)` to remove a subscription
before unloading when needed.

## Rules

- Never retain `SPluginContext*` or host-owned pointers beyond the callback.
- Keep update and UI callbacks fast and non-blocking.
- Release resources owned by the plugin before its library is unloaded.
- Keep callback function code and plugin descriptor storage in the library
  until the host has completed unloading it.

## Related docs

- [Plugin API](plugins.md)
- [UI System](ui.md)
- [Scene Management](scene.md)
- [Entities and Components](entities.md)
