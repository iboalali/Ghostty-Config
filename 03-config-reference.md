# Ghostty Configuration Reference

Source: https://ghostty.org/docs/config/reference

Grouped by topic. Run `ghostty +show-config --default --docs` for the full machine-generated list with descriptions.

## Font

| Key | Notes |
|---|---|
| `font-family`, `font-family-bold`, `font-family-italic`, `font-family-bold-italic` | Multiple fallbacks supported. List with `ghostty +list-fonts`. |
| `font-size` | Points, non-integer allowed. Linux GTK scales with DPI. |
| `font-style`, `font-style-bold`, `font-style-italic`, `font-style-bold-italic` | Named font styles. `false` disables that style. |
| `font-synthetic-style` | `true`, `false`, `no-bold`, `no-italic`, `no-bold-italic`. |
| `font-feature` | OpenType features. `-calt` disables ligatures. |
| `font-variation` | Variable font axes, e.g. `wght=600`. |
| `freetype-load-flags` | Linux only: `hinting`, `force-autohint`, `monochrome`, `autohint`, `light` (prefix `no-` to disable). |

## Text rendering tweaks

| Key | Notes |
|---|---|
| `alpha-blending` | `native`, `linear`, `linear-corrected`. macOS default: `native`; others: `linear-corrected`. |
| `adjust-cell-width`, `adjust-cell-height` | Integer or percent. |
| `adjust-font-baseline`, `adjust-underline-position`, `adjust-strikethrough-position`, `adjust-overline-position` | Position tweaks. |
| `adjust-cursor-thickness`, `adjust-cursor-height` | Cursor metrics. |
| `grapheme-width-method` | `unicode` (default) or `legacy`. |

## Theme & colors

| Key | Notes |
|---|---|
| `theme` | Name or absolute path. `light:NAME,dark:NAME` for auto-switching. |
| `background`, `foreground` | Hex `#RRGGBB` or X11 color name. |
| `palette` | 16+ entries: `palette = 0=#51576d`. |
| `palette-generate`, `palette-harmonious` | Auto-derive cohesive palette. |
| `cursor-color`, `cursor-text`, `cursor-opacity` | Special values: `cell-foreground`, `cell-background`. |
| `cursor-style` | `block`, `bar`, `underline`, `block_hollow`. |
| `cursor-style-blink` | Respects DEC Mode 12 if unset. |
| `minimum-contrast` | WCAG 1–21 enforcement. |
| `selection-foreground`, `selection-background` | Selection colors. |

## Background image & transparency

| Key | Notes |
|---|---|
| `background-image` | PNG or JPEG path. |
| `background-image-opacity`, `background-image-position`, `background-image-fit`, `background-image-repeat` | Fit: `contain`, `cover`, `stretch`, `none`. |
| `background-opacity` | 0–1. macOS native fullscreen disables it; requires restart. |
| `background-opacity-cells` | Apply opacity to colored cells too. Default false. |
| `background-blur` | 0+ or `true` (=20). macOS 26.0+: `macos-glass-regular`, `macos-glass-clear`. Linux: KDE only. |
| `unfocused-split-opacity`, `unfocused-split-fill` | Dim inactive splits. |

## Window

| Key | Notes |
|---|---|
| `window-decoration` | `none`, `auto`, `client`, `server`. `true`=`auto`, `false`=`none`. |
| `window-theme` | `auto`, `system`, `light`, `dark`, `ghostty`. |
| `window-colorspace` | macOS only: `srgb` or `display-p3`. |
| `window-title-font-family` | System default if unset. |
| `window-subtitle` | GTK: `false` or `working-directory`. |
| `window-width`, `window-height` | Initial size in cells. Both required. Min 10×4. |
| `window-position-x`, `window-position-y` | macOS only, pixels from top-left. |
| `window-padding-x`, `window-padding-y` | Points. `left,right` / `top,bottom` form supported. |
| `window-padding-balance` | Distribute extra padding evenly. |
| `window-padding-color` | `background`, `extend`, `extend-always`. |
| `window-save-state` | macOS: `default`, `never`, `always`. |
| `window-step-resize` | macOS: resize in cell increments. |
| `window-new-tab-position` | `current`, `end`. |
| `window-show-tab-bar` | GTK: `always`, `auto`, `never`. |
| `maximize`, `fullscreen` | Startup state. Fullscreen: `true`, `non-native`, `non-native-visible-menu`, `non-native-padded-notch`. |
| `title` | Spaces hide the title. |
| `class` | GTK app id (X11/Wayland/DBus). |
| `x11-instance-name` | GTK X11 WM_CLASS instance. |

## Interaction & input

| Key | Notes |
|---|---|
| `keybind` | `trigger=action`. See `04-keybindings.md`. |
| `key-remap` | e.g. `ctrl=super`. One-way, non-transitive. |
| `cursor-click-to-move` | Needs shell integration + OSC 133. |
| `mouse-hide-while-typing` | Hide cursor during keystrokes. |
| `mouse-shift-capture` | `true`, `false`, `always`, `never`. Default `false` (shift extends selection). |
| `mouse-reporting` | Allow programs to capture mouse. |
| `mouse-scroll-multiplier` | Speed factor. Prefixes `precision:` / `discrete:`. |
| `scroll-to-bottom` | `keystroke`, `output`. Prefix `no-` to disable. |
| `focus-follows-mouse` | Within same window only. |
| `selection-clear-on-typing`, `selection-clear-on-copy` | Auto-clear behavior. |
| `selection-word-chars` | Word-boundary chars for selection. |

## Search & links

| Key | Notes |
|---|---|
| `search-foreground`, `search-background` | Non-active match colors. |
| `search-selected-foreground`, `search-selected-background` | Active match colors. |
| `link-url` | Enable URL detection. |
| `link-previews` | `true`, `false`, `osc8`. |

## Command & shell

| Key | Notes |
|---|---|
| `command` | Shell or program. Prefixes: `direct:`, `shell:`. |
| `initial-command` | First terminal only. |
| `env` | `env=KEY=VALUE`. Empty resets. |
| `input` | `raw:text` or `path:file`. |
| `wait-after-command` | Keep terminal open on exit. |
| `abnormal-command-exit-runtime` | Threshold (ms) for quick-exit error UI. |
| `working-directory` | `home`, `inherit`, or absolute path. |
| `window-inherit-working-directory`, `tab-inherit-working-directory`, `split-inherit-working-directory` | Inherit from previous surface. |
| `window-inherit-font-size` | New windows/tabs inherit current size. |
| `notify-on-command-finish` | `never`, `unfocused`, `always`. |
| `notify-on-command-finish-action` | `bell`, `notify`. Prefix `no-`. |
| `notify-on-command-finish-after` | e.g. `5s`, `1m30s`. |

## Scrollback & display

| Key | Notes |
|---|---|
| `scrollback-limit` | Bytes. Affects new terminals only. |
| `scrollbar` | `system`, `never`. |
| `resize-overlay` | `always`, `never`, `after-first`. |
| `resize-overlay-position` | 9-point grid. |
| `resize-overlay-duration` | e.g. `750ms`. |
| `window-vsync` | macOS only. |

## Clipboard

| Key | Notes |
|---|---|
| `clipboard-read`, `clipboard-write` | `ask`, `allow`, `deny`. Defaults: ask/allow. |
| `clipboard-trim-trailing-spaces` | Strip trailing spaces on copy. |
| `clipboard-paste-protection` | Warn on unsafe paste. |
| `clipboard-paste-bracketed-safe` | Bracketed pastes always safe. Default true. |
| `clipboard-codepoint-map` | Replace chars on copy. Supports ranges. |

## Shell integration & misc

| Key | Notes |
|---|---|
| `shell-integration` | `detect`, `bash`, `zsh`, `fish`, `nu`, etc. |
| `shell-integration-features` | Toggle individual features. |
| `title-report` | Allow programs to read window title. |
| `split-divider-color` | Split border color. |
| `split-preserve-zoom` | `navigation` preserves zoom across splits. |
