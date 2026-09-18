# rustdesk-20260917

Custom RustDesk Windows client build, compiled on GitHub Actions.

## Build config

No connection values are kept in this repository. Every value is stored as an
encrypted **GitHub Secret** and read by the workflow only at build time:

| Secret | Purpose |
| --- | --- |
| `ID_SERVER` | ID / rendezvous server |
| `RELAY_SERVER` | Relay server |
| `API_SERVER` | API server (optional) |
| `PUBLIC_KEY` | Server public key (optional) |
| `FIXED_PASSWORD` | Fixed unattended-access password (optional) |

`Source` builds against upstream `rustdesk/rustdesk` (master). The compiled
client connects to the configured server with the fixed password.

## Customized default settings

The "Patch RustDesk custom config" step in
`.github/workflows/build-rustdesk-windows.yml` also patches
`libs/hbb_common/src/config.rs` so a freshly installed client starts with:

| Setting | Option key | Injected default |
| --- | --- | --- |
| 常规-启动时检查软件更新 (check updates on start) | `enable-check-update` | `N` (unchecked) |
| 常规-启用UDP打洞 (UDP hole punching) | `enable-udp-punch` | `Y` (enabled) |
| 安全-拒绝局域网发现 (deny LAN discovery) | `enable-lan-discovery` | `N` (denied) |
| 安全-允许远程修改配置 (allow remote config modification) | `allow-remote-config-modification` | `Y` (allowed) |

- The first two are injected into `LocalConfig::load()` (file `_local`), the
  last two into `Config2::load()` (file `"2"`) — the same places the app reads
  these options from (`LocalConfig::get_option`, `Config::get_option`).
- Each value is written only when the key is **absent** (a `contains_key`
  guard), so an existing user setting is never overwritten.
- `enable-lan-discovery = "N"` is the deny-LAN-discovery state: the UI shows
  the checkbox as `_denyLANDiscovery = !option2bool("enable-lan-discovery", …)`.
- `allow-remote-config-modification = "Y"` means "allowed" (checkbox checked).

## Build

Pushing to `main` or running the workflow manually (`workflow_dispatch`)
triggers `.github/workflows/build-rustdesk-windows.yml`, which builds a
portable Windows client in two jobs (`generate-bridge` → `build-windows`).
The `.exe` artifacts appear under the **Actions** tab of the triggering run.
