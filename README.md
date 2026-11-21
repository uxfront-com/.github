# uxfront-com/.github

This repository holds the public defaults for every repository in the uxfront-com organisation. It gives each repository a security policy, a code of conduct, a contributing guide and issue and pull request templates, unless the repository has its own. It's also the one place that says which repository takes which file from here.

GitHub applies these defaults because this repository is named `.github`. Read [GitHub's guide to default community health files](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/creating-a-default-community-health-file) for the full rules.

## Provided

| File                                                    | What it does                                                                                     |
| ------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| [`SECURITY.md`](SECURITY.md)                            | Tells people how to report a vulnerability privately, and what happens next.                     |
| [`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md)              | Contributor Covenant 3.0, with reports to `support@uxfront.com`.                                 |
| [`CONTRIBUTING.md`](CONTRIBUTING.md)                    | Walks contributors from an issue to a pull request, and points to each repository's own checks. |
| [`pull_request_template.md`](pull_request_template.md)  | Fills in a new pull request with what changed, how it was checked and a short checklist.        |
| [`.github/ISSUE_TEMPLATE/`](.github/ISSUE_TEMPLATE/)    | A bug report form, a feature request form and a link to the contributing guide.                 |

## Planned

These shared pieces come later, each in its own pull request:

- A reusable release workflow (planned).
- A shared CI setup action (planned).
- A shared `.editorconfig` (planned).

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
