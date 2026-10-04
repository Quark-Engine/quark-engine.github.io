# Entities and Components

Entity functions are provided by `SPluginContext` and operate on the
host-owned scene. Pass `pCtx->pScene` to entity functions. Valid entity indices
are normally in `[0, pCtx->entityCount)`; scene mutations can invalidate or
shift indices.

## Read and update entity state

The API provides `pfnEntityGetName`, `pfnEntityGetPosition`,
`pfnEntityGetRotation`, `pfnEntityGetScale`, and `pfnEntityGetColor` for
queries, and corresponding `pfnEntitySet...` functions for updates.
String results are host-owned; do not free or modify them.

```cpp
const int entityIndex = pCtx->pfnSceneGetSelected(pCtx->pScene);
if (entityIndex >= 0)
{
    float x, y, z;
    pCtx->pfnEntityGetPosition(pCtx->pScene, entityIndex, &x, &y, &z);
    pCtx->pfnEntitySetPosition(pCtx->pScene, entityIndex, x, y + 1.0f, z);
}
```

Color uses four `unsigned char` channels in the `[0, 255]` range. Entity
rotation uses the host's rotation unit.

## Hierarchy

`pfnEntityGetParent` returns a parent index, or `-1` for a root.
`pfnEntitySetParent` reparents while preserving world transform and returns
`false` for invalid indices or a parent that would create a cycle.

```cpp
const int parentIndex = pCtx->pfnEntityGetParent(pCtx->pScene, entityIndex);
const bool reparented = pCtx->pfnEntitySetParent(
    pCtx->pScene, entityIndex, parentIndex);
(void)reparented;
```

## Components

Plugins can inspect components with `pfnEntityGetComponentCount`,
`pfnEntityGetComponentType`, and `pfnEntityHasComponent`; add, remove, or toggle
them with `pfnEntityAddComponent`, `pfnEntityRemoveComponent`, and
`pfnEntitySetComponentEnabled`.

To make a plugin-defined component constructible after loading a scene or
applying undo, register a factory with `pfnRegisterComponentFactory`. The
factory returns a newly allocated default component instance for the host to
own.

```cpp
static void* CreateExampleComponent()
{
    return new ExampleComponent();
}

static void OnLoad(SPluginContext* pCtx)
{
    pCtx->pfnRegisterComponentFactory(
        pCtx, "ExampleComponent", CreateExampleComponent);
}
```

Built-in component type names cannot be overwritten. There is currently a
lifecycle limitation: `pfnUnregisterComponentFactory` needs a context, but
`pfnOnUnload` does not provide one. Do not retain `SPluginContext*` beyond a
callback to work around this. Until the host provides a context-aware teardown
hook, avoid unloading a plugin that registered component factories.

## Tags

Use `pfnEntityGetTagCount`, `pfnEntityGetTag`, and `pfnEntityHasTag` to query
tags. `pfnEntityAddTag` and `pfnEntityRemoveTag` mutate them. Tag strings
returned by the host must not be freed or modified.

## Selected entity

`pCtx->pSelected` is a pointer to the selected entity index and can be null.
Alternatively, use the multi-selection functions in
[Scene Management](scene.md).

```cpp
if (pCtx->pSelected != nullptr && *pCtx->pSelected >= 0)
{
    const int entityIndex = *pCtx->pSelected;
    const char* pName = pCtx->pfnEntityGetName(pCtx->pScene, entityIndex);
    (void)pName;
}
```

## Related docs

- [Plugin API](plugins.md)
- [Plugin Lifecycle](lifecycle.md)
- [UI System](ui.md)
- [Scene Management](scene.md)
