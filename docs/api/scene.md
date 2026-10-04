# Scene Management

Scene and editor functions are exposed as function pointers on
`SPluginContext`. Scene operations take the host-owned `pCtx->pScene`; asset
operations use `pCtx->pAssets`. Do not retain these borrowed handles beyond
the current callback.

## Save

`pfnSceneSave` serializes the scene to the project path supplied by the host:

```cpp
pCtx->pfnSceneSave(pCtx->pProjectPath, pCtx->pScene);
```

## Spawn and delete entities

Spawn a registered asset by name. The function returns the new entity index or
`-1` on failure. `pfnSceneSpawnEx` additionally sets the initial position.

```cpp
const int entityIndex = pCtx->pfnSceneSpawn(
    pCtx->pAssets, pCtx->pScene, "Cube");
if (entityIndex >= 0)
{
    pCtx->pfnEntitySetPosition(pCtx->pScene, entityIndex, 0.0f, 1.0f, 0.0f);
}
```

```cpp
const int entityIndex = pCtx->pfnSceneSpawnEx(
    pCtx->pAssets, pCtx->pScene, "Cube", 0.0f, 1.0f, 0.0f);
```

Delete an entity with `pfnSceneDelete`. Entity indices greater than the
deleted index can shift, so re-query selection and entity state after deletion;
do not keep indices across scene mutations.

```cpp
pCtx->pfnSceneDelete(pCtx->pScene, entityIndex);
```

## Selection

`pfnSceneGetSelected` returns the primary selected index or `-1`.
`pfnSceneGetSelectionCount` and `pfnSceneGetSelectedAt` expose multi-selection.
Use `pfnSceneSetSelected` to replace or add/toggle selection.

```cpp
const int selectionCount = pCtx->pfnSceneGetSelectionCount(pCtx->pScene);
for (int i = 0; i < selectionCount; ++i)
{
    const int entityIndex = pCtx->pfnSceneGetSelectedAt(pCtx->pScene, i);
    (void)entityIndex;
}
```

## Undoable plugin commands

Group related scene mutations into a host undo command:

```cpp
pCtx->pfnSceneBeginCommand(pCtx->pScene, "Move selected entity");
pCtx->pfnEntitySetPosition(pCtx->pScene, entityIndex, 1.0f, 2.0f, 3.0f);
pCtx->pfnSceneEndCommand(pCtx->pScene);
```

Call begin before the mutations and always pair it with end. Nested plugin
commands are ignored. The context also provides `pfnSceneUndo`,
`pfnSceneRedo`, and `pfnSceneIsDirty`.

## Assets and editor integration

The asset helpers `pfnAssetGetCount`, `pfnAssetGetName`, `pfnAssetGetType`,
and `pfnAssetExists` query assets through `pCtx->pAssets`.
`pfnEditorFocusEntity`, `pfnEditorSetStatusMessage`, and
`pfnEditorOpenAsset` provide editor-level actions. Check that an asset exists
before spawning it if the name may be unavailable.

## Related docs

- [Plugin API](plugins.md)
- [Plugin Lifecycle](lifecycle.md)
- [UI System](ui.md)
- [Entities and Components](entities.md)
