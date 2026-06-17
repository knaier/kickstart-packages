# kickstart-packages

Decentralized package + daemon catalog for [kickstart](https://github.com/knaier/kickstart).
No monolithic index — each package version is its own small YAML file, navigated
dynamically by path. Scales to thousands of packages; works on GitHub **or** a
plain static host (S3, internal mirror).

## Layout

```
packages/
  <name>/
    meta.yaml          # OPTIONAL: default pin + channels + metadata (type, source, …)
    <version>.yaml     # one file per published version: how to FETCH + how to RUN
```

- **`name`** is the folder; **`version`** is the filename. The path *is* the
  metadata — there is no central index to maintain.
- **`<version>.yaml`** has `targets` (per `os/arch`: `url`, `sha256`, `archive`,
  `bin`) and, for a runnable package, a **`daemon`** block (how the kickstart agent
  runs + supervises it: `exec` template, `health`, `post_start`, …). A pure tool
  (e.g. `mongosh`) has `targets` only.
- **`meta.yaml`** is optional. It does **not** list versions. It may pin a
  `default:` version, define **channels** (`stable`/`edge`/`8` → a version), and
  carry descriptive metadata (`type`, `source`, `homepage`).

## Version resolution

`ks pkg install mongodb` (no version):
1. if `packages/mongodb/meta.yaml` pins `default:` → use it;
2. else → the **highest published** `<version>.yaml` in the folder.

`ks pkg install mongodb@stable` / `@8` → the channel from `meta.yaml`.
`ks pkg install mongodb@8.0.4` → that file directly.

Local clones resolve by reading the directory; remote hosts resolve the default /
highest version by listing the folder (GitHub contents API, or the host's listing).
A `meta.yaml` `default:` pin removes the need for any listing — the static-host path.

## Using it

kickstart ships pointed at this repo by default. To use your own fork/mirror:

```
ks config pkg repo set https://raw.githubusercontent.com/<org>/kickstart-packages/main   # or a clone dir / S3 base
ks config policy lock packages.index_url --value <url>     # admin: forbid operator override
```

## Adding a package / version

Add `packages/<name>/<version>.yaml` (and optionally `meta.yaml`). That's it —
commit and it's live. `url`s point at the upstream origin by default; an optional
release-asset mirror (binaries as signed GitHub Release assets, `url`s rewritten to
them) is a future hardening step.
