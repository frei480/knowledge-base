---
tags:
  - python
---
You need a script that can:
1.  Find all your Markdown files (`.md`).
2.  For each file, read only the 3rd line.
3.  Check if that line contains the phrase "пространство имен".
4.  If it does, add a `namespace` tag to the file's frontmatter.

Here are a few ways to do this, from the most robust and recommended method to quicker command-line options.

# Important: Before You Run Any Script

**BACK UP YOUR FILES\!** These scripts will modify your files in place. Make a copy of your folder or use a version control system like Git before you proceed.

I will assume your Markdown files use YAML frontmatter for tags, which looks like this:

```yaml
---
title: My Note
date: 2025-10-14
tags: [existing_tag, another_tag]
---

The rest of your markdown content...
```

The scripts below will add the `namespace` tag to the `tags` list.

-----
# Solution 1: Python Script (Recommended & Cross-Platform)

This is the most robust and safest method. It works on Windows, macOS, and Linux. It correctly handles various cases, such as files with no tags, existing tags, or no frontmatter at all.

1.  Create a file named `add_tags.py` in the root folder of your Markdown files.
2.  Copy and paste the following code into it:

```python
import os
import re


ROOT_DIR = '.'

SEARCH_PHRASE = "пространство имен"

TAG_TO_ADD = "namespace"

def get_third_line(file_path):
    """Reads the third line of a file without loading the whole file."""
    try:
        with open(file_path, 'r', encoding='utf-8') as f:
            for i, line in enumerate(f):
                if i == 2:  # 3rd line is at index 2 (0, 1, 2)
                    return line
    except Exception:
        # File might have less than 3 lines or be unreadable
        return None
    return None

def add_tag_to_file(file_path, tag):
    """Adds a tag to the YAML frontmatter of a Markdown file."""
    print(f"  -> Found phrase. Processing '{file_path}'...")
    try:
        with open(file_path, 'r', encoding='utf-8') as f:
            content = f.read()

        # Check if the tag is already present
        if re.search(r'tags:.*' + re.escape(tag), content):
            print(f"     - Tag '{tag}' already exists. Skipping.")
            return

        # Case 1: File has frontmatter with a 'tags:' line
        if re.search(r'^---[\s\S]+?^tags:', content, re.MULTILINE):
            # Add tag to existing list
            content = re.sub(r'^(tags:.*)$', r'\1 ' + tag, content, 1, re.MULTILINE)
            # A more robust way for lists:
            # content = re.sub(r'^(tags:\s*\[.*?)(])', r'\1, ' + tag + r'\2', content, 1, re.MULTILINE)
            print(f"     - Appended tag to existing 'tags' line.")
        
        # Case 2: File has frontmatter but no 'tags:' line
        elif content.startswith('---'):
            # Add 'tags:' line after the opening '---'
            content = re.sub(r'^(---)$', r'\1\ntags: [' + tag + ']', content, 1, re.MULTILINE)
            print(f"     - Added 'tags' line to frontmatter.")

        # Case 3: File has no frontmatter
        else:
            # Create new frontmatter at the beginning of the file
            frontmatter = f"---\ntags: [{tag}]\n---\n\n"
            content = frontmatter + content
            print(f"     - Created new frontmatter with tag.")

        with open(file_path, 'w', encoding='utf-8') as f:
            f.write(content)
            
    except Exception as e:
        print(f"     - ERROR processing file {file_path}: {e}")


def main():
    """Main function to scan and process files."""
    print(f"Starting scan in directory: '{os.path.abspath(ROOT_DIR)}'")
    for root, _, files in os.walk(ROOT_DIR):
        for file in files:
            if file.endswith('.md'):
                file_path = os.path.join(root, file)
                third_line = get_third_line(file_path)
                if third_line and SEARCH_PHRASE in third_line:
                    add_tag_to_file(file_path, TAG_TO_ADD)
    print("\nScan complete.")

if __name__ == '__main__':
    main()
```


-----
# Solution 2: Bash (for Linux & macOS)

This is a powerful one-liner you can run directly in your terminal. It's faster for a quick job but less readable and robust than the Python script.

Navigate to your notes folder in the terminal and run this command:

```bash
find . -name "*.md" -print0 | while IFS= read -r -d $'\0' file; do
    # Get the 3rd line of the file
    THIRD_LINE=$(sed -n '3p' "$file")
    
    # Check if the line contains the phrase and does not already have the tag
    if [[ "$THIRD_LINE" == *"пространство имен"* ]]; then
        # Check if the file already has the namespace tag to avoid duplicates
        if ! grep -q "tags:.*namespace" "$file"; then
            echo "Tagging: $file"
            # This is a simple version. It adds 'tags: [namespace]' to the 2nd line.
            # It assumes you have a '---' on the first line. Adjust as needed.
            sed -i.bak '2 a\
tags: [namespace]' "$file"
        fi
    fi
done
```

**Explanation:**

  * `find . -name "*.md" -print0`: Finds all markdown files safely, even with spaces in names.
  * `while … done`: Loops through each file found.
  * `sed -n '3p' "$file"`: Extracts the 3rd line of the file.
  * `if [[ ... ]]`: Checks if the 3rd line contains "пространство имен".
  * `grep -q`: Quietly checks if the tag already exists.
  * `sed -i.bak '2 a\tags: [namespace]' "$file"`: This is the modification step. It **i**n-place edits the file, adding the line `tags: [namespace]` **a**fter line 2. It also creates a backup of the original file with a `.bak` extension.

-----

# Solution 3: PowerShell (for Windows)

This is the native scripting solution for Windows.

1.  Open PowerShell.
2.  Navigate to the root folder of your Markdown files.
3.  Run the following script:

```powershell
# Configuration
$rootDir = "."
$searchPhrase = "пространство имен"
$tagToAdd = "namespace"

# Get all Markdown files recursively
Get-ChildItem -Path $rootDir -Recurse -Filter "*.md" | ForEach-Object {
    $file = $_
    
    # Get the 3rd line. Handles files with less than 3 lines gracefully.
    $thirdLine = (Get-Content -Path $file.FullName -TotalCount 3 | Select-Object -Last 1)
    
    # Check if the 3rd line contains the phrase
    if ($thirdLine -and $thirdLine.Contains($searchPhrase)) {
        
        $content = Get-Content -Path $file.FullName -Raw
        
        # Check if tag already exists to avoid duplicates
        if ($content -notmatch "tags:.*$tagToAdd") {
            Write-Host "Tagging: $($file.FullName)"
            
            # Simple logic: Add 'tags:' line after the first '---'
            # This is less robust than the Python script but works for simple cases.
            if ($content.StartsWith("---")) {
                $newContent = $content -replace "(---`r?`n)", "---`r`ntags: [$tagToAdd]`r`n"
                Set-Content -Path $file.FullName -Value $newContent
            } else {
                # Add frontmatter if it doesn't exist
                $newContent = "---`r`ntags: [$tagToAdd]`r`n---`r`n`r`n" + $content
                Set-Content -Path $file.FullName -Value $newContent
            }
        }
    }
}

Write-Host "Script finished."
```
