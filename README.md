# uxfront-com/.github

This repository holds the public defaults for every repository in the uxfront-com organisation. It gives each repository a security policy, a code of conduct, a contributing guide and issue and pull request templates, unless the repository has its own. It's also the one place that says which repository takes which file from here.

GitHub applies these defaults because this repository is named `.github`. Read [GitHub's guide to default community health files](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/creating-a-default-community-health-file) for the full rules.

## Provided

| File                                                             | What it does                                                                                                                                                                                                                                                                                                                                           |
| ---------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [`SECURITY.md`](SECURITY.md)                                     | Tells people how to report a vulnerability privately, and what happens next.                                                                                                                                                                                                                                                                           |
| [`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md)                       | Contributor Covenant 3.0, with reports to `support@uxfront.com`.                                                                                                                                                                                                                                                                                       |
| [`CONTRIBUTING.md`](CONTRIBUTING.md)                             | Walks contributors from an issue to a pull request, and points to each repository's own checks.                                                                                                                                                                                                                                                        |
| [`pull_request_template.md`](pull_request_template.md)           | Fills in a new pull request with what changed, how it was checked and a short checklist.                                                                                                                                                                                                                                                               |
| [`.github/ISSUE_TEMPLATE/`](.github/ISSUE_TEMPLATE/)             | A bug report form, a feature request form and a link to the contributing guide.                                                                                                                                                                                                                                                                        |
| [`.github/workflows/release.yml`](.github/workflows/release.yml) | Versions and publishes a repository's packages with Changesets and npm trusted publishing. Read [Release workflow](#release-workflow).                                                                                                                                                                                                                 |
| [`.editorconfig`](.editorconfig)                                 | Sets the whitespace rules editors follow: tabs with no fixed width, so each reader picks their own, and two spaces for YAML. Also LF, UTF-8 and a final newline. It matches `@uxfront/oxfmt-config` 0.2 and later. EditorConfig has no `extends`, so each repository keeps an exact copy. Styleframe keeps its own until it moves to the shared style. |

## Planned

These shared pieces come later, each in its own pull request:

- A shared CI setup action (planned).

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
