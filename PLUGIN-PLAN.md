# ReVanced Manager as a droidtop plugin

Private plan. Upstream: https://github.com/ReVanced/revanced-manager
(GPL-3.0). Patches installed Android apps (YouTube etc.) using ReVanced
patch bundles, via `app.revanced.patcher`.

## What it would contribute

Not a library source and not a launcher surface — ReVanced Manager patches
*apps the user already has*, not games. Its droidtop-visible contribution is
narrower than the content-source plugin's: a Desktop-mode status tile ("N apps patched, M patches
available") plus an action on an app's own settings/details row ("Re-patch
with latest bundle", "Check for patch updates"). Search/browse-and-select
patch UI stays in the real app; droidtop only needs the summary state back
and a way to kick off a patch job and learn its result — which is exactly the
"something back" shape SPEC §12a reserves for the plugin half, unlike the content-source plugin's
fire-and-forget intent.

## Reuse as a native bundle

More promising than the content-source plugin, because this is already a Kotlin/JVM Android app,
not Flutter. The reusable core:
- `app/src/main/java/app/revanced/manager/domain/repository/{PatchBundleRepository,PatchOptionsRepository,InstalledAppRepository,DownloadedAppRepository}.kt`
  and `domain/sources/*` — bundle fetching/selection and patch-option state,
  no UI dependency.
- `domain/installer/RootInstaller.kt`, `service/RootService.kt`, and the
  `IRootSystemService` AIDL — the root install path (see below).
- The actual patcher invocation (wherever `app.revanced.patcher` is driven
  from `domain/manager` / `domain/repository`) — this is the part that does
  real work and is worth keeping.

Must be stripped: all of `ui/` (Compose screens, navigation, theming),
`di/` modules that wire Compose/Activity scope, and anything importing
Android `Activity`/`Application` context beyond what's needed to read/write
files and invoke the patcher. This is a substantial trim, not a rename —
expect the subplugin to keep maybe a third of `app/src/main/java`.

## Root

Optional enhancement, not the default, matching droidtop's rule. Two install
paths already exist upstream:
- **Root** (`RootInstaller.kt` + `IRootSystemService`): installs/replaces the
  patched APK directly, no user tap-through, survives re-patching cleanly.
- **Non-root**: the normal Android `PackageInstaller` session flow — user
  confirms an install/update dialog. This is upstream's default and must stay
  droidtop's default path too. (Upstream also has an unmerged
  `feat/shizuku` branch at `upstream/feat/shizuku`; worth checking during
  the sync whether it lands, since Shizuku is droidtop's own preferred
  privileged-access channel per the Shizuku fork's plan below — better than
  asking the user to grant root just for installs.)

## What it needs from droidtop's plugin API

- A capability shaped like "run job, poll/await result" — patch jobs run
  minutes, not milliseconds, so this can't be the same synchronous
  request/response shape a metadata lookup uses. SPEC §12a doesn't yet
  describe a long-running job capability; flagging this as a real gap, not
  just a detail.
- A way for the subplugin (running inside enginehost's process, per §12a) to
  hand off to Android's `PackageInstaller` — subplugins don't get a UI
  context of their own, so either droidtop/enginehost brokers the install
  intent on the plugin's behalf, or install stays entirely inside the real
  ReVanced Manager app and the subplugin only reports state. The second
  option is simpler and matches the content-source plugin's "don't reimplement the real app"
  principle — worth defaulting to unless the plugins agent wants otherwise.
- The closed `IntegrationCapability` set needs an entry for "per-app status
  tile" and "per-app action" if one doesn't exist yet — today's set (per
  §12/§12a as read) is oriented around library/system actions, not
  already-installed-app actions.
