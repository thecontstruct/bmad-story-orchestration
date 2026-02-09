# File Resolution Rules

**Purpose:** Rules for resolving input paths into the target file set.

---

## Folder Input Resolution

When input is a folder path:

1. **Traverse recursively** through all subdirectories
2. **Collect files** matching `file_extensions.target` from config (default: `.md`)
3. **Exclude** hidden directories (starting with `.`) and `node_modules`
4. **Sort** files by path (alphabetical) for deterministic ordering
5. **Return** the resolved file list

## Explicit File List Resolution

When input is an array of file paths:

1. **Validate** each file exists at the given path
2. **Resolve** relative paths from workspace root
3. **If any file missing:** Report which files are missing, ask user to confirm or correct
4. **Return** the validated file list

## Supporting Files

Supporting files (matching `file_extensions.supporting` from config) are NOT included in the target file set. They are:
- Discovered and read by the reviewing agent during per-file and cross-file reviews
- Only loaded when referenced by a target file
- Not tracked in the report's `file_paths` array

## Validation Rules

- At least 1 target file must be resolved (error if 0)
- Warn if more than 20 target files (large set may affect cross-file review quality)
- Display resolved file list to user before proceeding
