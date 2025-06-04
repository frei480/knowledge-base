---
tags:
  - git
---

`git blame` is a command used in Git to display detailed information about each line in a file, including:

1. **Who modified it**: The author of the change.
2. **When it was modified**: The commit timestamp.
3. **Which commit introduced it**: The commit hash.
4. **The line's current content**.

### Basic Usage
```bash
git blame <file-path>
```
This annotates every line in the specified file.

### Example Output
```
f3d8a7b1 (Alice Developer 2023-10-05 14:30:22 +0200 1) function calculate() {
a1b2c3d4 (Bob Coder    2023-10-04 09:15:45 +0200 2)   return a + b;
f3d8a7b1 (Alice Developer 2023-10-05 14:30:22 +0200 3) }
```
- `f3d8a7b1`: Commit hash where the line was last modified.
- `Alice Developer`: Author of the commit.
- `2023-10-05 14:30:22 +0200`: Timestamp of the commit.
- Line numbers and content follow.

### Key Options
| Option | Description |
|--------|-------------|
| `-L <start>,<end>` | Show blame for lines between `<start>` and `<end>`. |
| `-C` | Detect lines moved or copied within the same file (use `-C -C` or `-C -C -C` to check across files). |
| `--since=<date>` | Only consider commits older than `<date>`. |
| `-e` | Show the author's email instead of name. |
| `-w` | Ignore whitespace changes. |

### Use Cases
- **Debugging**: Identify when a bug was introduced by checking the commit history of specific lines.
- **Code Review**: Understand why a line was added by linking it to a commit message (`git show <commit-hash>`).
- **Tracking Refactoring**: Use `-C` to find where code was moved or copied from another file.

### Example Commands
1. Blame lines 10-20 of `app.js`:
   ```bash
   git blame -L 10,20 app.js
   ```
2. Detect code moved between files:
   ```bash
   git blame -C -C -C app.js
   ```
3. Blame only up to a specific commit:
   ```bash
   git blame --since="3 weeks ago" app.js
   ```

### Notes
- `git blame` does **not** track file renames by default (use `-C` to help with this).
- Combine with `git log -p` or `git show <commit-hash>` to dive deeper into changes.
- GUI tools (e.g., VS Code, GitKraken) often provide interactive blame annotations.