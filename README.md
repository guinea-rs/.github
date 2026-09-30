# guinea-rs/.github

Shared CI for the guinea-rs and uniproc-dev repositories. Every repository runs the same dependency checks from here; a repository may differ only where this file says so.

## What runs

`deps-audit.yml` runs on every push to master, every pull request and every Monday:

- `cargo deny check` (advisories, bans, licenses, sources) with cargo-deny pinned by version and sha256;
- one version of our own crates in the dependency graph: two copies of guinea, amethystate, ogurpchik, uniproc-protocol, uniproc-agent-kit, guicons or shown, whether two versions or the same version from two URLs, fail the build.

`deps-tags.yml` runs on Mondays and on demand. It reads the git dependencies from our two organisations that are pinned by tag, lists the tags of each repository and keeps one issue labelled `dependency-tags` with the newer ones. It closes the issue when everything is current. It never changes a pin: bumps stay announced by the owner, because a one-sided protocol bump breaks a host/agent pair.

Dependabot takes crates.io dependencies and GitHub Actions, weekly on Monday, grouped. It ignores our own crates: it would bump a git tag on one side of a host/agent pair, which is exactly what `deps-tags.yml` only reports. It also ignores `windows-reactor*` and `windows-canvas*`, which move in lockstep with guinea.

Unpublished workspace crates (`publish = false`) are left out of the license check.

## Adopting it

1. Copy `templates/deps.yml` to `.github/workflows/deps.yml` and replace `SHA` with the commit of this repository.
2. Copy `templates/dependabot.yml` to `.github/dependabot.yml`.
3. Copy `templates/deny.toml` to `deny.toml` at the repository root. cargo-deny runs from the root, so this one file governs every listed workspace; a copy next to another manifest is not read.
4. List every workspace in `manifests`, as a JSON array, in both jobs.

## Where a repository may differ

- `manifests`: more workspaces (for example `examples/*/Cargo.toml`, `tools/devtools/Cargo.toml`), and the same directories in Dependabot. Members of a listed workspace are found on their own.
- The branch in `deps.yml`, if the default branch is neither `master` nor `main`.
- Dependabot:
  - more ecosystems (npm, gradle);
  - `ignore` for crates that must move in lockstep with ours;
  - `ignore` for a git dependency outside our organisations pinned by `rev`, which Dependabot would move to the head of its default branch.
  Each with the reason in the pull request that adds it.
- `deny.toml`:
  - `advisories.ignore` entries, each with `reason`;
  - `bans.skip` for platform crates that cannot be deduplicated;
  - `sources.allow-git` for a repository outside our organisations, while it is not on crates.io;
  - `licenses.exceptions` for a named crate.

Nothing else. A change that every repository needs goes here, and the owners move their `SHA` when it is announced.
