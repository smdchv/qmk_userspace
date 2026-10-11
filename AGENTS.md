# AGENTS.md

## What this repo is
- A QMK External Userspace: keymaps, shared user code and build targets (`qmk.json`).
- It contains no keyboard definitions and no QMK core. Those come from upstream `qmk_firmware`, which is not part of this repo.
- All customisation lives here. Never modify `qmk_firmware`.

## Building
- The only host requirement is Docker. Do not install QMK tooling or toolchains on the host.
- Local builds need a checkout of upstream `qmk_firmware`. Use its Docker build wrapper, pointed at this userspace. See QMK's [Docker Quick Start](https://docs.qmk.fm/getting_started_docker) and [External Userspace](https://docs.qmk.fm/newbs_external_userspace) docs.
- Alternatively, push to GitHub: CI builds every `qmk.json` target and publishes the firmware as a GitHub Release.
- Build output is git-ignored.
- If a build can't find a keymap from this repo, the userspace wasn't detected. Check `qmk.json`.

## qmk.json
- Use the format QMK's own tools write
- An invalid `qmk.json` silently disables the whole userspace.

## Keymaps
- Use a unique keymap name, never `default`, because upstream keymaps win the lookup.
- The lookup checks, in order:
  1. `qmk_firmware` `keyboards/`
  2. userspace `keyboards/`
  3. `qmk_firmware` `layouts/`
  4. userspace `layouts/`

  The first match wins.
- A per-keyboard keymap shadows a layout keymap of the same name.
- A layout keymap applies to every board that lists that layout in `community_layouts`.
- Per-board conditionals:
  - C and `config.h`: `#ifdef KEYBOARD_<path_with_underscores>`
  - `rules.mk`: `ifeq ($(KEYBOARD),...)`

## Settings
- Override settings at keymap level, never in the keyboard definition.
- `config.h` is read after the keyboard's config. `#undef` a value before re-`#define`-ing it, because redefinitions break the build.
- `rules.mk` is read after the keyboard's `?=` rules. Only write `FEATURE = no` if the keyboard's JSON enables that feature.
- `rules.mk` aligns its comments with tabs.

## Verifying changes
- If this repo hasn't been updated in a while, verify the assumptions above and descriptions of QMK internals are still valid.
- Compare the full effective config before and after: keyboard definition, keymap and core defaults. The keymap files alone aren't enough.
- After a QMK update, check whether `EECONFIG_MAGIC_NUMBER` changed. If it did, the first boot after flashing resets saved settings.
- Use the Docker build wrapper to test that changes compile.
