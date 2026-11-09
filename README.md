# Herdr mods

Mods for [Herdr Desktop](https://herdr.dev). Each folder is one mod (or a pair that work together),
laid out like Herdr's home folder (`mods/<name>/`) with a `RECIPE.md` describing it.

| Folder | What it is |
| --- | --- |
| [issues](issues/) | GitHub issues, PRs and Projects in Herdr: a table, a Projects board, an in-app viewer (PR checks, diffs, merge), and a Create worktree dialog that starts work from a branch (including remote-only), an issue, a PR or a new name, with Claude's prompt prefilled. Two mods: `issues` and `new-worktree`. |
| [live-ports](live-ports/) | Listening ports grouped by the space that started them, as a status-bar button and popup with Open and Stop. |
| [usage-limits](usage-limits/) | Claude Code and Codex usage windows in a status bar, with a hover panel and session context. Reads the CLIs, never tokens. |
| [hibernate](hibernate/) | Put a space to sleep to free CPU and memory and wake it as it was (Claude and Codex resume the same conversation), and resume what a reboot stopped. Settings for waking on open or on start. |
| [resources](resources/) | CPU and memory of everything Herdr runs, as a status-bar meter and a per-space popup with Sleep and Wake (with hibernate). |
| [t3-actions](t3-actions/) | A one-click run button for the tab bar, after T3 Code's, with saved actions. Adapted from [JinJieBeWater/herdr-t3](https://github.com/JinJieBeWater/herdr-t3). |

They're also part of the [Orca layout](https://github.com/VladTemp27/herdr-orca).

## Install one

Ask your Herdr agent, with the folder's link:

> Read ~/.config/herdr-app/AGENTS.md, then add this mod: https://github.com/VladTemp27/herdr-mods/tree/main/issues

By hand: copy the folder's `mods/<name>/` into `~/.config/herdr-app/mods/`, run
`herdr-app check ~/.config/herdr-app/mods/<name>`, and place it as its `RECIPE.md` says
(`mod.open <name>`, a key, or a block in a layout you own).

## Updating this repo

`./sync.sh` copies these mods from your Herdr home into their folders.
`previews/render.sh` redraws each folder's `preview.png` from `previews/<folder>.html` with headless
Chrome: previews are HTML mockups with demo data, so no screenshot of your own work is published.

## License

Apache-2.0, see [LICENSE](LICENSE) and [NOTICE](NOTICE).
