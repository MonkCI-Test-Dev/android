# MonkCI Configuration and Hooks

This directory contains MonkCI-specific configuration and hooks to handle
artifact capture issues with checkov SARIF output.

## Issue

Checkov, when invoked with `--output-file-path checkov.sarif`, creates a
directory named `checkov.sarif` containing the SARIF results, rather than
a single file. MonkCI's artifact capture logic expects `checkov.sarif` to
be a file and attempts to copy it with `cp checkov.sarif "$CAPTURE_DIR/checkov.sarif"`,
which fails with "cp: -r not specified; omitting directory".

## Solution

### Hooks

- **`pre-artifact-capture`**: Runs before MonkCI captures artifacts. Converts
  the `checkov.sarif` directory to a single file by extracting the SARIF content
  from within the directory.

- **`post-job`**: Runs after the job completes. Handles copying the SARIF content
  to the capture directory if MonkCI hasn't already done so.

Both hooks are defensive and will:
1. Detect if `checkov.sarif` is a directory
2. Extract the actual SARIF file from within it
3. Replace the directory with a single file
4. Fallback to creating an empty valid SARIF file if no content is found

### Configuration

- **`.monkci.yml`** (in repository root): Attempts to configure MonkCI's behavior
  regarding checkov scanning and artifact capture.

## Related Files

- `.github/workflows/security-scan.yml`: Provides proper checkov scanning with
  correct SARIF handling, potentially replacing MonkCI's auto-scan.
- `.checkov.yml`: Checkov configuration file.
