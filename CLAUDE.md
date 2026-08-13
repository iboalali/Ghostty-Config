# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A **documentation + recipe** for restoring Ghostty terminal sessions
(tabs, panes, cwds) on Linux after a crash, update, or reboot — by
having Ghostty launch zellij and letting zellij own the session.
There is no source code, build, lint, or test step.

The repo contains two kinds of files:

- **Public-facing instructions** — `README.md`, `SETUP.md`,
  `config.example`. Users follow these to reproduce the setup.
- **Offline docs snapshot** — `01-overview.md` … `05-themes.md`,
  pulled from <https://ghostty.org/docs> for reference. Treat these as
  cached upstream content, not original prose.

## The live config files this repo describes are NOT in the repo

The setup actually takes effect through two files in the user's home,
which this repo only documents:

- `~/.config/ghostty/config.ghostty` — `command = zellij attach --create --force-run-commands main` plus theme/padding and a `mouse-scroll-multiplier` scroll tweak.
- `~/.config/zellij/config.kdl` — `session_serialization true`, `serialize_pane_viewport true`, `scrollback_lines_to_serialize 10000`, plus a `keybinds` block moving Session mode from `Ctrl+O` to `Ctrl+Y` and unbinding tmux compat mode from `Ctrl+B`.

When the user asks you to "change a setting", clarify whether they want
to edit their live config, update the published recipe, or both.

## Non-obvious gotchas (learned the hard way — keep them documented)

- **zellij must be on a system PATH, not `~/.local/bin`.** Desktop apps
  on Ubuntu/GNOME inherit `PATH` from the systemd user manager, which
  does **not** include `~/.local/bin`. Ghostty launched from the GUI
  fails with `sh: zellij: not found` if zellij is only in
  `~/.local/bin`. Install to `/usr/local/bin/zellij` instead.
- **Ghostty snap must be `classic` confinement.** Strict confinement
  would block access to `/usr/local/bin` and the user's
  `~/.config/zellij/`. Verify with
  `snap info ghostty | grep confinement`.
- **`command =` only applies to newly spawned Ghostty windows.**
  `ghostty +reload-config` does **not** re-shell windows that are
  already open. Tell the user to close every window and open a new one.
- **`theme = dark:X,light:Y` falls back to light when GNOME's
  `color-scheme` is `'default'`.** Check with
  `gsettings get org.gnome.desktop.interface color-scheme`. For a
  reliable always-dark setup, use a plain `theme = NAME` instead of the
  light/dark clause.
- **Ghostty's default `mouse-scroll-multiplier` is
  `precision:1,discrete:3`** — a mouse-wheel notch scrolls 3 lines,
  which reads as "too fast." `precision:` controls touchpad/smooth
  scroll, `discrete:` controls the wheel; lower either to slow it (the
  live config uses `precision:0.5,discrete:1.5`). Confirm the default
  with `ghostty +show-config --default | grep mouse-scroll`.
- **zellij 0.44.3 has `session_serialization` OFF by default.** This
  is contrary to older zellij docs that may say otherwise. Always
  verify with `zellij setup --dump-config | grep session_serialization`.
- **zellij swallows `Ctrl+ b c f g h n o p q s t` before the app in the
  pane sees them.** Mode keys are intercepted by zellij, so TUI programs
  lose those bindings — notably `Ctrl+O` (Session mode), which Claude
  Code uses to expand tool output / toggle the transcript, and `Ctrl+B`
  (tmux compat mode), which Claude Code uses to background a process.
  Two fixes, pick per mode:
  - *Move* the mode key when the mode is still wanted — edit **both** the
    `shared_except "<mode>" "locked"` block (enter) and the `<mode>`
    block (exit). The live config moves Session mode to `Ctrl+Y`.
  - *Unbind* the mode when it's redundant — only the `shared_except`
    block needs editing, since an unreachable mode's exit key is moot.
    The live config unbinds tmux compat mode this way; it duplicated
    Session/Pane/Tab/Scroll, and no comfortable `Ctrl` letter was left
    to move it to (readline owns most of what zellij doesn't).

  List taken keys with
  `zellij setup --dump-config | grep -oE 'bind "Ctrl [a-z]"' | sort -u`,
  validate with `zellij setup --check`, and note the change only affects
  **new** zellij sessions.
- **Inside a zellij mode, the mode's own key sends the literal byte.**
  tmux mode binds `Ctrl b` → `Write 2`, so pressing `Ctrl+B` twice passes
  a real `Ctrl+B` to the app. Useful as a zero-config workaround before
  reaching for a rebind.
- **zellij 0.44.3 detach is `Ctrl+O d` (Session mode), not `Ctrl+S d`.**
  `Ctrl+S` is Scroll mode in this version; older docs and earlier zellij
  releases had Session on `Ctrl+S`. The repo docs were wrong about this
  until corrected. On stock zellij `Ctrl+B d` (tmux compat mode) also
  detaches, but the live config unbinds that mode.
- **A systemd user service to "daemonize" zellij does NOT work** —
  zellij requires a TTY and has no `--daemon` mode. The reboot-survival
  story relies on zellij's own session serialization to disk, not on
  keeping a headless zellij process alive.

## Refreshing the docs snapshot (`01-…05-`*.md)

Source of truth is <https://ghostty.org/docs>. Context7 library IDs
used originally:

- `/ghostty-org/website` — best signal on config reference
- `/ghostty-org/ghostty` — source tree

Use WebFetch on the live docs pages or `mcp__context7__query-docs` for
re-pulls. Don't paraphrase — these files exist so an offline user gets
the same wording as the upstream site.

## Repo conventions

- Remote `origin` is SSH (`git@github.com:iboalali/Ghostty-Config.git`),
  branch `main`.
- `.claude/` is gitignored — it holds per-folder Claude permission
  state, not project content.
- Absolute home paths (`/home/<user>/…`) should not appear in any file
  meant for the public — use `~` or `$HOME`. Quick check before
  committing: `grep -rn -E '/home/|/Users/' . --exclude-dir=.git`.
