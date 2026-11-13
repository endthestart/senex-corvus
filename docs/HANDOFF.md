# Corvus handoff

Written 2026-11-13, before a break. It stands alone: no session transcript is needed to resume.

## What Corvus is

Corvus (SENEX Corvus, formerly Andermic Photo Importer) is a native macOS app that imports new
photo and video originals from camera cards, USB cameras and folders into ordinary date folders. It
never deletes from a card, never overwrites a destination file and never invents a capture date; it
has no account and no catalog. It is Swift and AppKit with no Xcode project, and it bundles a pinned
Perl and ExifTool for metadata. The [specification](SPEC.md), the [roadmap](ROADMAP.md), the
[validation log](VALIDATION.md), the [decisions](decisions/) and [AGENTS.md](../AGENTS.md) hold the
detail. It belongs to the SENEX photo suite with SENEX Pictor (a nightscape stacker in its own repo,
[endthestart/senex-pictor](https://github.com/endthestart/senex-pictor)); the two share
[design tokens v1](DESIGN-TOKENS.md).

## Current state

- Source: [endthestart/senex-corvus](https://github.com/endthestart/senex-corvus) (public, MIT),
  renamed from `andermic-photo-importer` on 2026-11-05; old URLs redirect.
- Released: the [0.2.0-alpha.1 source preview](https://github.com/endthestart/senex-corvus/releases/tag/v0.2.0-alpha.1),
  published 2026-11-05 from `737817f`. Source only; no app binaries are published anywhere.
- `main` is `737817f`. `./test.sh` passes 261 synthetic checks, and a fresh clone tests and builds
  on the Intel Mac Pro. CI builds, tests and package-verifies Apple silicon and Intel on every push
  to `main` and every pull request.
- Done in October and November: the rename to Corvus (bundle ID `com.senexapps.corvus`), the design
  shared with Pictor, the [October code review](audit/CLEANUP-2026-10.md) and its fixes, ExifTool
  13.60, the house-style formatter in CI, and the split of `MainWindow.swift` and `ImportCore.swift`.
- Deleted merged branches (2026-11-05), with their tips:
  `claude/corvus-pictor-design-0851ec` f7b3a0198c53fe41f45a9d01a07fc68dbb654d00,
  `claude/corvus-split-mainwindow` e49d5715c792002008e6b7b3c84a8e450e14334d,
  `claude/corvus-split-importcore` 58963701775675f8e800a4ec90fa8b70346c26ee and
  `claude/corvus-release-0.2.0-alpha.1` ef08cae10a5f1b4f70cd9af61355610ab32c8173.

## Build, test and release

```sh
xcode-select --install        # once: Apple's Command Line Tools
git clone https://github.com/endthestart/senex-corvus.git
cd senex-corvus
./test.sh                     # synthetic safety tests
./build.sh                    # dist/Corvus.app for this Mac's architecture
```

- The first build downloads the pinned Perl and ExifTool sources and checks them by SHA-256.
- Packaging: `./scripts/package.sh` then `./scripts/verify-package.sh`. Performance:
  `./scripts/performance-exercise.sh`. GUI: `./scripts/ui-exercise.sh`, which drives the app with
  generated images and needs an unlocked screen (a screensaver counts as locked).
- Install without building: each CI run on `main` keeps Apple silicon and Intel development ZIPs as
  artifacts for seven days (`gh workflow run ci.yml --ref main`, then
  `gh run download <run id> --name development-build-arm64`). They are ad hoc signed and not
  notarized, so macOS needs Open Anyway in System Settings → Privacy & Security on first launch.
- Release: set `VERSION`, push a tag `v<VERSION>-<label>`; CI builds both architectures and creates a
  draft source prerelease with no binaries; the owner reviews and publishes it
  ([docs/RELEASE.md](RELEASE.md)).
- Clone into an ordinary folder, not under `/tmp` or `/private/tmp`. Foundation shortens
  `/private/tmp/…` to `/tmp/…`, and `/tmp` is a symlink, which the import checks refuse, so
  `./test.sh` fails there. The same refusal applies to a source or destination chosen under `/tmp`.

## Where things live

| Thing | Where | After the move off the Mac Pro |
| --- | --- | --- |
| Source | GitHub | The same; any clone builds on a Mac |
| Build, tests, packaging | Mac Pro (Intel) and GitHub CI (`macos-15`, `macos-15-intel`) | Any Mac with macOS 13 or later and the Command Line Tools, plus CI. The MiniPC runs Debian and cannot build it |
| GUI exercise | Mac Pro, needs an unlocked screen | Any Mac with an unlocked screen; not run in CI |
| Real-card and camera checks | The owner's Mac and card reader | The same, only with the owner's authorization |
| Build output (`build/`, `dist/`) | Repo folder, ignored by git | Do not move; rebuild |
| Settings, presets and import reports | Application Support on the Mac that runs the app | Stay with the installed app |
| Backup of the 2026-10-05 app build | External storage | The same |

## Open items, in priority order

1. Record the owner's hands-on run (real card, card reinsertion, VoiceOver) in
   [validation](VALIDATION.md); it was planned for 2026-11-07 and no notes have reached the repo.
2. Run `./scripts/ui-exercise.sh` on Apple silicon, and on Intel against `main`; it last passed on
   Intel on PR #5's tree.
3. Decide on the Apple Developer Program. Developer ID signing, notarization and live
   qualification are required before any public app binary ([roadmap](ROADMAP.md)).
4. Background-scan heat: options in owner question 3 of the [audit](audit/CLEANUP-2026-10.md).
5. Later updates (history, filename templates, second copies, source deletion) stay in the backlog
   and need an explicit product decision.

## Owner steps still pending

- Merge PR #8, which adds this handoff and the links to the new repository name.
- Decide on `codex/sd-card-metadata-cache` on GitHub. Its PR #3 was closed and its ADRs 0011 to 0014
  are on `main`, but the branch has not been confirmed empty, so it was kept.
- Decide on the unpublished `v0.1.0-alpha.1` draft release, which still holds four development
  files from 2026-10-04.
- Remove the worktree `.claude/worktrees/corvus-pictor-design-0851ec` once PR #8 is merged
  (`git worktree remove`), and fast-forward the main checkout's `main`.
- Pictor items that touch this suite: rename its Gitea repo to `senex-pictor`, and name a NAS folder
  for its `.sources` copy (approved, not done for lack of a path).

## Known risks

- SSH to github.com often timed out from the Mac Pro in November; HTTPS with
  `gh auth git-credential` worked.
- Ad hoc builds change signature on every build, so macOS may ask again for removable-volume access.
- Real-camera metadata, grouping and real-card performance are not qualified; only synthetic
  fixtures are tested.
- CI artifacts expire after seven days; nothing installable is kept elsewhere.

## First three things on resume

1. Clone or pull `main`, then run `./test.sh` and `./build.sh` to confirm the toolchain still works.
2. Read the open items above and the owner's hands-on notes, if any were written down.
3. Run `./scripts/ui-exercise.sh` on an unlocked Mac before changing any import code.
