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
[ -r file ]       # File is readable
[ -w file ]       # File is writable
[ -x file ]       # File is executable
[ -s file ]       # File size > 0

# File operations
touch file.txt           # Create empty file
cp source destination    # Copy file/directory
mv old_name new_name    # Move/rename
rm file.txt             # Remove file
rm -rf directory        # Remove directory recursively
```

## Essential Commands

| Command | Description | Common Options |
|---------|-------------|----------------|
| `ls` | List directory contents | `-la` (long + hidden), `-lh` (human readable) |
| `cd` | Change directory | `cd -` (previous), `cd ~` (home) |
| `pwd` | Print working directory | |
| `find` | Search files/directories | `-name "*.txt"`, `-type f`, `-exec` |
| `grep` | Search text patterns | `-r` (recursive), `-i` (ignore case), `-n` (line numbers) |
| `sed` | Stream editor | `s/old/new/g` (substitute), `-i` (in-place) |
| `awk` | Text processing | `'{print $1}'` (first column) |
| `sort` | Sort lines | `-n` (numeric), `-r` (reverse), `-k2` (by column 2) |
| `uniq` | Remove duplicates | `-c` (count), requires sorted input |
| `cut` | Extract columns | `-d','` (delimiter), `-f1,3` (fields 1,3) |
| `head` | First lines | `-n 10` (first 10 lines) |
| `tail` | Last lines | `-n 10` (last 10), `-f` (follow) |
| `wc` | Word/line/char count | `-l` (lines), `-w` (words), `-c` (characters) |

## Process Management

```bash
# Process information
ps aux              # All processes
ps -ef              # All processes (alternative format)
top                 # Real-time process viewer
htop                # Enhanced top (if installed)
pgrep process_name  # Find process ID by name
pkill process_name  # Kill process by name

# Job control
command &           # Run in background
Ctrl+Z              # Suspend current job
jobs                # List jobs
fg %1               # Bring job 1 to foreground
bg %1               # Continue job 1 in background
nohup command &     # Run immune to hangups
```

## Control Structures

### Conditionals
```bash
# If statement
if [ condition ]; then
    echo "true"
elif [ other_condition ]; then
    echo "other true"
else
    echo "false"
fi

# Case statement
case "$variable" in
    pattern1)
        echo "matches pattern1"
        ;;
    pattern2|pattern3)
        echo "matches pattern2 or pattern3"
        ;;
    *)
        echo "default case"
        ;;
esac
```

### Loops
```bash
# For loop
for item in list item1 item2; do
    echo "$item"
done

# For loop with range
for i in {1..10}; do
    echo "Number: $i"
done

# While loop
while [ condition ]; do
    echo "running"
    # Don't forget to modify condition!
done

# Until loop
until [ condition ]; do
    echo "waiting"
done
```

## Functions

```bash
# Function definition
function_name() {
    local local_var="$1"  # First parameter, local scope
    echo "Processing: $local_var"
    return 0  # Exit status
}

# Function call
function_name "argument"
result=$?  # Capture return status

# Parameters
# $0 - script name
# $1, $2, $3... - positional parameters
# $# - number of parameters
# $@ - all parameters as separate words
# $* - all parameters as single word
# $? - exit status of last command
```

## I/O Redirection

```bash
# Output redirection
command > file.txt          # Redirect stdout to file (overwrite)
command >> file.txt         # Redirect stdout to file (append)
command 2> error.log        # Redirect stderr to file
command &> all_output.log   # Redirect both stdout and stderr
command > /dev/null 2>&1    # Discard all output

# Input redirection
command < input.txt         # Read from file
command << EOF              # Here document
Line 1
Line 2
EOF

# Pipes
command1 | command2         # Pipe stdout of cmd1 to stdin of cmd2
command1 |& command2        # Pipe both stdout and stderr
```

## String Operations

```bash
string="Hello World"

# Length
echo ${#string}             # 11

# Substring extraction
echo ${string:6:5}          # "World" (start at 6, length 5)
echo ${string:6}            # "World" (start at 6 to end)

# Pattern matching
echo ${string#Hello }       # "World" (remove shortest match from beginning)
echo ${string##*/}          # Remove longest match from beginning
echo ${string%World}        # "Hello " (remove shortest match from end)
echo ${string%%/*}          # Remove longest match from end

# Pattern replacement
echo ${string/World/Bash}   # "Hello Bash" (replace first match)
echo ${string//o/0}         # "Hell0 W0rld" (replace all matches)
```

## Arrays

```bash
# Array declaration
array=("item1" "item2" "item3")
declare -a indexed_array
declare -A associative_array

# Array operations
array[0]="new_item"         # Set element
echo "${array[0]}"          # Get element
echo "${array[@]}"          # All elements
echo "${#array[@]}"         # Array length
echo "${!array[@]}"         # All indices

# Associative arrays
associative_array["key"]="value"
echo "${associative_array[key]}"

# Loop through array
for item in "${array[@]}"; do
    echo "$item"
done
```

## Best Practices & Gotchas

### ✅ Do's
- Always quote variables: `"$variable"`
- Use `#!/bin/bash` shebang for bash-specific features
- Use `local` for function variables
- Check command success: `if command; then ... fi`
- Use `set -e` to exit on first error
- Use `set -u` to exit on undefined variables
- Use `shellcheck` to validate scripts

### ❌ Don'ts
- Don't use `ls` output in scripts, use globs or `find`
- Don't parse `ps` output, use `pgrep`/`pkill`
- Don't use `which`, use `command -v`
- Don't use backticks, use `$(command)`
- Don't use `cat file | command`, use `command < file`

### 🔧 Common Gotchas
- **Spaces in filenames**: Always quote paths and use arrays
- **Exit codes**: 0 means success, non-zero means failure
- **Variable scope**: Variables are global by default in functions
- **Command substitution**: `$(command)` strips trailing newlines
- **Test operators**: Use `[[` for advanced tests, `[` for POSIX compatibility

### 🛡️ Error Handling
```bash
#!/bin/bash
set -euo pipefail  # Exit on error, undefined vars, pipe failures

# Trap errors
error_handler() {
    echo "Error on line $1" >&2
    exit 1
}
trap 'error_handler $LINENO' ERR

# Check command success
if ! command_that_might_fail; then
    echo "Command failed" >&2
    exit 1
fi
```

### 📝 Script Template
```bash
#!/bin/bash
# Script description
# Usage: script.sh [options] arguments

set -euo pipefail

# Global variables
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
PROGRAM_NAME="$(basename "$0")"

# Function definitions
usage() {
    cat << EOF
Usage: $PROGRAM_NAME [OPTIONS] ARGS

Description of what this script does.

OPTIONS:
    -h, --help     Show this help message
    -v, --verbose  Enable verbose output

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

This cheatsheet covers the most essential bash scripting concepts and commands used in DevOps workflows.
