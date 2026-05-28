# ADR 001: Git Repository & Submodule Strategy

## Date
2026-05-28

## Status
Accepted & Implemented (Reference: Task T-005)

## Context
The overall workspace encompasses an AI-driven management structure containing project context, tasks, workspace scratchpads, and documentation, alongside the actual deliverable codebase (the Whisper Flutter app). Keeping both within a single, flat Git repository could lead to a cluttered commit history and complicate CI/CD pipelines, which should ideally trigger only on app-specific source code changes. 

Additionally, interactions with the designated GitHub remote require a specific SSH identity, to avoid conflicts with other local SSH keys.

## Decision
1. **Separate App Repository**: We initialized a distinct Git repository specifically for the Flutter source code inside `05_deliverables/whisper_app/`.
2. **Remote Link**: The app repository is linked directly to the remote GitHub repository using the custom SSH host: `ghub:Berbass/whisper.git`. The parent repository is linked to `ghub:Berbass/intel_project_template.git`.
3. **Git Submodule**: To maintain structural integrity and a unified entry point, `05_deliverables/whisper_app/` is integrated as a **Git submodule** into the parent AI-management project root (tracked via `.gitmodules`).
4. **SSH Authentication (`ghub` SSH Host)**: Interactions with the remote repository must utilize the dedicated SSH host alias `ghub` (configured via `~/.ssh/config` pointing to the correct identity file) in the remote URL (e.g., `git remote add origin ghub:...`) instead of the default `github.com`. This ensures pushes and pulls are successfully authenticated under the proper GitHub profile.

## Consequences
### Positive
* **Clean Separation of Concerns**: Project management artifacts (like completed task files and intermediate drafts) are kept entirely separate from the Flutter app's version control.
* **Streamlined CI/CD**: The remote GitHub repository only receives the clean application source code, making it simple to attach build pipelines.
* **Authentication Reliability**: The `ghub` SSH Host alias reliably maps to the correct deployment/developer keys transparently when using standard `git` commands, preventing authentication errors.

### Negative / Considerations
* **Cloning Complexity**: Future developers or automated systems cloning the parent workspace must use `git clone --recurse-submodules` or run `git submodule update --init` to populate the `05_deliverables/whisper_app` directory.
* **Two-Step Commits**: When source code is updated, agents/developers must first commit and push from within the `05_deliverables/whisper_app/` repository, and then commit the updated submodule reference hash in the parent repository.
