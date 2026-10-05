# Contribute to deploy

Contributions can fix behavior, improve documentation, or add focused tests.

## Before you start

Read [the support guide](SUPPORT.md) for questions and issue routing.
Search existing issues and pull requests. Discuss larger API, architecture, or dependency changes before implementation.

Work from `main` and target that branch in your pull request.
Keep each change focused. Avoid unrelated formatting and dependency updates.

## Prepare a checkout

Install the tools for the deployment path you change: Docker Compose, Podman, or kubectl/Kustomize.

Run the commands below from the repository root unless a command names another directory.
On Windows, use `gradlew.bat` in place of `./gradlew` for Gradle commands.

## Repository layout

- `agent/compose.yaml`: Docker Compose deployment.
- `agent/quadlet/`: rootless Podman units.
- `agent/kustomize/`: Kubernetes base and read-only overlay.

## Verify your change

```bash
ENDERDASH_AGENT_KEY=validation-placeholder docker compose -f agent/compose.yaml config --quiet
kubectl kustomize agent/kustomize/base > /tmp/enderdash-base.yaml
kubectl kustomize agent/kustomize/readonly > /tmp/enderdash-readonly.yaml
```

These commands render configuration without starting services or applying cluster changes. Validate changed Quadlet units with the generator installed by your Podman version. For behavior changes, use a disposable host or cluster. Verify the read-only overlay separately. Document mounts, sockets, privileges, persistence, and supported upgrade behavior. Never commit an agent key or a rendered manifest containing one.

Run the relevant checks before review. State the command and result in the pull request.
If a check cannot run, explain the missing dependency or service. Do not claim it passed.
Keep generated artifacts consistent with their source and review their diff.

## Style and documentation

Follow the existing code conventions and repository formatter. Keep commit hooks enabled.
Add focused tests for changed logic when practical. Avoid tests that only assert source strings.
Update documentation when commands, APIs, configuration, or expected behavior change.
Keep examples small and reproducible. Preserve exact identifiers, commands, and error messages.

## Open a pull request

Explain the problem and resulting behavior. Link related issues without a placeholder issue number.
Include rendered validation and any disposable runtime check. Explain changes to privileges, volumes, or RBAC.
Include commands and results. State any runtime checks that remain necessary.
Respond to review with a correction or concrete evidence.

Use Conventional Commits: `type(scope): description`, for example `docs(contributing): explain local validation`.
Use a meaningful scope, or omit it. Keep the subject concise and imperative.
Add a body when the reason or compatibility impact is not obvious.

For vulnerabilities, follow [the security reporting instructions](SECURITY.md).
Remove credentials and private data from examples, logs, and screenshots.
