# Contributing

Thanks for helping. This guide applies to every repository in the Kelpgate organization unless a repository says otherwise.

## Ways to contribute

- **Report a bug** with the bug form under Issues. Say what you expected, what happened, and your versions.
- **Suggest a change** with the feature form. Describe the problem first; the solution can follow.
- **Send a pull request.** For anything larger than a small fix, open an issue first so we can agree on the approach before you spend time on it.
- **Report a security problem** privately. See [SECURITY.md](SECURITY.md); do not use public issues.

## Pull requests

1. Fork the repository and branch from `main`.
2. Keep the change focused. One concern per pull request.
3. Add or update tests when behavior changes, and update the README or changelog when users would notice.
4. Make sure checks pass locally and in CI.
5. Describe what changed and why, and link the issue.

A maintainer reviews every pull request. We may ask for changes or decline a change that does not fit the project; we will explain why.

## Commits

Write a short imperative subject ("Fix token refresh on 401"), with detail in the body when it helps. Sign your work with a `Signed-off-by` line (`git commit -s`) to confirm you have the right to submit it under the project's license (the [Developer Certificate of Origin](https://developercertificate.org/)).

## License

Contributions are accepted under the license of the repository you contribute to, which for our projects is GPL-2.0-or-later. You keep your copyright.

## Conduct

Everyone taking part is expected to follow our [Code of Conduct](CODE_OF_CONDUCT.md).

New clients, plugins and modules, and changes to existing ones, follow the [Kelpgate client standard](docs/client-standard.md): the same operation, error and setting names in every language.

## Branches, pull requests and releases

The same rules apply to every Kelpgate repository.

- **`main` is protected.** No force pushes, no deletion, linear history. Changes arrive through a pull request, and a pull request merges only when CI is green and the branch is up to date.
- **One change per branch**, named `feat/...`, `fix/...`, `chore/...`, `docs/...` or `ci/...`, and squash-merged with the pull request title as the commit message. The branch is deleted after the merge.
- **CI runs on every pull request**: tests on every supported language version, lint and static analysis, CodeQL where the language is supported, and a dependency review. Actions are pinned by commit SHA.
- **Dependabot** opens one grouped pull request per ecosystem each week. Read the diff, let CI run, and merge it like any other change.
- **Secrets**: secret scanning and push protection are on. Never commit credentials; use environment variables or the repository's secrets.
- **Releases** are tags named `vX.Y.Z` that match the version in the package. Pushing the tag publishes it, and the release workflow refuses a tag whose version or changelog entry is missing. Every release gets a GitHub Release built from its changelog entry. The full policy is in [versioning and releases](docs/versioning-and-releases.md).
- **Security issues** go to the private channel in [SECURITY.md](SECURITY.md), not to a public issue.
