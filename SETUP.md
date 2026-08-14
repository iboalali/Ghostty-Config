# Setup Log — Session Restore for Ghostty on Linux

Date: 2026-05-21
Host: Ubuntu 24.04 (x86_64), zsh
Ghostty: snap v1.3.1 rev 718 (classic confinement)
Goal: Ghostty should reopen with all tabs/panes intact after a crash, an
update-triggered restart, or a full reboot.

---

## 1. Why we couldn't do this natively

Ghostty's reference docs list one relevant option:

```
window-save-state = default | never | always
```

But the docs say verbatim: *"Currently only supported on macOS. This has
no effect on Linux."* Upstream tracking issue is
[ghostty-org/ghostty#1847](https://github.com/ghostty-org/ghostty/issues/1847)
— still open, no GTK/Linux implementation.

So on Linux the only working approach today is a terminal multiplexer
that owns the session lifetime independently of Ghostty.

## 2. Why zellij over tmux

- Both work. We picked **zellij** because session resurrection is built
  in — no plugin manager, no `tmux-resurrect` / `tmux-continuum`.
- zellij's persistence layer writes layout + cwds + last-foreground
  command (and optionally viewport + scrollback) to
  `~/.cache/zellij/<version>/<session>/` and resurrects from there.

Trade-off accepted: tabs/panes inside Ghostty are now zellij's, not
Ghostty's. Ghostty's own Ctrl+Shift+T etc. won't be used.

## 3. Why we did NOT use a systemd user service

The first plan was a `~/.config/systemd/user/zellij.service` to start a
detached zellij session at login. We dropped it because:

- zellij requires a TTY to start. There is no `--daemon` mode.
- Wrapping it in `setsid` + a fake pty works but is fragile and offers
  no benefit over zellij's own serialization (which already survives
  reboots cleanly).
- The serialization-based approach gives the same user-visible outcome
  (after-reboot resurrection) with one less moving part.

## 4. What we installed

```
~/.local/bin/zellij   # zellij 0.44.3, official static-musl x86_64 build
```

Source: <https://github.com/zellij-org/zellij/releases/tag/v0.44.3>
Tarball: `zellij-x86_64-unknown-linux-musl.tar.gz`
SHA-256 of the extracted binary (matched the published `.sha256sum`):

```
397481870c4fc3bae646cd7613cde3a1cebdc204558a6cb9a7c603d4c852fc90
```

`~/.local/bin` was already on `$PATH`. No system-wide install, no sudo.

The Ghostty snap uses **classic** confinement, so it can see binaries
in `~/.local/bin` (strict confinement would have blocked this).

## 5. Files we wrote

### `~/.config/ghostty/config.ghostty`

```ini
# ~/.config/ghostty/config.ghostty
# Reload without restart: `ghostty +reload-config`

command = zellij attach --create --force-run-commands main

font-size = 13
theme = dark:Catppuccin Frappe,light:Catppuccin Latte

window-padding-x       = 8
window-padding-y       = 8
window-padding-balance = true

shell-integration = detect
```

The `command =` line is the load-bearing one. Flags:

| Flag | Effect |
|---|---|
| `--create` | Attach to session `main`, create it if it doesn't exist. |
| `--force-run-commands` | On resurrection, re-run each pane's last foreground command (e.g. `nvim foo.py`). Plain shells are unaffected. |

### `~/.config/zellij/config.kdl`

```kdl
session_serialization true
serialize_pane_viewport true
scrollback_lines_to_serialize 10000

// Move Session mode off Ctrl+O so that key reaches the running program.
keybinds {
    shared_except "session" "locked" {
        unbind "Ctrl o"
        bind "Ctrl y" { SwitchToMode "Session"; }
    }
    session {
        unbind "Ctrl o"
        bind "Ctrl y" { SwitchToMode "Normal"; }
    }

    // Drop tmux compatibility mode so Ctrl+B reaches the app too.
    shared_except "tmux" "locked" {
        unbind "Ctrl b"
    }
}
```

Defaults in zellij 0.44.3 have `session_serialization false` and
`serialize_pane_viewport false`, so this had to be set explicitly.

### Why the `keybinds` block: zellij eats the app's Ctrl keys

Because zellij now owns the terminal, its mode keys are intercepted
*before* the program running inside a pane ever sees them. zellij's
defaults claim **`Ctrl+ b c f g h n o p q s t`**.

That collides with TUI programs that use those keys. Two concrete cases,
both from Claude Code:

- `Ctrl+O` expands tool output / toggles the transcript, but zellij's
  default `shared_except "session" "locked" { bind "Ctrl o" ... }`
  swallows it and enters Session mode instead.
- `Ctrl+B` backgrounds a running process, but zellij's
  `shared_except "tmux" "locked" { bind "Ctrl b" ... }` swallows it and
  enters tmux compatibility mode.

There are two ways out, and which one fits depends on whether you still
want the zellij mode:

**Move the mode key** — Session mode goes to `Ctrl+Y` (unused by
zellij's defaults), so the session manager and detach stay reachable.
Both the `shared_except` block *and* the mode's own block need editing:
the latter holds the key that exits the mode again.

**Unbind the mode** — tmux compatibility mode is simply dropped. It's
pure duplication here (detach is `Ctrl+Y d`, panes `Ctrl+P`, tabs
`Ctrl+T`, scroll `Ctrl+S`), and with `b c f g h n o p q s t` taken by
zellij and the rest of the alphabet spoken for by readline (`Ctrl+A`,
`Ctrl+E`, `Ctrl+R`, `Ctrl+W`, …), there is no comfortable key left to
move it to. Only the `shared_except` block needs editing — once the
mode is unreachable, its exit key is moot.

One thing worth knowing before you change anything: inside tmux mode,
`Ctrl+B` is bound to `Write 2`, so pressing **`Ctrl+B` twice** sends a
literal `Ctrl+B` through to the app. That's the zero-config workaround
if you'd rather keep tmux mode.

Same recipe for any other collision: pick a letter outside that list
and rebind, or unbind the mode if you don't use it. Verify with:

```sh
zellij setup --dump-config | grep -oE 'bind "Ctrl [a-z]"' | sort -u
zellij setup --check          # → "[CONFIG FILE]: Well defined."
```

Config changes need a **new** zellij session to take effect for
already-running panes' mode keys.

## 6. How restore behaves now

| Scenario | What happens | Why |
|---|---|---|
| **Ghostty crash** | Next Ghostty window reattaches instantly to the live zellij session. Everything is exactly where it was, including running processes. | zellij is a separate process tree, unaffected by Ghostty's death. |
| **Ghostty self-update / restart** | Same as crash. | Same reason. |
| **Reboot / power loss** | First Ghostty window after boot resurrects the saved layout: same tabs, same panes, same cwds. Foreground commands re-run; background processes don't (impossible — they died with the kernel). | zellij's serializer writes state to `~/.cache/zellij/` continuously. `--force-run-commands` replays the last foreground command per pane. |
| **`zellij kill-session main`** | A fresh session is created on next attach. The serialized state may still exist on disk; use `--forget` to wipe it. | By design. |

## 7. Verification commands used

```sh
# Ghostty parses our config cleanly:
ghostty +show-config | grep '^command '
ghostty +show-config >/dev/null; echo $?     # → 0

# Zellij parses our config cleanly:
~/.local/bin/zellij setup --check             # → "[CONFIG FILE]: Well defined."

# Inspect Ghostty snap confinement (must be 'classic' to see ~/.local/bin):
snap info ghostty | grep -E '^(name|confinement)'

# List active zellij sessions (after first launch):
zellij list-sessions
```

## 8. Activation

`command =` only affects newly spawned terminals. To switch all windows:

1. Close every Ghostty window.
2. Open a new one. The first window creates the `main` zellij session.

`ghostty +reload-config` will not re-shell existing windows.

## 9. Day-to-day zellij usage (cheat sheet)

zellij is modal — you press a "mode key" then a single action key. The
bottom status bar always shows the current mode and available actions.

| Action | Keys |
|---|---|
| New tab | `Ctrl+t` then `n` |
| Next tab / previous tab | `Ctrl+t` then `l` / `h` |
| Jump to tab *n* | `Ctrl+t` then `1`–`9` |
| Rename tab | `Ctrl+t` then `r` |
| Close tab | `Ctrl+t` then `x` |
| New pane (right) | `Ctrl+p` then `r` |
| New pane (down) | `Ctrl+p` then `d` |
| Focus pane | `Ctrl+p` then arrow / `hjkl` |
| Resize pane | `Ctrl+n` then arrow / `hjkl` |
| Scrollback / search | `Ctrl+s` |
| Session mode (detach, session manager) | `Ctrl+y` (default: `Ctrl+o`) |
| Detach (leave session running) | `Ctrl+y` then `d` |
| Session manager | `Ctrl+y` then `w` |
| Quit zellij entirely | `Ctrl+q` |

Some actions have no mode key at all — they're bound directly under
`shared_except "locked"`, so they work from any mode without a prefix:

| Action | Keys |
|---|---|
| **Reorder tab left / right** | `Alt+i` / `Alt+o` |
| Focus pane, crossing to the next tab at the edge | `Alt+h` / `Alt+l` (or `Alt+←` / `Alt+→`) |
| Focus pane up / down | `Alt+k` / `Alt+j` |
| New pane | `Alt+n` |
| Toggle floating panes | `Alt+f` |
| Resize pane | `Alt+=` / `Alt+-` |

Reordering tabs is the one people hunt for in Tab mode and never find:
`Ctrl+t` only navigates and manages tabs, it can't move them. The same
actions are scriptable from any pane — e.g. `zellij action move-tab left`.

If the `Alt` bindings don't register, the terminal isn't forwarding
`Alt` as an escape prefix; rebind them in `~/.config/zellij/config.kdl`.

`Ctrl+y d` is the everyday "stop using this terminal but keep the work
alive" gesture — closing the Ghostty window does the same thing.

Detach is on `Ctrl+y` here only because of the rebind above; on a stock
zellij it is `Ctrl+o d`. Stock zellij also detaches with `Ctrl+b d`
(tmux compatibility mode) — this config unbinds that mode, so `Ctrl+y d`
is the only detach gesture here.

## 10. Rollback / uninstall

```sh
# Stop zellij from being launched by Ghostty:
#   Edit ~/.config/ghostty/config.ghostty and comment out the `command =` line.
# Then close all Ghostty windows.

# Kill any running zellij session:
zellij kill-all-sessions

# Remove serialized session state:
rm -rf ~/.cache/zellij

# Remove the binary and its config:
rm -f  ~/.local/bin/zellij
rm -rf ~/.config/zellij
rm -rf ~/.local/share/zellij
```

## 11. Things to revisit later

- Watch [ghostty-org/ghostty#1847](https://github.com/ghostty-org/ghostty/issues/1847)
  for native Linux session restore. If it lands, the whole zellij layer
  becomes optional.
- zellij itself moves fast; if a newer release becomes available,
  re-run the install (download tarball, verify SHA, replace
  `~/.local/bin/zellij`).
