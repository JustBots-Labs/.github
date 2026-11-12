# Versioning and releases

One policy for every Kelpgate repository. The [client standard](client-standard.md) covers names; this page
covers version numbers, where each package is published, and how a release is made.

## Version numbers

- **Semantic Versioning everywhere** (`MAJOR.MINOR.PATCH`). Git tags are `vMAJOR.MINOR.PATCH` and always equal the
  version inside the package.
- **Before launch, everything is `0.x`.** A minor bump may change behavior (and says so in the changelog); a patch
  bump only fixes. At launch the clients, plugins and module move to `1.0.0` together, and from then on breaking
  changes need a major version.
- **The API is versioned by path** (`/api/v1`). A breaking API change ships as `/api/v2` next to `v1`, with notice
  and an overlap period. Client versions are independent of the API version; each release states which API version
  it talks to.
- **The service and the console are versioned too** (`vMAJOR.MINOR.PATCH` tags on `main`). Every image is named by
  `git describe` (the latest tag, the commits since, and the short commit) and carries that in its
  `org.opencontainers.image.version` label. The service reports it in `GET /api/v1/health` (`version`) and in
  `kelpgate version`.

## What is published where

| Package | Repository | Registry | How a release is published |
|---|---|---|---|
| Node.js client `@kelpgate/client` | [kelpgate-node](https://github.com/KelpGate/kelpgate-node) | npm | Tag `v*`; the Publish workflow tests, then runs `npm publish` with provenance |
| Python client `kelpgate-client` | [kelpgate-python](https://github.com/KelpGate/kelpgate-python) | PyPI | Tag `v*`; the Publish workflow builds and uploads through PyPI trusted publishing (no stored token) |
| PHP client `kelpgate/kelpgate-php` | [kelpgate-php](https://github.com/KelpGate/kelpgate-php) | Packagist | Tag `v*`; Packagist reads the tag through its GitHub hook |
| WordPress plugin | [kelpgate-wordpress](https://github.com/KelpGate/kelpgate-wordpress) | WordPress.org | Tag `v*` builds the zip and a GitHub Release; the directory copy is updated through WordPress.org after review |
| Drupal module | [kelpgate-drupal](https://github.com/KelpGate/kelpgate-drupal) | Drupal.org | Released from the Drupal.org project page |
| API service | `KelpGate/kelpgate` (private) | ghcr.io image | Every merge to `main` builds and deploys; a tag names a release |
| Console | `KelpGate/kelpgate-console` (private) | ghcr.io image | Same as the service |

Current versions are shown by the badges in the [organization profile](../profile/README.md) and in each README,
not copied into documents that would go stale.

## How to make a release

1. **Merge the work through pull requests.** `main` must be green and up to date.
2. **Bump the version** in the one place the package keeps it (`package.json`, `src/kelpgate/_version.py`,
   `KelpgateClient::VERSION`, the plugin header and `readme.txt`, ...), and add a `## X.Y.Z` entry to
   `CHANGELOG.md`. Do both in a pull request, like any other change.
3. **Tag `main`**: `git tag vX.Y.Z && git push origin vX.Y.Z`.
4. **The release workflow checks, then publishes.** It fails early if the tag does not match the package version or
   if the changelog has no entry for it. Then it publishes to the registry and creates a GitHub Release whose notes
   are that changelog entry.
5. **Verify the registry** (the new version is listed, with the right author and repository links) and test an
   install from a clean directory.

For the service and console, step 2 is just the changelog and step 3 a tag; the deploy pipeline has already shipped
the code, and the tag gives the build a name.

## Rules that keep releases boring

- A tag is never moved or reused. A bad release is fixed by a new patch version; published registry metadata cannot
  be edited anyway.
- Fixes, including security fixes, go into the latest release only.
- Dependency updates arrive as one grouped Dependabot pull request per ecosystem each week, and merge like any other
  change.
- A release never needs a stored publishing secret where the registry supports trusted publishing.
