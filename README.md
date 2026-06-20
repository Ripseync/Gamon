# GamingAlliance content repo (Ripseync/Gamon)

This is the **content backend** for the GamingAlliance launcher. The launcher
fetches everything below at runtime, so updating mods or moving the server
endpoint is just a commit + a release here — no installer reissue needed.

## First-time seeding

A complete `manifest.json` has already been generated for you (146 mods on
MC 1.21.1 + Fabric 0.16.9, server `0.0.0.0:25566`). To seed the empty
`Ripseync/Gamon` repo:

```powershell
# 1. Clone the repo somewhere
git clone https://github.com/Ripseync/Gamon
cd Gamon

# 2. Copy the prepared files in from the launcher project
cp ..\MC_Launcher\gamingalliance\content-template\manifest.json .
cp ..\MC_Launcher\client_files\options.txt .

# 3. Commit
git add manifest.json options.txt
git commit -m "Initial manifest + options.txt"
git push

# 4. Authenticate the GitHub CLI (one-time)
gh auth login

# 5. Create the release that will hold every mod jar
gh release create mods-initial --title "Initial mod pack" --notes "146 mods (MC 1.21.1 Fabric)"

# 6. Upload every jar in client_files\mods (5-15 minutes; ~510 MB total)
gh release upload mods-initial C:\Users\NOVA-N\Documents\Anthropic\MC_Launcher\client_files\mods\*.jar
```

Once the upload finishes, the launcher can resolve every mod URL in
`manifest.json`. You can now run `installer\build.ps1` to produce the
Windows installer.

> ℹ️ The current `server.host` of `0.0.0.0` is a placeholder. When you're
> ready to point at your real MC server, edit `manifest.json` → `server.host`
> and `git push` — players' launchers will pick it up on next start. No
> installer rebuild needed.

## Repo layout

```
.
├── manifest.json              ← single source of truth for the launcher
├── patchnotes.md              ← Patch Notes tab
├── docs.md                    ← Documentation tab
├── rules.md                   ← Rules tab
├── patreon.md                 ← Patreon tiers/rewards (shown by the Patreon icon)
├── patreon/titles/            ← tier badge images referenced by patreon.md
├── announcements/             ← one .md per server announcement
├── mods/                      ← optional: store mod jars locally before uploading
└── README.md
```

## Launcher content pages

The launcher fetches these Markdown files raw on startup, so editing them on
GitHub updates every player on their next launch — no launcher rebuild:

| File | Where it shows |
|---|---|
| `patchnotes.md` | Patch Notes tab |
| `docs.md` | Documentation tab |
| `rules.md` | Rules tab |
| `patreon.md` | Patreon dialog (opened by the Patreon icon in the bottom dock) |
| `announcements/*.md` | Grouped announcements popup |

### Patreon page (`patreon.md`)

`patreon.md` renders a Markdown **comparison table** of tiers vs rewards. To set
it up:

1. Edit the numbers / ✓ / ✗ cells to match your server config.
2. Add the tier badge images under `patreon/titles/` — `supporter.png`,
   `bronze.png`, `silver.png`, `vip.png`, `elite.png`. They're referenced from
   the table by bare path (e.g. `![Supporter](patreon/titles/supporter.png)`),
   which resolves against this repo's raw root.
3. Set your real Patreon URL in `manifest.json` → `links.patreon` (the dialog's
   "Become a patron" button). Falls back to a placeholder until set.

The launcher reads `manifest.json` directly from
`https://raw.githubusercontent.com/Ripseync/Gamon/main/manifest.json`,
then downloads referenced files from the URLs you list inside it. Typically
those URLs point at **GitHub Releases assets in this same repo**.

## How updates work

1. **Mod or config change** → edit `manifest.json` on the main branch.
2. **New mod jar** → create a release (e.g. tag `mods-2026-05-30`) and attach
   the jar(s) as release assets. Copy the asset URL into `manifest.json`.
3. **New launcher version** → create a release with tag `launcher-latest` (or
   whatever tag you set in `manifest.json` → `launcher.releaseTag`) and attach
   the new `launcher.jar`. The launcher's `LauncherFetcher` will pick it up on
   next start.

## Quick start with the helper script

```powershell
# Drop your local mod jars into mods/, then:
./upload.ps1 -ReleaseTag "mods-2026-05-30" -Notes "First mod pack"
```

The script (next to this README) will:
1. Compute SHA-1 of every jar in `mods/`.
2. Create a GitHub release with those tag + notes via `gh release create`.
3. Upload each jar as a release asset.
4. Regenerate the `mods` section of `manifest.json` with the new asset URLs.

You can then `git commit && git push` the updated `manifest.json`.

## manifest.json schema

| Field | Required | Notes |
|---|---|---|
| `server.host` | yes | Hostname or IP for direct-connect |
| `server.port` | no  | Defaults to 25565 |
| `version.minecraft` | yes | e.g. `"1.20.1"` |
| `version.fabric` / `forge` / `neoforge` / `quilt` | one of | Modloader version string |
| `launcher.releaseTag` | no | Default `"launcher-latest"` |
| `launcher.assetName`  | no | Default `"launcher.jar"` |
| `mods[]` | no | Always-installed mods |
| `optionalMods[]` | no | User-toggleable mods (shown in Settings) |
| `extraFiles[]` | no | Arbitrary files placed at `<gameDir>/<path>` (configs, options.txt) |
| `links` | no | Free-form URL map (discord, website, patreon) shown in UI |

See [`manifest.example.json`](./manifest.example.json) for a fully-populated
example.

## Prerequisites for the helper script

- [`gh` CLI](https://cli.github.com/) installed and authenticated (`gh auth login`)
- This repo created on GitHub and cloned locally
- Mods placed under `mods/` (filename = the name you want on disk)
