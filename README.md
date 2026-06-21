# sing-box (v2ray_api builds)

Automated rebuilds of **unmodified upstream [`SagerNet/sing-box`](https://github.com/SagerNet/sing-box)**
with the **`with_v2ray_api`** build tag enabled, published as ready-to-use Linux binaries.

## Why this exists

The official sing-box release binaries are built **without** `with_v2ray_api`, so the
**V2Ray gRPC `StatsService`** (per-user traffic counters + management API) is not available
in stock builds. Some deployments need it. Rather than hand-compile sing-box every release,
this repo tracks upstream and rebuilds automatically.

There is **no source modification** — this is the exact upstream source at each release tag,
compiled with the official tag set **plus one addition** (`with_v2ray_api`):

```
with_gvisor,with_quic,with_dhcp,with_wireguard,with_utls,with_acme,with_clash_api,with_tailscale,with_v2ray_api
```

So you keep every feature of an official build and additionally get the V2Ray API.

## Downloads

Each release matches an upstream sing-box version tag (`vX.Y.Z`). Assets per release:

| asset | what |
|---|---|
| `sing-box-<ver>-linux-amd64.tar.gz` (+ `.sha256`, `.minisig`) | linux/amd64 binary |
| `sing-box-<ver>-linux-arm64.tar.gz` (+ `.sha256`, `.minisig`) | linux/arm64 binary |

Latest release: <https://github.com/river2heaven/sing-box-v2rayapi/releases/latest>

Download a specific asset:
```
https://github.com/river2heaven/sing-box-v2rayapi/releases/download/v<ver>/sing-box-<ver>-linux-amd64.tar.gz
```

Verify before use:
```bash
sha256sum -c sing-box-<ver>-linux-amd64.tar.gz.sha256
# minisign signature (recommended): public key is in minisign.pub (key ID 2EC45A4A4FB31B2A)
minisign -Vm sing-box-<ver>-linux-amd64.tar.gz -P RWQqG7NPSlrELtbB3OOjGwLo/9Sn8y436071SbKqijrMPFpmqCcXWX1U
```

**Signing public key** (minisign, key ID `2EC45A4A4FB31B2A`) — also committed as [`minisign.pub`](minisign.pub):
```
RWQqG7NPSlrELtbB3OOjGwLo/9Sn8y436071SbKqijrMPFpmqCcXWX1U
```

## How auto-tracking works

GitHub Actions **cannot be triggered by another repository's release event** (no cross-repo
release webhook exists). So this repo **polls**:

- **`schedule`** (daily cron) — checks `SagerNet/sing-box` *latest stable* release; builds + publishes if it's new (idempotent: existing versions are skipped). Pre-releases (alpha/beta/rc) are **not** auto-built.
- **`workflow_dispatch`** — manual trigger; optional `version` input to build a specific tag.
- **`repository_dispatch`** (`type: upstream-release`) — optional low-latency trigger from an external watcher of `…/releases.atom`.

## Maintenance

- **Signing**: **enabled** — releases are signed with minisign (key ID `2EC45A4A4FB31B2A`,
  public key in [`minisign.pub`](minisign.pub)). Signing uses repo secrets `MINISIGN_PRIVATE_KEY`
  + `MINISIGN_PASSWORD` (password-protected key). If those secrets are ever removed, the workflow
  falls back to `.sha256`-only (warns, doesn't fail).
- **First run / backfill**: `Actions → Build sing-box (v2ray_api) → Run workflow` (leave
  `version` blank for upstream latest, or pin a specific tag).
- **Schedule keep-alive**: GitHub disables scheduled workflows after 60 days of repo
  inactivity; a successful release counts as activity, but during long upstream-quiet periods
  re-enable via the Actions tab if needed.

## License

The published binaries are builds of sing-box, which is licensed under **GPL-3.0-or-later**.
The corresponding source is upstream `SagerNet/sing-box` at the tagged version; the only build
configuration applied is the tag set shown above (see `.github/workflows/build.yml`). This repo
distributes binaries under the same terms.
