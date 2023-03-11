# workflows.common
Contains common workflows like updating changelogs etc.

NOTE: All examples use @release. You should pin this to a major semver version.

Also have a look at the parameters of the workflows in their files. They are documented there. No need to duplicate it here.
All workflows that are prefixed with self_ are workflows that are for this repo only like release management and cannot be called.

## Workflows

### Calculate Version

This workflow calculates the version based on the latest tag and the current branch. It then sets the
version as an output which can be used in other workflows.

```yaml
name: Prepare Release
on:
  push:
    branches: [ release ]
    paths-ignore:
      - '**.md'

jobs:
  calculate_next_version:
    uses: codenamephp/workflows.common/.github/workflows/calculate-next-version.yml@relase
  update_changelog:
    uses: codenamephp/workflows.common/.github/workflows/update-changelog.yml@release
    needs: calculate_next_version
    with:
      ref: ${{github.ref_name}}
      future-release: ${{ needs.calculate_next_version.outputs.version }}
      release-branch: 'release'
```

### Update Changelog

This workflow updates the changelog file with the latest changes. This works when all your features are
developed in a feature branch and merged using a pull request.

```yaml
name: Update Changelog
on:
  push:
    branches: [ main ]
    paths-ignore:
      - "**.md"

jobs:
  update_changelog:
    uses: codenamephp/workflows.common/.github/workflows/update-changelog.yml@release
    with:
      ref: ${{github.ref_name}}
```

This works best in conjuction with the Calculate Version workflow.

### Draft Release

This workflow creates a draft release with the latest changes. This works when all your features are
developed in a feature branch and merged using a pull request.

This works well with the calculate version workflow.

```yaml
name: Prepare Release
on:
  push:
    branches: [ release ]
    paths-ignore:
      - '**.md'

jobs:
  calculate_next_version:
    uses: codenamephp/workflows.common/.github/workflows/calculate-next-version.yml@release
  draft_release:
    needs: calculate_next_version
    uses: codenamephp/workflows.common/.github/workflows/draft-release.yml@release
    with:
      version: ${{ needs.calculate_next_version.outputs.version }}
```
### Update Release Versions

This workflow is intended for reusable workflow repos (like this one). It will update the major and minor tags.

```yaml
name: Update Release Versions

on:
  release:
    types:
      - published

jobs:
  update_release_versions:
    uses: codenamephp/workflows.common/.github/workflows/updateReleaseVersions.yml@release
    with:
      version: ${{ github.event.release.tag_name }}
      release_ref: ${{ github.event.release.target_commitish }}
```

## Examples

### Prepare Relase

This workflow can be used to automatically prepare a release. It will calculate the next version, update
the changelog and create a draft release. This is based on a dedicated release branch.

```yaml
name: Prepare Release
on:
  push:
    branches: [ release ]
    paths-ignore:
      - '**.md'

jobs:
  calculate_next_version:
    uses: codenamephp/workflows.common/.github/workflows/calculate-next-version.yml@release
  draft_release:
    needs: calculate_next_version
    uses: codenamephp/workflows.common/.github/workflows/draft-release.yml@release
    with:
      version: ${{ needs.calculate_next_version.outputs.version }}
  update_changelog:
    uses: codenamephp/workflows.common/.github/workflows/update-changelog.yml@release
    needs: calculate_next_version
    with:
      ref: ${{github.ref_name}}
      future-release: ${{ needs.calculate_next_version.outputs.version }}
      release-branch: 'release'
```