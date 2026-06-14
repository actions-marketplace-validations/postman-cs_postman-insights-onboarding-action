# Release Policy

## Source of truth

Git tags are the public release source of truth for this action. `package.json` is used for npm packaging, but consumers should choose action versions by Git tag.

## Tags

- Immutable release tags use `v1.x.y`.
- The moving `v1` tag tracks the latest compatible `v1` release.
- Existing release tags are never force-pushed or rewritten.
- Every immutable release tag should have a GitHub release with generated notes.

## Validation

Before pushing an immutable release tag:

1. Confirm the working tree is clean.
2. Run `npm test`.
3. Run `npm run typecheck`.
4. Run `npm run lint`.
5. Run `npm run build`.
6. Run `npm run check:dist`.
7. Confirm `README.md` generated tables match `action.yml`.
8. Confirm `SECURITY.md`, `SUPPORT.md`, and this file still match the release surface.

## Publishing

The release workflow validates the tag, publishes the GitHub release, and publishes the npm package when the tag matches the package version. The rolling `v1` tag updates the action channel and skips npm publishing.

## Compatibility

This action expects the workspace, environment, deployed service, and Insights agent to exist before it runs. Changes that make this action deploy the agent, create workspaces, or manage environments are outside the release contract for this repository.
