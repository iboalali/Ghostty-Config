# Ghostty Config Syntax

## File location

- Linux: `~/.config/ghostty/config` (or `~/.config/ghostty/config.ghostty`)
- macOS: same XDG path, plus the app reads it when launched

## Syntax

```ini
# Comments start with `#` and must be on their own line.
# Blank lines are ignored.

# key = value  (whitespace around `=` doesn't matter)
background = 282c34
foreground = ffffff

# Quoted or unquoted values both work.
font-family = "JetBrains Mono"
font-size = 14

# Empty value resets the option to its default.
font-family =

# Some keys (like `keybind`, `palette`, `env`, `font-feature`) accept
# multiple entries — list them on separate lines.
keybind = ctrl+shift+t=new_tab
keybind = ctrl+shift+w=close_surface
```

## Reload without restarting

```sh
ghostty +reload-config
# or
kill -USR1 <ghostty-pid>
```

## Including other files

Use `config-file = path/to/other` to split the config across files. Theme files are a special case loaded with `theme = <name>`.
