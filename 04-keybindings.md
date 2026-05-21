# Ghostty Keybindings

Source: https://ghostty.org/docs/config/keybind/reference

## Syntax

```ini
keybind = trigger=action
```

Trigger pieces, joined with `+`:

- Modifiers: `shift`, `ctrl`, `alt`, `super`
- Logical key: `a`, `KeyA` (physical), or a Unicode codepoint
- Sequences (chords): `ctrl+a>b=action`
- Trigger prefixes:
  - `global:` — match even when Ghostty isn't focused (platform-dependent)
  - `all:` — match in all surfaces, including read-only
  - `unconsumed:` — match but still forward the key
  - `performable:` — only match when the action would actually do something
- Special: `keybind = clear` removes **all** bindings (then redefine your own).

## Key tables (modal bindings)

```ini
keybind = resize/arrow_up=resize_split:up,10
keybind = resize/arrow_down=resize_split:down,10
keybind = resize/escape=deactivate_key_table
keybind = resize/catch_all=ignore
keybind = ctrl+a=activate_key_table:resize
```

Lookup precedence is innermost-table-outward.

## Common actions

- **Input:** `ignore`, `unbind`, `csi:...`, `esc:...`, `text:...`, `cursor_key:...`
- **Clipboard:** `copy_to_clipboard`, `paste_from_clipboard`, `paste_from_selection`, `copy_url_to_clipboard`, `copy_title_to_clipboard`
- **Font:** `increase_font_size:1`, `decrease_font_size:1`, `reset_font_size`, `set_font_size:14`
- **Search:** `search`, `search_selection`, `start_search`, `navigate_search:next|previous`, `end_search`
- **Scroll/select:** page- and line-based scrolling, viewport selection adjustments
- **Tabs/windows:** `new_window`, `new_tab`, `close_surface`, `goto_tab:N`, `previous_tab`, `next_tab`, `move_tab:+1|-1`
- **Splits:** `new_split:right|down|left|up|auto`, `goto_split:...`, `resize_split:...`, `toggle_split_zoom`
- **Misc:** `toggle_fullscreen`, `toggle_quick_terminal`, `toggle_window_decorations`, `toggle_mouse_reporting`, `inspector:toggle`, `reload_config`, `undo`, `redo`

Run `ghostty +list-actions` for the full local set.

## Examples

```ini
keybind = ctrl+shift+t=new_tab
keybind = ctrl+shift+w=close_surface
keybind = ctrl+shift+enter=new_split:right
keybind = ctrl+shift+f=toggle_fullscreen
keybind = ctrl+plus=increase_font_size:1
keybind = ctrl+minus=decrease_font_size:1
keybind = ctrl+0=reset_font_size
```
