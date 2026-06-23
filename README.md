# Box CLI — Distribution

Public download host for the **Box CLI** (`box`).

This repository ships **prebuilt release artifacts only**. The CLI source lives
in a separate, private repository maintained by the internal team. Releases here
are published automatically by that repository's CI.

## Install

**macOS / Linux**

```sh
curl -fsSL https://get.withpotter.com/box.sh | sh
```

**Windows (PowerShell)**

```powershell
irm https://get.withpotter.com/box.ps1 | iex
```

**Homebrew**

```sh
brew install withpotter/tap/box
```

All three methods download the published release assets from this repository.
Box runs on your existing Node.js (20+); the installers do not install Node for
you.

## What's in a release

Each `v<version>` release attaches:

- `withpotter-box-<version>.tgz` — npm tarball used by the curl / PowerShell
  installers (`npm install -g <tgz>`).
- `box-bundle-<version>.tar.gz` — self-contained bundle (vendored dependencies)
  used by the Homebrew cask.

## Support

Issues are **not** tracked here. Licensed users have dedicated reporting
channels; please use those. This repository does not accept public
contributions.
