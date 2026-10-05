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
`pfnUiSameLine`, `pfnUiInputText`, `pfnUiSliderInt`, `pfnUiDragFloat`,
`pfnUiDragInt`, `pfnUiCombo`, `pfnUiProgressBar`, `pfnUiSetTooltip`,
`pfnUiCollapsingHeader`, `pfnUiSelectable`, `pfnUiInputInt`,
`pfnUiInputTextMultiline`, `pfnUiColorEdit4`, `pfnUiRadioButton`,
`pfnUiTextWrapped`, and `pfnUiBulletText`.

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

`pfnUiInputText` edits a caller-owned, null-terminated UTF-8 buffer. Pass its
capacity in bytes, including space for the final `'\0'`. `pfnUiCombo` accepts
an array of labels and an in-range selected index. `pfnUiProgressBar` expects a
fraction from `0.0` to `1.0`; its overlay may be null. `pfnUiSetTooltip`
attaches a tooltip to the most recently submitted item.

`pfnUiInputTextMultiline` edits a caller-owned writable buffer. Its capacity is
in bytes and includes the terminating null character. `pfnUiColorEdit4` edits
four normalized RGBA channels. Radio buttons that share an integer pointer
make a mutually exclusive selection. `pfnUiInputFloat2/3/4`,
`pfnUiInputInt2/3/4`, `pfnUiDragFloat2/3/4`, and `pfnUiColorPicker4` provide
multi-component editing and color picking. Each array must contain at least
the documented number of writable elements.

```cpp
static int s_Mode = 0;

static void DrawMode(SPluginContext* pCtx)
{
    pCtx->pfnUiRadioButton("Translate", &s_Mode, 0);
    pCtx->pfnUiRadioButton("Rotate", &s_Mode, 1);
    pCtx->pfnUiRadioButton("Scale", &s_Mode, 2);
}
```

## Layout helpers

The API also provides collapsible sections, scrollable child regions, tabs,
and disabled sections. Begin/end calls must be paired according to their
return values:

```cpp
if (pCtx->pfnUiCollapsingHeader("Advanced"))
{
    if (pCtx->pfnUiBeginChild("advanced_options", 0.0f, 180.0f))
    {
        pCtx->pfnUiText("Scrollable content");
    }
    pCtx->pfnUiEndChild();
}

if (pCtx->pfnUiBeginTabBar("settings_tabs"))
{
    if (pCtx->pfnUiBeginTabItem("General"))
    {
        pCtx->pfnUiBeginDisabled(false);
        pCtx->pfnUiText("General settings");
        pCtx->pfnUiEndDisabled();
        pCtx->pfnUiEndTabItem();
    }
    pCtx->pfnUiEndTabBar();
}
```

Always call `pfnUiEndChild()` after `pfnUiBeginChild()`, even when it returns
false. Call `pfnUiEndTabBar()` only after a successful tab-bar begin, and
`pfnUiEndTabItem()` only after a successful tab-item begin. Disabled scopes
must always be paired.

The layout helpers `pfnUiPushID`/`pfnUiPopID` distinguish repeated controls,
for example controls rendered in a loop. Pop every pushed ID before the
callback returns. `pfnUiSetNextItemWidth` applies only to the next widget;
`pfnUiSpacing`, `pfnUiIndent`, and `pfnUiUnindent` adjust the current layout.
`pfnUiSetNextWindowSize` and `pfnUiSetNextWindowPos` must be called before
`pfnUiBegin`; the host applies these initial values the first time the window
identifier is used. `pfnUiSetNextWindowCollapsed` requests the initial
collapsed state under the same condition. `pfnUiSetNextWindowFocus` requests
focus for the next window submission.

`pfnUiTreeNode` creates an expandable hierarchy entry. Pair every successful
call with `pfnUiTreePop()`:

```cpp
if (pCtx->pfnUiTreeNode("Renderer"))
{
    pCtx->pfnUiText("Active renderer settings");
    pCtx->pfnUiTreePop();
}
```

`pfnUiListBox` provides a fixed-label list selector. For a custom combo
contents layout, use `pfnUiBeginCombo`/`pfnUiEndCombo` and submit selectable
items yourself. Pair `EndCombo` only with a successful begin.

The item-state queries `pfnUiIsItemHovered`, `pfnUiIsItemActive`,
`pfnUiIsItemFocused`, `pfnUiIsItemClicked`, and `pfnUiIsItemVisible` inspect
the most recently submitted item. Call them immediately after that item,
before submitting another widget.

## Popups

Open a popup before beginning it, and submit its contents only when begin
returns true. End every successfully begun popup:

```cpp
if (pCtx->pfnUiButton("Delete"))
{
    pCtx->pfnUiOpenPopup("confirm_delete");
}

if (pCtx->pfnUiBeginPopupModal("confirm_delete", nullptr))
{
    pCtx->pfnUiTextWrapped("This action cannot be undone.");
    if (pCtx->pfnUiButton("Confirm"))
    {
        // Perform the deletion.
        pCtx->pfnUiCloseCurrentPopup();
    }
    pCtx->pfnUiSameLine();
    if (pCtx->pfnUiButton("Cancel"))
    {
        pCtx->pfnUiCloseCurrentPopup();
    }
    pCtx->pfnUiEndPopup();
}
```

`pfnUiBeginPopup` starts a non-modal popup. `pfnUiBeginPopupModal` starts a
modal popup that blocks interaction with the rest of the UI. The optional
`pOpen` pointer lets the host update plugin-owned open state when the modal's
close control is used.

`pfnUiBeginPopupContextItem` and `pfnUiBeginPopupContextWindow` open a popup
from a right-click on the last item or current window. They use the same
`pfnUiEndPopup` pairing rule. For hover tooltips with custom widgets rather
than a single string, use `pfnUiBeginTooltip`/`pfnUiEndTooltip`.

## Menus

Use `pfnUiBeginWithMenuBar` instead of `pfnUiBegin` when the window needs its
own menu bar. Pair a successful `pfnUiBeginMenuBar` with
`pfnUiEndMenuBar`. `pfnUiMenuItemEx` adds optional shortcut display text,
enabled state, and a plugin-owned selected/toggle value. Shortcut text is
display-only; it does not register a keyboard binding.

## Asset images

Images are drawn from textures already loaded in the host asset library. No
renderer texture handle is exposed to plugins. Enumerate textures with
`pfnAssetGetTextureCount` and `pfnAssetGetTextureName`, then pass an index to
`pfnUiImage` or `pfnUiImageButton`:

```cpp
for (int index = 0; index < pCtx->pfnAssetGetTextureCount(pCtx->pAssets); ++index)
{
    const char* pName = pCtx->pfnAssetGetTextureName(pCtx->pAssets, index);
    if (pName != nullptr)
    {
        pCtx->pfnUiText(pName);
        pCtx->pfnUiImage(pCtx->pAssets, index, 96.0f, 64.0f);
    }
}
```

Texture names are host-owned and texture indices may change when assets are
refreshed. Re-enumerate after a refresh; do not retain the returned name
pointers.

## Drag and drop

Drag sources and targets attach to the most recently submitted item. Use the
same payload type string on both ends. The target copies delivered bytes into a
plugin-owned buffer, so the API never exposes ImGui's temporary payload
pointer:

```cpp
static int s_DraggedValue = 0;
static int s_DroppedValue = 0;

static void DrawDragDrop(SPluginContext* pCtx)
{
    pCtx->pfnUiButton("Drag this value");
    if (pCtx->pfnUiBeginDragDropSource())
    {
        pCtx->pfnUiSetDragDropPayload("EXAMPLE_INT", &s_DraggedValue,
                                      sizeof(s_DraggedValue));
        pCtx->pfnUiText("Example integer");
        pCtx->pfnUiEndDragDropSource();
    }

    pCtx->pfnUiButton("Drop here");
    if (pCtx->pfnUiBeginDragDropTarget())
    {
        size_t payloadSize = 0;
        if (pCtx->pfnUiAcceptDragDropPayload("EXAMPLE_INT", &s_DroppedValue,
                                             sizeof(s_DroppedValue), &payloadSize) &&
            payloadSize == sizeof(s_DroppedValue))
        {
            // Use the received integer.
        }
        pCtx->pfnUiEndDragDropTarget();
    }
}
```

Every successful source/target begin must be paired with its end call. Payload
bytes are copied only when the drag is delivered; check the reported size and
validate the payload before interpreting it. Never send pointers to temporary
or plugin-private objects whose representation is not agreed upon by both
sides.

## Advanced table controls

Sortable tables support multiple sort keys; hold Shift while selecting
additional column headers. `pfnUiTableGetSortSpecs` copies the active key list
into caller-provided arrays. Its output count reports the total keys, even if
the arrays have smaller capacity. Apply the copied keys in order and clear the
dirty flag after handling the change. Use `pfnUiTableSetupScrollFreeze` before
submitting rows to keep leading rows or columns visible while scrolling.
`pfnUiTableSetColumnEnabled` and `pfnUiTableIsColumnVisible` control/query
column visibility inside a table.

## Style scopes

The style functions support a stable semantic subset of host color and style
slots. They do not expose raw ImGui enum values. Pair every color/style push
with its corresponding pop, and restore the stack before leaving the UI
callback:

```cpp
pCtx->pfnUiPushStyleColor(PLUGIN_UI_COLOR_BUTTON, 0.2f, 0.45f, 0.8f, 1.0f);
pCtx->pfnUiPushStyleVarFloat(PLUGIN_UI_STYLE_FRAME_ROUNDING, 5.0f);
pCtx->pfnUiButton("Styled");
pCtx->pfnUiPopStyleVar(1);
pCtx->pfnUiPopStyleColor(1);
```

Use `pfnUiPushStyleVarFloat` for scalar slots and
`pfnUiPushStyleVarVec2` for padding/spacing slots. The semantic identifiers are
declared by `EPluginUiColor` and `EPluginUiStyleVar` in `plugin.h`.

## Custom drawing

The draw helpers add primitives to the current window's draw list without
exposing an ImGui draw-list pointer. Coordinates are screen-space UI
coordinates; get the current content origin with `pfnUiGetCursorScreenPos`.
Drawing is clipped by the current window:

```cpp
float x = 0.0f;
float y = 0.0f;
pCtx->pfnUiGetCursorScreenPos(&x, &y);
pCtx->pfnUiDrawLine(x, y, x + 100.0f, y + 40.0f,
                    1.0f, 0.2f, 0.2f, 1.0f, 2.0f);
pCtx->pfnUiDrawCircleFilled(x + 50.0f, y + 20.0f, 6.0f,
                            0.2f, 0.8f, 0.3f, 1.0f);
```

These helpers draw overlays; they do not create layout items or automatically
reserve content space. Use a child region or other layout widget when space
needs to be reserved. `pfnUiGetItemRect` returns the last widget's screen-space
bounds for aligning custom drawing. `pfnUiPlotHistogram` takes the same
sample-array and axis-bound arguments as `pfnUiPlotLines`.

## Extended widgets and window options

The extended controls expose additional common ImGui widgets without
publishing ImGui types or enum values in the plugin ABI. Use
`pfnUiBeginEx` when a window needs a close state or specific window behavior.
Its `options` argument is a bitwise combination of `EPluginUiWindowOption`
values:

```cpp
static bool s_IsOpen = true;

static void DrawAdvancedWindow(SPluginContext* pCtx)
{
    const int windowOptions = PLUGIN_UI_WINDOW_NO_RESIZE |
                              PLUGIN_UI_WINDOW_MENU_BAR;
    if (pCtx->pfnUiBeginEx("Advanced", &s_IsOpen, windowOptions))
    {
        pCtx->pfnUiTextColored("Status: ready", 0.3f, 0.9f, 0.4f, 1.0f);
        pCtx->pfnUiSeparatorText("Controls");
        if (pCtx->pfnUiBeginChildEx("details", 0.0f, 120.0f, true))
        {
            pCtx->pfnUiText("Child content");
        }
        pCtx->pfnUiEndChild();
    }
    pCtx->pfnUiEnd();
}
```

Always call `pfnUiEnd` after `pfnUiBeginEx`, including when the begin returns
false. Likewise, pair `pfnUiEndChild` with every `pfnUiBeginChildEx` call.
For each child dimension, a negative value requests alignment relative to the
far edge of the available content region, zero uses the remaining space, and a
positive value requests an explicit size.
Available window options include hiding the title bar, resizing, moving,
scrollbar, collapse control, or background, disabling saved settings, enabling
a menu bar, and suppressing automatic focus raising.

Additional controls include compact and invisible buttons
(`pfnUiSmallButton`, `pfnUiInvisibleButton`), directional arrow buttons,
flag checkboxes, explicit-state radio buttons, range drags, multi-component
drag/slider controls, angle and vertical sliders, double input, hinted text
input, and a three-channel color picker. `pfnUiDragFloatN`,
`pfnUiSliderFloatN`, and `pfnUiSliderIntN` accept component counts of 2, 3, or
4 and require a writable array of at least that length. The range drag helpers
take separate writable pointers for their lower and upper endpoints.

`pfnUiTreeNodeEx` accepts a bitwise combination of `EPluginUiTreeNodeOption`
values (`PLUGIN_UI_TREE_SELECTED`, `PLUGIN_UI_TREE_DEFAULT_OPEN`,
`PLUGIN_UI_TREE_LEAF`, `PLUGIN_UI_TREE_BULLET`, `PLUGIN_UI_TREE_FRAMED`, and
`PLUGIN_UI_TREE_SPAN_AVAILABLE`). Pair every successful result with
`pfnUiTreePop`. `pfnUiSelectableEx` can enable double-click activation and
extend the selectable across table columns.

`pfnUiGetWindowRect` and `pfnUiGetContentRegionAvail` write position, size, or
available-region dimensions into caller-owned output pointers.
`pfnUiIsWindowAppearing` and `pfnUiIsWindowCollapsed` query the current window.
`pfnUiIsItemEdited`, `pfnUiIsItemActivated`, `pfnUiIsItemDeactivated`,
`pfnUiIsItemDeactivatedAfterEdit`, and `pfnUiGetItemRectSize` query the most
recently submitted widget; call them before submitting another item.

The plugin API intentionally does not expose ImGui context or frame lifecycle
management, renderer/backend functions, raw draw-list or viewport pointers,
callback-based size constraints, or raw input ownership and platform-I/O
interfaces. Those are owned by the host and crossing that boundary would
couple plugins to backend state or version-specific ImGui types.

## Window, layout, table, and input helpers

The extended API also provides one-shot window setup (`pfnUiSetNextWindowPosEx`,
`pfnUiSetNextWindowSizeEx`, `pfnUiSetNextWindowCollapsed`,
`pfnUiSetNextWindowFocus`, content-size, size-constraint, scroll, and
background-opacity helpers). Conditions use `EPluginUiCondition`, whose
values correspond to always, once, first use, and appearing. Call next-window
helpers before `pfnUiBegin`/`pfnUiBeginEx`; the condition-aware collapsed
variant is `pfnUiSetNextWindowCollapsedEx`. Current-window position, size,
collapse, focus, DPI scale, and scroll queries are available separately.

Cursor positioning is available in screen or window-local coordinates.
`pfnUiDummy`, `pfnUiNewLine`, `pfnUiBeginGroup`/`pfnUiEndGroup`,
`pfnUiAlignTextToFramePadding`, and `pfnUiGetLayoutMetrics` provide the
remaining common layout operations. Pair group scopes before returning from a
plugin UI callback. `pfnUiSameLineEx` accepts explicit offset and spacing.

For tree controls, `pfnUiSetNextItemOpen` sets the next node's open state;
`pfnUiCollapsingHeaderVisible` adds a plugin-owned close state, and
`pfnUiTreeNodeIsOpen` queries an existing node. For popups,
`pfnUiOpenPopupOnItemClick`, `pfnUiBeginPopupContextVoid`, and
`pfnUiIsPopupOpen` cover item/empty-area context menus and popup state. Every
successful popup begin still requires `pfnUiEndPopup`.

Table helpers can submit individual or angled headers, select a specific
column, query row/column indices, configure a column's initial width, and set
row/cell backgrounds using `EPluginUiTableColorTarget`. Legacy columns are
available for compatibility, but new plugin UI should prefer tables.
`pfnUiTabItemButton` and `pfnUiSetTabItemClosed` supplement the normal tab
bar/item helpers.

The main-screen menu bar has a separate
`pfnUiBeginMainMenuBar`/`pfnUiEndMainMenuBar` pair. `pfnUiBullet` and the
`pfnUiValueBool`, `pfnUiValueInt`, `pfnUiValueUInt`, and `pfnUiValueFloat`
helpers provide ImGui's compact display-only value controls.

Font-size and item-layout scopes are available as
`pfnUiPushFontSize`/`pfnUiPopFont` and
`pfnUiPushItemWidth`/`pfnUiPopItemWidth`. `pfnUiPushTextWrapPos` must be paired
with `pfnUiPopTextWrapPos`. The host's font pointer is never exposed to
plugins. `pfnUiImageWithBg` displays an asset-library texture with explicit
background and tint colors without exposing renderer texture handles.

For numeric representations beyond built-in `int`, `float`, and `double`
widgets, use `pfnUiDragScalar`, `pfnUiSliderScalar`,
`pfnUiInputScalar`, or `pfnUiVSliderScalar` with an `EPluginUiScalarType`.
The value, bound, and step pointers must point to correctly typed plugin-owned
storage and remain valid for the synchronous call. `pfnUiColorPackRgba` and
`pfnUiColorUnpackRgba` convert normalized RGBA values to/from the packed color
representation used by ImGui drawing helpers.

Input queries use stable `EPluginUiKey` and `EPluginUiMouseButton` values,
rather than ImGui numeric enums. `pfnUiGetKeyState`, `pfnUiGetKeyPressedAmount`,
`pfnUiGetKeyName`, and `pfnUiShortcut` provide common keyboard operations.
Mouse helpers query button state, click count, double-click/drag state,
position, popup-opening position, and screen-rectangle hovering. These are
observational queries; input ownership, backend event submission, and global
capture configuration remain host responsibilities.

`pfnUiPushClipRect`/`pfnUiPopClipRect` provide scoped UI clipping;
`pfnUiSetNextItemAllowOverlap` and `pfnUiSetItemDefaultFocus` support layout
and navigation. The utilities also include literal text measurement, RGB/HSV
conversion, frame/time queries, and configurable-size line/histogram plots.
`pfnUiTextLink` adds a clickable hyperlink-style label without allowing a
plugin to launch arbitrary external URLs. Mouse drag deltas can be queried or
reset using `pfnUiGetMouseDragDelta` and `pfnUiResetMouseDragDelta`; plugins
may request a standard cursor shape through `pfnUiSetMouseCursor`.
Logging helpers write literal text only to an active host-managed ImGui log;
the plugin API does not allow selecting arbitrary log files or writing to the
system clipboard.

When the host has enabled Dear ImGui docking, `pfnUiDockSpace` creates a
dockspace in the current plugin window. `pfnUiSetNextWindowDockId` accepts a
dock ID returned by that helper; docking node flags, window classes, and
viewport pointers are not exposed.

The linked ImGui build exposes its metrics, debug log, ID stack, and font
selector windows through the corresponding `pfnUiShow...` helpers.
`pfnUiSetBuiltinStyle` operates on the host's shared style and may affect all
editor windows; prefer scoped style overrides unless a global theme change is
intentional. `pfnUiGetVersion` copies the library version into plugin-owned
storage. Demo/about/style-editor/user-guide helpers are not included in the
host's linked ImGui build.

## Tables

Tables support resizable/reorderable columns and optional sorting. Declare all
columns before submitting headers or rows. Advance to each cell with
`pfnUiTableNextColumn()`:

```cpp
if (pCtx->pfnUiBeginTable("asset_table", 2, true))
{
    pCtx->pfnUiTableSetupColumn("Name", true);
    pCtx->pfnUiTableSetupColumn("Type", false);
    pCtx->pfnUiTableHeadersRow();

    int sortColumn = -1;
    int sortDirection = 0;
    bool sortDirty = false;
    if (pCtx->pfnUiTableGetSortSpec(&sortColumn, &sortDirection, &sortDirty) &&
        sortDirty)
    {
        // Sort plugin-owned data by sortColumn. Direction: 1 ascending, 2 descending.
        pCtx->pfnUiTableClearSortDirty();
    }

    pCtx->pfnUiTableNextRow();
    pCtx->pfnUiTableNextColumn();
    pCtx->pfnUiText("Example asset");
    pCtx->pfnUiTableNextColumn();
    pCtx->pfnUiText("Model");
    pCtx->pfnUiEndTable();
}
```

Only call table functions after `pfnUiBeginTable()` returns true, and always
pair it with `pfnUiEndTable()`. A column can be marked non-sortable even when
the table itself enables sorting.

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
