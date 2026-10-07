# Upstream contribution packaging

## Pull requests

All three draft PRs target `atchad/glance:main` at upstream base `b43d912`.
Each contains a single independent commit; none includes another PR's changes.

| PR | Branch | Package | Original local commits |
| --- | --- | --- | --- |
| [#68](https://github.com/atchad/glance/pull/68) | `madisonbullard/dashboard-keybindings` | Unified panel, configurable keyboard controls, focus dismissal, Snoozed collapse, development/signing checks | `6afe0f2`, `a930d14`, `3ea6785`, `53e692c` |
| [#70](https://github.com/atchad/glance/pull/70) | `madisonbullard/compact-pr-rows` | Two-line rows, attention icons, elapsed time, native tooltips and hover checks | `4977b72`, `94ba89f`, `f34f491`, `718ab98`, `dc41556` |
| [#69](https://github.com/atchad/glance/pull/69) | `madisonbullard/repository-colors` | Stable repository colors, settings/editor deep links, persistence/recovery checks | `b26816b` |

The early explicit window-height setting was superseded by native panel resizing;
the dashboard PR includes the final behavior, not that temporary intermediate UI.
The PRs retain the final behavior of all ten original commits, with existing
upstream callback APIs substituted where needed to make the packages independent.

## Recovery and integration

- Original local `main` and working tree were left unchanged at `dc41556`.
- `backup/local-changes-2026-10-07` preserves the original local tip and history.
- The `fork` remote is `https://github.com/madisonbullard/glance.git`.
- `origin` remains `https://github.com/atchad/glance.git`.
- `madisonbullard/integration-reference` preserves the combined implementation
  on current upstream, plus test-runner corrections and the color-window harness.
  Its entire production `Sources/` tree is byte-for-byte identical to `dc41556`.
- `madisonbullard/pr-previews` contains synthetic preview assets, kept out of
  feature diffs and normal Glance preferences.

The reference is a known combined tree, **not** an automatic merge of the three
independent PR branches. The PRs overlap in `Views.swift`, `Models.swift`, settings,
app wiring, documentation, and CI. After one is accepted, rebase the remaining
branches to keep their diffs focused. Resolve overlaps by retaining both feature
behaviors, using this reference for the intended combined command/UI wiring.
Do not replace upstream files wholesale if upstream has since changed them.

## Validation

| Package | XCTest | Additional checks |
| --- | --- | --- |
| Dashboard/keybindings | 129 tests; 0 failures; 2 intentional skips | 70 keybinding checks; 31 native panel checks |
| Compact rows/tooltips | 131 tests; 0 failures; 1 intentional skip | 12 rendered tooltip-caption checks across initial/reopened/refreshed windows |
| Repository colors | 134 tests; 0 failures; 1 intentional skip | 12 actual Settings-window deep-link checks |
| Combined reference | 150 tests; 0 failures; 2 intentional skips | 36 native panel checks; 12 Settings-window deep-link checks |

The intentional skips are the opt-in benchmark and, where present, the GUI test
wrapper inside an unbundled xctest process. Bundled GUI runners execute separately.

Each standalone PR built as a universal arm64/x86_64 release app and passed
`codesign --verify --deep --strict`, shell syntax checks, and `git diff --check`.

Local toolchain details:

- The default macOS 27 SDK cannot load SwiftUI macros in this installation.
  Compilation and tests were pinned to the installed macOS 26.5 SDK.
- XCTest used `DEVELOPER_DIR=/Applications/Xcode.app/Contents/Developer` and
  `swift test --build-system native --sdk /Library/Developer/CommandLineTools/SDKs/MacOSX26.5.sdk`.
- GUI scripts used `GLANCE_SDK_PATH` with the same SDK. The upstream build script
  on the row/color branches does not support that variable, so those builds used
  a temporary local Swift wrapper to supply `--sdk`; no wrapper was committed.
- The local `lipo -verify_arch` invocation reports an argument error, even for
  the successfully built universal app. `lipo -info` confirmed both architectures.

The **combined** tooltip-hover runner stopped before checking any captions because
the dashboard lost focus. The standalone row PR's 12 rendered-caption checks
passed. Combined focus dismissal and tooltip registration are covered separately,
but the combined end-to-end hover run is not claimed as passing; rerun it in an
idle GUI session before marking that integrated scenario verified. Failed
experimental changes to its startup timing were removed.

At PR creation, upstream CI and CodeQL runs reported `action_required` for these
fork contributions. An upstream maintainer must approve their workflows; the
author has read-only access to upstream. No CI success is claimed.
