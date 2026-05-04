# Функциональные возможности

## Интерфейс (UI)

Движок предоставляет Immediate-mode UI систему. Все вызовы должны находиться внутри колбэка `on_draw_ui`.

| Функция | Описание |
|---|---|
| `ui_begin(title)` / `ui_end()` | Открыть / закрыть окно |
| `ui_button(label)` | Кнопка, возвращает `true` при нажатии |
| `ui_slider_float(label, &val, min, max)` | Слайдер для вещественных чисел |
| `ui_color_edit3(label, color[3])` | Выбор цвета в формате RGB |

```asm
EXTERN ui_begin:PROC
EXTERN ui_slider_float:PROC
EXTERN ui_color_edit3:PROC
EXTERN ui_end:PROC

.data
    ui_title  db "Настройки", 0
    lbl_speed db "Скорость", 0
    lbl_color db "Цвет", 0
    
    speed_val dd 1.0
    speed_min dd 0.0
    speed_max dd 10.0
    
    color_vec dd 1.0, 0.0, 0.0  ; RGB (Red)

.code
on_draw_ui PROC
    sub rsp, 32                 ; Shadow space для вызова функций

    lea rcx, ui_title
    call ui_begin

    lea rcx, lbl_speed
    lea rdx, speed_val
    movss xmm2, [speed_min]
    movss xmm3, [speed_max]
    call ui_slider_float

    lea rcx, lbl_color
    lea rdx, color_vec
    call ui_color_edit3

    call ui_end

    add rsp, 32
    ret
on_draw_ui ENDP
```

## Работа с сущностями

Полный контроль над объектами в сцене через `PluginContext`.

### Чтение

```asm
; RCX = ctx, RDX = entity_id
call entity_get_position
; Результат (Vec3) обычно возвращается через XMM0-XMM2 или по указателю в RAX

; RCX = ctx, RDX = entity_id
call entity_get_name
; RAX = указатель на строку (const char*)
```

### Мутация

```asm
EXTERN entity_set_scale
EXTERN entity_set_rotation

section .data
    scale_vec   dd 2.0, 2.0, 2.0
    rot_vec     dd 0.0, 90.0, 0.0

section .text

; RCX = ctx
; RDX = entity_id
mutate_entity:
    sub rsp, 32

    ; entity_set_scale(ctx, entity_id, &scale_vec)
    lea r8, [rel scale_vec]   ; третий аргумент
    call entity_set_scale

    ; entity_set_rotation(ctx, entity_id, &rot_vec)
    lea r8, [rel rot_vec]
    call entity_set_rotation

    add rsp, 32
    ret
```

### Управление сценой

```asm
EXTERN scene_spawn
EXTERN scene_delete

section .data
    cube_name db "cube", 0

section .bss
    new_entity resq 1

section .text

; RCX = ctx
spawn_and_delete:
    sub rsp, 32

    ; new_entity = scene_spawn(ctx, "cube")
    lea rdx, [rel cube_name]
    call scene_spawn
    mov [rel new_entity], rax

    ; scene_delete(ctx, entity_id)
    ; предполагаем, что entity_id уже в RDX
    call scene_delete

    add rsp, 32
    ret
```

> **Внимание:** Не сохраняйте указатель `PluginContext` между вызовами функций. Движок может пересоздавать контекст в каждом кадре.
