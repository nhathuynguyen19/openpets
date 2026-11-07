# Agent Guidelines

Commits should use conventional commits.

Examples:

```text
feat: add pet adoption search
fix: handle empty pet profile response
docs: add agent contribution guidelines
style: format Swift files with swift-format
refactor: split pet loading logic into PetService
test: add adoption flow unit tests
chore: update Swift package dependencies
build: require Swift 6 toolchain
ci: run swift test on pull requests
perf: cache rendered pet thumbnails
feat(auth): add Sign in with Apple
fix(api): retry transient network failures
feat(api)!: replace legacy pet response model
```

## Linux Debian Package Workflow

When asked to build or build-and-install the Linux `.deb`, use the current
checkout/worktree as the source; do not switch branches or use a stale package.

1. Check the installed version with `dpkg-query -W -f='${Version}\n' openpets`
   and choose a Debian-compatible package version that sorts newer than it.
2. Build outside the repository so artifacts do not dirty the checkout:
   `bash scripts/package-linux.sh VERSION /tmp/opencode/openpets-debs`.
3. Inspect the resulting package with `dpkg-deb -I PACKAGE.deb` and simulate
   installation with `apt-get -s install PACKAGE.deb` before installing.
4. Before running any `sudo apt install PACKAGE.deb`, show the exact package and
   command and obtain explicit user confirmation. Do not install, publish, or
   upload the package without that authorization.
5. After installation, verify the installed version with `dpkg-query` and check
   the launcher with `openpets --help`. Do not change user settings or stop an
   already-running OpenPets process as part of installation.

For build-only requests, stop after package inspection; do not ask to install.

## Approved Main and Linux Release Workflow

Run this workflow only after the user says the feature is approved and
explicitly asks to commit/push it to `main` and publish a tagged release. Never
commit, push, tag, or publish merely because implementation or local tests are
complete.

1. Review `git status` and the complete diff. Include only the user-approved
   feature and release-workflow changes; preserve unrelated or untracked work.
   Run the relevant project tests and the same Linux/GTK checks required by CI.
2. Check the actual GitHub Actions workflows before release. Currently
   `.github/workflows/linux.yml` builds a temporary `.deb` artifact on PRs,
   pushes to `main`, or manual dispatch; it does not publish a Linux package
   release. `.github/workflows/release.yml` is tag-triggered but publishes
   macOS assets only. Do not claim a Linux release is supported until CI has a
   tag-triggered Linux job that builds the `.deb` from that tag and uploads it
   to the corresponding GitHub Release.
3. Add or update that Linux release job as needed, derive the package version
   from the release tag, and validate its workflow/package configuration before
   committing. Use a unique version/tag; never overwrite or force-move a
   published tag.
4. Commit approved changes with a Conventional Commit message. Push the
   resulting commit to `origin/main` without force-pushing. Wait for the
   relevant main-branch CI runs to finish successfully before tagging.
5. Create and push the approved `vX.Y.Z` tag at the green commit. Wait for the
   tag-triggered Linux release CI, then verify the GitHub Release contains the
   expected `.deb` and that its package metadata/version matches the tag.
6. If CI fails, inspect the failing job/log, fix the root cause, run the
   relevant checks, commit and push the fix, then repeat the CI/release checks
   until green. Do not blindly rerun deterministic failures. For transient
   infrastructure failures, rerun the existing job; if blocked by permissions,
   unavailable secrets, or an external outage, report the blocker instead of
   claiming success. Never force-update an existing tag to recover a release.

This project uses Swift 6.

## Coding Style

We use Swift 6.x:

- All code should be modern Swift 6 friendly.
- Respect separation of concerns. Prefer dedicated classes for business logic, UI, and other responsibilities.
- Create new files when needed. We prefer more small files over a few large files.
- Respect the DRY principle and avoid code duplication.
- Use actors when appropriate.
- Avoid `Timer`, DispatchQueue-based concurrency models, and `NSLock`. Prefer async/await, `Task`, `ContinuousClock`, and actors.

### Required Standards

- Delete old code when replacing it.
- Use meaningful names: `userID`, not `id`.
- Use early returns to reduce nesting.
- Leave no warnings after compilation unless you can justify them.

## Performance & Security

### Measure First

- Do not optimize prematurely.
- Benchmark before claiming something is faster.

## Communication Protocol

### Progress Updates

```text
✓ Implemented authentication (all tests passing)
✓ Added rate limiting
✗ Found issue with token expiration - investigating
```

### Suggesting Improvements

```text
The current approach works, but I notice [observation].
Would you like me to [specific improvement]?
```

### Reviewing

After you are done implementing:

1. Build the project to check for compilation errors.
2. Start a subagent to review your changes and ensure the initial issues are addressed.
