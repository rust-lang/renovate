# rust-lang Renovate presets

Shared [Renovate](https://docs.renovatebot.com/) presets for repositories in the Rust Project, maintained
by the Infrastructure Team.

To learn more about Renovate and how to use it in Rust Project repositories, see the
[Rust Forge documentation](https://forge.rust-lang.org/infra/docs/renovate.html).

> [!WARNING]
> These presets are intended for use within the Rust Project. We don't support using them outside it.

## Quickstart

To use the [`default` preset](#default), add the following to your Renovate configuration file
(e.g. `.github/renovate.json5`):

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>rust-lang/renovate"]
}
```

If you want to learn how to customize Renovate's behavior, keep reading!

## Presets

Here's a diagram of the presets and how they extend each other
(e.g. `default` extends both `actions` and `lockfile`):

```mermaid
graph TD
    recommended["config:recommended"] --> base["base"]
    base --> actions["actions"]
    base --> lockfile["lockfile"]
    actions --> default["default"]
    lockfile --> default
```

### `base`

Extends [`config:recommended`](https://docs.renovatebot.com/presets-config/#configrecommended)
and enables [vulnerability alert](https://docs.renovatebot.com/configuration-options/#vulnerabilityalerts) PRs.

This preset keeps Renovate's default behavior: raising one PR per dependency update.
If you think this is too noisy, have a look at the other presets and the [personalization](#personalization) section below.

> [!NOTE]
> To receive vulnerability alert PRs, an admin needs to enable the
> settings [Dependency graph](https://docs.github.com/en/code-security/concepts/supply-chain-security/about-the-dependency-graph)
> and [Dependabot alerts](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/managing-security-and-analysis-settings-for-your-repository).

### `actions`

Enables GitHub Actions updates, pinning action digests to SemVer-compatible refs.
All GitHub Actions updates are grouped into a single PR and scheduled weekly.

### `lockfile`

Enables weekly lock file updates (e.g. `cargo update`) and disables PRs for non-breaking updates to Rust, JavaScript, and Python ecosystem packages.
This is because lock file updates already include non-breaking updates. Breaking updates (e.g. `1.2.3` to `2.0.0`) are handled in separate PRs.

### `default`

This is the recommended preset for most repositories in the Rust Project.

## Use a preset in a repository

The [quickstart](#quickstart) section above shows how to use the `default` preset in a repository.

To use a different preset (e.g. `actions`), add the following to your Renovate configuration file:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>rust-lang/renovate:actions"]
}
```

## Personalization

Presets won't work for every repository. You can adopt them and customize them in your
repository by overriding specific configuration options.

### Disable PRs for breaking changes

The [`lockfile`](#lockfile) and the [`default`](#default) presets disable PRs for non-breaking updates,
updating them in the lock file maintenance PR instead.

You will still get one PR for every update to a new incompatible version (e.g. `0.1.2` to `0.2.0`).
If you also want to disable these PRs, you can use
[`matchJsonata`](https://docs.renovatebot.com/configuration-options/#packagerulesmatchjsonata) in your configuration file:

```json
"packageRules": [
  {
    "matchCategories": ["rust", "js", "python"],
    "matchJsonata": ["isBreaking == true"],
    "enabled": false
  }
]
```

### Schedule

Both the `actions` and `lockfile` presets default to Renovate's
[`schedule:weekly`](https://docs.renovatebot.com/presets-schedule/#scheduleweekly)
preset, which resolves to before 4 AM on Monday.

To override them in a repository:

```json5
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>rust-lang/renovate"],
  "lockFileMaintenance": {
    "extends": ["schedule:monthly"]
  },
  "packageRules": [
    {
      "matchManagers": ["github-actions"],
      "extends": ["schedule:monthly"]
    }
  ],
}
```

> [!NOTE]
> `schedule:monthly` means *"on the first day of the month, before 4 AM"*.
> See the [Schedule Presets](https://docs.renovatebot.com/presets-schedule/#schedulemonthly) documentation for more details.

### Minimum release age

The [minimum release age](https://docs.renovatebot.com/key-concepts/minimum-release-age/)
option allows you to specify a minimum age for releases before Renovate considers them for updates.

```json5
{
  "minimumReleaseAge": "7 days",
}
```

We didn't set it by default because:

* You could miss security updates. If a GitHub repository isn't set up correctly
  or a security vulnerability isn't properly reported, Renovate won't update
  to the latest patched version.
* It could be confusing for maintainers.

We might enable this option in the future.

### Automerge

The [automerge](https://docs.renovatebot.com/configuration-options/#automerge)
option tells Renovate to automatically merge PRs when CI checks pass and there are no conflicts.

```json5
{
  "automerge": true,
}
```

> [!NOTE]
> The `automerge` option is only available in the `renovate` GitHub App.
> If you use `forking-renovate`, this option is not available. See the [Forge](https://forge.rust-lang.org/infra/docs/renovate.html?highlight=renovate#1-install-the-renovate-github-app) for more details.

Automerge is disabled by default.

### Security only updates

To disable all updates except security updates, you can use
[`matchPackageNames`](https://docs.renovatebot.com/configuration-options/#packagerulesmatchpackagenames)
in your configuration file:

```json5

{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>rust-lang/renovate"],
  "packageRules": [
    {
      "enabled": false,
      "matchPackageNames": [
        "*"
      ]
    }
  ]
}
```

Alternatively, you can disable only certain [categories](https://docs.renovatebot.com/modules/manager/#supported-managers)
using [`matchCategories`](https://docs.renovatebot.com/configuration-options/#packagerulesmatchcategories).
For example, to disable updates for Rust and JavaScript:

```json5
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>rust-lang/renovate"],
  "packageRules": [
    {
      "enabled": false,
      "matchCategories": ["rust", "js"]
    }
  ]
}
```

> [!NOTE]
> Some projects won't report their security vulnerabilities.
> We therefore recommend keeping dependencies up to date without relying only on security updates.

## References

- [Shareable Config Presets](https://docs.renovatebot.com/config-presets/)
- [Schedule Presets](https://docs.renovatebot.com/presets-schedule/)
- [Configuration Options](https://docs.renovatebot.com/configuration-options/)
