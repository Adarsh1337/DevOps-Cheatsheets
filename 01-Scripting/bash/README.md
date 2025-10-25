# 🐚 Bash Cheatsheet

[← Back to Main ToC](../../README.md)

## Core Concepts

### Variables and Environment

```bash
# Variable assignment (no spaces around =)
VARIABLE_NAME="value"
export GLOBAL_VAR="value"  # Make available to child processes

# Variable expansion
echo "$VARIABLE_NAME"     # Standard expansion
echo "${VARIABLE_NAME}"   # Safer expansion
echo "${VAR:-default}"    # Default value if unset
echo "${VAR:=default}"    # Set and return default if unset
echo "${VAR:?error msg}"  # Error if unset
```

### File Operations

```bash
# File tests
[ -f file ]       # File exists and is regular
[ -d directory ]  # Directory exists
[ -e path ]       # Path exists
[ -r file ]       # Readable
[ -w file ]       # Writable
[ -x file ]       # Executable

# File manipulation
cp source dest    # Copy
mv source dest    # Move/rename
rm file           # Remove
mkdir -p path     # Create directory (with parents)
```

---

## Essential Commands

| Command | Description | Common Flags |
| :--- | :--- | :--- |
| `ls` | List directory contents | `-la` (long + hidden) |
| `cat` | Display file content | `-n` (line numbers) |
| `grep` | Search patterns | `-r` (recursive) |
| `find` | Find files | `-name`, `-type` |
| `chmod` | Change permissions | `755`, `644` |
| `chown` | Change ownership | `user:group` |

---

## Process Management

```bash
# Process info
ps aux              # All processes
top / htop          # Interactive process viewer
pgrep process_name  # Find process ID by name

# Process control
kill PID            # Send SIGTERM
kill -9 PID         # Force kill (SIGKILL)
killall name        # Kill by name
bg                  # Resume in background
fg                  # Bring to foreground
nohup cmd &         # Run detached from terminal
```

---

## Text Processing

```bash
# awk - pattern scanning and processing
awk '{print $1}' file         # Print first column
awk -F':' '{print $1}' file   # Custom delimiter

# sed - stream editor
sed 's/old/new/' file         # Replace first occurrence
sed 's/old/new/g' file        # Replace all occurrences
sed -i 's/old/new/g' file     # In-place edit

# cut - remove sections
cut -d':' -f1 file            # Cut by delimiter
cut -c1-10 file               # Cut by character position

# sort & uniq
sort file                     # Sort lines
sort -u file                  # Sort and remove duplicates
uniq file                     # Remove adjacent duplicates
```

---

## Conditionals

```bash
if [ condition ]; then
    # commands
elif [ condition ]; then
    # commands
else
    # commands
fi

# String comparisons
[ "$str1" = "$str2" ]    # Equal
[ "$str1" != "$str2" ]   # Not equal
[ -z "$str" ]            # Empty
[ -n "$str" ]            # Not empty

# Numeric comparisons
[ $num1 -eq $num2 ]      # Equal
[ $num1 -ne $num2 ]      # Not equal
[ $num1 -lt $num2 ]      # Less than
[ $num1 -gt $num2 ]      # Greater than
```

---

## Loops

```bash
# For loop
for item in list; do
    echo "$item"
done

# While loop
while [ condition ]; do
    # commands
done

# Until loop
until [ condition ]; do
    # commands
done

# C-style for
for ((i=0; i<10; i++)); do
    echo "$i"
done
```

---

## Functions

```bash
function_name() {
    local local_var="value"    # Local variable
    echo "Arguments: $@"
    return 0                   # Exit status
}

# Call function
function_name arg1 arg2
```

---

## Error Handling

```bash
# Exit on error
set -e                    # Exit on any error
set -u                    # Error on undefined variable
set -o pipefail           # Pipe fails if any command fails
set -euo pipefail         # Combine all

# Error checking
if ! command; then
    echo "Command failed" >&2
    exit 1
fi

# Trap errors
trap 'echo "Error on line $LINENO"' ERR
```

---

## Input/Output

```bash
# Redirection
command > file            # Overwrite
command >> file           # Append
command < file            # Input from file
command 2> error.log      # Redirect stderr
command &> all.log        # Redirect stdout and stderr

# Pipes
command1 | command2       # Pipe stdout
command1 |& command2      # Pipe stdout and stderr

# Here documents
cat <<EOF
Multi-line
text
EOF
```

---

## Best Practices Template

```bash
#!/usr/bin/env bash
set -euo pipefail

# Script metadata
SCRIPT_DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
SCRIPT_NAME="$(basename "$0")"

# Usage function
usage() {
    cat <<EOF
Usage: $SCRIPT_NAME [OPTIONS]

Description of what the script does.

Options:
    -h, --help      Show this help message
    -v, --verbose   Enable verbose output
EOF
}

# Main logic
main() {
    local verbose=false
    
    while [[ $# -gt 0 ]]; do
        case $1 in
            -h|--help)
                usage
                exit 0
                ;;
            -v|--verbose)
                verbose=true
                shift
                ;;
            *)
                echo "Unknown option: $1" >&2
                usage >&2
                exit 1
                ;;
        esac
    done
    
    # Script logic here
    [[ $verbose == true ]] && echo "Running in verbose mode"
}

# Call main with all arguments
main "$@"
```

This cheatsheet covers the most essential bash scripting concepts and
commands used in DevOps workflows.
