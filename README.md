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
| `version` | | Specifies the version whose release notes you wish to extract. **REQUIRED** |
| `release-notes-file` | `release-notes.txt` | Specifies the file the release notes will be extracted to, this filename will also be made available as an [output](#outputs) |

## Outputs

The following outputs are produced by this action:

| Output | Description |
|--------|-------------|
| `release-notes-files` | The path to the extracted release notes, generally identical to the `release-notes-file` input **UNLESS** no Change Log file was configured, or no release notes were extractable for the provided version. |
| `auto-release-notes` | A boolean indicating whether [automatic GitHub release note generation][GitHubAutoRelease] is configured for the repository. |

# Changelog Format

The action assumes that the Change Log file is formatted in Markdown and uses `#` style headings.

In order to find the release notes for a release it looks for a line starting with one or more `#` characters and
containing the specified input `version` e.g. both `# 1.2.3` and `### 1.2.3` would be acceptable.  Once this line is
found it then reads all the subsequent lines until the end of file is reached, or another header line (starting with a
`#`) character is reached.

Thus, if you use subheaders within your Change Log only the first portion of the release notes from your release will
currently be extracted.


[GitHubAutoRelease]: https://docs.github.com/en/repositories/releasing-projects-on-github/automatically-generated-release-notes