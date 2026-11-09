# Hibernate

Put a Herdr space to sleep to free its CPU and memory, and wake it later exactly as it was: Claude
and Codex come back in the same conversation, dev servers start again, and the panes and tabs never
move. After a reboot (or a lost herdr-core), it offers to resume the agents and programs that were
running, never the ones you exited yourself.

![Hibernate](preview.png)

## What you see

- Nothing docked: the mod runs in a 0px layout block and draws only dialogs and toasts.
- **Sleep confirm** (dialog, 440px wide): shown when an agent in the space is working or blocked.
  "Sleep <space>?", "Still busy:" and a line per agent ("Claude Code is working"), then
  **Cancel** and **Sleep anyway** (solid). Enter confirms, Esc cancels.
- **Resume after restart** (dialog, 560px wide, up to 620px tall): "These were running before
  their terminals were lost". A row per program, ticked by default: a checkbox, its name in bold, the
  space and address dimmed, and the exact command it will run in the code font
  (`cd ~/code/storefront && claude --resume 8f3c…`). Below: **Resume automatically next time**, then
  **Not now**, **Resume N** (when some are unticked) and **Resume all** (solid). Enter resumes the
  ticked ones, Esc is Not now.
- Toasts: "<space> is asleep · 3 programs stopped", "<space> is awake · 3 programs resumed", or a
  warning naming what didn't stop or start.

## How it behaves

- **Sleep a space**: every terminal in its tabs (and their sub-agents) is recorded to
  `~/.config/herdr-app/data/hibernate/state.json`: the foreground argv, its folder, and for Claude
  its session id (`~/.claude/sessions/<pid>.json`), for Codex the id of its open rollout file. Then
  each program stops and its shell stays: agents get an interrupt (when busy) and `/exit` (`/quit`
  for Codex); anything else gets SIGTERM to its process group, then SIGKILL as a last resort. A
  space with nothing running says so and isn't marked asleep.
- **Wake a space**: each recorded program is started again by typing
  `cd <folder> && <command>` into its shell, only when that shell is idle:
  `claude --resume <id>` (keeping `--model`, `--permission-mode`, `--add-dir`, `--agent`,
  `--dangerously-skip-permissions`, `--verbose`), `codex resume --no-daemon <id>` (or `--last`), or
  the original argv. A terminal that's gone gets a new shell labelled into the same tab.
- **After a restart**: every 30s the mod records what each Herdr terminal runs, writing only on a
  change. A reboot or a lost core brings kept terminals back as shells at the same address with a
  **new** shell pid; those are offered. A program you exited keeps its shell pid, so it never is.
  Terminals still being recovered are held for 15 minutes.
- **Settings** (Better Config's Mod settings, or `settings` in `mod.json`):
  - `wake`: `manual` (default: only Wake), `open` (when you go to the space), `start` (once per
    Herdr launch, each space when its machine has loaded). It spots a launch with macOS's `ps`, so
    it doesn't fire yet in the Windows or Linux app.
  - `after_restart`: `ask` (default, the dialog), `auto` (resume without asking), `off`.
  Ticking "Resume automatically next time" writes `after_restart: "auto"`.
- **Asking it from elsewhere**: keys or `herdr-app do` with `hibernate.sleep` / `hibernate.wake`
  (the focused space, or `hibernate.sleep:<space id>`) and `hibernate.resume` (everything waiting).
  Another mod sets shared `hibernate.request` to `{ nonce, action: "sleep" | "wake", space }`
  (`confirmed: true` skips the busy dialog); the [resource meter](../resources/) does this.
- It publishes shared `hibernate.sleeping` as `{ <space id>: { at, count } }`, so a sidebar can mark
  sleeping spaces (the Orca layout's shows a moon and "Asleep").
- A space on another machine (WSL behind the Windows app, a remote host) works too: its programs
  are found, stopped and started on that machine through herdr-core. The Windows app's own spaces
  can't sleep (no `ps` or `kill`). A space whose machine is offline stays asleep, and a wake that
  can't restart everything keeps the rest asleep.

## Built from

- `mods/hibernate/`: `main.js` (the queue, sleep, wake, the rolling record and restart detection,
  settings), `core.js` (what runs in a terminal, how to stop it, how to start it again), `ui.js`
  (the two dialogs), `settings.js` (reads `mod.json` settings over defaults), `mod.schema.json`
  (the two settings for Mod settings), `moon.svg`, `sun.svg` (for menu items).
- `mod.json`: `{ "place": "none", "title": "Hibernate" }`. Put it in a 0px block so it always runs:
  `{ "id": "hibernate", "type": "block", "size": { "px": 0 }, "content": { "mod": "hibernate" } }`.
- Menu items, in your own clone of Herdr's `menu`: a space's items can add "Sleep space" or "Wake
  space" (when `shared("hibernate.sleeping")?.[id]`) that do
  `setShared("hibernate.request", { nonce: Date.now(), action, space: id })`.
- Herdr API: `hc` (`get --json`, `ls --json`, `send`, `interrupt`, `run --bg --keep`), `exec`,
  `spaces`, `machines`, `focused`, `shared`/`setShared`, `notify`, `openOverlay`/`closeOverlay`,
  `nextCommand`, `readTextFile`/`writeTextFile`, `fs/promises`.
- Programs: `kill`, `pgrep` and `lsof` (Codex sessions) on the space's machine, `ps` (Herdr's start
  time) on this one.

## Watch out

- Never stop a program with ctrl+c: Herdr starts agents through
  `zsh -lic '"$@"; exec "$0" -li'`, and SIGINT makes that wrapper abort and close the pane instead of
  returning to a prompt.
- Go by pid → session id, never "the latest session in this folder": several agents can share one.
- "On start" compares Herdr.app's process start time with the last one seen, because the mod also
  restarts on any edit or layout switch.
- Jobs run one at a time (a queue), so a sleep never races the rolling record.
- Sub-agents end with their parent: they're recorded but not resumed.
- If someone exits a program while Herdr is closed and then reboots, it's still offered (an offer,
  never an automatic run unless `after_restart` is `auto`).

## Done when

- `herdr-app check mods/hibernate` passes and `herdr-app status` lists no errors.
- In a scratch space running `sleep 600`, `herdr-app do hibernate.sleep:<id>` leaves an idle shell
  with the same pid and `hibernate.sleeping` lists the space; `hibernate.wake:<id>` runs
  `sleep 600` again in that shell.
- With `wake: "open"`, switching to a sleeping space wakes it.
- A Claude agent slept and woken comes back in the same conversation.
- `kill -9` a test terminal's shell: herdr-core recovers it with a new pid, and the resume dialog
  lists it but not a program you exited by hand.
