# Contributing

Thank you for wanting to help. This guide covers every repository in the uxfront-com organisation that doesn't have a contributing guide of its own. Each repository differs in how you install, build and test it, so its README (and its `AGENTS.md`, where it has one) is where you'll find those steps.

Everyone taking part agrees to follow the [code of conduct](https://github.com/uxfront-com/.github/blob/main/CODE_OF_CONDUCT.md).

## Ask a question or report a problem

Let's start with issues, since most contributions begin there:

- **Search first.** Someone may already have reported the problem or suggested the idea, and adding to that issue keeps the discussion in one place.
- **Open an issue.** Pick a bug report or a feature request in the repository's **Issues** tab, or a blank issue for anything else.
- **Report a vulnerability privately.** Never in an issue: follow the [security policy](https://github.com/uxfront-com/.github/blob/main/SECURITY.md) instead.

## Discuss a large change first

For a new feature, a change to a public API or anything that touches many files, please open an issue before you write the code. A short discussion up front can save you work on an approach that doesn't fit the project. Small fixes, like a typo or a broken link, can go straight to a pull request.

## Make a change

Here's how a change usually goes:

1. Fork the repository and create a branch from its default branch.
2. Make your change, and add or update tests where the repository has them.
3. Run the checks the repository's README lists, like formatting, linting, type checks and tests, and fix anything they report.
4. If the repository has a `.changeset/` folder, add a changeset that describes your change for the people who install the package.
5. Update the docs if your change affects how something behaves or how people use it.
6. Open a pull request against the default branch, and fill in its template.

## Title your pull request

We use [Conventional Commits](https://www.conventionalcommits.org) for pull request titles, like `fix(button): keep focus after submit` or `docs: fix a broken link`. The prefix tells reviewers what kind of change it is at a glance, and a repository that squash-merges uses the title as the commit message.

## Licence

By contributing, you agree that your contribution is released under the licence of the repository you contribute to.
