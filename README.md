# kittine-registry

The package registry backing `kittine-compiler add` / `install` / `publish`
(see [Kittine](https://github.com/buildsbybuchanan/kittine) and its
`docs/CLI.md`).

There's no server here on purpose. The registry is just this repo:

- **`index/<name>.json`** — one file per published package, listing every
  version, its tarball URL, and its sha256 checksum. `kittine-compiler
  add`/`install` read this over plain HTTPS via
  `raw.githubusercontent.com` — no auth, no server, works for anyone.
- **Tarballs** are hosted as [GitHub Release](../../releases) assets on
  this same repo, tagged `<name>-v<version>`.

## Index format

```json
{
  "name": "kittine-http",
  "versions": [
    {
      "version": "0.1.0",
      "tarball": "https://github.com/buildsbybuchanan/kittine-registry/releases/download/kittine-http-v0.1.0/kittine-http-0.1.0.tar.gz",
      "sha256": "…"
    }
  ]
}
```

`install` always sha256-verifies a downloaded tarball against this file
before extracting it, and refuses to extract on a mismatch.

## Publishing

Publishing is the one maintainer-only operation, and it's not done by
hand-editing this repo: `kittine-compiler publish` (run from a package
directory with a `kittine.toml` and a `lib.kitty` entry point) packs the
directory into a tarball, creates the GitHub Release + uploads the
tarball asset, and updates `index/<name>.json` here — all via the `gh`
CLI, authenticated with write access to this repo. See `docs/CLI.md` in
the [kittine](https://github.com/buildsbybuchanan/kittine) repo for the
full command reference.

## Package layout

A published package is just `.kitty` source files. The one required
convention: a `lib.kitty` at the package root, which is what a
dependent's bare-name import (`import { X } from 'your-package'`)
resolves to.
