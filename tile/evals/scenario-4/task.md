# Extension Installation with specify-cli

Write a Python script that drives the `specify` CLI to install spec-kit extensions and verifies the results under different source conditions.

## Capabilities

### Extension Installation

- Installs an extension from a local directory and confirms that the extension's files appear in the project's extensions folder [@test](./test_extension.py)
- Installs an extension from a local ZIP file and confirms that the extension's files are correctly extracted into the project's extensions folder [@test](./test_extension.py)
- Attempts to install from a path that does not exist and confirms the CLI reports an error and exits with a non-zero code [@test](./test_extension.py)
- Installs two extensions sequentially and confirms both are present without either overwriting the other [@test](./test_extension.py)

## Implementation

[@generates](./solution.py)

## API

```python { #api }
def run_install_extension(
    project_dir: str,
    source: str,
) -> subprocess.CompletedProcess:
    """
    Invoke the specify CLI to install a spec-kit extension into project_dir.

    Args:
        project_dir: Root directory of an initialized specify project.
        source:      Path to the extension source — either a directory or a
                     ZIP file.

    Returns:
        The completed subprocess result (returncode, stdout, stderr).
    """
    ...

def list_installed_extensions(project_dir: str) -> list[str]:
    """
    Return the names of extensions currently installed in project_dir.
    """
    ...
```

## Dependencies { .dependencies }

### specify-cli 0.3.2 { .dependency }

Provides the `specify` CLI. Use its extension-installation subcommand to install spec-kit extensions from local directories or ZIP archives into an initialized project.

[@satisfied-by](specify-cli)
