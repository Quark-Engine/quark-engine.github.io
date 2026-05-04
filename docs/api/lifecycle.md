# Жизненный цикл плагина

Quark Engine вызывает четыре колбэка в течение жизни плагина. Все они опциональны — передайте `nullptr` если колбэк не нужен.

## on\_load

Вызывается один раз при загрузке библиотеки. Используйте для инициализации ресурсов: выделения памяти, загрузки файлов конфигурации, регистрации команд.

## on\_update

Вызывается каждый кадр перед рендерингом. Доступен `delta_time` для вычислений, не привязанных к частоте кадров.

```asm
; void on_update(PluginContext* ctx)
on_update PROC
    ; RCX = ctx

    ; float dt = ctx->delta_time;
    movss xmm0, dword ptr [rcx]

    ; // логика движения, физики и т.д.

    ret
on_update ENDP
```

## on\_draw\_ui

Специальный проход для отрисовки интерфейса плагина через встроенные функции UI. Вызывается после `on_update`.

```asm
EXTERN ui_begin:PROC
EXTERN ui_button:PROC
EXTERN ui_end:PROC

.data
ui_title db "Мой плагин",0
btn_text db "Сбросить",0

.code

; void on_draw_ui(PluginContext* ctx)
on_draw_ui PROC
    ; RCX = ctx

    ; ui_begin("Мой плагин");
    lea rcx, ui_title
    call ui_begin

    ; if (ui_button("Сбросить")) {
    lea rcx, btn_text
    call ui_button

    test eax, eax
    jz skip_button

    ; // обработка нажатия

skip_button:

    ; ui_end();
    call ui_end

    ret
on_draw_ui ENDP
```

## on\_unload

Вызывается перед выгрузкой плагина. Обязательно освободите всю выделенную память и закройте открытые ресурсы.

```asm
EXTERN free:PROC

.data
my_buffer dq 0

.code

; void on_unload(PluginContext* ctx)
on_unload PROC
    ; RCX = ctx

    ; // освобождение ресурсов
    ; free(my_buffer);

    mov rcx, my_buffer
    test rcx, rcx
    jz skip_free

    call free

skip_free:

    ret
on_unload ENDP
```
