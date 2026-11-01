# Repository workflow

This repository contains portable third-party service manifests and their documentation. The HomeStack engine is maintained separately in `../homestack` and at https://github.com/hipotures/homestack.

For each completed logical task:

1. Make the simplest change that meets the current requirements. Do not introduce unsolicited features, compatibility layers, custom images, or workarounds.
2. Review the complete task diff and perform the focused checks described below.
3. Commit the task's changes and push to the current branch's configured upstream.

Commit and push are mandatory default actions and require no additional confirmation. Commit at the end of a logical task, not after every edit. If Git synchronization is blocked, report the blocker explicitly; do not silently skip it.

Commit only changes belonging to the current task. Never commit secrets, private keys, runtime credentials, or unrelated local changes. Never force-push or rewrite history. If a push is rejected, stop and report it.

Use English for all repository content, including documentation, manifests, configuration descriptions, comments, and commit messages. Conversation may use another language.

# Scope and manifest authoring

- Use the repository-local `homestack-service` skill when authoring or reviewing service manifests. The manifest specification in the main HomeStack repository is the source of truth; do not invent unsupported fields or capabilities.
- Keep manifests portable. Supply private IP addresses, container IDs, storage names, secrets, and device identifiers through deployment configuration rather than committing them here.
- Prefer official upstream OCI images and preserve their standard startup behavior and native data directories where supported.
- Keep persistent application data separate from the disposable root filesystem. Changing an existing deployment's mount path requires a separate, explicit data migration plan.
- Do not modify `../homestack` as part of a task scoped to this repository unless the user explicitly requests changes there.
- Committing and pushing manifests does not authorize deployment, application updates, or changes to running infrastructure. This repository does not publish HomeStack engine images or PVE agent packages.

# Verification

For manifest changes, check YAML syntax, compliance with the current HomeStack manifest contract, and consistency between the service's manifest files. YAML parsing alone does not establish manifest validity. Use available validation tools; report any checks that could not be performed.

For documentation-only changes, review the diff and run `git diff --check`. Rust compilation and application builds are not required for this repository.

Do not automatically add permanent unit, integration, or regression tests, test frameworks, or broad verification tooling. Permanent test additions require explicit user authorization.

Do not run Podman application tests or Proxmox E2E tests unless the user explicitly requests verification of a specific service. These tests are not prerequisites for ordinary manifest or documentation commits.

# Explicitly requested runtime tests

- Use isolated, disposable resources. Never modify existing production containers, volumes, or application data as part of testing.
- Restrict cleanup to resources created and positively identified by the current test execution.
- Distinguish OCI runtime tests, simulations, and actual Proxmox LXC E2E evidence. Never claim Proxmox lifecycle or recovery verification based only on Podman or mocked operations.
- Store large downloads, images, archives, test data, and other large temporary artifacts in an ignored repository-local `./tmp/` directory, not `/tmp`, `/dev/shm`, or another RAM-backed filesystem. Verify the backing filesystem and available space first.

# Instruction handling

Follow the applicable instruction hierarchy. Explicit user instructions for the current task take precedence over these repository defaults. Ask for clarification only when a material ambiguity remains; do not request confirmation for ordinary authorized work, including commits and pushes.
