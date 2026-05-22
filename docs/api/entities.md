# Entities

Entities are the objects that make up a scene. Each one is identified by an integer index in the range `[0, entity_count)`. Use the read functions to inspect state and the write functions to modify it.

> **Note:** Entity indices can shift after a `scene_delete()` call. Never cache an index across frames — always re-query if needed.

## Reading Entity State

### Name

Returns a pointer to the entity's name string. The string is owned by the host — do not free or modify it.

```cpp
const char* name = ctx->entity_get_name(index);
ctx->ui_text(name);
```

```cpp
const char* (*entity_get_name)(int index);
```

### Position

Writes the entity's world-space position into three output floats.

```cpp
float x, y, z;
ctx->entity_get_position(index, &x, &y, &z);
```

```cpp
void (*entity_get_position)(int index, float* x, float* y, float* z);
```

### Rotation

Writes the entity's Euler rotation into three output floats. The unit (degrees or radians) is defined by the host.

```cpp
float rx, ry, rz;
ctx->entity_get_rotation(index, &rx, &ry, &rz);
```

```cpp
void (*entity_get_rotation)(int index, float* x, float* y, float* z);
```

### Scale

Writes the entity's scale into three output floats.

```cpp
float sx, sy, sz;
ctx->entity_get_scale(index, &sx, &sy, &sz);
```

```cpp
void (*entity_get_scale)(int index, float* x, float* y, float* z);
```

### Color

Writes the entity's RGBA tint into four output bytes in the range `[0, 255]`.

```cpp
unsigned char r, g, b, a;
ctx->entity_get_color(index, &r, &g, &b, &a);
```

```cpp
void (*entity_get_color)(int index, unsigned char* r, unsigned char* g,
                         unsigned char* b, unsigned char* a);
```

## Modifying Entity State

### Position

```cpp
ctx->entity_set_position(index, 0.0f, 1.0f, 0.0f);
```

```cpp
void (*entity_set_position)(int index, float x, float y, float z);
```

### Rotation

```cpp
ctx->entity_set_rotation(index, 0.0f, 90.0f, 0.0f);
```

```cpp
void (*entity_set_rotation)(int index, float x, float y, float z);
```

### Scale

```cpp
ctx->entity_set_scale(index, 2.0f, 2.0f, 2.0f);
```

```cpp
void (*entity_set_scale)(int index, float x, float y, float z);
```

### Color

```cpp
ctx->entity_set_color(index, 255, 128, 0, 255); // opaque orange
```

```cpp
void (*entity_set_color)(int index, unsigned char r, unsigned char g,
                         unsigned char b, unsigned char a);
```

### Name

The host copies the string internally, so the caller can free its buffer immediately after this returns.

```cpp
ctx->entity_set_name(index, "Player");
```

```cpp
void (*entity_set_name)(int index, const char* name);
```

## Iterating All Entities

Use `entity_count` from `PluginContext` to loop over every entity in the scene.

```cpp
for (int i = 0; i < ctx->entity_count; i++) {
    const char* name = ctx->entity_get_name(i);
    ctx->ui_text(name);
}
```

## Working with the Selected Entity

`ctx->selected` points to the index of the currently selected entity, or `nullptr` if nothing is selected. Always null-check before dereferencing.

```cpp
if (ctx->selected) {
    int i = *ctx->selected;
    float x, y, z;
    ctx->entity_get_position(i, &x, &y, &z);
    ctx->ui_input_float("X", &x);
    ctx->entity_set_position(i, x, y, z);
}
```

## Related Docs

- [Plugin Overview](api/plugins.md)
- [Plugin Lifecycle](api/lifecycle.md)
- [UI System](api/ui.md)
- [Scene Management](api/scene.md)