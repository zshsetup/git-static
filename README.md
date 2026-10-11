# git-static

Portable, relocatable builds of [git](https://git-scm.com) with zlib, Mbed TLS, and curl linked
in statically, built with [zig](https://ziglang.org) as the C toolchain.

- Linux glibc: only glibc is linked dynamically, and only symbols up to glibc 2.17, so the
  binaries run on anything from CentOS 7 onwards
- Linux musl (`--static`): fully static, no runtime dependencies at all (e.g. Alpine)
- macOS: only libSystem and system frameworks are dynamic (macOS has no fully static binaries)

Client-only: perl, python, Tcl/Tk, gettext, and the server-side programs (daemon, http-backend,
shell, cvsserver) are left out.

## Prerequisites

- `zig`, GNU make (`make` 3.81 of macOS is enough), `perl`, `git`
- `autoconf`, `automake`, `libtool` (the git tree of curl ships no `configure` script)
- On macOS: Xcode Command Line Tools (SDK)

## Installation

Clone the repo with submodules (shallow, the history of the vendored projects is not needed):

```shell
git clone --recurse-submodules --shallow-submodules https://github.com/zshsetup/git-static
cd git-static
```

## Usage

```shell
./build.sh
```

This builds zlib, Mbed TLS, and curl from `vendor/` as static libraries, then builds git against
them. Output is installed under `dist/`:

- `dist/bin`, `dist/libexec/git-core`, `dist/share`: a relocatable git install; copy the tree to
  any prefix (e.g. `~/.local`) and run `bin/git`
- `dist/share/man`: the man pages, which git publishes prebuilt for each release on kernel.org
- `dist/share/bash-completion/completions/git`: the bash completion (zsh and fish ship their own),
  and `dist/share/git-core/contrib/completion/git-prompt.sh` for the branch in the prompt
- `dist/lib/libgit.a`: static library of git

Pass `--static` for the fully static musl build on Linux (output in `dist-static/`), `--clean` to
remove previous build output first, `--jobs N` to control parallelism, and `--rust` to also build
optional Rust parts of git.

Pass `--tls openssl` to use OpenSSL instead of Mbed TLS (output in `dist-openssl/`). Released
binaries use Mbed TLS because it is available under GPL-2.0-or-later, while the Apache-2.0
license of OpenSSL is considered incompatible with GPL-2.0-only of git; building the OpenSSL variant for your
own use is fine.

HTTPS reads the CA certificates in `/etc/ssl/certs` on Linux and `/etc/ssl/cert.pem` on macOS.
Systems without them can use the Mozilla bundle from [curl.se](https://curl.se/docs/caextract.html)
in `share/git-core/certs`, e.g. `export GIT_SSL_CAPATH=~/.local/share/git-core/certs`; only set it
there, as it replaces the certificates of the system.

With `--tls openssl`, the default locations of OpenSSL are used instead, which miss RHEL/CentOS 7 and 8;
point git at their bundle with `git config --global http.sslCAInfo /etc/pki/tls/cert.pem` there, or
at the Mozilla bundle with `GIT_SSL_CAINFO=<prefix>/share/git-core/certs/cacert.pem`.

To run the smoke tests against the build:

```shell
./tests/run_smoke_tests.sh            # dist/
./tests/run_smoke_tests.sh --static   # dist-static/
```

## Releases

Publishing a GitHub release tagged with the git version (e.g. `v2.55.0`, matching the tag checked
out in `vendor/git`) builds and attaches `git-static-<platform>.tar.gz` for `macos-arm64`,
`linux-amd64`, `linux-arm64`, `linux-musl-amd64`, and `linux-musl-arm64`, together with the
sources they were built from (`git-static-sources.tar.gz`).

To update a vendored project, check out a new release tag in its submodule and commit it, e.g.
`git -C vendor/git fetch --depth 1 origin tag v2.56.0 && git -C vendor/git checkout v2.56.0`.

## Acknowledgments

This repository builds and vendors the following upstream projects, unmodified, as git
submodules under `vendor/`. Credit goes to their respective authors:

- [git](https://git-scm.com) by Linus Torvalds, Junio C Hamano, and contributors
- [curl](https://curl.se) by Daniel Stenberg and contributors
- [Mbed TLS](https://www.trustedfirmware.org/projects/mbed-tls/) by the Mbed TLS Contributors
- [OpenSSL](https://www.openssl.org) by the OpenSSL Project Authors (only with `--tls openssl`)
- [zlib](https://zlib.net) by Jean-loup Gailly and Mark Adler

## License

MIT for the code of this repository. See NOTICE for the licenses of the upstream projects and the
resulting binaries, which are GPL-2.0 (git).
