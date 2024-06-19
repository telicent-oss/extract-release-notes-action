# Extract Release Notes Action

This repository provides a GitHub Action that tries to automatically extract release notes based on a provided Change
Log file, e.g. `CHANGELOG.md`, or automatically using GitHub's automatic release notes functionality.

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
| `version` | | Specifies the version whose release notes you wish to extract. |
| `release-notes-file` | `release-notes.txt` | Specifies the file the release notes will be extracted to, this filename will also be made available as an [output](#outputs) |

## Outputs

TODO

# Changelog Format

TODO

