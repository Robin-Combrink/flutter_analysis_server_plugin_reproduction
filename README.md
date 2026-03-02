# Standalone reproduction of the analysis_server_plugin not respecting `hosted` plugins
The new analyzer plugin config supports specifying a hosted URL for plugin packages, but the plugin manager appears to ignore it when generating the synthetic plugin_entrypoint package.
As a result, plugin resolution falls back to https://pub.dev and fails for private registries.

## Docs that describe this support:

https://github.com/dart-lang/sdk/blob/main/pkg/analysis_server_plugin/doc/using_plugins.md
(mentions: “a package version constraint and a hosted key specifying pub server URL…”)

## Environment
OS: Windows 11
Dart 3.11.0 (stable) (Mon Feb 9 00:38:07 2026 -0800) on "windows_x64"
Flutter: 3.41.2
Analyzer plugin system: new top-level plugins config (not legacy analyzer.plugins)
SDK constraint: '>=3.10.0 <4.0.0'

## Reproduction
Host the plugin in a private registry, e.g., [Cloudsmith](https://cloudsmith.com)
Use this in analysis_options.yaml:

```yaml
plugins:
  custom_lint_package:
    hosted: https://path.to.private.registry
    version: ^1.0.0
```

Run `dart analyze`

## Actual behavior
Analyzer plugin setup fails with:

```
Because plugin_entrypoint depends on custom_lint_package any which doesn't exist
(could not find package custom_lint_package at https://pub.dev), version solving failed.
```

The generated synthetic plugin pubspec.yaml contains:

```yaml
dependencies:
  analysis_server_plugin: ^0.3.0
  custom_lint_package: ^1.0.0
```
So hosted is dropped, and resolution goes to pub.dev.

## Expected behavior
Generated synthetic plugin dependencies should preserve hosted source, equivalent to:

```yaml
dependencies:
  custom_lint_package:
    version: ^1.0.0
    hosted: https://path.to.private.registry
```

## Workaround

Path-based plugin source works as expected

```
plugins:
  custom_lint_package:
    path: ../custom_lint_package
```
This means that local analysis has an option, but CI analysis is very annoying
