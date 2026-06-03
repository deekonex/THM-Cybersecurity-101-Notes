# Windows Command Line

## General Overview & Basics

### What is it?
The **Command Line Interface (CLI)** is a text-based interaction method for Windows. The default shell is `cmd.exe`.

### Why use it?
- Faster than GUI
- Uses fewer system resources
- Essential for automation and remote management

---

## System Checks

| Command | Purpose |
|---------|---------|
| `ver` | Check Operating System version |
| `systeminfo` | Detailed system overview (OS, CPU, Memory, etc.) |
| `help [command]` | View options for any command |
| `cls` | Clear the screen |
| `driverquery` | List installed device drivers |

### Usage Tip
Append `/` or `/?` to most commands to display the help page.

---

## Network Troubleshooting & Data Lookup

### Basic Network Info

| Command | Purpose |
|---------|---------|
| `ipconfig` | Simple check (IP, Subnet) |
| `ipconfig /all` | **Critical.** Comprehensive network data (MAC address, DNS servers, DHCP status) |

**Need MAC Address?** Use `ipconfig /all`

### Connectivity & Mapping

| Command | Purpose |
|---------|---------|
| `ping [target]` | Test basic connection using ICMP packets |
| `tracert [target]` | Map entire route (hops) to destination. Essential for localizing network failures |
| `nslookup [domain]` | DNS lookup - find IP address from domain name |
| `netstat -abon` | Display active connections, listening ports, and associated PID/Program |

### Data Flow: Using the Pipe Operator

The **pipe** (`|`) operator redirects output:

```cmd
some_command | more
```
Feeds output page by page for easier reading.

---

## File & Directory Management

### Navigation

| Command | Purpose |
|---------|---------|
| `cd` | Change directory or display current path |
| `dir` | List contents of current folder |
| `dir /s` | List contents recursively (includes subfolders) |
| `dir /a` | Show hidden files |
| `mkdir [name]` | Create a new folder |
| `rmdir [name]` | Remove an empty folder |
| `tree` | Visual representation of directory structure |

### File Operations

| Command | Purpose |
|---------|---------|
| `type [file]` | Display text content immediately (best for reading logs/flags) |
| `more [file]` | Display large files page by page |
| `copy [file] [dest]` | Copy a file |
| `move [file] [dest]` | Move a file |
| `del [file]` | Delete a file (Handle with care) |
| `copy *.md [dest]` | Use wildcards to copy all matching files |

---

## Process Control & Management

### Core Concept
Processes require a unique **PID (Process ID)** for management.

### Listing Processes

| Command | Purpose |
|---------|---------|
| `tasklist` | Show all running processes |
| `tasklist /FI "imagename eq [process]"` | Filter process list (e.g., find all notepad.exe) |

### Process Actions

| Command | Purpose |
|---------|---------|
| `taskkill /PID [PID]` | Terminate a process using its PID |

---

## Quick Workflow Summary

### Network Failure Diagnosis
Use `tracert` before assuming the issue is local. It pinpoints the exact "hop" where connection fails.

### Resource Control
To kill a program:
1. `tasklist /FI` - Find PID
2. `taskkill /PID` - Terminate

### Data Access
- Use `type` for quick viewing
- Use `more` for large content files

### Shutdown Commands

| Command | Purpose |
|---------|---------|
| `shutdown /r` | Restart system |
| `shutdown /a` | Abort shutdown |

---

**Last Updated:** 2026-06-03  
**Source:** TryHackMe Cybersecurity 101 Learning Path
