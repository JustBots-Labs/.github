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
