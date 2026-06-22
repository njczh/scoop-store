# Scoop Manifest Guide

Use this reference after the `scoop-manifest-creator` skill triggers and the task
needs concrete field, installation, persistence, or validation decisions.

## Official References

- App manifest reference:
  https://github.com/ScoopInstaller/Scoop/wiki/App-Manifests
- Creating a manifest:
  https://github.com/ScoopInstaller/Scoop/wiki/Creating-an-app-manifest
- Persistent data:
  https://github.com/ScoopInstaller/Scoop/wiki/Persistent-data
- Pre/post install script variables:
  https://github.com/ScoopInstaller/Scoop/wiki/Pre-Post-%28un%29install-scripts
- Autoupdate and checkver:
  https://github.com/ScoopInstaller/Scoop/wiki/App-Manifest-Autoupdate
- Buckets:
  https://github.com/ScoopInstaller/Scoop/wiki/Buckets
- Folder layout:
  https://github.com/ScoopInstaller/Scoop/wiki/Scoop-Folder-Layout
- Main bucket examples:
  https://github.com/ScoopInstaller/Main/tree/master/bucket

## Research Checklist

Collect evidence before writing JSON:

- Current stable version, release date, and whether prereleases are ignored.
- Official Windows artifacts for `64bit`, `32bit`, and `arm64`.
- Whether artifacts are portable archives, self-extracting archives, MSI,
  Inno Setup, NSIS/Electron installers, app stores, winget manifests, or online
  installers.
- Official checksums, signatures, release assets, or hash files.
- License identifier or official license URL.
- Portable mode, config flags, environment variables, CLI options, and documented
  user-data locations.
- Archive or installer payload layout, including top-level directories, nested
  package names, architecture-specific subdirectories, and generated filenames.
- Runtime dependencies, optional integrations, file associations, shell
  extensions, services, drivers, scheduled tasks, registry usage, and self-update
  behavior.
- Start menu target, CLI entry points, PATH directories, and icon/shortcut needs.

If an answer is not available from official sources, state the gap and make the
manifest conservative.

## Installation Strategy Matrix

Prefer strategies in this order:

1. Versioned portable archive.
   Use direct `url`, `hash`, optional `extract_dir`, `extract_to`, `bin`, and
   `shortcuts`.

2. Portable `.exe` or self-extracting archive.
   Use the URL fragment rename pattern `#/dl.7z` when the executable can be
   extracted by 7-Zip. Clean installer residue in `pre_install` or `post_install`
   only when residue is understood.

3. MSI as archive.
   Treat MSI files as extractable archives when possible and use `extract_dir`.
   Do not add the deprecated `msi` property.

4. Inno Setup or NSIS/Electron installers.
   Use `innosetup: true` when appropriate. For Electron/NSIS app payloads, inspect
   `$PLUGINSDIR` and extract `app-64.7z`, `app-32.7z`, or equivalent with
   `Expand-7zipArchive`.

5. True installer execution.
   Use `installer` only when the app cannot work from extracted files. Require
   documented silent args or a clear PowerShell script. Include a matching
   `uninstaller` when the installer writes outside `$dir`.

6. Unsuitable or special-case distribution.
   Pause or clearly document limits when the only official path is an online
   bootstrapper, MS Store package, driver/service installer, paid authenticated
   download, or install flow that cannot be made reproducible.

## Side-Effect Policy

Default to no automatic system integration. If an app supports optional shell
integration, file associations, services, font installation, or registry changes:

- Generate scripts or `.reg` files from bucket `scripts/<app>` templates when
  the integration is useful but optional.
- Put the manual command in `notes`.
- Make uninstall cleanup conditional with `$cmd -eq 'uninstall'` so update flows
  do not remove integrations unexpectedly.
- Respect `$global`: per-user installs should target HKCU/user locations, and
  global installs should target HKLM/machine locations only when the manifest
  explicitly supports that path.
- Prefer not to register services or drivers automatically. Document manual steps
  unless the whole app is unusable without them.

## Persistence Strategy

Scoop's `persist` links files/directories from the app install directory into
`~/scoop/persist/<app>`. The manifest can use `$persist_dir` in scripts.

Use this decision order:

1. App already stores mutable state inside the install directory.
   Add those files/directories to `persist`.

2. App supports a portable/user-data flag or config.
   Configure the app to use a directory inside `$dir`, then persist that
   directory. Example pattern: shortcut args like `--user-data-dir="User Data"`
   plus `"persist": "User Data"`.

3. App config lives in a generated config file.
   Use `post_install` to write config pointing cache/prefix/state to
   `$persist_dir` or a persisted subdirectory.

4. App mutates a file that cannot be hard-linked safely.
   Copy from `$persist_dir` in `pre_install`, then save back in `pre_uninstall`,
   as seen in complex official manifests.

5. App is an SDK, package manager, plugin host, or toolchain.
   Separate bundled baseline files from user-installed artifacts. Direct `persist`
   is best for stable user-data directories, but use copy/move scripts when the
   app rewrites files, replaces directories, or installs packages beside current
   version files. Avoid persisting generated baseline directories wholesale.

6. App uses `%APPDATA%`, `%LOCALAPPDATA%`, registry, services, or scheduled tasks.
   Prefer official redirection. If unavailable, document the limitation in
   `notes`, or add narrow scripts only when the side effects are well understood.

Do not use `persist` for caches that are safe to recreate unless preserving them
is part of the user value, such as package caches, plugins, or user-installed
extensions.

## Field Guidance

- `version`: Exact version installed by the URLs.
- `description`: One-line program description; do not repeat the app name when
  it is redundant with the manifest filename.
- `homepage`: Official homepage or repository.
- `license`: Prefer SPDX identifiers. Use an object with `identifier` and `url`
  for proprietary/freeware or non-obvious licenses.
- `architecture`: Put architecture-specific `url`, `hash`, `extract_dir`,
  scripts, `installer`, and `shortcuts` here.
- `url`: Use official HTTPS release/download URLs. Use arrays only when multiple
  files are needed.
- `extract_dir`: Use when the archive contains the real app under a stable
  subdirectory.
- `extract_to`: Use when the whole archive must be placed under a chosen
  directory, then optionally normalize nested layout in `pre_install`.
- `hash`: SHA256 by default. Use official hash files in `autoupdate.hash` when
  available.
- `bin`: CLI shims. For alias shims, use nested arrays, for example
  `[ [ "program.exe", "alias" ] ]`.
- `shortcuts`: GUI entry points. Keep labels stable and user-facing.
- `env_add_path`: Relative paths inside `$dir`; use only for real executable
  directories.
- `env_set`: App-specific environment variables. Avoid broad PATH changes here.
- `depends`: Required runtime dependencies. Use `suggest` when the app can be
  installed without the dependency or only some workflows need it.
- `notes`: User-visible limitations or optional manual integration steps.
- `pre_install` / `post_install`: Idempotent setup and cleanup.
- `pre_uninstall` / `post_uninstall`: Save state or remove controlled side
  effects.
- `installer` / `uninstaller`: Prefer `script` for deterministic PowerShell when
  installer behavior is not simple args.
- `checkver`: Use GitHub mode, JSONPath, XPath, regex, or script based on the
  most stable official source.
- `autoupdate`: Use `$version`, `$cleanVersion`, `$matchName`, `$matchTag`, and
  other checkver capture variables only after verifying they exist.
- `autoupdate.hash`: Prefer official checksum files or release-page checksums.
  Use `$baseurl`, `$basename`, and hash variables like `$sha256` to avoid
  duplicating URL logic. If no stable checksum source exists, omit this field and
  let the maintenance flow compute the hash after download.

## Reconstructed Main-Bucket Lessons

These are distilled from official main-bucket manifests and are useful failure
checks before submitting a new custom manifest:

- `fd.json`: Simple GitHub CLI releases still need all supported Windows
  architectures and correct `extract_dir` values. A manifest that only covers
  x64 is often functional but not main-bucket grade when upstream ships x86 and
  arm64 artifacts.
- `nodejs.json`: A direct archive manifest is not enough. npm writes global
  packages and cache data; the manifest persists `bin` and `cache`, then writes
  npm config to redirect prefix/cache into `$persist_dir`. The autoupdate hash
  comes from Node's official `SHASUMS256.txt.asc` via `$baseurl`.
- `7zip.json`: Upstream recommends `.exe`, but Scoop uses extractable MSI for
  x86/x64 to avoid installer side effects. arm64 needs a helper extractor.
  Context-menu registry integration is generated as optional `.reg` files and
  shown in `notes`; install does not silently mutate Explorer integration.
- `git.json`: PortableGit is a self-extracting archive, so `#/dl.7z` is correct.
  `etc/gitconfig` is copied instead of persisted because Git rewrites it in a
  way that does not work reliably through hard links. File associations and
  context menus are generated as optional scripts, not applied automatically.
- `android-clt.json`: The archive layout is nested and must be normalized to
  `cmdline-tools/latest`. `ANDROID_HOME` points at `$dir`, while SDK packages
  installed later are user value and need persistence/migration. Java is a
  suggestion plus warning, not a hard dependency.

## Local Example Mining

Useful searches:

```pwsh
rg -n '"persist"|"pre_install"|"post_install"|"installer"|"uninstaller"|"innosetup"|"\$PLUGINSDIR"|"#/dl\.7z"' bucket
rg -n '"persist"|"pre_install"|"post_install"|"installer"|"uninstaller"|"innosetup"|"\$PLUGINSDIR"|"#/dl\.7z"' 'D:\Users\NJczh\Applications\Scoop\buckets\main\bucket'
rg -n '"checkver"|"autoupdate"|"jsonpath"|"github"|"regex"' bucket
rg -n '"checkver"|"autoupdate"|"jsonpath"|"github"|"regex"' 'D:\Users\NJczh\Applications\Scoop\buckets\main\bucket'
```

Examples worth inspecting:

- `7zip.json`: MSI/archive extraction, architecture split, `persist`, registry
  helper generation, and uninstall-time registry cleanup.
- `git.json`: portable self-extracting archive with `#/dl.7z`, config migration
  to `$persist_dir`, `env_add_path`, `env_set`, shortcuts, and complex checkver.
- `nodejs.json`: archive extraction, npm prefix/cache persistence, JSONPath
  checkver, and hash file autoupdate.
- Current custom bucket `mineru.json`: Electron/NSIS payload extraction from
  `$PLUGINSDIR`, user-data redirection, and self-updater cleanup.
- Current custom bucket `geist-font.json`: true installer script with registry
  writes and uninstall cleanup. Treat this as a high-side-effect pattern, not the
  default.

## Validation Commands

Adapt validation to the bucket. In this repository, the helper scripts wrap
Scoop's own tools:

```pwsh
Get-Content .\bucket\<app>.json | ConvertFrom-Json | Out-Null
.\bin\formatjson.ps1
.\bin\checkver.ps1 <app>
.\bin\checkurls.ps1 <app>
.\bin\checkhashes.ps1 <app>
.\bin\test.ps1
```

For install behavior, use a local manifest path:

```pwsh
scoop install .\bucket\<app>.json
scoop uninstall <app>
```

Ask before running install/uninstall when it may overwrite user data, touch
global locations, register fonts/shell extensions/file associations, start
services, or remove an already installed app.
