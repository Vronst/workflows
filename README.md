# 🚀 Multi-Lang Core Project

A high-performance, polyglot repository supporting **Python, Rust, JavaScript, and PHP**. This project uses fully automated CI/CD pipelines to handle linting, pull request formatting, documentation generation, and semantic releases.

---

## 🛠️ Tech Stack & Language Support

* **Python** (Managed via `uv` / `ruff`)
* **Rust** (Managed via `cargo`)
* **JavaScript / Node.js** (Managed via `npm`)
* **PHP** (Built-in runtime analysis)

---

## 🔄 Automated CI/CD Workflows

This repository features zero-maintenance automation pipelines. Below is a description of what each workflow does and exactly how to trigger it in your repository:

### 1. Code Style & Quality (`Lint Check`)
* **What it does:** Automatically analyzes your codebase to catch bugs, syntax issues, and formatting errors before they make it into production. By default it auto-detects the languages present and runs the appropriate engine (`Ruff` for Python, `cargo fmt` & `clippy` for Rust, `ESLint` & `Prettier` for JS/TS, and PHP native analysis).
* **How to trigger it:** Push any commit to an active branch, or open/update a Pull Request. The pipeline will automatically isolate and lint only the files you changed.
* **Optional inputs:**
  * `validate_all_codebase` (boolean, default `false`) — lint the whole repo instead of just the changed files.
  * `linter_env` (string, default `""`) — extra `KEY=VALUE` lines (one per line) forwarded as environment variables to Super-Linter. Use this to opt specific linters in or out, e.g.:
    ```yaml
    with:
      linter_env: |
        VALIDATE_PYTHON=true
        VALIDATE_JAVASCRIPT_ES=false
    ```
    Leave it unset to keep the safe default of auto-detecting and linting every language present.

### 2. Pull Request Automation (`Create Pull Request`)
* **What it does:** Automatically checks if an open Pull Request already exists for your active development branches. If no PR exists, it safely creates a new automated Pull Request targeting your base branch, maps your input titles cleanly, and manages flags without introducing security vulnerabilities.
* **How to trigger it:** This is a reusable `workflow_call` pipeline. It is triggered automatically by your main or test workflow configurations as a downstream job as soon as your unit/integration test suites complete successfully.

### 3. Living API Documentation (`Build and Deploy Docs`)
* **What it does:** Automatically parses code comments and docstrings across your Python, Rust, and JavaScript files, compiles them alongside static markdown layout guides, and publishes a premium, searchable documentation website directly to GitHub Pages.
* **How to trigger it:** Merging any Pull Request or pushing a commit directly into your project's main default branch triggers this pipeline. It will immediately rebuild the documentation site and publish the changes live.

### 4. Semantic Releases & Changelogs (`Create Release on Tag`)
* **What it does:** Intercepts newly pushed versions, reads the Git commit and Pull Request history since your last release, splits them into clean visual groups (like Features, Bug Fixes, and Maintenance), and automatically generates an official GitHub Release with a beautifully formatted changelog.
* **How to trigger it:** Tag a commit matching standard semantic versioning rules and push it up to GitHub from your command line:
    ```bash
    git tag v1.0.0
    git push origin v1.0.0
    ```

---

## 📜 License

This project is licensed under the Apache License - see the [LICENSE](LICENSE) file for details.

## Example usage
```bash
name: Centralized DevOps Pipeline # Your name for it

on:
  push:
    branches: [main, stage] # branches that should trigger it
    tags:
      - "[0-9]+.[0-9]+.[0-9]+" # tags that should trigger it
      - "v[0-9]+.[0-9]+.[0-9]+"
  pull_request:
    types: [opened, edited, synchronize] # pr that should trigger it

permissions:
  pull-requests: write # For Job 1 (Labeler) & Job 4 (Auto-PR)
  contents: write # For Job 6 (Releases)
  statuses: write # For commit status checks
  packages: read # For Job 2 (Linter package fetching)
  pages: write # For Job 5 (GitHub Pages deployment)
  id-token: write # For Job 5 (OIDC authentication token)

jobs:
  # 1. PR Labeling: Runs only when PRs are opened/edited
  pr-labeling:
    if: github.event_name == 'pull_request' && (github.event.action == 'opened' || github.event.action == 'edited')
    uses: Vronst/workflows/.github/workflows/autolabel.yml@main # DO NOT EDIT

  # 2. Linting: Runs on normal branch pushes and PRs
  linting:
    if: github.ref_type != 'tag'
    uses: Vronst/workflows/.github/workflows/lint.yml@main # DO NOT EDIT

  # 3. Testing: Runs on normal branch pushes and PRs
  testing:
    if: github.ref_type != 'tag'
    uses: Vronst/workflows/.github/workflows/test.yml@main # DO NOT EDIT
    with:
      runner_type: "local" # you can chose docker, podman or local
      test_command: "uv run --group test pytest && cargo test" # Command that should run on the env EDIT IT
      setup_command: |   # Command that runs before test_command EDIT IT
        pip install uv
        uv sync
        curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
        echo "$HOME/.cargo/bin" >> $GITHUB_PATH

  # 4. Auto PR Creation: Runs ONLY on pushes to 'stage' after testing/linting pass successfully
  auto-pr-generation:
    needs: [linting, testing]
    if: github.event_name == 'push' && github.ref == 'refs/heads/stage' # EDIT IT - to your branch
    uses: Vronst/workflows/.github/workflows/cd.yml@main # DO NOT EDIT
    with:
      head: "stage" # EDIT IT
      base: "main" # EDIT IT
      title: "Automated Release Pipeline: Merge stage into main"
      body: "This PR was automatically generated by the CI pipeline after linting and testing suites cleared successfully."
    secrets:
      TOKEN: ${{ secrets.GITHUB_TOKEN }}

  # 5. Documentation: Runs only when pushing to the main branch
  documentation:
    needs: [linting, testing]
    if: github.ref == 'refs/heads/main' && github.event_name == 'push' # EDIT BRANCH
    uses: Vronst/workflows/.github/workflows/docs.yml@main # DO NOT EDIT

  # 6. Release Job: Runs ONLY when a version tag is pushed
  release:
    if: github.ref_type == 'tag'
    uses: Vronst/workflows/.github/workflows/release.yml@main # DO NOT EDIT
```
