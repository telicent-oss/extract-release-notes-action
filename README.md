# Extract Release Notes Action

This repository provides a GitHub Action that tries to automatically extract release notes based on a provided Change
Log file, e.g. `CHANGELOG.md`, or automatically using [GitHub's automatic release notes][GitHubAutoRelease]
functionality.

It produces two outputs, a `string` with a path to a file containing the extracted release notes (if any), and a
`boolean` indicating whether automatic release note generation has been configured.

# Requirements

- This action requires a runner with a Bash shell available

# Usage

At its most basic the action is used as follows:

```yaml
name: Extract Release Notes Example
on: 
  push:
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
        uses: telicent-oss/extract-release-notes-action@v1
        with:
          changelog-file: CHANGELOG.md
          version: 1.2.3 

      # Add steps that use the output as necessary e.g.

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