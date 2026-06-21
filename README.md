# Host Health Dashboard (sshPilot plugin)

A page that lists every saved connection and shows its reachability, refreshed
automatically in the background. Each host has three states:

- **Up** — reachable (and, for SSH connections with "Check SSH" on, the SSH
  handshake succeeds).
- **TCP only** — the port is open but SSH isn't ready (sshd not responding, auth
  refused, …); hover for the reason. This distinguishes "port reachable" from
  "actually usable."
- **Down** — no TCP connection at all.

With **Check SSH** enabled (default) the SSH probe runs through your
`~/.ssh/config` using the connection's nickname, so **ProxyJump/bastioned hosts
are tested correctly** (a plain TCP probe would wrongly show them Down). Turn it
off for a faster, port-only check.

A worked example of doing network work off the UI thread and shutting worker
threads down cleanly on quit.

## Requirements

- sshPilot with plugin **API ≥ 1.4** (provides `ctx.list_connections()`). On an
  older app the page shows an "update required" notice instead of failing.

## Install

Copy this directory to your user plugin dir and enable it in
**Preferences ▸ Plugins** (then restart sshPilot):

- Linux: `~/.local/share/sshpilot/plugins/health/`
- Flatpak: `~/.var/app/io.github.mfat.sshpilot/data/sshpilot/plugins/health/`

Or install the released `.zip` from **Preferences ▸ Plugins ▸ Install plugin…**.

## Notes

- A TCP connect only tells you the port is *reachable and accepting* — it does
  not authenticate. Default timeout is 3s; auto-refresh every 30s.
- All probes run on a bounded background thread pool; results are marshalled back
  to the UI with `ctx.run_on_ui_thread`. The pool is stopped on `deactivate()`
  and on the `app_shutdown` event, so nothing outlives the app.

## Permissions

`connections`, `network`, `ui` — declared for transparency; sshPilot plugins run
unsandboxed with full app privileges. Only install plugins you trust.

## Develop / test

```sh
pip install pytest
pip install "sshpilot @ git+https://github.com/mfat/sshpilot" --no-deps
pytest -ra
```

`tcp_check` is unit-tested by monkeypatching `socket`; `gi` is imported lazily
inside the page factory.
