# DemLeagueOS plugin workflows

The shared release workflow for DemLeagueOS plugins. A plugin is only installed in DemLeagueOS if it was
built, attested and published by [`plugin-release.yml`](.github/workflows/plugin-release.yml) from this
repository's `main` branch.

Each plugin lives in its own repository in the `unifyamerica` org, named like its package
(`unifyamerica/demleague-plugin-<vendor>-<module>` for `@unifyamerica/demleague-plugin-<vendor>-<module>`),
and calls the workflow on a version tag:

```yaml
# .github/workflows/release.yml
name: Release
on:
  push:
    tags: ['v*']
permissions:
  contents: read
  packages: write
  id-token: write
  attestations: write
jobs:
  release:
    uses: unifyamerica/demleague-plugin-workflows/.github/workflows/plugin-release.yml@main
```

The workflow runs in two jobs:

1. **build** runs the plugin's own code with read-only access: it checks that the repository name, package
   name, `repository` field and tag agree, then runs lint, type-check, tests and the build, and packs the
   package.
2. **publish** runs none of the plugin's code: it checks the tarball's name and version again, attests it
   (SLSA build provenance, signed through GitHub) and publishes it to GitHub Packages.

Because the plugin's scripts never run in the job that holds the signing identity, a plugin can't get any
other file attested as this workflow's build.

The example plugins use it: [demleague-plugin-acme-notes](https://github.com/unifyamerica/demleague-plugin-acme-notes),
the template for new plugins, and
[demleague-plugin-acme-tasks](https://github.com/unifyamerica/demleague-plugin-acme-tasks), which shows one module
calling another's exported operations. The workflow treats them alike: calls between modules are approved in
DemLeagueOS' namespace registry, not at release time. The SDK is
[demleague-plugin-sdk](https://github.com/unifyamerica/demleague-plugin-sdk).

Attestations need the plugin repository to be public on the organization's current GitHub plan.
