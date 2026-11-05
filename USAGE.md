# Designing website templates with `box design`

> **Alpha.** The commands, the template format and the SDK can still change
> between releases. Pin the CLI version you build against.

`box design` is the part of the Box CLI for building **website templates**:
standalone Next.js projects whose text, images and links a site owner can edit
later, without touching code. This guide takes you from install to a template
that passes the checks.

## What you need

- **Node.js 20 or newer** (`node --version`).
- **Docker**, running. `check` and `dev` build and run your template in a
  container, so nothing but Docker has to be set up on your machine.
- Nothing else. `box design new` works offline.

## 1. Install

macOS / Linux:

```sh
curl -fsSL https://github.com/withpotter/box-distribution/releases/latest/download/install.sh | sh
```

Windows (PowerShell):

```powershell
irm https://github.com/withpotter/box-distribution/releases/latest/download/install.ps1 | iex
```

Check it worked:

```sh
box design --help
```

To install with npm instead, point it at the tarball of a release, for example
`npm install -g https://github.com/withpotter/box-distribution/releases/download/v0.1.0-alpha.85/withpotter-box-0.1.0-alpha.85.tgz`.

## 2. Create a template

```sh
box design new acme-cafe-01 --name "Acme Cafe" --category restaurant
cd acme-cafe-01
```

- The **id** is lowercase, hyphen-separated, and ends in a two-digit number
  (`acme-cafe-01`). It is the template's permanent name.
- Other options: `--out <dir>`, `--description`, `--author`, `--version`.
- It refuses to write into a directory that is not empty.

You get a complete, working site: a header, hero, story, features, contact
section and footer, with its own tests, checks and Dockerfile.

## 3. See it

```sh
box design dev            # http://localhost:3000
box design dev --port 4000
```

`dev` runs your template in a container against a built-in **mock tenant**, a
made-up business, so you see a realistic filled-in site without any backend.
Edit a file under `src/` and the page reloads.

## What a new template already does for you

You do not have to build these; they work from the first run, and the contract
tests in `test/contract/` fail `check` if a change breaks one:

- **Fonts.** The site owner picks the typefaces, and can give any one text its
  own. Draw owner-editable words with the SDK's `Text` and `Link` so the pick
  applies. A design that must keep its own type can opt out with
  `customFonts: false` and a reason.
- **Footer.** `<SiteFooter>` draws the brand, contact lines, social icons, link
  columns, legal links and the small print. Change a part by passing it as a
  prop; do not write a footer from scratch.
- **Pictures.** Print every picture with `Picture`, so an empty slot is still
  clickable in the editor. A bundled fallback shows until the owner uploads one.
- **Colours and buttons** reach the page as the platform expects.

The guides inside the generated project (`AGENT.md` first) explain each one.

## 4. How a template is laid out

| Path | What it is |
| --- | --- |
| `src/strings.ts` | **The only place for default text, links, image paths and design values.** Page and section definitions, with every editable field's label, type and default. |
| `src/components/` | The sections (`Header`, `Hero`, `Story`, `Features`, `Contact`, `Footer`) and `SiteView`, which assembles them. |
| `src/app/` | Next.js app files: layout, global CSS, error pages. |
| `template.json` | The manifest the editor reads (fields, sections, pages). **Generated** from `src/strings.ts`; never edit by hand. |
| `fixtures/` | Sample tenants used by `dev` and the tests (a full one, an empty one, a multi-page one). |
| `public/` | Static files such as the default images. |
| `vendor/` | The bundled Box render SDK. Leave it alone. |
| `scripts/` | The validators, manifest builder and smoke tests that `check` runs. |
| `src/lib/`, `src/pb/`, `src/proxy.ts` | The runtime contract with the Box website engine. Leave these alone unless you know why. |

### Content model

A site is a set of **pages**, each with **sections**, each with **fields**. A
field has a `name`, a human `label`, a `type` and a `default`:

```ts
{ name: 'title', label: 'Headline', type: 'text', default: 'A good business deserves a good first impression' },
{ name: 'lede',  label: 'Under the headline', type: 'textarea', default: '…' },
{ name: 'ctaLink', label: 'Main button link', type: 'url', default: '#contact' },
{ name: 'image', label: 'Picture', type: 'image', default: '/images/hero.svg' },
{ name: 'background', label: 'Background colour', type: 'color', default: '' },
```

The owner of a site overrides any of these; whatever they leave alone falls
back to your default.

### Rendering a field

Components never print a string directly. They render through the SDK, which
marks each piece so the editor knows what it is:

```tsx
import { Block, Image, Link, Text } from '@withpotter/render-react';

<Block markers={markers} blockId="hero" page="home" className="hero">
  <Text markers={markers} field="title" value={f.title ?? ''} as="h1" className="display" />
  <Link markers={markers} field="cta" hrefField="ctaLink" href={f.ctaLink ?? ''} label={f.cta ?? ''} className="button" hideWhenEmpty />
  <Image markers={markers} field="image" src={f.image} alt={f.imageAlt ?? ''} width={800} height={600} />
</Block>
```

Use `Text` for words, `Link` for links and buttons, `Image` for pictures, and
`Block` around each section. Copy the pattern of an existing component such as
`src/components/Hero.tsx` when you add a new one.

## 5. The rules (and why)

Everything that makes a template editable depends on one idea: **the site's
content comes from the owner's data, and your code only supplies defaults.**
`check` enforces this, so these are not style suggestions:

- **No hardcoded copy.** Words, links and image paths in components or CSS
  are rejected. Put them in `src/strings.ts` and render them through the SDK.
- **No hardcoded design values** where the rules call for a token (for example
  colours and sizes belong in the design values in `strings.ts`, not scattered
  in components).
- **No network calls.** A template never fetches anything itself. The engine
  hands it its content; there is no API URL to configure.
- **Manifest in step with code.** `template.json` must match `src/strings.ts`.
  After you change fields run:

  ```sh
  npm run manifest
  ```

- **Previews and static files stay portable.** Use the SDK's `Link` and
  `Image` rather than raw `<a href="/…">` or `<img src="/…">`, so the same
  build works under a preview path and under a custom domain.

Each rule has an id such as `copy/*`, `value/*`, `network/*`, `data/*` or
`manifest/*`, shown in the failure message with the file and line.

## 6. Check it

```sh
box design check            # validate, then typecheck + tests + build in Docker
box design check --verbose  # show the full output of each step
box design check ./other-dir
box design check --local    # run on this machine instead of in Docker
```

`check` first validates `template.json` against the v1 schema and the rules
above, then runs the TypeScript check, the unit tests and a production build.
Fix what it reports, run it again, and it should end green before you
publish.

**Changing defaults changes expected output.** The tests compare rendered
output with saved "golden" files. When you intentionally change defaults,
refresh them:

```sh
UPDATE_GOLDEN=1 npm test
```

and review the diff before keeping it.

## 7. Publishing

> Hosted publishing is **not available yet** in the alpha.

- `box design publish --local --engine <dir>` registers the template with a
  website-engine checkout on your machine (or set `BOX_WEBSITE_ENGINE`). The
  engine is not part of this download, so this is for people we have given
  access to it.
- `box design unpublish <id> --local --engine <dir>` removes it again.
- `box design setup` and `box design legal` are for the legal documents an
  author will accept before publishing to the hosted platform. That service is
  **not available yet**, so both fail with an HTTP 403 today. You do not need
  them to build, check or run a template.

Until then, the useful loop is `new` → `dev` → `check`: you can design a
complete, verified template and keep it in your own repository.

## What the alpha does not do yet

- **No hosted publish**, and no marketplace listing.
- **`check` does not yet verify that every visible string is editable**; it
  catches hardcoded copy, but you should still open the editor on a staging
  site and click through your fields.
- **The SDK is copied into the project** (`vendor/`) and there is no upgrade
  command yet. A template keeps the SDK version it was created with.
- **Stores need the platform.** Content comes in as pages, sections and fields.
  A shop template's cart, checkout and member area go through the platform's
  own operations, so you can only try them with the mock tenant, not live data.

## Troubleshooting

| Symptom | Likely cause |
| --- | --- |
| `Cannot connect to the Docker daemon` | Start Docker Desktop (or your Docker service) and retry. |
| `check` fails on the manifest | You changed `src/strings.ts` without running `npm run manifest`. |
| `refusing to write … not empty` | `new` only writes into an empty or new directory; choose another `--out`. |
| Build fails pulling the base image | Probably a Docker Hub rate limit. Retry later. |
| Port already in use | `box design dev --port 4000`. |
| A copy/value rule flags text you need | Move the text into `src/strings.ts` as a field default; do not suppress the rule. |

## Getting help

Issues are not tracked in this repository. Licensed users have dedicated
reporting channels; please use those.
