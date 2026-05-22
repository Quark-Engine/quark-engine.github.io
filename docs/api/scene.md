# Scene Management

The scene API lets plugins save the current scene, spawn new entities from assets, and delete existing ones. All functions are available as function pointers on `PluginContext`.

## The Scene Object

`ctx->scene` is a pointer to the current scene state owned by the host. It is available for the duration of the callback but should not be stored beyond it.

```cpp
Scene* scene;
```

## Functions

### Save

Serializes the current scene to disk using the host's default save path. Equivalent to the user pressing Ctrl-S.

```cpp
ctx->scene_save();
```

```cpp
void (*scene_save)();
```

### Spawn

Instantiates an asset by name and adds it to the scene. Returns the new entity's index on success, or `-1` if the asset name is not found in the host's asset registry.

```cpp
int index = ctx->scene_spawn("crate_01");
if (index >= 0) {
    ctx->entity_set_position(index, 0.0f, 0.0f, 0.0f);
}
```

```cpp
int (*scene_spawn)(const char* asset_name);
```

### Delete

Permanently removes an entity from the scene by index.

```cpp
ctx->scene_delete(index);
```

```cpp
void (*scene_delete)(int index);
```

> **Warning:** After `scene_delete()`, indices for all entities above the deleted index may shift. Treat any cached index as invalid and re-query `entity_count` before accessing entities again.

## Common Patterns

### Spawn and configure in one step

```cpp
int index = ctx->scene_spawn("point_light");
if (index >= 0) {
    ctx->entity_set_position(index, 0.0f, 5.0f, 0.0f);
    ctx->entity_set_name(index, "Key Light");
}
```

### Delete the selected entity

```cpp
if (ctx->selected) {
    int i = *ctx->selected;
    ctx->scene_delete(i);
    // Do not dereference ctx->selected or use i after this point.
}
```

### Auto-save on a condition

```cpp
void on_update(PluginContext* ctx) {
    if (should_autosave()) {
        ctx->scene_save();
    }
}
```

## Related Docs

- [Plugin Overview](api/plugins.md)
- [Plugin Lifecycle](api/lifecycle.md)
- [UI System](api/ui.md)
- [Entities](api/entities.md)