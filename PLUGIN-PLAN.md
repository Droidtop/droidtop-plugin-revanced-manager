# ReVanced Manager as a droidtop plugin

Private plan. Upstream: https://github.com/ReVanced/revanced-manager
(GPL-3.0). Patches installed Android apps (YouTube etc.) using ReVanced
patch bundles, via `app.revanced.patcher`.

**Plugin model this plan is written against (2026-09-25):** droidtop plugins
run in **droidtop's own process/context**, never through enginehost. A
plugin is Python, a native Kotlin/`.so` bundle (arm64-v8a + x86_64), or
another kind droidtop's host adds support for. The sandbox exists for
stability, not security. Root is an optional enhancement only, never
required, never the default path.

## What it would contribute

Not a library source and not a launcher surface -- ReVanced Manager patches
*apps the user already has*, not games. Its droidtop-visible contribution is
narrower than the content-source plugin's: a Desktop-mode status tile ("N apps patched, M patches
available") plus an action on an app's own settings/details row ("Re-patch
with latest bundle", "Check for patch updates"). Search/browse-and-select
patch UI stays in the real app; droidtop only needs the summary state back
and a way to kick off a patch job and learn its result.

## Reuse as a native bundle

Good fit: this is already a Kotlin/JVM Android app, not Flutter, so it maps
directly onto droidtop's native Kotlin/`.so` plugin kind. The reusable core:
- `app/src/main/java/app/revanced/manager/domain/repository/{PatchBundleRepository,PatchOptionsRepository,InstalledAppRepository,DownloadedAppRepository}.kt`
  and `domain/sources/*` -- bundle fetching/selection and patch-option state,
  no UI dependency.
- `domain/installer/RootInstaller.kt`, `service/RootService.kt`, and the
  `IRootSystemService` AIDL -- the root install path (see below).
- The actual patcher invocation (wherever `app.revanced.patcher` is driven
  from `domain/manager` / `domain/repository`) -- this is the part that does
  real work and is worth keeping.

Must be stripped: all of `ui/` (Compose screens, navigation, theming),
`di/` modules that wire Compose/Activity scope, and anything importing
Android `Activity`/`Application` context beyond what's needed to read/write
files and invoke the patcher. This is a substantial trim, not a rename --
expect the plugin to keep maybe a third of `app/src/main/java`.

## Root

Optional enhancement, not the default, matching droidtop's rule. Two install
paths already exist upstream:
- **Root** (`RootInstaller.kt` + `IRootSystemService`): installs/replaces the
  patched APK directly, no user tap-through, survives re-patching cleanly.
- **Non-root**: the normal Android `PackageInstaller` session flow -- user
  confirms an install/update dialog. This is upstream's default and must
  stay droidtop's default path too. Upstream also has an unmerged
  `feat/shizuku` branch (`upstream/feat/shizuku`); worth checking during the
  sync whether it lands, since Shizuku is droidtop's own preferred
  privileged-access channel (see the Shizuku fork's plan) -- better than
  asking the user to grant root just for installs.

## What it needs from droidtop's plugin API

- A capability shaped like "run job, poll/await result" -- patch jobs run
  minutes, not milliseconds, so this can't be the same synchronous
  request/response shape a metadata lookup uses. Flagging this as a real
  gap for the plugins agent, not just a detail.
- A way for the plugin to hand off to Android's `PackageInstaller` when
  running non-root: either droidtop brokers the install intent (the plugin
  runs in droidtop's own process now, per the 2026-09-25 decision, so this
  is more direct than it would have been under an enginehost-hosted model --
  the plugin can likely just start the installer intent itself, in-process,
  the same way any droidtop code would), or the plugin only reports state
  and the real ReVanced Manager app performs the install. Worth the plugins
  agent settling which, since it changes what permissions the plugin needs.
- A per-installed-app status tile / action capability, distinct from the
  per-system/per-library-entry rows most of droidtop's plugin surface is
  built around.
