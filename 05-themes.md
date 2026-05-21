# Ghostty Themes

Source: https://ghostty.org/docs/features/theme

## How themes work

A theme is just a config file that mostly sets color options. Themes load **before** your config, so anything you set in your own config overrides the theme.

## Built-in themes

Ghostty ships hundreds of themes from [iterm2-color-schemes](https://iterm2colorschemes.com), refreshed weekly.

```sh
ghostty +list-themes
```

Use one:

```ini
theme = Catppuccin Frappe
```

## Light / dark switching

```ini
theme = dark:Catppuccin Frappe,light:Catppuccin Latte
```

Ghostty follows the OS appearance setting.

## Custom theme files

Place a file in `$XDG_CONFIG_HOME/ghostty/themes/<name>` (or `$PREFIX/share/ghostty/themes/<name>`) and reference it by name, or use an absolute path.

Example custom theme:

```ini
palette = 0=#51576d
palette = 1=#e78284
palette = 2=#a6d189
palette = 3=#e5c890
palette = 4=#8caaee
palette = 5=#f4b8e4
palette = 6=#81c8be
palette = 7=#a5adce
palette = 8=#626880
palette = 9=#e67172
palette = 10=#8ec772
palette = 11=#d9ba73
palette = 12=#7b9ef0
palette = 13=#f2a4db
palette = 14=#5abfb5
palette = 15=#b5bfe2
background = #303446
foreground = #c6d0f5
cursor-color = #f2d5cf
cursor-text = #c6d0f5
selection-background = #626880
selection-foreground = #c6d0f5
```

A custom theme file can set anything except `theme` and `config-file`. Review themes from untrusted sources before using.
