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
* **What it does:** Automatically analyzes your codebase to catch bugs, syntax issues, and formatting errors before they make it into production. It auto-detects the languages present and runs the appropriate engine (`Ruff` for Python, `cargo fmt` & `clippy` for Rust, `ESLint` & `Prettier` for JS/TS, and PHP native analysis).
* **How to trigger it:** Push any commit to an active branch, or open/update a Pull Request. The pipeline will automatically isolate and lint only the files you changed.

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
name: Continuous Integration

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:

  # 1. Example of triggering your shared Lint workflow
  
  run-lint-check:
    uses: Vronst/workflows/.github/workflows/lint.yml@main

  # 2. Example of triggering your shared Documentation workflow
  
  run-docs-build:
    uses: Vronst/workflows/.github/workflows/docs.yml@main
    permissions:
      contents: read
      pages: write
      id-token: write

  # 3. Example of triggering your shared Pull Request creation workflow
  # This runs only after the lint check passes successfully
  
  run-pr-creation:
    needs: run-lint-check
    uses: Vronst/workflows/.github/workflows/cd.yml@main
    secrets:
      token: ${{ secrets.GITHUB_TOKEN }}
```
