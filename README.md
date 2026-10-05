# stasis-macos-ci

Source-free CI inputs for building the Stasis macOS bundle on GitHub's macOS runners.

This repository contains **no plaintext source**. Each `ci-macos/csrc/<pkg>.c` embeds the
obfuscated, compressed package; the Action compiles them to stable-ABI `.so` libraries on
macOS and packages the bundle. Private by design.

Build: Actions > build-macos > Run workflow (or push). Artifacts: `stasis-<ver>-macos-<arch>`.
