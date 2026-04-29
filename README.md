# provide-virtual-package

## Feature exercised

This probe exercises Composer's `provide` field, where a concrete package declares that it satisfies a virtual interface package name. The root project explicitly requires `psr/log-implementation` (a virtual interface defined by PHP-FIG) alongside `monolog/monolog ^3.0`. Monolog's own `composer.json` declares `"provide": {"psr/log-implementation": "3.0.0"}`, so Composer satisfies the virtual requirement through monolog — no separate `psr/log-implementation` package is ever downloaded or locked.

## Virtual package mechanism

A virtual package in Composer is a name that exists only as an interface contract, never as an installable package on Packagist. When a root project lists a virtual name in `require`, Composer searches the dependency graph for any real package whose `provide` map includes that name with a compatible version. If one is found, the requirement is satisfied internally and the virtual name is never written into `composer.lock` as a package entry.

In this probe:

1. Root requires `psr/log-implementation: *` and `monolog/monolog: ^3.0`.
2. Composer resolves `monolog/monolog 3.10.0`.
3. Monolog's metadata contains `"provide": {"psr/log-implementation": "3.0.0"}`.
4. Composer marks `psr/log-implementation` as satisfied; it does not add it to `packages[]`.
5. `composer.lock` contains exactly 2 entries: `monolog/monolog` and its transitive dependency `psr/log`.

## Assertion contract

| Assertion | Expected |
|-----------|----------|
| `psr/log-implementation` appears as a `name` in `packages[]` | ABSENT — probe is broken if present |
| `monolog/monolog` appears in `packages[]` | PRESENT |
| `psr/log` appears in `packages[]` | PRESENT (transitive dep of monolog) |
| Total `packages[]` entries | 2 |
| `packages-dev[]` entries | 0 |

## Failure modes this probe targets

- Virtual package name appears as an unresolved dependency node in the Mend detection tree.
- Provider package (`monolog/monolog`) not linked to the virtual requirement it satisfies.
- Scanner treats `psr/log-implementation` as a real installable package and reports it as unresolved or missing.
- Provide relationship not understood; both virtual name and provider appear simultaneously as separate nodes.

## Probe metadata

| Field | Value |
|-------|-------|
| pattern | provide-virtual-package |
| pm | composer |
| schema_version | 1.0 |
| composer_min | 2.6 |
| php_min | 8.1 |
| direct_deps | monolog/monolog 3.10.0 |
| transitive_deps | psr/log 3.0.2 |
| virtual_satisfied | psr/log-implementation (via monolog/monolog provide) |
| total_locked_packages | 2 |