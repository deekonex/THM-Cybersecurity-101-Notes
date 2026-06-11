# 4.03 Linux Shell

## Shell Overview & Core Concepts

The **Shell** is the primary interface between the user and the Operating System (OS). Using CLI (Command Line Interface) commands is inherently more efficient and resource-friendly than relying on a Graphical User Interface (GUI).

**Bash (Bourne Again Shell)** is the default shell for most Linux distributions. It is a command interpreter that reads commands from the user and executes them.

### Key Features:
- **Command Execution**: Run individual commands or scripts
- **Scripting**: Automate complex tasks using shell scripts
- **Environment Variables**: Store configuration and system information
- **Job Control**: Manage multiple processes
- **History**: Command history and recall

---

## Navigation

### pwd
**Print Working Directory** - Shows the user's current location in the file system.
```bash
$ pwd
/home/user/documents
```

### cd [directory]
**Change Directory** - Navigates to a specified folder path.
```bash
cd /home/user                    # Absolute path
cd documents                     # Relative path
cd ..                           # Go to parent directory
cd ~                            # Go to home directory
cd -                            # Go to previous directory
```

### ls
**List Files** - Displays the contents of the current directory.
```bash
ls                              # Basic listing
ls -l                           # Long format with details
ls -la                          # Include hidden files
ls -lh                          # Human-readable file sizes
ls -R                           # Recursive listing (subdirectories)
```

---

## File I/O & Search

### cat [filename]
**Concatenate/Display** - Reads and displays the entire text content of a file directly to the console.
```bash
cat filename.txt                # Display file contents
cat file1.txt file2.txt         # Display multiple files
cat -n filename.txt             # Show line numbers
cat > newfile.txt               # Create file (Ctrl+D to save)
```

### grep [pattern] [filename]
**Global Search** - Highly powerful search tool. Searches within files for specific patterns or keywords, outputting only matching lines.
```bash
grep "error" logfile.txt                # Search for "error"
grep -i "error" logfile.txt             # Case-insensitive search
grep -n "error" logfile.txt             # Show line numbers
grep -c "error" logfile.txt             # Count matching lines
grep -r "error" /var/log                # Recursive search
grep -v "error" logfile.txt             # Invert match (exclude)
grep -E "pattern|another" file.txt      # Extended regex (OR)
grep "^ERROR" logfile.txt               # Lines starting with ERROR
grep -q "error" logfile.txt             # Quiet mode (no output)
```

---

## System Administration Commands

### Documentation

#### man [command]
**Manual Page** - The built-in manual page system provides comprehensive documentation.
```bash
man cat                         # Display manual for cat command
man -k keyword                  # Search manual pages by keyword
man -s 1 command                # Search specific section
man 5 passwd                    # Read about /etc/passwd format
```

### Permissions & Users

#### chmod [permissions] [filename]
**Change Mode** - Sets file/directory permissions.
```bash
chmod +x scriptname.sh          # Add executable permission
chmod 755 scriptname.sh         # rwxr-xr-x permissions
chmod 644 file.txt              # rw-r--r-- permissions
chmod -R 755 directory/         # Recursive permission change
chmod u+x file.sh               # Add execute for user only
chmod g-w file.txt              # Remove write for group
chmod o-r file.txt              # Remove read for others
```

**Permission Breakdown:**
- `u` = user (owner)
- `g` = group
- `o` = others
- `r` = read (4)
- `w` = write (2)
- `x` = execute (1)

#### sudo [command]
**Super User Do** - Execute commands with root (administrative) privileges.
```bash
sudo apt update                 # Update package list
sudo nano /etc/config           # Edit system files
sudo su                         # Switch to root user
sudo -l                         # List available commands
```

#### su - [username]
**Switch User** - Changes the current user context.
```bash
su - root                       # Switch to root user
su - username                   # Switch to specific user
exit                            # Exit to previous user
```

---

## Scripting & Automation Constructs

Shell scripting builds power by combining core commands into an executable file (.sh).

### Script Structure

#### Shebang
Must be the first line of the script. Defines which interpreter must run the code.
```bash
#!/bin/bash                     # Use bash interpreter
#!/bin/sh                       # Use POSIX shell
#!/usr/bin/python              # Use Python interpreter
```

#### Variables
Stores data values used throughout the script, preventing repetition.
```bash
name="John"                     # Assign variable
age=25
echo "Hello, $name"             # Use variable
echo "Name: ${name}"            # Safer syntax
readonly constant="VALUE"       # Read-only variable
unset variable                  # Remove variable
```

#### Command Substitution
Execute command and use output as variable value.
```bash
current_date=$(date)            # Modern syntax
current_date=`date`             # Legacy syntax
files=$(ls *.txt)               # Store command output
```

### Control Structures

#### Loops

**for loop:**
```bash
for i in {1..5}; do
    echo "Iteration $i"
done

for file in *.txt; do
    echo "Processing $file"
done

for ((i=1; i<=5; i++)); do
    echo "Count: $i"
done
```

**while loop:**
```bash
while [ $count -lt 5 ]; do
    echo "Count: $count"
    ((count++))
done
```

**until loop:**
```bash
until [ $count -ge 5 ]; do
    echo "Count: $count"
    ((count++))
done
```

#### Conditional Statements
Controls flow logic by executing code only if certain rules are met.

**if statement:**
```bash
if [ condition ]; then
    echo "Condition is true"
elif [ other_condition ]; then
    echo "Other condition is true"
else
    echo "No conditions met"
fi
```

**Comparison Operators:**
```bash
-eq                             # Equal
-ne                             # Not equal
-lt                             # Less than
-le                             # Less than or equal
-gt                             # Greater than
-ge                             # Greater than or equal
-z                              # String is empty
-n                              # String is not empty
-f                              # File exists
-d                              # Directory exists
-x                              # File is executable
```

**Example:**
```bash
if [ -f "/path/to/file.txt" ]; then
    echo "File exists"
else
    echo "File does not exist"
fi
```

**case statement:**
```bash
case $variable in
    value1)
        echo "First case"
        ;;
    value2)
        echo "Second case"
        ;;
    *)
        echo "Default case"
        ;;
esac
```

#### Functions
Reusable blocks of code.
```bash
function greet() {
    echo "Hello, $1"
}

greet "World"                   # Call function with argument
```

---

## Advanced Workflow Concepts

### Flag Hunting Scripting
Scripts can automate complex research tasks. The workflow often involves:

1. **Define target variables** - specify the directory and flag to search for
2. **Iterate through files** - use for loop to process all files
3. **Search with grep** - use grep to find flag existence
4. **Log successful finds** - record matches without unnecessary output

**Example Script:**
```bash
#!/bin/bash

TARGET_DIR="/var/log"
FLAG="ERROR_CODE_42"

for file in "$TARGET_DIR"/*.log; do
    if grep -q "$FLAG" "$file"; then
        echo "Flag found in: $file"
        grep "$FLAG" "$file" >> results.txt
    fi
done

echo "Search complete. Results saved to results.txt"
```

**Key Techniques:**
- `grep -q` : Quiet mode - checks for pattern existence without printing
- `grep -n` : Show line numbers where pattern is found
- `grep -c` : Count occurrences
- `>` : Redirect output to file
- `>>` : Append output to file

---

## Output Redirection & Piping

### Redirection

**Redirect to file (overwrite):**
```bash
command > file.txt              # Redirect stdout
command 2> error.txt            # Redirect stderr
command > output.txt 2>&1       # Redirect both stdout and stderr
```

**Redirect to file (append):**
```bash
command >> file.txt             # Append stdout
command 2>> error.txt           # Append stderr
```

**Discard output:**
```bash
command > /dev/null             # Discard stdout
command 2> /dev/null            # Discard stderr
```

### Piping
Pass output from one command as input to another.
```bash
cat file.txt | grep "error"             # Pipe cat to grep
ls -l | grep "\.txt"                    # List text files only
cat file.txt | sort | uniq               # Sort and remove duplicates
ps aux | grep "processname"              # Find specific process
```

---

## Text Processing Tools

### sort
Sort lines in a file.
```bash
sort file.txt                   # Sort alphabetically
sort -n file.txt                # Sort numerically
sort -r file.txt                # Reverse sort
sort -u file.txt                # Remove duplicates while sorting
```

### uniq
Remove duplicate lines (works on sorted input).
```bash
uniq file.txt                   # Remove adjacent duplicates
sort file.txt | uniq            # Sort then remove duplicates
uniq -c file.txt                # Count occurrences
uniq -d file.txt                # Show only duplicates
```

### cut
Extract columns from text.
```bash
cut -d: -f1 /etc/passwd         # Extract first field (username)
cut -c1-10 file.txt             # Extract first 10 characters
cut -d',' -f2,3 data.csv        # Extract fields 2 and 3 from CSV
```

### sed
Stream editor for text manipulation.
```bash
sed 's/old/new/' file.txt       # Replace first occurrence per line
sed 's/old/new/g' file.txt      # Replace all occurrences
sed '5d' file.txt               # Delete line 5
sed -n '1,5p' file.txt          # Print lines 1-5
```

### awk
Text processing and pattern scanning.
```bash
awk '{print $1}' file.txt       # Print first field
awk -F: '{print $1}' /etc/passwd    # Print using colon delimiter
awk '{sum+=$1} END {print sum}' file.txt  # Sum first column
```

---

## 🛠 Command Repository (Cheat Sheet)

### Linux Shell Commands (Bash/CLI)

| Command | Full Name | Function | Purpose |
|---------|-----------|----------|---------|
| `pwd` | Print Working Directory | `pwd` | Shows current location in file system |
| `cd [dir]` | Change Directory | `cd /path` | Navigate to specified folder |
| `ls` | List | `ls` or `ls -la` | Display directory contents |
| `man [cmd]` | Manual | `man cat` | Provides command documentation |
| `cat [file]` | Concatenate | `cat file.txt` | Output file contents |
| `grep [pattern]` | Global Search | `grep "text" file.txt` | Search for text patterns |
| `chmod +x` | Change Mode | `chmod +x script.sh` | Make file executable |
| `sudo` | Super User Do | `sudo command` | Execute with root privileges |
| `su` | Switch User | `su username` | Change user context |
| `echo` | Echo | `echo "text"` | Print text to console |
| `mkdir [dir]` | Make Directory | `mkdir folder` | Create new directory |
| `rm [file]` | Remove | `rm file.txt` | Delete file |
| `cp [src] [dst]` | Copy | `cp file.txt copy.txt` | Copy file |
| `mv [src] [dst]` | Move | `mv file.txt /path/` | Move/rename file |
| `touch [file]` | Touch | `touch newfile.txt` | Create empty file |
| `find [path]` | Find | `find / -name "*.txt"` | Search for files |
| `tar [options]` | Tape Archive | `tar -czf archive.tar.gz folder/` | Compress/archive |
| `wget [url]` | Web Get | `wget http://example.com/file` | Download files |
| `curl [url]` | Client URL | `curl http://example.com` | Transfer data via URL |
| `ps` | Process Status | `ps aux` | List running processes |
| `kill [pid]` | Kill | `kill 1234` | Terminate process |
| `top` | Top | `top` | Monitor system resources |
| `df` | Disk Free | `df -h` | Show disk space usage |
| `du` | Disk Usage | `du -sh folder/` | Show folder size |
| `whoami` | Who Am I | `whoami` | Display current user |
| `exit` | Exit | `exit` | Close terminal/shell |

---

## Practical Examples

### Example 1: Search and Count Errors in Log Files
```bash
#!/bin/bash
# Script to find and count errors in log files

LOG_DIR="/var/log"
ERROR_PATTERN="ERROR"

echo "Searching for errors in $LOG_DIR..."

total_errors=0
for logfile in "$LOG_DIR"/*.log; do
    if [ -f "$logfile" ]; then
        error_count=$(grep -c "$ERROR_PATTERN" "$logfile" 2>/dev/null)
        if [ "$error_count" -gt 0 ]; then
            echo "$logfile: $error_count errors"
            total_errors=$((total_errors + error_count))
        fi
    fi
done

echo "Total errors found: $total_errors"
```

### Example 2: Backup Directory with Timestamp
```bash
#!/bin/bash
# Backup script with timestamp

SOURCE_DIR="/home/user/important"
BACKUP_DIR="/home/user/backups"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

tar -czf "$BACKUP_DIR/backup_$TIMESTAMP.tar.gz" "$SOURCE_DIR"
echo "Backup created: backup_$TIMESTAMP.tar.gz"
```

### Example 3: Process Monitoring Script
```bash
#!/bin/bash
# Monitor if process is running, restart if not

PROCESS_NAME="apache2"
SERVICE_NAME="apache2"

if ! pgrep -x "$PROCESS_NAME" > /dev/null; then
    echo "$PROCESS_NAME is not running. Restarting..."
    sudo systemctl restart $SERVICE_NAME
    echo "Service restarted at $(date)"
else
    echo "$PROCESS_NAME is running"
fi
```

---

**Last Updated**: 2026-06-11
