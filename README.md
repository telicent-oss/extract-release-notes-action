# Extract Release Notes Action

This repository provides a GitHub Action that tries to automatically extract release notes based on a provided Change
Log file, e.g. `CHANGELOG.md`, or automatically using [GitHub's automatic release notes][GitHubAutoRelease]
functionality.

You can optionally fail the build if no release notes can be extracted using the [`fail-if-missing`](#inputs) input.

It produces several [outputs](#outputs) detailing the availability of release notes, the release notes get written to a
local file in the workspace by default, but can be optionally uploaded as a build artifact by setting the
[`attach-release-notes`](#inputs) to `true` to enable sharing the extracted release notes between multiple jobs in a
workflow.

# Requirements

- This action requires a runner with a Bash shell available

# Usage

At its most basic the action is used as follows:

```yaml
name: Extract Release Notes Example
on: 
  push:
    tags:
      - '**'
  workflow_dispatch:

jobs:
  example:
    runs-on: ubuntu-latest
    permissions:
      contents: read

    steps:
      - name: Checkout
        uses: actions/checkout@v3

      # Run the Extract Release Notes
      - name: "Extract Release Notes"
        id: release-notes
        uses: telicent-oss/extract-release-notes-action@v1
        with:
          changelog-file: CHANGELOG.md
          version: 1.2.3 

      # Add steps that use the output as necessary e.g.
      - name: Print the Release Notes
        run: |
          cat "${{ steps.release-notes.output.release-notes-file }}
```

A more advanced usage might look like the following:

```yaml
name: Advanced Extract Release Notes Example
on: 
  push:
    tags:
      - '**'

jobs:
  example:
    runs-on: ubuntu-latest
    permissions:
      contents: write

    steps:
      - name: Checkout
        uses: actions/checkout@v3

      # Run the Extract Release Notes Action
      - name: "Extract Release Notes"
        id: release-notes
        uses: telicent-oss/extract-release-notes-action@v1
        with:
          changelog-file: CHANGELOG.md
          # Use the tag as the version input
          version: ${{ github.ref_name }}
          # Fail the build if no release notes are found for this tag
          fail-if-missing: true
          # Add the release notes as GitHub Job Summary content
          job-summary: true 
          # Attach the release notes as a build artifact
          attach-release-notes: true

      # Generate a GitHub release with the release notes
      - name: Create GitHub Release
        uses: softprops/action-gh-release@v2
        with:
          body_path: ${{ steps.release-notes.outputs.release-notes-file }}
          generate_release_notes: ${{ steps.release-notes.outputs.auto-release-notes }}
          name: ${{ github.ref_name }}
```

## Inputs

The following inputs are supported by this action:

| Input | Default | Purpose |
|-------|---------|---------|
| `changelog-file` | `CHANGELOG.md` | Specifies the file from which release notes should be extracted, see [format](#changelog-format) notes below. |
| `version` | | **REQUIRED:** Specifies the version whose release notes you wish to extract. |
| `release-notes-file` | `release-notes.txt` | Specifies the file the release notes will be extracted to, this filename will also be made available as an [output](#outputs). |
| `fail-if-missing` | `false` | Whether to fail the action if no release notes can be extracted for the version from the Change Log file. |
| `attach-release-notes` | `false` | Whether to attach the extracted release notes as a build artifact, this will be named `<job-name>-release-notes-<version><suffix>` where `<job-name>` is the name of the calling job, `<version>` the supplied `version` input, and `<suffix>` the optional `attachment-suffix` input. |
| `attachment-suffix` |  | An optional suffix used for the attached build artifact, generally only needed for matrix jobs if the job name and input `version` are not unique for each job within the matrix. |
| `job-summary` | `true` | Whether to add the extracted release notes as a job summary per https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions#adding-a-job-summary |

## Outputs

The following outputs are produced by this action:

| Output | Description |
|--------|-------------|
| `release-notes-files` | The path to the extracted release notes, generally identical to the `release-notes-file` input **UNLESS** no Change Log file was configured, or no release notes were extractable for the provided version. |
| `release-notes-artifact` | The name of the release notes artifact attached to the build, while this output is always set this artifact only exists if the `attach-release-notes` input was set to `true`. |
| `auto-release-notes` | A boolean indicating whether [automatic GitHub release note generation][GitHubAutoRelease] is configured for the repository. |

# Changelog Format

The action assumes that the Change Log file is formatted in Markdown and uses `#` style headings.

In order to find the release notes for a release it looks for a line starting with one or more `#` characters, followed
by some space characters, followed by the specified input `version` and then a new line character.

For example if `version` was set to `1.2.3` then both `# 1.2.3` and `### 1.2.3` would be acceptable, but `## 1.2.3.0`
would not.  Similarly, if your Change Log file has headers like `### Version 1.2.3` then this won't match either.

Once this line is found it then reads all the subsequent lines until the end of file is reached, or another
header line, any line starting with a `#` character, is reached.

Thus, if you use subheaders within your Change Log only the first portion of the release notes from your release will
currently be extracted.

[GitHubAutoRelease]: https://docs.github.com/en/repositories/releasing-projects-on-github/automatically-generated-release-notes