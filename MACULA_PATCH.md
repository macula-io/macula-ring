# Macula fork of `ring` 0.17.14

This is a vendored fork of [briansmith/ring](https://github.com/briansmith/ring)
at version `0.17.14`, with a single patch applied so the crate's `build.rs`
recognises bare-metal targets (`os = "none"`) and compiles its assembly
files the same way it does for Linux / Redox / other LINUX_ABI hosts.

Used by [macula-kernel](https://codeberg.org/macula-internal/macula-kernel)
via `[patch.crates-io]` to crypto-back the kernel-resident Quinn QUIC
implementation. Upstream `ring` produces undefined-symbol link errors on
`x86_64-unknown-none` targets because its `ASM_TARGETS` list keys on
`CARGO_CFG_TARGET_OS` and excludes `"none"`.

## The patch

```diff
 const LINUX_ABI: &[&str] = &[
     ...
     "linux",
+    "none",
     "redox",
     ...
 ];
```

That's it. The assembly itself is OS-agnostic position-independent x86_64
machine code in ELF format; the only thing the upstream gate prevents is
the build script from invoking `cc-rs` to compile those `.S` files for
us.

## Upstream contribution

The intent is to file an upstream PR adding `"none"` to `LINUX_ABI` so
no future kernel projects (Redox-style, Theseus-style, Hermit-style, or
Macula-style) need their own fork for the same reason. Until that lands,
this fork is what `macula-kernel` consumes.

## Versioning policy

Track `ring` upstream releases of 0.17.x. Re-roll this fork by:

1. Downloading the new `ring-X.Y.Z.crate` tarball from crates.io.
2. Extracting into a fresh checkout.
3. Re-applying the single LINUX_ABI patch.
4. Tagging as `vX.Y.Z-macula1`.
5. Bumping the `git` ref in `macula-kernel`'s `Cargo.toml`.

When the upstream PR lands, retire this fork entirely.

## License

Inherited from upstream `ring` — see `LICENSE`, `LICENSE-BoringSSL`,
`LICENSE-other-bits`. No new code, only the three-character LINUX_ABI
addition.
