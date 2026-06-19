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

Direct (GitHub release): `https://github.com/<owner>/<repo>/releases/latest`

Via jsdelivr CDN (good for restricted networks):
```
https://cdn.jsdelivr.net/gh/<owner>/<repo>@v<ver>/...   # (release assets are served from the release, see Releases tab)
```

Verify before use:
```bash
sha256sum -c sing-box-<ver>-linux-amd64.tar.gz.sha256
minisign -Vm sing-box-<ver>-linux-amd64.tar.gz -P <minisign-public-key>   # if signed
```

## How auto-tracking works

GitHub Actions **cannot be triggered by another repository's release event** (no cross-repo
release webhook exists). So this repo **polls**:

- **`schedule`** (daily cron) — checks `SagerNet/sing-box` *latest stable* release; builds + publishes if it's new (idempotent: existing versions are skipped). Pre-releases (alpha/beta/rc) are **not** auto-built.
- **`workflow_dispatch`** — manual trigger; optional `version` input to build a specific tag.
- **`repository_dispatch`** (`type: upstream-release`) — optional low-latency trigger from an external watcher of `…/releases.atom`.

## Maintenance

- **Signing**: set repo secrets `MINISIGN_PRIVATE_KEY` (and `MINISIGN_PASSWORD` if the key is
  password-protected) to enable `.minisig` signatures. Without them, releases ship with
  `.sha256` only (the workflow warns, doesn't fail). Publish the matching minisign **public**
  key here so consumers can verify.
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
