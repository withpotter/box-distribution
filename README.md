# Box CLI — Distribution

Public download host for the **Box CLI** (`box`).

This repository ships **prebuilt release artifacts only**. The CLI source lives
in a separate, private repository maintained by the internal team.

The CLI does two separate jobs. Pick the part you need; they share only the install.

| | **Part 1: Cloud** | **Part 2: Design** |
| --- | --- | --- |
| For | Running your own **Box**: the Potter commerce engine plus its console, on your own infrastructure under your own brand | Making a **template**: a standalone website or store design that a Box serves |
| Commands | `box create`, `auth`, `install`, `start`, `deploy`, `config`, `status`, `update`, `upgrade` | `box design new`, `check`, `dev`, `demo`, `publish --local` |
| You need | A license key and a control plane address | Docker. No license key and no control plane to build |
| Start at | [Part 1](#part-1-cloud-run-your-own-box) | [Part 2](#part-2-design-make-a-template) |

## Install

**macOS / Linux**

```sh
curl -fsSL https://github.com/withpotter/box-distribution/releases/latest/download/install.sh | sh
```

Or with Homebrew: `brew tap withpotter/tap && brew install withpotter/tap/box`.

**Windows (PowerShell)**

```powershell
irm https://github.com/withpotter/box-distribution/releases/latest/download/install.ps1 | iex
```

The installers download the published release assets from this repository. Box
runs on your existing Node.js (20+); the installers do not install Node for you.
Check the install with `box --version`, and update later with `box upgrade`.

---

## Part 1: Cloud, run your own Box

`box` scaffolds, activates, installs and runs a **Box**: the full Potter commerce
engine plus its console. The scaffold ships empty; your license entitlements are
pulled in when you activate. The only credential you need is your **license key**
(no GitHub account is required).

You need Node.js 20+, a license key, PostgreSQL 14+ (your own, or an isolated
database the CLI starts in Docker), two free TCP ports (defaults 4000 and 3010)
and about 2 GB of disk.

```sh
box create my-box --control-plane <your-control-plane-url>   # scaffold an empty box
cd my-box
box auth --key <license-key>                                  # activate the license
box install                                                   # install the entitled packages
box start my-box 4000 3010                                    # run the engine and console locally
box deploy --register-domains                                 # route your domains via the control plane
```

| Command | What it does |
| --- | --- |
| `box create <name>` | Create a new box from the bundled scaffold. |
| `box auth --key <license>` | Authenticate the license and connect to the registry. |
| `box install` | Install the entitled engines and dependencies. |
| `box start <name> <backend-port> <dashboard-port> [db-name]` | Run the box (engine and console). |
| `box config` | View or set the box's service URLs and deploy defaults. |
| `box deploy` | Generate deploy artifacts, or route domains with `--register-domains`. |
| `box status` | Show the activation state and entitlements. |
| `box update` | Update the entitled engine packages to the latest release. |
| `box upgrade` | Upgrade the CLI itself. |

Ask your Box vendor for the control plane address and your license key.

---

## Part 2: Design, make a template

A **template** is a standalone Next.js project whose words, pictures, colours and
links the site owner can edit later without touching code. Everything runs in
Docker, so Docker is the only thing to set up. No license key and no control plane
are needed to build one.

```sh
mkdir templates && cd templates
box design new acme-cafe-01 --name "Acme Cafe" --category food   # scaffold ./acme-cafe-01 (works offline)
cd acme-cafe-01
box design dev --port 3000    # the template with a mock tenant; prints the address
box design check              # schema copy, install, validate, typecheck, tests, build
```

A new template already handles the platform for you: owner-editable everything
(`check` fails on hardcoded copy, colours, pictures or links), site-wide and
per-text typefaces, a footer with social icons and link columns, clickable
picture slots, and colours and buttons that follow the owner's choices. A set of
contract tests ships in the project and runs in `check`, so a change that breaks
one of these fails the build.

Read the [step-by-step guide](USAGE.md) for the full walk-through. This is an
**alpha**: publishing to a hosted platform is not available yet (`publish --local`
runs a template in an engine you run yourself), and the SDK is copied into each
template, so a template keeps the version it was made with.

---

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
