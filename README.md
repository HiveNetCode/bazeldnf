# bazeldnf (Hive patch of brianmcarey/bazeldnf v0.5.9-2)

Source tree of [brianmcarey/bazeldnf](https://github.com/brianmcarey/bazeldnf)
release **v0.5.9-2**, with one surgical patch in `pkg/repo/cache.go`:
repository metadata (repomd primary/filelists) is read transparently as gzip
*or* zstd, sniffing the magic bytes instead of assuming gzip.

## Why

CentOS Stream 10 mirrors now serve repodata in zstd compression only
(`*.primary.xml.zst`, no `.gz` variant in `repomd.xml`). bazeldnf v0.5.9-2
downloads the file it is told to (it follows the repomd href) but then
unconditionally `gzip.NewReader`s the cached file, failing with
`gzip: invalid header` on any zstd-only repo. This broke every
`make rpm-deps-cs10` run against the live CS10 repos.

Upstream `rmohr/bazeldnf` fixed the same problem in v0.99.x, but that line
requires Bazel 8; the KubeVirt tree (and this pin) runs Bazel 6.5, so the
patch is applied on top of the v0.5.9-2 source instead.

## Changes vs v0.5.9-2

- `pkg/repo/cache.go`: new `openCompressed` helper (gzip/zstd sniff) used by
  `CurrentPrimary` and `CurrentFilelistsForPackages`; the gzip-only reads are
  replaced by it.
- `go.mod` / `go.sum`: `github.com/klauspost/compress` promoted to a direct
  requirement (same version as before, `v1.11.1`); `zstdReadCloser` adapter
  for the older `*zstd.Decoder.Close()` signature.

No CLI, rule, or BUILD-file behavior changed — everything else is identical
to v0.5.9-2, so WORKSPACE wiring (`defs.bzl`, `repositories.bzl`) is
unchanged.

## Usage (WORKSPACE)

```starlark
http_archive(
    name = "bazeldnf",
    sha256 = "<sha256 of the tag archive>",
    strip_prefix = "bazeldnf-0.5.9-2-hc1",
    url = "https://github.com/HiveNetCode/bazeldnf/archive/refs/tags/v0.5.9-2-hc1.tar.gz",
)
```