# Box CLI — Distribution

Public download host for the **Box CLI** (`box`).

This repository ships **prebuilt release artifacts only**. The CLI source lives
in a separate, private repository maintained by the internal team. Releases here
are published automatically by that repository's CI.

## Install

**macOS / Linux**

```sh
curl -fsSL https://github.com/withpotter/box-distribution/releases/latest/download/install.sh | sh
```

**Windows (PowerShell)**

```powershell
irm https://github.com/withpotter/box-distribution/releases/latest/download/install.ps1 | iex
```

Both installers download the published release assets from this repository.
Box runs on your existing Node.js (20+); the installers do not install Node for
you.

## Guides

- [Designing website templates with `box design`](USAGE.md) — create, preview
  and check a template (alpha).

## What's in a release

Each `v<version>` release attaches:

- `install.sh` / `install.ps1` — the installers above.
- `withpotter-box-<version>.tgz` — npm tarball used by the installers
  (`npm install -g <tgz>`).
- `box-bundle-<version>.tar.gz` — self-contained bundle (vendored dependencies).

## Support

Issues are **not** tracked here. Licensed users have dedicated reporting
channels; please use those. This repository does not accept public
contributions.
