# Preset Template Resolution

specify-cli supports installing multiple presets, each of which can provide templates. When more than one installed preset supplies a template with the same name, specify-cli resolves the conflict using a numeric priority: a lower number means higher priority. Your task is to implement logic that, given a list of installed presets with assigned priorities and their provided templates, returns the correct template content for a requested template name by applying priority-based resolution.

## Capabilities

### Preset Template Resolution

- When only one preset provides a template, that template is returned regardless of priority [@test](./test_preset_resolution.py)
- When two presets provide the same template name, the one with the lower priority number wins [@test](./test_preset_resolution.py)
- When no installed preset provides the requested template name, the resolver raises an appropriate error [@test](./test_preset_resolution.py)
- When multiple presets have equal priority, the first one in installation order is used [@test](./test_preset_resolution.py)

## Implementation

[@generates](./solution.py)

## API

```python { #api }
def resolve_template(
    template_name: str,
    installed_presets: list[dict],
) -> str:
    """
    Resolve and return the content of a template from installed presets.

    Resolution order: lowest priority number wins; ties broken by list order.

    Args:
        template_name: Name of the template to resolve (e.g. "component").
        installed_presets: List of preset dicts, each with:
            {
                "name": str,
                "priority": int,       # lower = higher precedence
                "templates": {         # mapping of template_name -> content
                    "<template_name>": str,
                    ...
                }
            }

    Returns:
        The resolved template content string.

    Raises:
        TemplateNotFoundError: If no installed preset provides the template.
    """
    ...
```

## Dependencies { .dependencies }

### specify-cli 0.3.2 { .dependency }

specify-cli manages presets that bundle templates. When multiple presets are installed, it resolves template conflicts using a priority system where lower numeric priority values take precedence. The package exposes APIs for installing presets and resolving templates programmatically.

[@satisfied-by](specify-cli)
