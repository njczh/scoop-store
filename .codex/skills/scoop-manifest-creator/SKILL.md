---
name: scoop-manifest-creator
description: Create, update, review, or debug custom Scoop app manifests and bucket entries. Use when the user asks for a Scoop manifest JSON, bucket manifest, checkver/autoupdate rules, persist handling, installer-to-portable conversion, or app packaging research for Scoop on Windows.
---

# Scoop Manifest Creator

## Core Rule

Research before writing. For every target app, inspect official app sources first:
official homepage, documentation, download page, release feed, repository/releases,
license, checksum/signature information, installer documentation, and portable-mode
or config-location documentation. Do not rely on third-party download mirrors unless
the official project names them as the canonical distribution channel.

When a task needs detailed manifest semantics, read
`references/scoop-manifest-guide.md`.

## Workflow

1. Establish the bucket context.
   - Locate the bucket root and target manifest path, usually `bucket/<app>.json`.
   - Inspect existing bucket conventions, especially `bucket/app-name.json.template`
     when present.
   - Compare against official main-bucket examples. Prefer the user's local main
     bucket at `D:\Users\NJczh\Applications\Scoop\buckets\main\bucket` when it is
     available; otherwise use the online ScoopInstaller/Main repository.

2. Build an evidence table before authoring.
   - List official installation methods: portable archive, portable executable,
     MSI, Inno Setup, NSIS/Electron installer, winget/MS Store only, online
     bootstrapper, service/driver installer, or source build.
   - Mark which methods Scoop can manage directly and which require conversion.
   - Identify persistent state locations: files under the app directory, `%APPDATA%`,
     `%LOCALAPPDATA%`, `%USERPROFILE%`, registry keys, service data, caches, plugins,
     and self-updater metadata.
   - Decide how Scoop should own or preserve each state location.
   - Inspect archive or installer payload layout when the manifest depends on
     `extract_dir`, `extract_to`, `$PLUGINSDIR`, nested directories, or generated
     filenames. Do not infer internal paths from URL names alone.

3. Choose the least invasive install strategy.
   - Prefer versioned portable archives or app-managed portable mode.
   - If an installer is only a container, prefer extraction with `#/dl.7z`,
     `extract_dir`, `extract_to`, `innosetup`, or `Expand-7zipArchive`.
   - Run an installer only when extraction cannot produce a working app. Require
     documented silent arguments or a clear `installer.script`/`uninstaller.script`.
   - Avoid admin prompts, machine-global writes, background services, drivers, and
     self-updaters unless the manifest explicitly documents and controls them.
   - For shell integration, file associations, services, fonts, and registry
     changes, prefer generating optional scripts plus `notes` over applying side
     effects automatically during install.

4. Model persistence explicitly.
   - Use `persist` only for files or directories inside the install directory.
   - Prefer official flags or environment variables that move app data under a
     persisted directory, such as a user-data-dir argument in a shortcut.
   - Use scripts to copy, seed, migrate, or clean data only when `persist` and
     official app settings are insufficient.
   - For SDKs, package managers, plugins, and toolchains, separate versioned app
     baseline files from user-installed extensions; persist only user value and
     migrate side directories around upgrades when hard links are unsafe.
   - Never imply that registry or external AppData paths are captured by `persist`
     unless the manifest redirects or synchronizes them.

5. Write the manifest.
   - Use valid JSON with four-space indentation and no placeholders.
   - Include `version`, `description`, `homepage`, `license`, `url`, and `hash`.
   - Use `architecture` for architecture-specific URLs/hashes/extraction.
   - Add only necessary `bin`, `shortcuts`, `env_add_path`, `env_set`, `notes`,
     `depends`, `suggest`, `pre_install`, `post_install`, `pre_uninstall`,
     `uninstaller`, `persist`, `checkver`, and `autoupdate` fields.
   - Keep scripts idempotent, scoped to `$dir`, `$persist_dir`, and documented app
     locations; gate uninstall-only side effects with `$cmd` when needed.
   - Use `checkver` and `autoupdate` variables from a stable official source.
     Prefer official hash files or release checksums; omit `autoupdate.hash` only
     when no stable upstream checksum source exists and Scoop must compute hashes.

6. Validate.
   - At minimum, parse JSON and run formatting/checkver checks available in the
     bucket.
   - For meaningful changes, also verify URL/hash behavior and install/uninstall
     behavior when that will not risk user data or global system state.
   - Report any validation not run and why.

## Output Expectations

When delivering a manifest, include:

- The manifest path changed or created.
- A concise install-method decision and why it fits Scoop.
- A persistence decision covering all known state locations.
- The exact validation commands run and their result.
- Source links used for app facts when web research was required.
