# Kelpgate client standard

Every Kelpgate client, SDK, plugin and module uses the same names for the same things. A developer who
knows one should recognise the others. This page is the reference: when a package differs from it, the
package is wrong.

Only two things change between languages: **casing** (`scanFile` in Node and PHP, `scan_file` in Python)
and the **error suffix** (`...Error` in Node and Python, `...Exception` in PHP, as each language expects).

## Packages

| Package | Name | Class |
|---|---|---|
| Node.js | `@kelpgate/client` | `KelpgateClient` |
| Python | `kelpgate-client` (import `kelpgate`) | `KelpgateClient`, `AsyncKelpgateClient` |
| PHP | `kelpgate/kelpgate-php` | `Kelpgate\KelpgateClient` |
| WordPress | `kelpgate-malware-scanner` | Kelpgate Malware Scanner for WordPress |
| Drupal | `kelpgate` | Kelpgate Malware Scanner for Drupal |

## Operations

| Operation | Node, PHP | Python |
|---|---|---|
| Scan a file | `scanFile` | `scan_file` |
| Scan bytes held in memory | `scanBytes` | `scan_bytes` |
| Scan a remote URL | `scanUrl` | `scan_url` |
| Start an async file scan | `scanFileAsync` | `scan_file_async` |
| Start an async URL scan | `scanUrlAsync` | `scan_url_async` |
| Read an async result | `getAsyncResult` | `get_async_result` |
| Wait for an async result | `waitForResult` | `wait_for_result` |
| Check the service | `health` | `health` |
| Throw if infected | `assertClean` | `assert_clean` |
| Build from environment | `fromEnv` | `from_env` |

## Result fields

`infected`, `status` (`CLEAN` or `INFECTED`), `signature`, `sha256`, `ruleAuthors`, `filename`, `scanId`,
`timestamp`, `s3Path`, `downloadUrl`, `requestId`, `policyRejected`, `degraded`, `raw`.
In Python the same names are snake_case (`rule_authors`, `scan_id`, and so on).

A scan answers 200 (clean) or 406 (infected); both are verdicts, not errors. `policyRejected` means the
file was refused by a policy (signature starts with `POLICY:`), and `degraded` means an engine did not
finish (signature contains `scan coverage reduced`).

## Errors

| Class | When | Stable code |
|---|---|---|
| `KelpgateError` | Base class of everything below | `KELPGATE_ERROR` |
| `ConfigError` | Wrong options, before any request | `CONFIG` |
| `AuthError` | 401, 403, or no access token | `UNAUTHORIZED`, `FORBIDDEN`, `TOKEN_REQUEST_FAILED` |
| `QuotaExceededError` | 402, plan quota used up | `QUOTA_EXCEEDED` |
| `PayloadTooLargeError` | 413, or over the client's `maxBytes` | `PAYLOAD_TOO_LARGE` |
| `UnsupportedFileError` | 415, type not allowed or encrypted archive | `UNSUPPORTED_FILE` |
| `RateLimitedError` | 429 | `RATE_LIMITED` |
| `ScanError` | 400 or 412 | `SCAN_ERROR` |
| `ServerError` | 5xx | `SERVER_ERROR` |
| `NetworkError` | Connection failed | `NETWORK` |
| `TimeoutError` | Request timed out (Python: `RequestTimeout`, which avoids hiding the built-in) | `TIMEOUT` |
| `InfectedError` | Raised by `assertClean` | `INFECTED` |

Errors carry `code`, `status`, `requestId` and the decoded response `body`. PHP exceptions end in
`Exception` and expose the same data through `getErrorCode()`, `getStatus()`, `getRequestId()` and `getBody()`.

## Client options

| Option | Default | Meaning |
|---|---|---|
| `apiUrl` | `https://api.kelpgate.cloud` (from the environment) | Base URL of the API |
| `tokenUrl` | `https://auth.kelpgate.cloud/oauth2/token` (from the environment) | OAuth2 token endpoint |
| `clientId`, `clientSecret` | none | OAuth2 client credentials |
| `bearerToken` | none | A static token, instead of client credentials |
| `scope` | `scan` | OAuth2 scope |
| `timeout` | 60 seconds (Node: `timeoutMs`, milliseconds) | Per request |
| `maxRetries` | 2 | Retries after a transient failure |
| `retryDelayMs` | 500 | Base delay between retries |
| `maxBytes` | 128 MiB | Largest file the client will send |

Behavior every client shares:
- the token is cached and renewed 120 seconds before it expires, and a 401 triggers one fresh token and one retry;
- network errors, 429, 502, 503 and 504 are retried with exponential backoff, and `Retry-After` is honored up to 30 seconds;
- every scan sends an `Idempotency-Key`, the same one on each retry of a call (see Idempotency);
- `apiUrl` and `tokenUrl` must use https (plain http only for localhost), and redirects are never followed;
- the `User-Agent` is `kelpgate-<language>/<version>`, and credentials never appear in logs or `var_dump`-style output.

## Idempotency

A retry must never be charged twice. Every scan sends an `Idempotency-Key` header, and the service counts
a request with the same key, method, path and size only once.

| | |
|---|---|
| Option | `idempotencyKey` (Node, PHP) or `idempotency_key` (Python), optional, on every scan operation: `scanFile`, `scanBytes`, `scanUrl`, `scanFileAsync`, `scanUrlAsync` |
| Default | the client generates a random key (UUID v4) for each call and sends the **same key on every retry of that call**, including the retry after a 401 |
| Plugins | the WordPress plugin and the Drupal module generate one key per scan automatically; there is no setting |
| Format | 1 to 128 characters from `A-Za-z0-9._:-`; anything else is a `ConfigError` before any request is made |
| Not sent | `getAsyncResult`, `waitForResult`, `health` and the token request: they are not counted |

What the service does with it:
- a repeat of the same request under the same key is not counted again, up to two repeats within 24 hours;
- the key is bound to the file: the same key on a **different file** is counted as a new scan, so a key cannot
  be used to scan for free, and a third repeat is counted too;
- a scan that fails on the service side (5xx, 413, 422) is refunded and its key is cleared, so the retry is
  counted once.

Pass your own key only to make your own retries safe, for example a job or upload id. Never reuse one key
for different uploads. Servers that do not know the header ignore it.

## Environment variables

Clients read these through `fromEnv`; the plugins and modules read them as their overrides.

| Variable | Meaning |
|---|---|
| `KELPGATE_API_URL` | API base URL |
| `KELPGATE_TOKEN_URL` | OAuth2 token endpoint |
| `KELPGATE_CLIENT_ID`, `KELPGATE_CLIENT_SECRET` | OAuth2 client credentials |
| `KELPGATE_TOKEN` | Static bearer token |

## Plugin and module settings (WordPress, Drupal)

| Setting | Values | Default |
|---|---|---|
| `enabled` | on or off | on |
| `api_url`, `token_url` | https URLs | empty |
| `auth_mode` | `oauth2` or `token` | `oauth2` |
| `client_id`, `client_secret` | strings | empty |
| `api_token` | string, used when `auth_mode` is `token` | empty |
| `fail_action` | `block` or `allow`: what happens when a file cannot be scanned | `block` |
| `max_mb` | whole number: largest upload scanned, in megabytes | 50 |
| `timeout_seconds` | whole number | 45 |

Each setting can be forced from the environment or a constant with the same name in capitals and the
`KELPGATE_` prefix: `KELPGATE_API_URL`, `KELPGATE_TOKEN_URL`, `KELPGATE_AUTH_MODE`, `KELPGATE_CLIENT_ID`,
`KELPGATE_CLIENT_SECRET`, `KELPGATE_TOKEN` (for `api_token`), `KELPGATE_FAIL_ACTION`, `KELPGATE_MAX_MB`,
`KELPGATE_TIMEOUT_SECONDS`. A forced value wins over the stored setting, and secrets are never exported with
site configuration.

Both products also share one threat label format for the incident log and admin messages, so the same
detection reads the same everywhere.
