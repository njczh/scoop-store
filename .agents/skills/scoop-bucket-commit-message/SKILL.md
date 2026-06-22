---
name: scoop-bucket-commit-message
description: Format Scoop bucket Git commit messages and pull request titles. Use when committing, amending, reviewing, or generating messages for Scoop bucket manifest additions, app version updates, manifest fixes, bucket maintenance, or any change under bucket/, scripts/, bin/, or Scoop bucket workflow files.
---

# Scoop Bucket Commit Message

## Core Rule

For Scoop bucket changes, use the Scoop bucket title style, not generic
Conventional Commits. Scoop core asks for Conventional Commits, but Scoop bucket
contributions have their own PR title rules and official main-bucket commits
follow the same style.

Official source:
https://github.com/ScoopInstaller/.github/blob/main/.github/CONTRIBUTING.md

## Decision Tree

1. New manifest under `bucket/<app>.json`:

   ```text
   <app name>: Add version <version>
   ```

   Example:

   ```text
   open-design: Add version 0.11.0
   ```

2. Existing manifest version bump:

   Prefer the main-bucket commit-history style:

   ```text
   <app name>: Update to version <version>
   ```

   Example:

   ```text
   opencode: Update to version 1.17.9
   ```

   If matching the exact official PR-title template is more important than
   matching recent commit history, use:

   ```text
   <app name>@<version>: <small description>
   ```

3. Existing manifest fix without changing the app version:

   ```text
   <app name>@<version>: <small description>
   ```

   Keep the description short and concrete, for example:

   ```text
   open-design@0.11.0: Fix autoupdate hash extraction
   ```

4. Bucket maintenance not tied to one app version:

   ```text
   (chore): <small description>
   ```

   Examples include CI, helper scripts, repository metadata, and broad bucket
   maintenance.

## Scope Rules

- Use the manifest filename stem as `<app name>`, without `.json`.
- Use the exact manifest `version` value as `<version>`.
- Keep the subject as a single line.
- Do not add a body unless the user explicitly needs rationale in the commit.
- Do not use prefixes such as `feat(bucket):`, `fix:`, or Chinese conventional
  commit subjects for ordinary Scoop bucket manifest commits.
- If a commit contains multiple unrelated app changes, split the commit when
  possible. If it cannot be split, use a maintenance message only when the change
  is genuinely bucket-wide.

## Before Committing

1. Confirm the staged paths match the message.
2. Read the target manifest version from `bucket/<app>.json`.
3. Prefer a path-limited commit when unrelated staged changes exist.
4. If amending a wrong message, use a soft reset or `git commit --amend` only
   after confirming the commit has not already been shared.

## Evidence Examples

Official main-bucket history uses:

- `agentsview: Add version 0.33.1`
- `lean-ctx: Add version 3.8.9`
- `vim: Update to version 9.2.0680`
- `opencode: Update to version 1.17.9`
