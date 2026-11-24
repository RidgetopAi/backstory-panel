# Backstory — Omarchy panel

Memory for your AI coding agents. The panel shows each project's latest handoff,
what needs your attention, and a one-click **Continue** that resumes the work in
your agent. It also adds a bar widget showing the project you are in.

**Every agent starts where you left off.**

![Backstory panel](preview.png)

This repo is the shell plugin only. The memory itself comes from the
[`backstory`](https://github.com/RidgetopAi/backstory) daemon. The plugin is a
client: it runs the `backstory` CLI and reads its output, and it never opens the
store directly.

## Install

Backstory has two parts: the `backstory` daemon and CLI, and this panel. Nothing
needs root; everything goes under your home directory.

**1. The daemon**, from the prebuilt release (x86_64):

```sh
cd "$(mktemp -d)"
curl -LO https://github.com/RidgetopAi/backstory/releases/download/v1.3.4/backstory-1.3.4-linux-amd64.tar.gz
curl -LO https://github.com/RidgetopAi/backstory/releases/download/v1.3.4/SHA256SUMS
sha256sum -c SHA256SUMS
tar -xzf backstory-1.3.4-linux-amd64.tar.gz
install -Dm755 backstory-1.3.4-linux-amd64/backstory ~/.local/bin/backstory
install -Dm644 backstory-1.3.4-linux-amd64/backstory.service ~/.config/systemd/user/backstory.service
systemctl --user daemon-reload
systemctl --user enable --now backstory
```

**2. Hook up your agents** (claude, codex, hermes, pi: whichever it finds):

```sh
backstory install
backstory backfill claude    # optional: import your existing Claude Code history
```

**3. The panel:**

```sh
omarchy plugin add https://github.com/RidgetopAi/backstory-panel --enable
```

Optional: a Hyprland snippet that sizes and places the panel window and binds
**Ctrl+Shift+B** to open it:

```sh
curl -fLo ~/.config/hypr/backstory.lua https://raw.githubusercontent.com/RidgetopAi/backstory/v1.3.4/ops/hyprland/backstory.lua
```

then add `require("hypr.backstory")` to your `hyprland.lua`.

**Or build from source** instead of steps 1 and 3. This needs Go 1.27 or newer
(`omarchy-install-dev-env go`, or Install > Development > Go in the Omarchy menu):

```sh
git clone --branch v1.3.4 https://github.com/RidgetopAi/backstory
cd backstory && make install
```

`make install` installs the daemon, this panel and the Hyprland snippet in one go.
Then run step 2.

**Updating:** repeat step 1 with the new version, then
`systemctl --user restart backstory`, `backstory install` (refreshes the agent hooks and
skills) and `omarchy plugin update ridgetopai.backstory`. From source, check out the new tag and
run `make install` again; it updates both.

## What it runs

Every command the plugin can run is listed in `js/launchers.js`:

- `backstory this-week`, `records`, `tools`: reads the memory, all with `--json`.
- `backstory affirm`, `done`, `delete`, `purge`: only when you press the matching
  button. Edit opens `backstory edit` in a terminal through `xdg-terminal-exec`.
- `omarchy agent prompt`, `omarchy agent --pick`, `omarchy-default-agent`: used by
  Continue / Resume to start your agent.
- Continue on a specific agent's row starts that agent directly, in a terminal at the
  project, with the same flags Omarchy's own `omarchy-agent` uses: `claude --permission-mode
  auto`, `codex --approve-for-me`, `hermes chat --yolo`, or `pi`. Those modes let the agent run
  commands without asking each time. The opening prompt is the project's last handoff, which
  an earlier agent session wrote (possibly on another of your machines, if you sync).
- `xdg-terminal-exec -- sh -c …`: opens a terminal in the project's directory. The
  path is passed as an argument and is never inserted into the shell script.
- `hyprctl dispatch focuswindow`, `tmux select-window` / `select-pane`: brings an
  already-running agent to the front instead of starting a new one.
- `omarchy-shell`: toggles and focuses the panel itself.

The plugin makes no network calls, and neither does the daemon. The only network use is
the optional sync between your own machines over ssh: `backstory install sync` adds a
systemd timer that runs `backstory sync`.

## Remove

```sh
omarchy plugin remove ridgetopai.backstory
backstory install --remove                # unhook the agents it finds
backstory install sync --remove           # only if you set up sync
backstory install bash --remove           # only if you added shell capture
backstory install agents --remove         # only if you installed the generic agents entry
systemctl --user disable --now backstory
rm -f ~/.local/bin/backstory ~/.config/systemd/user/backstory.service ~/.config/hypr/backstory.lua
```

Remove the `require("hypr.backstory")` line from `hyprland.lua` if you added it.

Your memory stays in `~/.local/share/backstory` until you delete it.

## License

MIT. This repo is the `panel/` directory of
[RidgetopAi/backstory](https://github.com/RidgetopAi/backstory) at the release named
in `manifest.json`. Report issues there.
