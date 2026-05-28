# Linux Notes — Task 1

## Overview
Linux is an open-source, Unix-like operating system kernel. It powers servers, embedded systems, and is widely used in cybersecurity and development environments.

---

## File System Structure

| Directory | Purpose |
|-----------|---------|
| `/`       | Root of the entire filesystem |
| `/home`   | User home directories |
| `/etc`    | System configuration files |
| `/var`    | Variable data (logs, databases) |
| `/tmp`    | Temporary files |
| `/bin`    | Essential user binaries |
| `/sbin`   | System administration binaries |
| `/usr`    | User programs and data |
| `/proc`   | Virtual filesystem for process info |
| `/dev`    | Device files |

---

## Essential Commands

### Navigation
```bash
pwd           # Print working directory
ls -la        # List all files with details
cd /path      # Change directory
cd ..         # Go up one level
```

### File Operations
```bash
touch file.txt          # Create empty file
mkdir dirname           # Create directory
cp source dest          # Copy file
mv source dest          # Move/rename file
rm file.txt             # Remove file
rm -rf dirname          # Remove directory recursively
cat file.txt            # View file contents
less file.txt           # Paginated file view
head -n 10 file.txt     # First 10 lines
tail -n 10 file.txt     # Last 10 lines
```

### Permissions
```bash
chmod 755 file          # Set permissions (rwxr-xr-x)
chmod +x script.sh      # Make executable
chown user:group file   # Change ownership
ls -l                   # View permissions
```

Permission notation:
- `r` = read (4)
- `w` = write (2)
- `x` = execute (1)
- Example: `chmod 644` → owner rw, group r, others r

### Process Management
```bash
ps aux                  # List all running processes
top                     # Real-time process monitor
htop                    # Interactive process viewer
kill PID                # Kill process by ID
kill -9 PID             # Force kill
killall processname     # Kill by name
```

### Searching
```bash
find / -name "file.txt"         # Find file by name
grep "pattern" file.txt         # Search in file
grep -r "pattern" /dir          # Recursive search
locate filename                 # Fast file search (uses db)
which command                   # Find command location
```

### Networking Commands
```bash
ifconfig                # Network interface info (older)
ip a                    # Network interface info (modern)
ping host               # Test connectivity
netstat -tulnp          # Active connections/ports
ss -tulnp               # Socket statistics
curl url                # HTTP request
wget url                # Download file
```

### User Management
```bash
whoami                  # Current user
id                      # User/group IDs
sudo command            # Run as superuser
su - username           # Switch user
adduser username        # Add new user
passwd username         # Change password
```

### Package Management (Debian/Ubuntu)
```bash
apt update              # Update package list
apt upgrade             # Upgrade installed packages
apt install package     # Install package
apt remove package      # Remove package
dpkg -l                 # List installed packages
```

---

## Shell Scripting Basics

```bash
#!/bin/bash
# This is a comment

# Variables
NAME="World"
echo "Hello, $NAME"

# Conditionals
if [ "$NAME" == "World" ]; then
    echo "Name is World"
fi

# Loops
for i in 1 2 3; do
    echo "Number: $i"
done

# Functions
greet() {
    echo "Hello, $1"
}
greet "Sayan"
```

---

## Key Concepts

- **stdin / stdout / stderr**: Standard input (0), output (1), error (2)
- **Piping**: `command1 | command2` — pass output of one to another
- **Redirection**: `>` overwrite, `>>` append, `<` input from file
- **Environment variables**: `export VAR=value`, `echo $PATH`
- **Cron jobs**: Scheduled tasks via `crontab -e`
- **Symbolic links**: `ln -s target linkname`

---

## Notes
- Always use `sudo` carefully — it runs commands as root
- Use `man command` to read the manual for any command
- `.bashrc` / `.bash_profile` — shell configuration files loaded on login
