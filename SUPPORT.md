# Support

## Before opening an issue

Confirm these basics first:

- The Insights agent is already running in the target cluster and has discovered live traffic for the service.
- The workspace ID and environment ID already exist.
- `postman-region` matches the Postman region that owns the workspace.
- `postman-access-token` comes from [postman-resolve-service-token-action](https://github.com/postman-cs/postman-resolve-service-token-action), or you are intentionally using the documented legacy fallback.
- `credential-preflight` is set to `warn` or `enforce`.

## Where to ask

- Use GitHub issues for action bugs, documentation gaps, and reproducible contract drift.
- Use Postman support for account access, org membership, plan, or product availability questions.
- Use [SECURITY.md](SECURITY.md) for vulnerability reports or accidental secret exposure.

## What to include

- The action tag, for example `v1.0.2`.
- The workflow step with secrets redacted.
- The `status` output and the relevant warning or error lines.
- Whether the token came from the service-token action or the legacy CLI fallback.
- The Postman region, workspace ID shape, and environment ID shape, with sensitive values redacted.

Do not paste API keys, access tokens, team verification tokens, or full unredacted workflow logs.
