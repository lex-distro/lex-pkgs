# lex-pkgs — recipe repository for the lex package manager

Recipes are JSON `*.lexbuild` files (`kind = recipe`).

## Layout

```
pkg/
  x86_64/          # arch-specific recipes (lex.conf Include points here)
  aarch64/         # same layout per arch
  any/             # arch-independent recipes (future Include)
groups/
  base.txt         # named sets, one package per line, file order = install order
```

## Recipe schema

```json
{
  "kind": "recipe",
  "name": "hello",
  "version": "2.12",
  "arch": ["any"],
  "deps": [],
  "homepage": "https://www.gnu.org/software/hello/",
  "description": "...",
  "binary": {
    "url": "https://.../pkg.tar.gz",
    "sha256": "<sha256 of the tarball>",
    "install": ["cp -a ... /usr/..."]
  },
  "source": {
    "url": "https://.../pkg-1.0.tar.gz",
    "sha256": "<sha256 of the tarball>",
    "patches": [],
    "build": ["./configure --prefix=/usr", "make"],
    "install": ["DESTDIR=$DESTDIR make install"],
    "post_remove": ["ldconfig"]
  },
  "post_remove": ["ldconfig"]
}
```

- At least one of `binary` / `source` is required. `lex` prefers
  `binary` unless `lex.conf` says `Prefer = source` (or `--method=`).
- `sha256` must be the real tarball checksum — it is verified on every
  install. Get it with `sha256sum file.tar.gz`.
- `install` commands run with a `DESTDIR` staging dir for source builds;
  `remove` deletes the recorded manifest, so no `remove` stanza is needed.
  `post_remove` is only for non-file side effects (`ldconfig`, cache updates).
- Package `name` must match `^[A-Za-z0-9._-]+$` and the file name.

## Client config (`lex/lex.conf`)

```ini
[stable]
Server = https://github.com/mikuripa/lex-pkgs
Include = pkg/x86_64/

[options]
Prefer = binary
InfoMode = combined
```

`lex sync` clones/pulls `Server` into `cache/repo/<name>`, then reads
`<repo>/<Include>/*.lexbuild`. Validate locally with
`lex info <pkg> --lexbuild` after syncing.
