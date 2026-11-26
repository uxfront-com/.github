# uxfront-com/.github

This repository holds the public defaults for every repository in the uxfront-com organisation. It gives each repository a security policy, a code of conduct, a contributing guide and issue and pull request templates, unless the repository has its own. It's also the one place that says which repository takes which file from here.

GitHub applies these defaults because this repository is named `.github`. Read [GitHub's guide to default community health files](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/creating-a-default-community-health-file) for the full rules.

## Provided

| File                                                                   | What it does                                                                                                                                                                                                                                                                              |
| ---------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`SECURITY.md`](SECURITY.md)                                           | Tells people how to report a vulnerability privately, and what happens next.                                                                                                                                                                                                              |
| [`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md)                             | Contributor Covenant 3.0, with reports to `support@uxfront.com`.                                                                                                                                                                                                                          |
| [`CONTRIBUTING.md`](CONTRIBUTING.md)                                   | Walks contributors from an issue to a pull request, and points to each repository's own checks.                                                                                                                                                                                           |
| [`pull_request_template.md`](pull_request_template.md)                 | Fills in a new pull request with what changed, how it was checked and a short checklist.                                                                                                                                                                                                  |
| [`.github/ISSUE_TEMPLATE/`](.github/ISSUE_TEMPLATE/)                   | A bug report form, a feature request form and a link to the contributing guide.                                                                                                                                                                                                           |
| [`.github/workflows/release.yml`](.github/workflows/release.yml)       | Versions and publishes a repository's packages with Changesets and npm trusted publishing. Read [Release workflow](#release-workflow).                                                                                                                                                    |
| [`.github/workflows/lighthouse.yml`](.github/workflows/lighthouse.yml) | Runs Lighthouse on a repository's site each night, and only when `main` has moved since the last audit. Read [Lighthouse workflow](#lighthouse-workflow).                                                                                                                                 |
| [`.github/actions/setup/`](.github/actions/setup/)                     | Sets up pnpm and Node.js, restores the pnpm store and installs a repository's dependencies in one CI step. Read [Setup action](#setup-action).                                                                                                                                            |
| [`.editorconfig`](.editorconfig)                                       | Sets the whitespace rules editors follow: tabs with no fixed width, so each reader picks their own, and two spaces for YAML. Also LF, UTF-8 and a final newline. It matches `@uxfront/oxfmt-config` 0.2 and later. EditorConfig has no `extends`, so each repository keeps an exact copy. |

## Release workflow

The release workflow lets a repository version and publish its packages to npm without an npm token. On a push to `main`, it opens or updates the "Version Packages" pull request while changesets are pending. Once that pull request merges, it builds, packs and publishes the new versions, then tags them and creates GitHub releases. Only its `publish` job gets `id-token: write`, the permission npm's [trusted publishing](https://docs.npmjs.com/trusted-publishers) needs, so the build never runs next to the publish credential.

A repository calls it from its own `.github/workflows/changesets.yml`:

```yaml
name: Changesets
on:
  push:
    branches: [main]
  workflow_dispatch:
concurrency:
  group: changesets-${{ github.ref }}
  cancel-in-progress: false
permissions: {}
jobs:
  release:
    permissions:
      contents: write
      pull-requests: write
      id-token: write
    uses: uxfront-com/.github/.github/workflows/release.yml@<40-char SHA> # main
    with:
      build: <the repository's build command>
```

The calling job must grant all three permissions, since a called workflow can only lower the permissions it gets, and npm needs `id-token: write` in both workflows. Keep `concurrency` in the caller too: a called workflow that uses the caller's group waits on itself.

| Input               | Type    | Default                            | What it does                                                                                                |
| ------------------- | ------- | ---------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `build`             | string  | required                           | The command that builds the packages, like `pnpm turbo run build --filter='./packages/*'`.                  |
| `node-version`      | string  | `''`                               | The Node.js version. When set, it wins over `node-version-file`.                                            |
| `node-version-file` | string  | `.node-version`                    | The file to read the Node.js version from when `node-version` is empty.                                     |
| `pnpm-version`      | string  | `''`                               | The pnpm version. Empty reads `packageManager` from `package.json`, and setting both fails the job.         |
| `version-script`    | string  | `''`                               | The command that versions packages. Empty runs `changeset version`.                                         |
| `commit-message`    | string  | `chore(release): version packages` | The commit message of the "Version Packages" pull request.                                                  |
| `pr-title`          | string  | `chore(release): version packages` | The title of the "Version Packages" pull request.                                                           |
| `publish`           | boolean | `true`                             | With `false`, the workflow only opens or updates the "Version Packages" pull request and publishes nothing. |

Two rules keep releases working:

- **Keep the filename.** npm's trusted publisher checks the calling workflow's filename, not this one, so keep `changesets.yml` as it is. Renaming or moving it revokes the publish credential, and the release then fails with a misleading E404.
- **Pin the full SHA.** Point `uses:` at the full 40-character SHA of a commit on this repository's `main`, with a `# main` comment, never at a branch or a tag. That way a change here reaches a repository only when it moves its pin.

The publish job also checks that npm is 11.5.1 or later, which trusted publishing needs. It installs nothing, so if the check fails, move the repository to a Node.js release that bundles a newer npm.

## Lighthouse workflow

The Lighthouse workflow audits a repository's site once a night, and only when `main` has moved since the last audit. Running Lighthouse on every pull request slows each one down, while a nightly run that skips an unchanged branch costs almost nothing on a quiet day. When it audits, it installs, builds and runs the repository's own Lighthouse command, then uploads the reports as the `lighthouse-reports` artifact for 14 days. A failed audit fails the run.

A repository calls it from its own `.github/workflows/lighthouse.yml`:

```yaml
name: Lighthouse

on:
  schedule:
    - cron: "<minute> <hour> * * *"
  workflow_dispatch:
  # Tests the caller itself, and each later pin bump, before it merges.
  pull_request:
    paths: [.github/workflows/lighthouse.yml]

concurrency:
  group: lighthouse-${{ github.ref }}
  cancel-in-progress: false

permissions: {}

jobs:
  lighthouse:
    name: Lighthouse
    permissions:
      actions: read # the change check reads earlier runs
      contents: read
    uses: uxfront-com/.github/.github/workflows/lighthouse.yml@<40-char SHA> # main
    with:
      build: <the repository's build command>
      audit: <the repository's audit command>
      reports: <the report directory>
```

The `pull_request` trigger only fires for a pull request that changes the caller itself, like the one that adds it or moves its pin, so each change to it runs once before it merges. No other pull request runs Lighthouse. Pin the full SHA with a `# main` comment, as for the release workflow.

The calling job must grant `actions: read`, since the change check reads the workflow's earlier runs, and `contents: read`, since the audit checks out the code. A called workflow can only lower the permissions it gets. Keep `concurrency` in the caller too, for the same reason as the release workflow.

| Input               | Type   | Default                          | What it does                                                                                                         |
| ------------------- | ------ | -------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| `install`           | string | `pnpm install --frozen-lockfile` | The command that installs dependencies. Empty skips the step.                                                        |
| `build`             | string | `''`                             | The command that builds the site. Empty skips the step.                                                              |
| `audit`             | string | required                         | The command that runs Lighthouse and exits non-zero when it fails. It runs from the repository root, so it can `cd`. |
| `reports`           | string | `.lighthouseci`                  | The path to upload as the `lighthouse-reports` artifact.                                                             |
| `node-version`      | string | `''`                             | The Node.js version. When set, it wins over `node-version-file`.                                                     |
| `node-version-file` | string | `.node-version`                  | The file to read the Node.js version from when `node-version` is empty.                                              |
| `pnpm-version`      | string | `''`                             | The pnpm version. Empty reads `packageManager` from `package.json`, and setting both fails the job.                  |
| `timeout-minutes`   | number | `30`                             | The timeout of the `Audit` job, in minutes. Raise it for a site whose build takes longer.                            |

Each command runs in Bash, from an environment variable rather than the script text, so the caller's input is never expanded into the script.

The `Changes` job decides whether the `Audit` job runs, and writes its decision and the reason to the run's summary:

- **A scheduled run audits only when the branch moved.** It looks back through the workflow's completed runs on the same branch, newest first, for the last one whose `Audit` job passed, failed or timed out. When the branch is still at that run's commit, it skips the audit and the run stays green.
- **Every other run audits.** A manual run from the **Actions** tab, or a pull request run, always audits.
- **A skipped or cancelled audit doesn't count**, so a cancelled audit runs again the next night.
- **A failed lookup fails the run.** When the history can't be read, like without `actions: read`, the run goes red instead of skipping.

The check looks back at most 100 completed runs, so after 100 skipped nights in a row it audits the same commit again. GitHub can delay or drop a scheduled run when it's busy, and the next night catches up, since the check compares against the last audit rather than the last night.

Keep in mind who hears about a failed night: GitHub sends the [notification for a scheduled run](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#schedule) to whoever last changed the cron line in the caller. To receive them yourself, be the last one to change it.

`lighthouse-test.yml` tests this workflow on each pull request that changes it. It calls the pull request's own copy against the fixture page in `test/lighthouse/`, which must score at least 0.9 in every category. This repository has no `package.json` or `.node-version`, so the self-test also passes the Node.js and pnpm versions and an empty `install`.

## Setup action

The setup action gives every CI job one step that sets up pnpm and Node.js, restores the pnpm store and installs the dependencies. Without it, each job repeats the same three or four steps, and their pins and options drift apart between jobs and repositories. It reads the pnpm version from `packageManager` in `package.json` and the Node.js version from `.node-version`, so a repository keeps both in one place.

A job calls it right after its own checkout:

```yaml
steps:
  - name: Checkout
    uses: actions/checkout@<40-char SHA> # vX
  - name: Setup
    uses: uxfront-com/.github/.github/actions/setup@<40-char SHA> # main
    # with:                      # only where the job needs it
    #   filter: "@uxfront/testlane"
    #   node-version: ${{ matrix.node-version }}
```

Keep the checkout first and in the job, since the action reads `.node-version`, `package.json` and `pnpm-lock.yaml` from it, and each job picks its own checkout options, like `fetch-depth: 0`. Pin the full SHA with a `# main` comment, as for the release workflow. The action is read from here at run time, so this repository must stay public, or every caller's `Setup` step fails.

| Input               | Default         | What it does                                                                                                                      |
| ------------------- | --------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| `filter`            | `''`            | One pnpm filter selector, like `@uxfront/testlane`. When set, the install runs with `--filter` and only installs what it selects. |
| `node-version`      | `''`            | The Node.js version, like `22` for a matrix leg. When set, it wins over `node-version-file`.                                      |
| `node-version-file` | `.node-version` | The file to read the Node.js version from when `node-version` is empty.                                                           |

The install always runs `pnpm install --frozen-lockfile`, so a lockfile that doesn't match `package.json` fails the job. The filter comes in through an environment variable rather than the script text, so the caller's input is never expanded into the script.

The pnpm store is cached once a week, per runner OS and architecture and per filter. A job restores its own store when the lockfile matches, and otherwise the newest store from the same week, so pnpm downloads only the difference. Each week starts from a cold install, which keeps old packages from piling up in the store.

`setup-test.yml` tests the action on each pull request that changes it. It copies the pnpm workspace in `test/setup/` to the repository root and calls the pull request's own copy three times: with no inputs, with a `filter` and with a `node-version`, each followed by a check of the Node.js and pnpm versions and the installed packages. Dependabot keeps the action's pins current, since `.github/dependabot.yml` covers `.github/actions/*`.

## How defaults apply

A repository's own file always wins, so a default only fills a gap:

- **One file at a time.** A repository with its own `SECURITY.md` keeps it and still gets every other default it has no file for.
- **Issue templates are all or nothing.** A repository with any issue template or `config.yml` of its own in `.github/ISSUE_TEMPLATE/` uses none of the defaults from that folder.
- **This repository must stay public.** GitHub only applies the defaults from a public `.github` repository.

## What each repository takes from where

This table is as of this README's last update. "Default" means the file comes from here, and "own" means the repository has its own.

| Repository      | Security policy | Code of conduct | Contributing guide | Issue templates | Pull request template |
| --------------- | --------------- | --------------- | ------------------ | --------------- | --------------------- |
| uxfront         | default         | own             | own                | own             | own                   |
| Unframework     | default         | own             | own                | own             | own                   |
| Styleframe      | own             | own             | own                | own             | own                   |
| Inkline         | own             | own             | own                | own             | own                   |
| Open Components | default         | own             | own                | own             | own                   |
| Testlane        | default         | default         | default            | default         | default               |

Every other repository in the organisation gets each default it has no file of its own for.
