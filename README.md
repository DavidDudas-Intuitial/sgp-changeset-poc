# sgp-changeset-poc

Proof of concept repository for creating and managing independent releases in a polyglot monorepo containing both Python and Node.js packages. This setup integrates Changesets, pnpm, and GitHub Actions to support semantic versioning, tagging, changelog generation, and optional release publishing.

---

## Purpose

This repository demonstrates how to:

- Use Changesets to track and apply semantic version bumps
- Manage versioning independently for Python and Node packages in a single monorepo
- Tag and optionally release each package with GitHub Actions
- Support publishing and package validation via CI/CD workflows

---

## Directory Structure

```
sgp-changeset-poc/
├── .changeset/                      # Markdown files describing version changes
├── .github/workflows/              # GitHub Actions workflows for CI/CD
├── packages/                       # Node.js packages 
│   └── core/
├── packages-python/                # Python packages
│   └── common_grants_sdk/
├── scripts/                        # Helper scripts (e.g. Python version bump)
│   └── bump_python_version.py
├── package.json                    # pnpm root config with dev dependencies
├── pnpm-workspace.yaml            # Workspace config for pnpm (includes Python)
└── README.md                       # This documentation file
```

---

## Capabilities

### Changeset-Aware Versioning

- Tracks version bumps via Changesets (`.changeset/*.md`)
- Node packages: managed via `pnpm changeset version`
- Python packages: handled via a custom script

### Tagging

- Git tags are automatically created and pushed
  - Format: `common_grants_sdk@0.5.0`

### GitHub Releases

- Optional workflow (`create-release-from-tag.yml`) can be manually triggered

### CI/CD

| Workflow File                 | Description                                        |
|-------------------------------|----------------------------------------------------|
| `build-check-python.yml`      | Lint/build/validate Python packages                |
| `create-release-from-tag.yml` | Creates GitHub Releases from pushed tags           |
| `publish-node.yml`            | Dry-run publish to npm                             |
| `publish-python.yml`          | Dry-run publish to TestPyPI                        |
| `version.yml`                 | Unified versioning and tagging                     |

---

## Tools Used

- [Changesets](https://github.com/changesets/changesets)
- [pnpm](https://pnpm.io)
- [GitHub Actions](https://docs.github.com/en/actions)
- [softprops/action-gh-release](https://github.com/softprops/action-gh-release)
- [twine](https://pypi.org/project/twine/), [build](https://pypi.org/project/build/), [semver](https://pypi.org/project/semver)

---

## Adapting Changesets for Python Packages

Because Changesets is designed for Node.js and expects each package to contain a package.json, we include a minimal “fake” Node package under `packages-python/` to satisfy this requirement. Without it, Changesets would not recognize Python-only changes in the monorepo.

This adaptation enables the versioning workflow to trigger correctly even when only the Python package has been modified.

---

## Developer Instructions

### Step 1: Make a Change

Edit a Python or Node package file.

### Step 2: Generate a Changeset

Run:

```bash
pnpm changeset
```

Then follow the CLI prompts:

- Select affected package(s)
- Choose version bump type (`patch`, `minor`, or `major`)
- Add a short summary

This generates a `.md` file in `.changeset/`.

Example output:

```markdown
---
"common_grants_sdk": patch
---

Fix grant lookup edge case.
```

### Step 3: Commit and Push Changes

Include the `.changeset/*.md` file in your PR.

### Step 4: Merge the PR

Upon merging to `main`, the following actions are triggered automatically via the `version.yml` workflow:

- The Python package version is bumped using a custom script (`scripts/bump_python_version.py`)
- The Node packages are versioned using `pnpm changeset version`
- Corresponding `CHANGELOG.md` files are updated for each affected package (Python and/or Node)
- All changes (version bumps and changelogs) are committed and pushed back to `main`
- Git tags are created for each versioned package and pushed to the repository

### Step 5: Trigger GitHub Release from Tags (Optional)

Once tags have been automatically created and pushed by the `version.yml` workflow, you can manually trigger the `create-release-from-tag.yml` workflow to generate GitHub Releases based on those tags.

To do this:

1. Navigate to the **Actions** tab on GitHub.
2. Select the **Create GitHub Release from Tag** workflow from the left-hand menu.
3. Click **Run workflow**.
4. Enter the tag you want to release (e.g. `common_grants_sdk@0.2.0`).
5. Click the **Run workflow** button to initiate the release.

This workflow uses the pushed tags to create GitHub Releases with autogenerated release notes.

---

## Manual Validation Steps (Python)

After a PR is merged:

1. **Verify version bump**  
   - Check version in `common_grants_sdk/setup.py`
   - Check version in `common_grants_sdk/package.json`

2. **Verify changelog**  
   - Check top entry in `common_grants_sdk/CHANGELOG.md`

3. **Check tags**  
   - Navigate to the GitHub repo > Tags tab  
   - Ensure tags like `common_grants_sdk@0.5.0` appear

4. **Check commits**  
   - Look for a `chore: version packages [skip ci]` commit from `github-actions[bot]`

---

