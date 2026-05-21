# Ghostty Session Restore on Linux

Make [Ghostty](https://ghostty.org) reopen with all tabs, panes, and working
directories intact after a crash, a Ghostty update, or a full reboot —
on Linux, where Ghostty's native `window-save-state` is not yet
implemented (tracked in
[ghostty-org/ghostty#1847](https://github.com/ghostty-org/ghostty/issues/1847)).

The idea: let [zellij](https://zellij.dev) own the session and have
Ghostty attach to it on every launch.

## Minimum requirements

- Linux with a graphical session
- **Ghostty** with the `command = …` config option (any modern release)
- **zellij ≥ 0.40** with session serialization support (tested with 0.44.3)
- Ghostty must be able to launch `zellij` from its `PATH`
  - If Ghostty is installed as a snap, it must be **classic** confinement,
    not strict — check with `snap info ghostty | grep confinement`

## Install zellij

Pick whichever you prefer:

```sh
# Option A — official static binary, no root
curl -fsSL -o /tmp/z.tar.gz \
  https://github.com/zellij-org/zellij/releases/latest/download/zellij-x86_64-unknown-linux-musl.tar.gz
tar -xzf /tmp/z.tar.gz -C /tmp
install -m 755 /tmp/zellij ~/.local/bin/zellij

# Option B — package manager
sudo pacman -S zellij        # Arch
# brew install zellij        # Linuxbrew
# cargo install --locked zellij
```

Confirm `zellij --version` works and that `which zellij` is on Ghostty's
`PATH`.

## Configure

### `~/.config/ghostty/config.ghostty`

```ini
command = zellij attach --create --force-run-commands main
```

- `--create` — attach to a session named `main`, create it if missing.
- `--force-run-commands` — on resurrection, re-run each pane's last
  foreground command (e.g. `nvim file.py`). Plain shells are unaffected.

### `~/.config/zellij/config.kdl`

```kdl
session_serialization true
serialize_pane_viewport true
scrollback_lines_to_serialize 10000
```

These are off by default in zellij 0.44.3 and must be set explicitly.
Serialized state lives in `~/.cache/zellij/<version>/<session>/`.

## Activate

Close every Ghostty window and open a new one. The `command =` option
only applies to newly spawned terminals; `ghostty +reload-config` will
not re-shell windows that are already open.

## How it behaves

| Scenario | Result |
|---|---|
| Ghostty crash | Next window reattaches instantly; processes still running. |
| Ghostty self-update / restart | Same as crash. |
| Reboot | Layout, tabs, panes, cwds, and last foreground commands are restored. In-flight background processes are gone — impossible to preserve across a kernel restart. |

## Trade-off

Tabs and splits now belong to zellij, not Ghostty. zellij is modal —
the bottom status bar shows the current mode and available keys.

| Action | Keys |
|---|---|
| New tab | `Ctrl+t` then `n` |
| New pane (right / down) | `Ctrl+p` then `r` / `d` |
| Move focus | `Ctrl+p` then arrow / `hjkl` |
| Detach (keep session alive) | `Ctrl+s` then `d` |
| Quit zellij | `Ctrl+q` |

## Uninstall

```sh
# 1. In ~/.config/ghostty/config.ghostty, comment out the `command =` line.
# 2. Close all Ghostty windows.

zellij kill-all-sessions
rm -rf ~/.cache/zellij ~/.config/zellij ~/.local/share/zellij
rm -f  ~/.local/bin/zellij
```

## Local docs

This repo also includes a snapshot of Ghostty's configuration docs for
offline reference:

- `01-overview.md` — docs map
- `02-config-syntax.md` — config file location and `key = value` syntax
- `03-config-reference.md` — full option reference
- `04-keybindings.md` — keybind syntax and actions
- `05-themes.md` — themes
- `config.example` — annotated starter config
- `SETUP.md` — full setup log with decisions and rationale
