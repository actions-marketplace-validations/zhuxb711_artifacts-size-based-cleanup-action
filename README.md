![CI](https://img.shields.io/github/actions/workflow/status/zhuxb711/artifacts-size-based-cleanup-action/ci.yml)

# Artifacts Size-based Cleanup Action

Cleanup artifacts base on the size limit to make sure the storage space is not exhausted.

This action helps you cleanup the oldest/newest artifacts when space is not enough for the pending upload artifacts.

| limit | fixedReservedSize / calcalated size | removeDirection | Total size of existing artifacts                         | Behavior                                      |
| ----- | ----------------------------------- | --------------- | -------------------------------------------------------- | --------------------------------------------- |
| 10MB  | 5MB                                 | oldest          | 6MB --> Artifact 1 (Older): 2MB, Artifact 2 (Newer): 4MB | Artifact 1 will be deleted                    |
| 10MB  | 5MB                                 | newest          | 6MB --> Artifact 1 (Older): 2MB, Artifact 2 (Newer): 4MB | Artifact 2 will be deleted                    |
| 10MB  | 5MB                                 | oldest          | 5MB --> Artifact 1 (Older): 2MB, Artifact 2 (Newer): 3MB | None (Space is enough to place new artifacts) |
| 10MB  | 5MB                                 | oldest          | 4MB --> Artifact 1 (Older): 2MB, Artifact 2 (Newer): 2MB | None (Space is enough to place new artifacts) |
| 10MB  | 5MB                                 | oldest / newest | 9MB --> Artifact 1 (Older): 3MB, Artifact 2 (Newer): 6MB | Artifact 1 & Artifact 2 will be deleted       |
| 10MB  | 12MB                                | oldest / newest | <Any>                                                    | Exception throw                               |

#### **_Make sure you run this cleanup action before upload the artifacts_**

## Authentication

### Default GITHUB_TOKEN

If you don't provide the `token` input, this action will automatically use the default `GITHUB_TOKEN` that GitHub Actions provides for each workflow run.

### Custom Token

You can provide a custom GitHub token if you need additional permissions or want to use a different token:

```yml
with:
  token: ${{ secrets.MY_CUSTOM_TOKEN }} # Should use your Personal Access Token. Token must be granted access permission with 'workflow' scope
```

However, if you do not provide the token, and want to use the `GITHUB_TOKEN` in the workflow context by default. The write access to 'action' must be grant for your `GITHUB_TOKEN`.

```yml
jobs:
  YourJobName:
    permissions:
      actions: write # Must have this permission
      contents: read # Optional. But you would need this to check out the code which is necessary in most case
```

## Glob Pattern Support

This action supports glob patterns (wildcards) in `artifactPaths` for flexible file matching:

### Supported Patterns

- `*` - Matches any number of characters (except path separators)
- `?` - Matches exactly one character
- `**` - Matches any number of directories recursively
- `[abc]` - Matches any character in brackets
- `{a,b,c}` - Matches any of the alternatives

### Examples

- `dist/**/*` - All files in dist directory and subdirectories
- `build/*.zip` - All .zip files in build directory
- `logs/app-*.log` - Log files matching pattern like app-debug.log, app-error.log
- `output/**/*.{tar.gz,zip}` - All .tar.gz and .zip files in output directory tree
- `temp/cache-[0-9]*.tmp` - Temporary files with numeric prefixes

**Note:** Glob patterns are resolved at runtime, so you can use dynamic patterns that match different files based on your build output.

## Usage

See also [action.yml](https://github.com/zhuxb711/artifacts-size-based-cleanup-action/blob/main/action.yml)

### Example without explicit token (uses default GITHUB_TOKEN)

```yml
- name: Run cleanup action
  uses: zhuxb711/artifacts-size-based-cleanup-action@v1
  with:
    limit: 1GB
    artifactPaths: <Your path to the files or directories that pending uploads>
```

### Example with multiple artifact paths

```yml
- name: Run cleanup action
  uses: zhuxb711/artifacts-size-based-cleanup-action@v1
  with:
    limit: 1GB
    artifactPaths: |
      dist/index.js
      src/main.ts
```

### Example with glob patterns (wildcards)

```yml
- name: Run cleanup action
  uses: zhuxb711/artifacts-size-based-cleanup-action@v1
  with:
    limit: 1GB
    artifactPaths: |
      dist/**/*
      build/*.zip
      output/**/*.tar.gz
      logs/app-*.log
```

### Example with a fixed size that need to be reserved

```yml
- name: Run cleanup action
  uses: zhuxb711/artifacts-size-based-cleanup-action@v1
  with:
    limit: 1GB
    fixedReservedSize: 512MB # Will delete the artifacts until this size satisfied. Which means all the artifacts remain will less than (1GB - 512MB = 512MB)
```

### Example that cleanup all the artifacts

```yml
- name: Run cleanup action
  uses: zhuxb711/artifacts-size-based-cleanup-action@v1
  with:
    limit: 0KB
```

### Complete example

```yml
- name: Run cleanup action
  uses: zhuxb711/artifacts-size-based-cleanup-action@v1
  with:
    token: ${{ secrets.MY_CUSTOM_TOKEN }} # Optional: Token must be granted access permission with 'workflow' scope. Will use default GITHUB_TOKEN if not provided.
    limit: 1GB # Could also set to 1024MB/512KB/2.5GB or size in bytes.
    fixedReservedSize: 512MB # Optional. Fixed size you want to reserved for the new artifacts. Must set 'artifactPaths' or 'fixedReservedSize'.
    failOnError: true # Optional. Reports failure if meet any exception.
    removeDirection: oldest # Optional. Remove the oldest artifact first or the newest one first.
    simulateCompressionLevel: 9 # Optional. Only works if artifactPaths used. Should be the same value as you specific in the upload artifacts action. This parameter is used to calculate the actual artifact size that would be uploaded.
    artifactPaths:
      | # Optional. The file paths that pending uploads. Supports glob patterns. Could be file path or directory path. Must set 'artifactPaths' or 'fixedReservedSize' unless 'limit' is less than or equal to zero.
      dist/**/*
      build/*.{zip,tar.gz}
      logs/app-*.log
```
