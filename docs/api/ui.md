# UI System

Plugins draw UI through the function pointers in `SPluginContext`, declared in
`include/plugins/plugin.h`. UI functions are intended for use during
`pfnOnDrawUI` or a registered UI-region callback.

## Plugin window

Every `pfnUiBegin()` must be paired with `pfnUiEnd()`, including when begin
returns `false`.

```cpp
static void OnDrawUI(SPluginContext* pCtx)
{
    if (pCtx->pfnUiBegin("My Plugin"))
    {
        pCtx->pfnUiText("Hello from my plugin.");
        if (pCtx->pfnUiButton("Reset"))
        {
            // Perform the action.
        }
    }
    pCtx->pfnUiEnd();
}
```

```cpp
bool (*pfnUiBegin)(const char* pTitle);
void (*pfnUiEnd)();
```

## Widgets

The available widgets are `pfnUiText`, `pfnUiButton`, `pfnUiCheckbox`,
`pfnUiSliderFloat`, `pfnUiInputFloat`, `pfnUiColorEdit3`, `pfnUiSeparator`,
and `pfnUiSameLine`.

```cpp
static bool s_ShowDetails = false;
static float s_Speed = 1.0f;
static float s_Color[3] = { 1.0f, 1.0f, 1.0f };

static void DrawSettings(SPluginContext* pCtx)
{
    pCtx->pfnUiText("Settings");
    pCtx->pfnUiCheckbox("Show details", &s_ShowDetails);
    pCtx->pfnUiSliderFloat("Speed", &s_Speed, 0.0f, 10.0f);
    pCtx->pfnUiInputFloat("Speed value", &s_Speed);
    pCtx->pfnUiColorEdit3("Tint", s_Color);
    pCtx->pfnUiSeparator();
    pCtx->pfnUiButton("Apply");
}
```

Buttons and checkboxes return `true` when activated or changed. Slider and
input functions return whether their value changed. Color edit takes three
floats in the `[0.0, 1.0]` range. The UI helper functions operate inside the
current host UI context; call them from a drawing callback.

## Menus and editor regions

Register an `FPluginUICallback` with `pfnRegisterUICallback` (usually from
`pfnOnLoad`) to draw when the host renders a particular `EUIRegion`.

```cpp
static void DrawFileMenu(SPluginContext* pCtx)
{
    if (pCtx->pfnUiBeginMenu("Export"))
    {
        if (pCtx->pfnUiMenuItem("Export as JSON"))
        {
            // Handle the menu action.
        }
        pCtx->pfnUiEndMenu();
    }
}

static void OnLoad(SPluginContext* pCtx)
{
    pCtx->pfnRegisterUICallback(pCtx, UI_MENU_FILE, DrawFileMenu);
}
```

| Region | Location |
|---|---|
| `UI_MENU_FILE` | File menu |
| `UI_MENU_EDIT` | Edit menu |
| `UI_MENU_HELP` | Help menu |
| `UI_HIERARCHY` | Scene hierarchy |
| `UI_INSPECTOR` | Inspector |
| `UI_SCENE` | Scene viewport |

`pfnUiBeginMenu`, `pfnUiMenuItem`, and `pfnUiEndMenu` are the menu helpers.
They should be used while drawing an appropriate menu region. The host removes
registered callbacks when the plugin is unloaded.

## Related docs

- [Plugin API](plugins.md)
- [Plugin Lifecycle](lifecycle.md)
- [Scene Management](scene.md)
- [Entities and Components](entities.md)
