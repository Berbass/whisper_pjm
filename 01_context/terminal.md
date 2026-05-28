# Terminal & Git Configuration Guidelines

This document outlines terminal usage guidelines and aliases configured for this project.

## Git Configuration & Remotes

To ensure authentication with the remote GitHub repository succeeds with the correct credentials, an SSH Host alias named `ghub` has been configured in `~/.ssh/config`. 

This means that **all AI agents must use standard `git` commands, but ensure the remote URL uses `ghub` instead of `github.com`.**

*   **Example Remote URL:** `ghub:Berbass/intel_project_template.git` (or `ghub:Berbass/whisper.git` for the submodule).

### Example Usage:
```bash
# To fetch remote changes (assuming 'origin' is configured with the 'ghub' host):
git fetch origin

# To push your commits:
git push origin main
```
