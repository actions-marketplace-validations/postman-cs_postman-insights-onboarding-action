# Credentials and Identity

This action uses a service-account credential pair:

- `postman-api-key` (optional for this action, `PMAK-*`): a service-account [Postman API key](https://learning.postman.com/docs/reference/postman-api/authentication/) used for the application binding call. If it is omitted or rejected with `401` or `403`, the action creates a short-lived replacement through the Postman identity service.
- `postman-access-token` (required): a service-account access token used for API Catalog onboarding calls.
- `postman-team-id` (recommended): the team ID emitted by the service-token action, used as `x-entity-team-id` for org-mode integration calls.

## Primary path: service-token action

Use [postman-resolve-service-token-action](https://github.com/postman-cs/postman-resolve-service-token-action) before this action. It mints the access token from a service-account PMAK and emits both the token and team ID.

```yaml
- id: postman_token
  uses: postman-cs/postman-resolve-service-token-action@v1
  with:
    postman-api-key: ${{ secrets.POSTMAN_API_KEY }}
    postman-region: us

- uses: postman-cs/postman-insights-onboarding-action@v1
  with:
    project-name: af-cards-activation
    workspace-id: ${{ vars.POSTMAN_WORKSPACE_ID }}
    environment-id: ${{ vars.POSTMAN_ENVIRONMENT_ID }}
    postman-region: us
    postman-api-key: ${{ secrets.POSTMAN_API_KEY }}
    postman-access-token: ${{ steps.postman_token.outputs.token }}
    postman-team-id: ${{ steps.postman_token.outputs.team-id }}
```

`POSTMAN_API_KEY` should be a [Postman service account](https://learning.postman.com/docs/administration/service-accounts/) PMAK from the same parent org as the workspace and environment. A personal user PMAK can fail token minting or trigger the non-service-account warning described below.

## Legacy fallback: Postman CLI credential store

Use this only when service-account token minting is not available yet. The fallback reads the [Postman CLI credential store](https://learning.postman.com/docs/postman-cli/postman-cli-auth/) populated by `postman login`; do not paste copied cookies, DevTools values, or manually harvested session credentials into CI secrets.

```bash
postman login
jq -r '.login._profiles[].accessToken' ~/.postman/postmanrc | gh secret set POSTMAN_ACCESS_TOKEN --repo <owner>/<repo>
```

CLI login tokens are session-scoped and expire. Prefer the service-token action for CI because it mints a service-account token at runtime and avoids long-lived session secrets.

## API key auto-creation

If `postman-api-key` is omitted or the `/me` validation call returns `401` or `403`, the action creates a new API key via the Postman identity service using `postman-access-token`. Network failures and unexpected validation responses fail the action instead of silently rotating credentials.

Auto-created API keys are not used as evidence for a credential mismatch. The preflight can still warn about unresolved identity, but it does not fail only because the original API key was missing or rejected.

## Credential preflight (`credential-preflight`)

Before any onboarding write, the action can probe both credentials and compare the parent organization each one resolves to. Mismatched credentials are a common source of duplicate-link errors and workspaces that are visible to one credential but not the other.

- `warn` (default): logs a note and continues when `postman-api-key` and `postman-access-token` resolve to different parent orgs.
- `enforce`: fails the run on that condition before any onboarding write.

Those are the only public modes. There is no public opt-out. A rejected or auto-created `postman-api-key` is never failed on.

## Non-service-account warning

When the access-token session reports a `consumerType` other than `service_account`, the action logs a warning and continues according to the selected preflight mode. That warning means the run is using a user/session token path. Re-mint the token with [postman-resolve-service-token-action](https://github.com/postman-cs/postman-resolve-service-token-action) for CI.

## Team scope (`postman-team-id`)

Supply `postman-team-id` for org-mode tokens that require an explicit team header. When set, it is sent as `x-entity-team-id` on integration requests. For non-org tokens, leave it unset so Postman can infer team context from the access token. The `POSTMAN_TEAM_ID` environment variable is honored when the input is empty. Postman's [roles and permissions](https://learning.postman.com/docs/administration/roles-and-permissions/) docs cover the team and workspace role model.
