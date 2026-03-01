---
layout: default
title: "Module 03 — Kali Linux Fundamentals"
nav_order: 4
---

# Module 03 — Kali Linux Fundamentals
{: .no_toc }

Kali Linux is a Debian-based distribution purpose-built for penetration testing, security auditing, and digital forensics. Before you can wield its 600+ pre-installed security tools, you need to be completely at home on the Linux command line. This module takes you from shell fundamentals to writing your own hacking scripts, covering every essential concept along the way.

> This module is long and dense by design. Bookmark it and return to it often — command-line fluency is the single biggest differentiator between a script kiddie and a professional penetration tester.
{: .note }

<details open markdown="block">
  <summary>Table of Contents</summary>
  {: .text-delta }
- TOC
{:toc}
</details>

---

## 1 — The Linux Command Line

The command line is your primary interface in Kali. Every tool, every exploit, every scan starts here. Mastering the shell is not optional — it is foundational.

### 1.1 Shell Basics

A **shell** is a program that interprets your commands and passes them to the operating system kernel. Kali ships with two major shells:

| Shell | Path | Notes |
|:------|:-----|:------|
| **Bash** (Bourne Again Shell) | `/bin/bash` | Default on most Linux distros, POSIX-compliant with extensions |
| **Zsh** (Z Shell) | `/bin/zsh` | Default on Kali 2020.4+, advanced tab completion, plugin ecosystem |

Check your current shell:

```bash
echo $SHELL
# /bin/zsh  (typical on modern Kali)

echo $0
# -zsh  (currently running shell)
```

Switch shells temporarily:

```bash
bash          # Drop into bash
exit          # Return to previous shell
```

Change your default shell permanently:

```bash
chsh -s /bin/bash    # Switch to bash
chsh -s /bin/zsh     # Switch to zsh
```

> Both Bash and Zsh share 95% of their syntax. This guide uses Bash syntax, which works in both shells. Where Zsh differs meaningfully, it will be noted.
{: .tip }

### 1.2 Navigation

```bash
# Print current working directory
pwd
# /home/kali

# Change directory
cd /etc                  # Absolute path
cd ..                    # Parent directory
cd ~                     # Home directory (same as cd alone)
cd -                     # Previous directory (toggle)
cd ../..                 # Two levels up

# List directory contents
ls                       # Basic listing
ls -l                    # Long format (permissions, size, date)
ls -la                   # Include hidden files (dotfiles)
ls -lah                  # Human-readable sizes (K, M, G)
ls -lt                   # Sort by modification time (newest first)
ls -lS                   # Sort by size (largest first)
ls -R                    # Recursive listing

# Directory stack (push/pop directories like a stack)
pushd /var/log           # Push current dir, change to /var/log
pushd /etc               # Push /var/log, change to /etc
dirs                     # Show the stack
popd                     # Pop back to /var/log
popd                     # Pop back to original directory
```

> `pushd` and `popd` are incredibly useful during a pentest when you are jumping between exploit directories, loot folders, and tool locations. Build the habit early.
{: .tip }

### 1.3 File Operations

```bash
# Create files and directories
touch newfile.txt                 # Create empty file (or update timestamp)
mkdir reports                     # Create directory
mkdir -p loot/target1/creds      # Create nested directories

# Copy files and directories
cp source.txt dest.txt            # Copy file
cp -r source_dir/ dest_dir/       # Copy directory recursively
cp -v file1 file2 /destination/   # Verbose copy (shows what's happening)
cp -p file.txt backup.txt         # Preserve permissions and timestamps

# Move and rename
mv old_name.txt new_name.txt      # Rename file
mv report.txt /root/Desktop/      # Move file
mv -i file.txt /dest/             # Interactive (prompt before overwrite)

# Remove files and directories
rm unwanted.txt                   # Delete file
rm -f stubborn.txt                # Force delete (no prompts)
rm -r old_directory/              # Delete directory recursively
rm -rf /tmp/junk/                 # Force recursive delete

# Reading file contents
cat /etc/hostname                 # Print entire file
less /var/log/syslog              # Paginated viewer (q to quit)
head -n 20 /etc/passwd            # First 20 lines
tail -n 10 /var/log/auth.log     # Last 10 lines
tail -f /var/log/syslog          # Follow file in real time (live updates)
```

> **Never** run `rm -rf /` or `rm -rf *` in the root directory. Triple-check destructive commands, especially when running as root. There is no recycle bin on Linux.
{: .warning }

### 1.4 File Permissions

Linux permissions control who can read, write, and execute files. Understanding them is critical for both securing systems and exploiting misconfigurations.

```
-rwxr-xr-- 1 kali kali 4096 Jan 15 10:30 exploit.py
│├─┤├─┤├─┤
│ │   │  │
│ │   │  └── Others: r-- (read only)
│ │   └───── Group: r-x (read + execute)
│ └───────── Owner: rwx (read + write + execute)
└──────────── File type: - (regular file), d (directory), l (symlink)
```

**Permission values:**

| Symbol | Octal | Meaning |
|:-------|:------|:--------|
| `r` | 4 | Read |
| `w` | 2 | Write |
| `x` | 1 | Execute |
| `-` | 0 | No permission |

**Octal notation** sums the values for owner, group, and others:

| Octal | Symbolic | Meaning |
|:------|:---------|:--------|
| `755` | `rwxr-xr-x` | Owner full, group/others read+execute |
| `644` | `rw-r--r--` | Owner read+write, group/others read only |
| `777` | `rwxrwxrwx` | Everyone can do everything (dangerous!) |
| `600` | `rw-------` | Owner read+write only (good for SSH keys) |
| `400` | `r--------` | Owner read only |

```bash
# Change permissions
chmod 755 exploit.sh              # Set exact permissions (octal)
chmod +x script.sh                # Add execute permission for all
chmod u+x script.sh               # Add execute for owner only
chmod go-w sensitive.txt          # Remove write from group and others
chmod -R 750 /opt/tools/          # Recursive permission change

# Change ownership
chown kali exploit.py             # Change owner
chown kali:kali exploit.py        # Change owner and group
chown -R root:root /opt/tools/    # Recursive ownership change

# Change group
chgrp pentesters report.txt       # Change group only
```

### 1.5 Text Processing

These tools are the Swiss Army knives of the command line. Master them, and you can extract any data from any output.

#### grep — Pattern Matching

```bash
# Basic search
grep "password" config.txt                 # Find lines containing "password"
grep -i "password" config.txt              # Case-insensitive search
grep -r "admin" /etc/                      # Recursive search in directory
grep -n "root" /etc/passwd                 # Show line numbers
grep -c "Failed" /var/log/auth.log         # Count matches
grep -v "comment" config.txt              # Invert match (lines NOT containing)
grep -l "password" *.txt                   # List filenames with matches only
grep -w "root" /etc/passwd                 # Whole word match only

# Regular expressions
grep -E "^root" /etc/passwd                # Lines starting with "root"
grep -E "bash$" /etc/passwd                # Lines ending with "bash"
grep -E "[0-9]{1,3}\.[0-9]{1,3}" scan.txt # IP address pattern
grep -oE "[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+" output.txt  # Extract IPs only

# Practical pentest examples
grep -i "password\|passwd\|pwd" config.txt # Multiple patterns (OR)
grep -A 2 -B 2 "error" /var/log/syslog    # Show 2 lines before and after match
```

#### sed — Stream Editor

```bash
# Substitution (find and replace)
sed 's/old/new/' file.txt                  # Replace first occurrence per line
sed 's/old/new/g' file.txt                 # Replace ALL occurrences (global)
sed -i 's/old/new/g' file.txt             # Edit file in place
sed 's/http:/https:/g' urls.txt           # Fix URLs

# Delete lines
sed '/^#/d' config.txt                     # Delete comment lines
sed '/^$/d' file.txt                       # Delete empty lines
sed '1,5d' file.txt                        # Delete lines 1-5

# Print specific lines
sed -n '10,20p' file.txt                   # Print lines 10-20
sed -n '/pattern/p' file.txt              # Print matching lines (like grep)

# Practical examples
sed 's/\r$//' windows_file.txt             # Remove Windows carriage returns
```

#### awk — Pattern Processing

```bash
# Print specific columns (fields)
awk '{print $1}' file.txt                  # Print first column
awk '{print $1, $3}' file.txt             # Print first and third columns
awk -F: '{print $1}' /etc/passwd          # Use : as delimiter, print usernames
awk -F: '{print $1 ":" $3}' /etc/passwd   # Print username:UID

# Conditional processing
awk -F: '$3 >= 1000 {print $1}' /etc/passwd    # Users with UID >= 1000
awk -F: '$7 ~ /bash/ {print $1}' /etc/passwd   # Users with bash shell
awk '{if ($1 > 100) print $0}' data.txt         # Lines where field 1 > 100

# Calculations
awk '{sum += $1} END {print sum}' numbers.txt   # Sum a column
awk '{print NR, $0}' file.txt                   # Add line numbers
awk 'END {print NR}' file.txt                   # Count lines
```

#### Other Text Tools

```bash
# cut — extract columns
cut -d: -f1 /etc/passwd                   # Cut first field (delimiter :)
cut -d: -f1,3 /etc/passwd                 # Cut fields 1 and 3
cut -c1-10 file.txt                       # Cut first 10 characters per line

# sort — sort lines
sort file.txt                              # Alphabetical sort
sort -n numbers.txt                        # Numeric sort
sort -r file.txt                           # Reverse sort
sort -t: -k3 -n /etc/passwd              # Sort by 3rd field numerically

# uniq — deduplicate (requires sorted input)
sort file.txt | uniq                       # Remove duplicates
sort file.txt | uniq -c                    # Count occurrences
sort file.txt | uniq -d                    # Show only duplicates

# wc — word/line/character count
wc -l file.txt                             # Count lines
wc -w file.txt                             # Count words
wc -c file.txt                             # Count bytes

# tr — translate/delete characters
echo "HELLO" | tr 'A-Z' 'a-z'            # Convert to lowercase
echo "hello" | tr 'a-z' 'A-Z'            # Convert to uppercase
cat file.txt | tr -d '\r'                 # Delete carriage returns
cat file.txt | tr -s ' '                  # Squeeze repeated spaces
```

### 1.6 I/O Redirection

Every command has three standard streams: **stdin** (0), **stdout** (1), and **stderr** (2). Redirection lets you control where data flows.

```bash
# Output redirection
echo "Hello" > file.txt           # Write to file (overwrite)
echo "World" >> file.txt          # Append to file
ls /nonexistent 2> errors.txt     # Redirect stderr to file
ls /etc /nonexistent > out.txt 2>&1   # Redirect both stdout and stderr
ls /etc /nonexistent &> all.txt       # Shorthand for the above (Bash)

# Input redirection
sort < unsorted.txt               # Read input from file
wc -l < /etc/passwd               # Count lines in file

# Pipes — connect stdout of one command to stdin of another
cat /etc/passwd | grep "bash" | cut -d: -f1 | sort
#  Read file  → Filter bash users → Extract usernames → Sort them

# tee — write to file AND stdout simultaneously
nmap -sV 10.10.10.1 | tee scan_results.txt
# You see the output AND it's saved to a file

nmap -sV 10.10.10.1 | tee -a scan_results.txt
# Append mode (don't overwrite)

# /dev/null — the black hole (discard output)
command 2>/dev/null               # Suppress error messages
command > /dev/null 2>&1          # Suppress ALL output
```

> During a pentest, **always** use `tee` to save your tool output. You will need it for reporting, and re-running scans wastes time and generates noise.
{: .tip }

### 1.7 Process Management

```bash
# View processes
ps aux                            # All running processes (BSD syntax)
ps -ef                            # All running processes (System V syntax)
ps aux | grep nmap                # Find specific process
pgrep -a nmap                     # Find process by name
top                               # Real-time process monitor (q to quit)
htop                              # Better top (interactive, colored)

# Process control
Ctrl+C                            # Kill foreground process (SIGINT)
Ctrl+Z                            # Suspend foreground process (SIGTSTP)
bg                                # Resume suspended process in background
fg                                # Bring background process to foreground
jobs                              # List background jobs

# Run in background
nmap -sV 10.10.10.0/24 &         # Start in background (& at end)
nohup long_scan.sh &              # Persist after terminal closes
disown %1                         # Detach job 1 from terminal

# Kill processes
kill 1234                         # Send SIGTERM to PID 1234
kill -9 1234                      # Force kill (SIGKILL) — last resort
killall nmap                      # Kill all processes named "nmap"
pkill -f "python3 exploit.py"     # Kill by command pattern
```

> `kill -9` should be a last resort. It does not give the process a chance to clean up (close files, release locks). Use `kill` (SIGTERM) first and only escalate to `-9` if the process does not respond.
{: .warning }

### 1.8 Environment Variables and PATH

```bash
# View environment
env                               # List all environment variables
echo $HOME                        # Print specific variable
echo $PATH                        # Print executable search path
echo $USER                        # Current username
echo $SHELL                       # Current shell

# Set variables
export MY_VAR="value"             # Set for current session + child processes
MY_VAR="value"                    # Set for current session only (no export)
unset MY_VAR                      # Remove variable

# PATH — where the shell looks for executables
echo $PATH
# /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# Add a directory to PATH
export PATH=$PATH:/opt/my_tools   # Append
export PATH=/opt/my_tools:$PATH   # Prepend (searched first)

# Make changes permanent — add to shell config file
echo 'export PATH=$PATH:/opt/my_tools' >> ~/.bashrc   # For Bash
echo 'export PATH=$PATH:/opt/my_tools' >> ~/.zshrc    # For Zsh
source ~/.bashrc                  # Reload config without restarting shell
```

### 1.9 Shell Scripting Basics

```bash
#!/bin/bash
# The shebang line (^) tells the system which interpreter to use

# Variables (no spaces around =)
NAME="Kali"
PORT=443
echo "Hello from $NAME on port $PORT"

# Command substitution
CURRENT_IP=$(hostname -I | awk '{print $1}')
echo "My IP is: $CURRENT_IP"

# User input
read -p "Enter target IP: " TARGET
echo "Scanning $TARGET..."

# Conditionals
if [ -f "/etc/shadow" ]; then
    echo "Shadow file exists"
elif [ -f "/etc/passwd" ]; then
    echo "Only passwd exists"
else
    echo "Neither exists"
fi

# Numeric comparisons
if [ $PORT -eq 443 ]; then
    echo "HTTPS"
fi
# -eq (equal), -ne (not equal), -gt (greater), -lt (less),
# -ge (greater or equal), -le (less or equal)

# String comparisons
if [ "$NAME" = "Kali" ]; then
    echo "Running Kali"
fi
# = (equal), != (not equal), -z (empty), -n (not empty)

# File test operators
# -f (file exists), -d (directory exists), -r (readable),
# -w (writable), -x (executable), -s (non-empty)

# For loop
for IP in 192.168.1.1 192.168.1.2 192.168.1.3; do
    ping -c 1 -W 1 $IP && echo "$IP is up"
done

# For loop with range
for i in $(seq 1 254); do
    ping -c 1 -W 1 192.168.1.$i > /dev/null 2>&1 && echo "192.168.1.$i is alive"
done

# While loop
COUNT=1
while [ $COUNT -le 10 ]; do
    echo "Attempt $COUNT"
    COUNT=$((COUNT + 1))
done

# While read loop (process file line by line)
while IFS= read -r line; do
    echo "Processing: $line"
done < targets.txt

# Functions
scan_target() {
    local target=$1
    local ports=$2
    echo "[*] Scanning $target on ports $ports"
    nmap -p $ports $target
}
scan_target "10.10.10.1" "80,443,8080"

# Exit codes
# 0 = success, non-zero = failure
ping -c 1 10.10.10.1 > /dev/null 2>&1
if [ $? -eq 0 ]; then
    echo "Host is up"
else
    echo "Host is down"
fi
```

### 1.10 Useful Keyboard Shortcuts

| Shortcut | Action |
|:---------|:-------|
| `Ctrl+C` | Kill current process (send SIGINT) |
| `Ctrl+Z` | Suspend current process |
| `Ctrl+D` | Exit shell / send EOF |
| `Ctrl+L` | Clear screen (same as `clear`) |
| `Ctrl+R` | Reverse search command history |
| `Ctrl+A` | Move cursor to beginning of line |
| `Ctrl+E` | Move cursor to end of line |
| `Ctrl+U` | Delete from cursor to beginning of line |
| `Ctrl+K` | Delete from cursor to end of line |
| `Ctrl+W` | Delete word before cursor |
| `Alt+.` | Insert last argument of previous command |
| `Tab` | Auto-complete commands, filenames, and paths |
| `Tab Tab` | Show all possible completions |
| `!!` | Repeat last command (useful: `sudo !!`) |
| `!$` | Last argument of previous command |
| `!nmap` | Repeat last command starting with "nmap" |

> `Ctrl+R` is one of the most powerful shortcuts. Start typing any part of a previous command and it searches backward through your history. Press `Ctrl+R` again to cycle through matches. Press `Enter` to execute or `Ctrl+G` to cancel.
{: .tip }

---

## 2 — File System Hierarchy

Understanding where things live in Linux is essential. When you land on a compromised system, you need to know exactly where to look for credentials, logs, configurations, and sensitive data.

### 2.1 Directory Structure Overview

```
/                     Root of the entire filesystem
├── bin/              Essential user binaries (ls, cp, cat, grep)
├── sbin/             System binaries (iptables, fdisk, mount)
├── usr/              User programs and data
│   ├── bin/          Non-essential user binaries
│   ├── sbin/         Non-essential system binaries
│   ├── lib/          Libraries
│   ├── local/        Locally installed software
│   └── share/        Architecture-independent data (man pages, wordlists)
├── etc/              System configuration files (THE goldmine)
├── home/             User home directories (/home/kali, /home/user)
├── root/             Root user's home directory
├── var/              Variable data
│   ├── log/          System and application logs
│   ├── www/          Web server document root (Apache/Nginx)
│   ├── mail/         User mailboxes
│   └── tmp/          Temporary files (preserved across reboots)
├── tmp/              Temporary files (cleared on reboot) — world-writable
├── opt/              Optional/third-party software
├── dev/              Device files
│   ├── null          Black hole (discards all data)
│   ├── zero          Infinite source of null bytes
│   ├── random        Random number generator
│   └── urandom       Non-blocking random number generator
├── proc/             Virtual filesystem — kernel and process info (not on disk)
│   ├── [PID]/        Per-process information
│   ├── cpuinfo       CPU information
│   ├── meminfo       Memory information
│   ├── version       Kernel version
│   └── net/          Network information (tcp, udp, arp, route)
├── sys/              Virtual filesystem — hardware and driver info
├── mnt/              Temporary mount points
├── media/            Removable media mount points
├── srv/              Service data (FTP, HTTP)
└── boot/             Boot loader files, kernel images
```

### 2.2 Critical Security Files

These are the files you will look at on every single engagement, both on your own Kali box and on compromised targets.

#### /etc/passwd — User Account Database

```bash
cat /etc/passwd
# root:x:0:0:root:/root:/bin/bash
# daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
# kali:x:1000:1000:Kali,,,:/home/kali:/bin/zsh
```

**Format**: `username:password:UID:GID:comment:home_dir:shell`

| Field | Meaning | Security Note |
|:------|:--------|:-------------|
| `username` | Login name | Check for unusual accounts |
| `password` | `x` means stored in /etc/shadow | If actual hash here, system is misconfigured |
| `UID` | User ID, 0 = root | Any non-root user with UID 0 is suspicious |
| `GID` | Primary group ID | |
| `comment` | Full name / description | |
| `home_dir` | User's home directory | Check for unusual paths |
| `shell` | Login shell | `/sbin/nologin` or `/bin/false` = cannot log in |

> During a pentest, always check for users with UID 0 besides root: `awk -F: '$3 == 0 {print $1}' /etc/passwd`. Any match besides `root` is a backdoor account.
{: .warning }

#### /etc/shadow — Password Hashes

```bash
sudo cat /etc/shadow
# root:$y$j9T$abc123...:19500:0:99999:7:::
# kali:$y$j9T$def456...:19500:0:99999:7:::
```

**Format**: `username:hash:last_changed:min_age:max_age:warn:inactive:expire`

**Hash formats** (the `$id$` prefix identifies the algorithm):

| Prefix | Algorithm | Strength |
|:-------|:----------|:---------|
| `$1$` | MD5 | Weak (crackable in seconds) |
| `$5$` | SHA-256 | Moderate |
| `$6$` | SHA-512 | Strong (default on most systems) |
| `$y$` | yescrypt | Very strong (default on modern Debian/Kali) |
| `$2b$` | bcrypt | Strong (common on BSD) |

```bash
# Extract hashes for cracking with John the Ripper
sudo unshadow /etc/passwd /etc/shadow > hashes.txt
john hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

#### /etc/sudoers — Sudo Privileges

```bash
sudo cat /etc/sudoers
# root    ALL=(ALL:ALL) ALL
# %sudo   ALL=(ALL:ALL) ALL
# kali    ALL=(ALL) NOPASSWD: ALL
```

> Never edit `/etc/sudoers` directly. Always use `visudo`, which validates syntax before saving. A malformed sudoers file can lock you out of `sudo` entirely.
{: .warning }

**What to look for during privilege escalation:**

```bash
# Check what YOU can run with sudo
sudo -l
# (root) NOPASSWD: /usr/bin/vim     ← This is exploitable!
# (root) NOPASSWD: /usr/bin/find    ← This too!

# Misconfigured sudoers entries are one of the most common privesc vectors.
# Check GTFOBins (https://gtfobins.github.io) for exploitation techniques.
```

#### /etc/hosts and /etc/resolv.conf

```bash
# /etc/hosts — local DNS overrides
cat /etc/hosts
# 127.0.0.1   localhost
# 10.10.10.1  target.htb    # Common in CTF environments

# /etc/resolv.conf — DNS server configuration
cat /etc/resolv.conf
# nameserver 8.8.8.8
# nameserver 8.8.4.4
```

#### /var/log/ — System Logs

```bash
# Key log files for forensics and detection
/var/log/syslog          # General system log (Debian/Ubuntu)
/var/log/messages        # General system log (RHEL/CentOS)
/var/log/auth.log        # Authentication events (logins, sudo, SSH)
/var/log/kern.log        # Kernel messages
/var/log/dpkg.log        # Package installation history
/var/log/apache2/        # Apache web server logs
/var/log/nginx/          # Nginx web server logs
/var/log/mysql/          # MySQL database logs
/var/log/fail2ban.log    # Brute force protection logs
/var/log/lastlog         # Last login for all users (use `lastlog` command)
/var/log/wtmp            # Login history (use `last` command)
/var/log/btmp            # Failed login attempts (use `lastb` command)

# Useful log analysis commands
grep "Failed password" /var/log/auth.log              # Failed SSH logins
grep "Accepted" /var/log/auth.log                     # Successful SSH logins
grep "sudo:" /var/log/auth.log | tail -20             # Recent sudo usage
last -20                                               # Last 20 logins
lastb -20                                              # Last 20 FAILED logins
journalctl -u ssh --since "1 hour ago"                # Systemd journal query
```

#### /proc/net/ — Network Intelligence

```bash
cat /proc/net/tcp         # Active TCP connections (hex format)
cat /proc/net/udp         # Active UDP connections
cat /proc/net/arp         # ARP table
cat /proc/net/route       # Routing table

# More readable alternatives
ss -tulnp                 # All listening ports with process info
netstat -tulnp            # Same (older tool)
```

### 2.3 Finding Files

```bash
# find — powerful but slow (walks the filesystem in real time)
find / -name "password*" 2>/dev/null           # Find files named password*
find / -name "*.conf" -type f 2>/dev/null      # Find configuration files
find / -perm -4000 -type f 2>/dev/null         # Find SUID binaries (privesc!)
find / -perm -2000 -type f 2>/dev/null         # Find SGID binaries
find / -writable -type f 2>/dev/null           # Find world-writable files
find / -user root -perm -4000 2>/dev/null      # Root-owned SUID files
find /home -name "*.txt" -readable 2>/dev/null # Readable .txt in /home
find / -name "id_rsa" 2>/dev/null              # Find SSH private keys
find / -name ".bash_history" 2>/dev/null       # Find bash histories
find / -newer /etc/passwd -type f 2>/dev/null  # Files modified after passwd
find / -mtime -1 -type f 2>/dev/null           # Files modified in last 24h
find / -size +100M -type f 2>/dev/null         # Files larger than 100MB

# locate — fast (uses pre-built database)
locate password.txt                # Search database for files
sudo updatedb                      # Update the locate database

# which — find executable in PATH
which nmap                         # /usr/bin/nmap
which python3                      # /usr/bin/python3

# whereis — find binary, source, and man pages
whereis nmap                       # nmap: /usr/bin/nmap /usr/share/man/man1/nmap.1.gz
```

> `find / -perm -4000` is one of the first commands you should run after gaining access to a system. SUID binaries run with the privileges of the file owner, and misconfigured ones are a direct path to root.
{: .tip }

### 2.4 File Types and the `file` Command

Linux does not rely on file extensions to determine type. The `file` command reads the file header (magic bytes) to identify it.

```bash
file exploit.py               # Python script, ASCII text executable
file /usr/bin/nmap            # ELF 64-bit LSB pie executable, x86-64
file mystery_file             # data / PDF document / JPEG image / etc.
file -i document.pdf          # MIME type: application/pdf
file /dev/null                # character special (1/3)

# This is invaluable when analyzing unknown files found during an engagement.
# Files can be disguised — a .jpg might actually be a PHP shell.
```

---

## 3 — User and Permission Management

### 3.1 Root vs Regular Users

| Aspect | Root | Regular User |
|:-------|:-----|:-------------|
| UID | 0 | 1000+ (typically) |
| Prompt symbol | `#` | `$` |
| Home directory | `/root` | `/home/username` |
| Permissions | Unrestricted | Limited by file permissions |
| Danger level | Can destroy the entire system | Limited blast radius |

```bash
# Check current user
whoami                    # kali
id                        # uid=1000(kali) gid=1000(kali) groups=...

# Become root
sudo su                   # Switch to root (requires password)
sudo -i                   # Login shell as root
sudo su -                 # Login shell as root (resets environment)

# Run single command as root
sudo nmap -sV 10.10.10.1
```

> On Kali Linux, the default user `kali` has full sudo privileges with `NOPASSWD`. This is by design for a pentesting distro but would be a critical misconfiguration on a production system.
{: .note }

### 3.2 Managing Users and Groups

```bash
# Add a user
sudo useradd -m -s /bin/bash newuser     # -m creates home dir, -s sets shell
sudo useradd -m -G sudo,pentesters bob   # Add with supplementary groups

# Set password
sudo passwd newuser

# Modify user
sudo usermod -aG sudo bob               # Add bob to sudo group
sudo usermod -s /bin/zsh bob             # Change bob's shell
sudo usermod -L bob                      # Lock account (disable login)
sudo usermod -U bob                      # Unlock account

# Delete user
sudo userdel bob                         # Delete user (keep home dir)
sudo userdel -r bob                      # Delete user AND home directory

# Group management
sudo groupadd pentesters                 # Create group
sudo groupdel pentesters                 # Delete group
groups kali                              # Show kali's groups
id kali                                  # Detailed user info
```

### 3.3 SUID, SGID, and Sticky Bit

These special permissions are critically important for privilege escalation.

| Permission | Octal | On Files | On Directories |
|:-----------|:------|:---------|:--------------|
| **SUID** (Set User ID) | `4000` | Executes as the file **owner** | No effect |
| **SGID** (Set Group ID) | `2000` | Executes as the file **group** | New files inherit directory's group |
| **Sticky Bit** | `1000` | No effect | Only file owner can delete files |

```bash
# SUID example — runs as root regardless of who executes it
ls -la /usr/bin/passwd
# -rwsr-xr-x 1 root root ... /usr/bin/passwd
#    ^
#    The 's' in owner execute position = SUID is set

# Set/remove SUID
chmod u+s /path/to/binary                # Set SUID
chmod 4755 /path/to/binary               # Set SUID (octal)
chmod u-s /path/to/binary                # Remove SUID

# Find all SUID binaries (CRITICAL for privilege escalation)
find / -perm -4000 -type f 2>/dev/null

# Common SUID binaries that can be exploited (check GTFOBins):
# /usr/bin/find, /usr/bin/vim, /usr/bin/python3, /usr/bin/nmap (old versions)
# /usr/bin/bash, /usr/bin/less, /usr/bin/nano, /usr/bin/cp

# Sticky bit example — /tmp
ls -ld /tmp
# drwxrwxrwt 12 root root ... /tmp
#          ^
#          The 't' in others execute position = Sticky bit

# Users can write files to /tmp but cannot delete other users' files.
```

> If you find a SUID binary owned by root that is not a standard system utility (like `passwd`, `ping`, or `sudo`), investigate it immediately. Custom SUID binaries are one of the most common privilege escalation vectors in CTFs and real engagements.
{: .warning }

### 3.4 Understanding /etc/passwd and /etc/shadow Format

```bash
# /etc/passwd format breakdown:
# username:password:UID:GID:GECOS:home:shell
#
# password field:
#   x     = password stored in /etc/shadow (normal)
#   *     = account is disabled
#   (empty) = no password required (DANGEROUS)
#   hash  = password hash stored directly (legacy, insecure)

# /etc/shadow format breakdown:
# username:hash:lastchanged:minimum:maximum:warn:inactive:expire:reserved
#
# hash field:
#   $id$salt$hash = password hash
#   !             = account locked
#   !!            = password never set
#   *             = account disabled
#   (empty)       = no password required

# Quick checks during a pentest:
awk -F: '$2 == "" {print $1}' /etc/passwd     # Users with empty passwords
awk -F: '$3 == 0 {print $1}' /etc/passwd      # Users with root UID
awk -F: '$7 != "/sbin/nologin" && $7 != "/bin/false" {print $1,$7}' /etc/passwd
# ^ Users who CAN log in (have a real shell)
```

---

## 4 — Networking Fundamentals in Linux

Networking is the backbone of penetration testing. You need to understand how to configure, query, and manipulate network settings on Linux.

### 4.1 Network Interfaces

```bash
# Modern tool (ip command — part of iproute2)
ip addr                           # Show all interfaces and IPs
ip addr show eth0                 # Show specific interface
ip link                           # Show interface status (up/down)
ip link set eth0 up               # Bring interface up
ip link set eth0 down             # Bring interface down

# Assign IP address
sudo ip addr add 192.168.1.100/24 dev eth0
sudo ip addr del 192.168.1.100/24 dev eth0

# Legacy tool (ifconfig — deprecated but still widely seen)
ifconfig                          # Show all interfaces
ifconfig eth0                     # Show specific interface
sudo ifconfig eth0 192.168.1.100 netmask 255.255.255.0 up

# MAC address manipulation (useful for bypassing MAC filters)
sudo ip link set eth0 down
sudo ip link set eth0 address aa:bb:cc:dd:ee:ff
sudo ip link set eth0 up

# Or use macchanger (pre-installed on Kali)
sudo macchanger -r eth0           # Random MAC
sudo macchanger -p eth0           # Reset to permanent MAC
```

### 4.2 Routing

```bash
# View routing table
ip route                          # Modern
ip route show                     # Same
route -n                          # Legacy (numeric, no DNS resolution)

# Add/delete routes
sudo ip route add 10.10.10.0/24 via 192.168.1.1 dev eth0
sudo ip route del 10.10.10.0/24

# Default gateway
sudo ip route add default via 192.168.1.1
ip route | grep default           # Check current default gateway
```

### 4.3 DNS

```bash
# DNS configuration
cat /etc/resolv.conf
# nameserver 8.8.8.8
# search example.com

# DNS lookup tools
nslookup example.com              # Basic DNS lookup
nslookup -type=MX example.com     # Mail server records
nslookup -type=NS example.com     # Name server records

dig example.com                   # Detailed DNS lookup
dig example.com ANY               # All record types
dig @8.8.8.8 example.com         # Query specific DNS server
dig example.com +short            # Just the answer
dig -x 93.184.216.34             # Reverse DNS lookup
dig example.com AXFR @ns1.example.com  # Zone transfer attempt (recon!)

host example.com                  # Simple DNS lookup
host -t MX example.com           # Mail exchange records
```

> Zone transfer (`AXFR`) queries are an important reconnaissance technique. A misconfigured DNS server may respond with every hostname and IP in the entire zone. Always try `dig AXFR` against discovered name servers.
{: .tip }

### 4.4 Connectivity Testing

```bash
# Ping — ICMP echo request/reply
ping -c 4 10.10.10.1             # Send 4 pings
ping -c 1 -W 1 10.10.10.1       # Single ping, 1 second timeout

# Traceroute — trace the path packets take
traceroute 10.10.10.1            # UDP-based (default)
traceroute -T 10.10.10.1        # TCP-based (more likely to pass firewalls)
traceroute -I 10.10.10.1        # ICMP-based

# mtr — combines ping and traceroute (real-time)
mtr 10.10.10.1                   # Interactive mode
mtr -r -c 10 10.10.10.1         # Report mode (10 cycles, then print report)
```

### 4.5 Ports and Connections

```bash
# ss — Socket Statistics (modern replacement for netstat)
ss -tulnp                        # TCP/UDP listening ports with process info
# -t TCP | -u UDP | -l listening | -n numeric | -p process

ss -tan                          # All TCP connections (including established)
ss -tan state established        # Only established connections
ss -s                            # Socket statistics summary

# netstat — legacy but still commonly used
netstat -tulnp                   # Same as ss -tulnp
netstat -an                      # All connections, numeric

# lsof — list open files (including network sockets)
sudo lsof -i                     # All network connections
sudo lsof -i :80                 # What's using port 80?
sudo lsof -i TCP                 # Only TCP connections
sudo lsof -i -P -n              # Numeric ports and IPs (faster)
sudo lsof -u kali               # Files opened by user kali
```

### 4.6 Network Configuration Files

```bash
# Debian/Kali static network config
cat /etc/network/interfaces
# auto eth0
# iface eth0 inet static
#   address 192.168.1.100
#   netmask 255.255.255.0
#   gateway 192.168.1.1
#   dns-nameservers 8.8.8.8

# Restart networking
sudo systemctl restart networking

# NetworkManager (GUI-capable, common on desktop Kali)
nmcli device status               # Show device status
nmcli connection show             # Show connections
nmcli device wifi list            # List available WiFi networks
nmcli device wifi connect "SSID" password "pass"  # Connect to WiFi
```

### 4.7 Firewall Basics

```bash
# iptables — the classic Linux firewall
sudo iptables -L -n -v           # List all rules (verbose, numeric)
sudo iptables -L -n --line-numbers  # With rule numbers (for deletion)

# Basic iptables rules
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT    # Allow SSH
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT    # Allow HTTP
sudo iptables -A INPUT -j DROP                         # Drop everything else
sudo iptables -D INPUT 3                               # Delete rule #3
sudo iptables -F                                       # Flush all rules

# ufw — Uncomplicated Firewall (easier frontend for iptables)
sudo ufw status                   # Check status
sudo ufw enable                   # Enable firewall
sudo ufw disable                  # Disable firewall
sudo ufw allow 22/tcp            # Allow SSH
sudo ufw allow from 192.168.1.0/24  # Allow entire subnet
sudo ufw deny 23/tcp             # Deny Telnet
sudo ufw delete allow 22/tcp     # Remove rule

# nftables — modern replacement for iptables
sudo nft list ruleset             # Show all rules
```

### 4.8 ARP

The Address Resolution Protocol maps IP addresses to MAC addresses. ARP poisoning is a fundamental network attack technique.

```bash
# View ARP table
arp -a                            # Show ARP cache
ip neigh                          # Modern equivalent
ip neigh show                     # Same

# Add/delete static ARP entries
sudo arp -s 192.168.1.1 aa:bb:cc:dd:ee:ff   # Static entry
sudo arp -d 192.168.1.1                      # Delete entry
```

### 4.9 Packet Capture with tcpdump

```bash
# Basic capture
sudo tcpdump -i eth0                         # Capture all traffic on eth0
sudo tcpdump -i any                          # Capture on all interfaces
sudo tcpdump -c 100 -i eth0                 # Capture 100 packets then stop

# Filtered capture
sudo tcpdump -i eth0 host 10.10.10.1        # Traffic to/from specific host
sudo tcpdump -i eth0 port 80                # HTTP traffic only
sudo tcpdump -i eth0 src 192.168.1.1        # Only from source IP
sudo tcpdump -i eth0 dst port 443           # Only to destination port 443
sudo tcpdump -i eth0 'tcp port 80 or tcp port 443'  # HTTP or HTTPS

# Save and read captures
sudo tcpdump -i eth0 -w capture.pcap        # Save to file
tcpdump -r capture.pcap                      # Read from file
tcpdump -r capture.pcap -A                   # ASCII output (show HTTP content)
tcpdump -r capture.pcap -X                   # Hex + ASCII output

# Useful flags
sudo tcpdump -i eth0 -n -v                  # No DNS resolution, verbose
sudo tcpdump -i eth0 -nn                    # No DNS or port name resolution
```

### 4.10 Common Protocols and Ports

Memorize these. You will see them constantly.

| Port | Protocol | Service | Notes |
|:-----|:---------|:--------|:------|
| 20 | TCP | FTP Data | File Transfer Protocol (data channel) |
| 21 | TCP | FTP Control | FTP command channel — often allows anonymous login |
| 22 | TCP | SSH | Secure Shell — encrypted remote access |
| 23 | TCP | Telnet | Unencrypted remote access (avoid; cleartext credentials) |
| 25 | TCP | SMTP | Email sending — can sometimes be used for user enumeration |
| 53 | TCP/UDP | DNS | Domain Name System — zone transfers on TCP |
| 67/68 | UDP | DHCP | Dynamic Host Configuration Protocol |
| 69 | UDP | TFTP | Trivial FTP — no authentication |
| 80 | TCP | HTTP | Unencrypted web traffic |
| 88 | TCP | Kerberos | Active Directory authentication |
| 110 | TCP | POP3 | Email retrieval (unencrypted) |
| 111 | TCP/UDP | RPCbind | Remote Procedure Call — enumerate services |
| 135 | TCP | MSRPC | Microsoft RPC — Windows endpoint mapper |
| 137-139 | TCP/UDP | NetBIOS | Windows networking (legacy) |
| 143 | TCP | IMAP | Email retrieval (unencrypted) |
| 161/162 | UDP | SNMP | Network management — often poorly secured |
| 389 | TCP | LDAP | Directory services (Active Directory) |
| 443 | TCP | HTTPS | Encrypted web traffic |
| 445 | TCP | SMB | Server Message Block — file sharing, EternalBlue |
| 465 | TCP | SMTPS | Encrypted SMTP |
| 514 | UDP | Syslog | System logging |
| 587 | TCP | SMTP (submission) | Email submission with authentication |
| 636 | TCP | LDAPS | Encrypted LDAP |
| 993 | TCP | IMAPS | Encrypted IMAP |
| 995 | TCP | POP3S | Encrypted POP3 |
| 1433 | TCP | MSSQL | Microsoft SQL Server |
| 1521 | TCP | Oracle | Oracle database |
| 2049 | TCP | NFS | Network File System — mount remote shares |
| 3306 | TCP | MySQL | MySQL / MariaDB database |
| 3389 | TCP | RDP | Remote Desktop Protocol (Windows) |
| 5432 | TCP | PostgreSQL | PostgreSQL database |
| 5900+ | TCP | VNC | Virtual Network Computing (remote desktop) |
| 5985 | TCP | WinRM HTTP | Windows Remote Management |
| 5986 | TCP | WinRM HTTPS | Windows Remote Management (encrypted) |
| 6379 | TCP | Redis | In-memory database — often no auth |
| 8080 | TCP | HTTP Proxy | Common alternative HTTP port |
| 8443 | TCP | HTTPS Alt | Common alternative HTTPS port |
| 8888 | TCP | HTTP Alt | Alternative HTTP port |
| 27017 | TCP | MongoDB | Document database — often no auth |

> Print this table and keep it at your desk. During CTFs and engagements, instantly recognizing what service a port belongs to saves enormous time. When nmap reveals open ports, you should know immediately what to try.
{: .tip }

---

## 5 — Package Management

### 5.1 APT (Advanced Package Tool)

APT is the primary package manager on Debian-based systems including Kali Linux.

```bash
# Update package lists (always do this first)
sudo apt update

# Upgrade all installed packages
sudo apt upgrade                  # Upgrade with confirmation
sudo apt full-upgrade             # Upgrade + remove obsolete packages
sudo apt dist-upgrade             # Same as full-upgrade

# Install packages
sudo apt install nmap             # Install single package
sudo apt install nmap nikto dirb  # Install multiple packages
sudo apt install -y nmap          # Auto-confirm (non-interactive)

# Remove packages
sudo apt remove nmap              # Remove package (keep config)
sudo apt purge nmap               # Remove package AND config files
sudo apt autoremove               # Remove unused dependencies

# Search and info
apt search metasploit             # Search for packages
apt show nmap                     # Show package details
apt list --installed              # List all installed packages
apt list --installed | grep -i nmap   # Check if installed
apt list --upgradable             # Packages with available updates

# Cache management
sudo apt clean                    # Remove downloaded .deb files
sudo apt autoclean                # Remove old downloaded .deb files
```

### 5.2 dpkg — Low-Level Package Manager

```bash
# Install .deb file
sudo dpkg -i package.deb         # Install local .deb package
sudo apt install -f               # Fix broken dependencies after dpkg

# Query installed packages
dpkg -l                           # List all installed packages
dpkg -l | grep nmap               # Search installed packages
dpkg -L nmap                      # List files installed by package
dpkg -S /usr/bin/nmap            # Which package owns this file?
dpkg --status nmap                # Package info
```

### 5.3 Kali Repositories

```bash
# View configured repositories
cat /etc/apt/sources.list
# deb http://http.kali.org/kali kali-rolling main contrib non-free non-free-firmware

# Kali metapackages (install entire tool categories)
sudo apt install kali-linux-default       # Default tools
sudo apt install kali-linux-large         # Large set (most tools)
sudo apt install kali-linux-everything    # Every tool (several GB)
sudo apt install kali-tools-web           # Web application tools
sudo apt install kali-tools-exploitation  # Exploitation tools
sudo apt install kali-tools-passwords     # Password attack tools
sudo apt install kali-tools-wireless      # Wireless tools
sudo apt install kali-tools-forensics     # Forensic tools
sudo apt install kali-tools-sniffing-spoofing  # Network tools
```

> Never add random third-party repositories to your Kali system unless you absolutely trust them. Malicious repositories can serve backdoored packages. Stick to the official Kali repos whenever possible.
{: .warning }

### 5.4 Installing Tools From Other Sources

```bash
# From GitHub
git clone https://github.com/author/tool.git
cd tool
pip3 install -r requirements.txt   # If Python
./install.sh                       # If it has an installer
sudo make install                  # If C/C++

# Python tools via pip
pip3 install impacket              # Install Python package
pip3 install --user impacket       # Install for current user only
pip3 install --upgrade impacket    # Upgrade package

# Go tools
go install github.com/author/tool@latest

# Ruby tools (gems)
sudo gem install wpscan

# Standalone binaries
wget https://example.com/tool
chmod +x tool
sudo mv tool /usr/local/bin/      # Move to a directory in PATH
```

---

## 6 — Essential Kali Tools Overview

Kali comes pre-loaded with hundreds of security tools. Here is a categorized reference of the most important ones you will use throughout this guide.

### Information Gathering

| Tool | Purpose | Quick Example |
|:-----|:--------|:-------------|
| **nmap** | Network scanner and service detection | `nmap -sC -sV -oN scan.txt 10.10.10.1` |
| **masscan** | Fastest port scanner (async) | `masscan 10.10.10.0/24 -p1-65535 --rate=1000` |
| **maltego** | OSINT and relationship mapping (GUI) | Link analysis between entities |
| **recon-ng** | Web reconnaissance framework | `recon-ng` then `marketplace install all` |
| **theHarvester** | Email, subdomain, and IP harvesting | `theHarvester -d example.com -b google` |
| **amass** | Subdomain enumeration | `amass enum -d example.com` |
| **dnsrecon** | DNS enumeration | `dnsrecon -d example.com -t std` |
| **enum4linux** | SMB/Windows enumeration | `enum4linux -a 10.10.10.1` |
| **nbtscan** | NetBIOS name scanner | `nbtscan 192.168.1.0/24` |
| **smbclient** | SMB client (file shares) | `smbclient -L //10.10.10.1 -N` |
| **whatweb** | Web technology fingerprinting | `whatweb http://10.10.10.1` |
| **wafw00f** | WAF detection | `wafw00f http://example.com` |

### Vulnerability Analysis

| Tool | Purpose | Quick Example |
|:-----|:--------|:-------------|
| **OpenVAS/GVM** | Full vulnerability scanner | Web GUI at `https://127.0.0.1:9392` |
| **Nessus** | Commercial vulnerability scanner | Web GUI (requires license) |
| **nikto** | Web server vulnerability scanner | `nikto -h http://10.10.10.1` |
| **wpscan** | WordPress vulnerability scanner | `wpscan --url http://10.10.10.1 -e ap,at,u` |
| **sqlmap** | SQL injection detection & exploitation | `sqlmap -u "http://10.10.10.1/?id=1" --dbs` |
| **searchsploit** | Local Exploit-DB search | `searchsploit apache 2.4` |
| **nmap NSE scripts** | Vulnerability scanning scripts | `nmap --script vuln 10.10.10.1` |

### Web Application Testing

| Tool | Purpose | Quick Example |
|:-----|:--------|:-------------|
| **Burp Suite** | Web app proxy and scanner (GUI) | Intercept and modify HTTP requests |
| **OWASP ZAP** | Open-source web app scanner | Automated and manual web testing |
| **sqlmap** | Automated SQL injection | `sqlmap -r request.txt --batch` |
| **dirb** | Web directory brute forcing | `dirb http://10.10.10.1` |
| **gobuster** | Directory/DNS/vhost brute forcing | `gobuster dir -u http://10.10.10.1 -w wordlist` |
| **feroxbuster** | Fast recursive content discovery | `feroxbuster -u http://10.10.10.1 -w wordlist` |
| **ffuf** | Fast web fuzzer | `ffuf -u http://10.10.10.1/FUZZ -w wordlist` |
| **wfuzz** | Web application fuzzer | `wfuzz -c -z file,wordlist -u http://10.10.10.1/FUZZ` |
| **nikto** | Web server scanner | `nikto -h http://10.10.10.1` |
| **whatweb** | Technology fingerprinting | `whatweb 10.10.10.1` |
| **curl** | HTTP client | `curl -v http://10.10.10.1` |

### Password Attacks

| Tool | Purpose | Quick Example |
|:-----|:--------|:-------------|
| **John the Ripper** | Offline hash cracker | `john --wordlist=rockyou.txt hashes.txt` |
| **Hashcat** | GPU-accelerated hash cracker | `hashcat -m 0 hashes.txt rockyou.txt` |
| **Hydra** | Online brute force (SSH, FTP, HTTP, etc.) | `hydra -l admin -P rockyou.txt ssh://10.10.10.1` |
| **Medusa** | Online brute force | `medusa -h 10.10.10.1 -u admin -P pass.txt -M ssh` |
| **CeWL** | Custom wordlist generator (from website) | `cewl http://10.10.10.1 -w wordlist.txt` |
| **Crunch** | Pattern-based wordlist generator | `crunch 8 8 -t ,@@^^%%% -o wordlist.txt` |
| **hash-identifier** | Identify hash type | `hash-identifier` then paste hash |
| **hashid** | Identify hash type | `hashid '$6$rounds=5000$...'` |

### Wireless Attacks

| Tool | Purpose | Quick Example |
|:-----|:--------|:-------------|
| **aircrack-ng** | WEP/WPA/WPA2 cracker | `aircrack-ng -w rockyou.txt capture.cap` |
| **airmon-ng** | Enable monitor mode | `airmon-ng start wlan0` |
| **airodump-ng** | Wireless traffic capture | `airodump-ng wlan0mon` |
| **aireplay-ng** | Packet injection | `aireplay-ng -0 5 -a BSSID wlan0mon` |
| **reaver** | WPS brute force | `reaver -i wlan0mon -b BSSID -vv` |
| **wifite** | Automated wireless auditing | `wifite` |
| **bettercap** | Network attack and monitoring | `bettercap -iface wlan0` |

### Exploitation

| Tool | Purpose | Quick Example |
|:-----|:--------|:-------------|
| **Metasploit Framework** | Exploit development and delivery | `msfconsole` then `search eternalblue` |
| **msfvenom** | Payload generation | `msfvenom -p linux/x64/shell_reverse_tcp LHOST=x LPORT=y -f elf -o shell` |
| **searchsploit** | Local exploit database search | `searchsploit -m 42315` (copy exploit) |
| **BeEF** | Browser exploitation framework | Hook and control browsers |
| **Social Engineering Toolkit (SET)** | Phishing and SE attacks | `setoolkit` |
| **crackmapexec / NetExec** | Network service exploitation | `crackmapexec smb 10.10.10.0/24` |
| **evil-winrm** | WinRM shell | `evil-winrm -i 10.10.10.1 -u admin -p pass` |
| **impacket** | Python network protocol tools | `impacket-psexec domain/user:pass@10.10.10.1` |

### Sniffing and Spoofing

| Tool | Purpose | Quick Example |
|:-----|:--------|:-------------|
| **Wireshark** | Network protocol analyzer (GUI) | Capture and analyze packets |
| **tcpdump** | CLI packet capture | `tcpdump -i eth0 -w capture.pcap` |
| **Responder** | LLMNR/NBT-NS/MDNS poisoner | `responder -I eth0 -wrf` |
| **ettercap** | ARP spoofing and MITM | `ettercap -T -M arp:remote /target1/ /target2/` |
| **bettercap** | Network attack suite | `bettercap -iface eth0` |
| **mitmproxy** | Intercepting HTTP/HTTPS proxy | `mitmproxy -p 8080` |
| **arpspoof** | ARP spoofing tool | `arpspoof -i eth0 -t target gateway` |

### Post-Exploitation

| Tool | Purpose | Quick Example |
|:-----|:--------|:-------------|
| **Mimikatz** | Windows credential extraction | `mimikatz.exe "sekurlsa::logonpasswords"` |
| **Empire / Starkiller** | Post-exploitation framework | Agent-based persistence and pivoting |
| **BloodHound** | Active Directory attack path mapping | Visualize domain relationships |
| **LinPEAS** | Linux privilege escalation scanner | `./linpeas.sh` |
| **WinPEAS** | Windows privilege escalation scanner | `winpeas.exe` |
| **pspy** | Monitor Linux processes without root | `./pspy64` |
| **Chisel** | TCP/UDP tunnel over HTTP | `chisel server -p 8000 --reverse` |
| **Ligolo-ng** | Tunneling/pivoting | Establish tunnels through compromised hosts |
| **Socat** | Multipurpose relay tool | `socat TCP-LISTEN:8080,fork TCP:target:80` |

### Forensics

| Tool | Purpose | Quick Example |
|:-----|:--------|:-------------|
| **Autopsy** | Digital forensics platform (GUI) | Analyze disk images |
| **Volatility** | Memory forensics | `vol.py -f memory.dmp imageinfo` |
| **binwalk** | Firmware analysis and extraction | `binwalk -e firmware.bin` |
| **foremost** | File carving from disk images | `foremost -i disk.img -o output/` |
| **exiftool** | Image/document metadata extraction | `exiftool image.jpg` |
| **steghide** | Steganography detection/extraction | `steghide extract -sf image.jpg` |
| **strings** | Extract printable strings from binaries | `strings suspicious_binary` |

> You do not need to memorize every tool. Focus on learning 2-3 tools per category deeply. For most engagements, `nmap`, `Burp Suite`, `Metasploit`, `John/Hashcat`, and `Wireshark` will cover 80% of your needs.
{: .tip }

---

## 7 — Bash Scripting for Hackers

Scripts automate the tedious parts of penetration testing. A good script can turn a 30-minute manual process into a 30-second automated one.

### 7.1 Your First Recon Script

This script takes a target domain and performs basic reconnaissance:

```bash
#!/bin/bash
# recon.sh — Basic domain reconnaissance
# Usage: ./recon.sh example.com

if [ -z "$1" ]; then
    echo "Usage: $0 <domain>"
    exit 1
fi

DOMAIN=$1
OUTPUT_DIR="recon_$DOMAIN"
mkdir -p $OUTPUT_DIR

echo "========================================="
echo "  Reconnaissance: $DOMAIN"
echo "  Started: $(date)"
echo "========================================="

# DNS lookup
echo -e "\n[*] DNS Records..."
dig $DOMAIN ANY +noall +answer | tee $OUTPUT_DIR/dns.txt

# Subdomains via DNS
echo -e "\n[*] Name Servers..."
dig NS $DOMAIN +short | tee -a $OUTPUT_DIR/dns.txt

# WHOIS
echo -e "\n[*] WHOIS Lookup..."
whois $DOMAIN | tee $OUTPUT_DIR/whois.txt

# Web technology detection
echo -e "\n[*] Web Technologies..."
whatweb $DOMAIN 2>/dev/null | tee $OUTPUT_DIR/whatweb.txt

# HTTP headers
echo -e "\n[*] HTTP Headers..."
curl -sI https://$DOMAIN | tee $OUTPUT_DIR/headers.txt

# SSL certificate info
echo -e "\n[*] SSL Certificate..."
echo | openssl s_client -connect $DOMAIN:443 2>/dev/null | openssl x509 -noout -subject -issuer -dates | tee $OUTPUT_DIR/ssl.txt

echo -e "\n========================================="
echo "  Reconnaissance complete!"
echo "  Results saved to: $OUTPUT_DIR/"
echo "========================================="
```

```bash
chmod +x recon.sh
./recon.sh example.com
```

### 7.2 Ping Sweep Script

Discover live hosts on a network:

```bash
#!/bin/bash
# ping_sweep.sh — Discover live hosts on a subnet
# Usage: ./ping_sweep.sh 192.168.1

if [ -z "$1" ]; then
    echo "Usage: $0 <first-three-octets>"
    echo "Example: $0 192.168.1"
    exit 1
fi

SUBNET=$1
ALIVE=0
TOTAL=254

echo "[*] Ping sweep on $SUBNET.0/24"
echo "[*] Started: $(date)"
echo ""

for ip in $(seq 1 254); do
    ping -c 1 -W 1 $SUBNET.$ip > /dev/null 2>&1
    if [ $? -eq 0 ]; then
        echo "[+] $SUBNET.$ip is ALIVE"
        ALIVE=$((ALIVE + 1))
    fi
done &

# Wait for background processes
wait

echo ""
echo "[*] Sweep complete: $ALIVE hosts alive out of $TOTAL"
```

**Faster version using parallel execution:**

```bash
#!/bin/bash
# fast_ping_sweep.sh — Parallel ping sweep
# Usage: ./fast_ping_sweep.sh 192.168.1

if [ -z "$1" ]; then
    echo "Usage: $0 <first-three-octets>"
    exit 1
fi

SUBNET=$1

echo "[*] Fast ping sweep on $SUBNET.0/24"

for ip in $(seq 1 254); do
    (ping -c 1 -W 1 $SUBNET.$ip > /dev/null 2>&1 && echo "[+] $SUBNET.$ip is ALIVE") &
done

wait
echo "[*] Sweep complete."
```

> The parallel version runs all 254 pings simultaneously using subshells `( )` and `&`. It completes in roughly 1 second instead of 4 minutes.
{: .tip }

### 7.3 Port Scanner Script

A simple TCP port scanner written entirely in Bash:

```bash
#!/bin/bash
# port_scanner.sh — Simple TCP port scanner
# Usage: ./port_scanner.sh 10.10.10.1 [start_port] [end_port]

TARGET=$1
START=${2:-1}
END=${3:-1024}

if [ -z "$TARGET" ]; then
    echo "Usage: $0 <target_ip> [start_port] [end_port]"
    echo "Example: $0 10.10.10.1 1 65535"
    exit 1
fi

echo "[*] Scanning $TARGET ports $START-$END"
echo "[*] Started: $(date)"
echo ""

OPEN=0

for port in $(seq $START $END); do
    # Use /dev/tcp (Bash built-in) to test connection
    (echo >/dev/tcp/$TARGET/$port) 2>/dev/null
    if [ $? -eq 0 ]; then
        echo "[+] Port $port is OPEN"
        OPEN=$((OPEN + 1))
    fi
done

echo ""
echo "[*] Scan complete: $OPEN open ports found"
echo "[*] Finished: $(date)"
```

> This scanner uses Bash's built-in `/dev/tcp/host/port` pseudo-device, which attempts a TCP connection. It is slow for large ranges — for real-world scanning, always use nmap or masscan.
{: .note }

### 7.4 Automating Repetitive Tasks

#### Auto-enumerate a list of targets

```bash
#!/bin/bash
# batch_scan.sh — Scan multiple targets from a file
# Usage: ./batch_scan.sh targets.txt

if [ ! -f "$1" ]; then
    echo "Usage: $0 <targets_file>"
    echo "File should contain one IP per line"
    exit 1
fi

TARGETS_FILE=$1
OUTPUT_DIR="batch_scan_$(date +%Y%m%d_%H%M%S)"
mkdir -p $OUTPUT_DIR

while IFS= read -r target; do
    # Skip empty lines and comments
    [[ -z "$target" || "$target" =~ ^# ]] && continue

    echo "[*] Scanning $target..."
    nmap -sC -sV -oN "$OUTPUT_DIR/${target}.txt" "$target" &

    # Limit parallel scans to avoid overwhelming the network
    if [ $(jobs -r | wc -l) -ge 5 ]; then
        wait -n  # Wait for any one job to finish
    fi
done < "$TARGETS_FILE"

wait  # Wait for all remaining jobs
echo "[*] All scans complete. Results in $OUTPUT_DIR/"
```

### 7.5 Parsing Tool Output

```bash
# Extract live hosts from nmap ping scan
nmap -sn 192.168.1.0/24 -oG - | awk '/Up$/{print $2}'

# Extract open ports from nmap output
grep "^[0-9]" nmap_scan.txt | grep "open" | cut -d/ -f1 | sort -n | uniq

# Extract URLs from a web page
curl -s http://target.com | grep -oE 'https?://[^"]+' | sort -u

# Parse Hydra results for successful logins
grep "\[22\]\[ssh\]" hydra_output.txt | awk '{print $5, $7}'

# Extract email addresses from harvested data
grep -oE '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}' data.txt | sort -u

# Count unique IPs in a log file
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -20

# Extract hashes from a hashdump
cat hashdump.txt | cut -d: -f4  # NTLM hash (4th field in SAM dump)

# Process nmap XML output with grep
grep -oP 'portid="\K[^"]+' scan.xml  # Extract port numbers from XML

# One-liner: scan, extract IPs, then scan each for services
nmap -sn 10.10.10.0/24 -oG - | awk '/Up$/{print $2}' | while read ip; do
    echo "=== $ip ===" && nmap -sV --top-ports 20 $ip
done
```

---

## 8 — tmux — Terminal Multiplexer

### 8.1 Why tmux Is Essential for Pentesting

When you are running multiple tools simultaneously during an engagement — a listener in one pane, a scan in another, a shell in a third — tmux is indispensable. It lets you:

- **Split your terminal** into multiple panes and windows
- **Keep sessions alive** even if your SSH connection drops
- **Organize your workflow** across different phases of a pentest
- **Switch between tasks** without losing state

### 8.2 Session Management

```bash
# Start a new session
tmux                              # Anonymous session
tmux new -s pentest               # Named session "pentest"

# Detach from session (session keeps running)
Ctrl+B, then D                    # Detach

# List sessions
tmux ls

# Reattach to session
tmux attach -t pentest            # By name
tmux attach -t 0                  # By number
tmux a                            # Attach to last session

# Kill session
tmux kill-session -t pentest
tmux kill-server                  # Kill ALL sessions
```

### 8.3 Windows and Panes

All tmux commands start with the **prefix key**: `Ctrl+B` (press and release), then the command key.

#### Window Management

| Shortcut | Action |
|:---------|:-------|
| `Ctrl+B, c` | Create new window |
| `Ctrl+B, n` | Next window |
| `Ctrl+B, p` | Previous window |
| `Ctrl+B, 0-9` | Switch to window by number |
| `Ctrl+B, ,` | Rename current window |
| `Ctrl+B, w` | List all windows (interactive picker) |
| `Ctrl+B, &` | Kill current window |

#### Pane Management

| Shortcut | Action |
|:---------|:-------|
| `Ctrl+B, %` | Split pane vertically (left/right) |
| `Ctrl+B, "` | Split pane horizontally (top/bottom) |
| `Ctrl+B, arrow keys` | Navigate between panes |
| `Ctrl+B, x` | Kill current pane |
| `Ctrl+B, z` | Toggle pane zoom (fullscreen) |
| `Ctrl+B, {` | Swap pane with previous |
| `Ctrl+B, }` | Swap pane with next |
| `Ctrl+B, Space` | Cycle through pane layouts |
| `Ctrl+B, q` | Show pane numbers (press number to switch) |

#### Scrolling and Copy Mode

| Shortcut | Action |
|:---------|:-------|
| `Ctrl+B, [` | Enter copy/scroll mode |
| `q` | Exit copy mode |
| Arrow keys / PgUp/PgDn | Scroll in copy mode |
| `Space` | Start selection (in copy mode) |
| `Enter` | Copy selection and exit (in copy mode) |
| `Ctrl+B, ]` | Paste copied text |

### 8.4 Recommended tmux Configuration

Create or edit `~/.tmux.conf`:

```bash
# Remap prefix from Ctrl+B to Ctrl+A (easier to reach)
unbind C-b
set-option -g prefix C-a
bind-key C-a send-prefix

# Split panes using | and - (more intuitive)
bind | split-window -h -c "#{pane_current_path}"
bind - split-window -v -c "#{pane_current_path}"
unbind '"'
unbind %

# Navigate panes with Alt+arrow (no prefix needed)
bind -n M-Left select-pane -L
bind -n M-Right select-pane -R
bind -n M-Up select-pane -U
bind -n M-Down select-pane -D

# Enable mouse support (click to switch panes, scroll, resize)
set -g mouse on

# Start window numbering at 1 (not 0)
set -g base-index 1
setw -g pane-base-index 1

# Increase scrollback buffer
set -g history-limit 50000

# Reload config with prefix + r
bind r source-file ~/.tmux.conf \; display "Config reloaded!"

# Visual styling
set -g status-style bg=black,fg=white
set -g pane-active-border-style fg=green
set -g status-left "#[fg=green]#S "
set -g status-right "#[fg=yellow]%H:%M "

# Reduce escape time (important for Vim users)
set -sg escape-time 0

# New windows/panes open in current directory
bind c new-window -c "#{pane_current_path}"
```

After saving, reload with:

```bash
tmux source-file ~/.tmux.conf
```

### 8.5 Pentest tmux Workflow Example

```bash
# Start a pentest session
tmux new -s htb_target

# Window 1: Scanning (rename with Ctrl+A, ,)
# Pane 1: nmap scan
# Pane 2: directory brute force

# Window 2: Exploitation
# Pane 1: Metasploit
# Pane 2: Listener (nc -lvnp 4444)

# Window 3: Post-exploitation
# Pane 1: Reverse shell
# Pane 2: LinPEAS output

# Window 4: Notes
# Pane 1: vim notes.md
```

> If your SSH connection drops during a pentest, your tmux session (and all running tools) stay alive on the remote machine. Just SSH back in and run `tmux attach`. This alone makes tmux worth learning.
{: .tip }

---

## 9 — Knowledge Check

Test your understanding of Kali Linux fundamentals with these questions.

<details>
<summary><strong>Question 1:</strong> What command would you use to find all SUID binaries on a Linux system, and why is this important for penetration testing?</summary>

**Answer:**

```bash
find / -perm -4000 -type f 2>/dev/null
```

SUID (Set User ID) binaries execute with the permissions of the file **owner**, not the user who runs them. If a SUID binary is owned by root and has a vulnerability or can be abused to spawn a shell, it provides a direct privilege escalation path from a regular user to root. This is one of the most common and reliable privilege escalation techniques, which is why checking for SUID binaries is one of the first things you do after gaining access to a system.

Common exploitable SUID binaries include `find`, `vim`, `python`, `bash`, `nmap` (old versions), `less`, and `cp`. Check GTFOBins (https://gtfobins.github.io) for exploitation techniques.
</details>

<details>
<summary><strong>Question 2:</strong> Explain the difference between <code>></code>, <code>>></code>, <code>2></code>, and <code>2>&1</code> in Bash.</summary>

**Answer:**

| Operator | Function | Example |
|:---------|:---------|:--------|
| `>` | Redirect **stdout** to a file, **overwriting** the file | `echo "hello" > file.txt` |
| `>>` | Redirect **stdout** to a file, **appending** to the file | `echo "world" >> file.txt` |
| `2>` | Redirect **stderr** (error messages) to a file | `ls /nonexistent 2> errors.txt` |
| `2>&1` | Redirect **stderr** to wherever **stdout** is going | `command > output.txt 2>&1` (both to file) |

In the last example, `2>&1` means "send file descriptor 2 (stderr) to the same place as file descriptor 1 (stdout)." This is essential when you want to capture all output (both normal and error) in a single file, or discard everything with `> /dev/null 2>&1`.
</details>

<details>
<summary><strong>Question 3:</strong> What is the difference between <code>/etc/passwd</code> and <code>/etc/shadow</code>? Why are they separated?</summary>

**Answer:**

`/etc/passwd` contains user account information (username, UID, GID, home directory, shell) and is **world-readable** — any user on the system can read it. This is necessary because many programs need to look up user information.

`/etc/shadow` contains the actual **password hashes** and is only readable by root (`-rw-r----- root shadow`). It also stores password aging information (last changed, min/max age, expiration).

They are separated for security. In early Unix, password hashes were stored directly in `/etc/passwd`. Since that file must be world-readable, anyone on the system could extract the hashes and crack them offline. By moving hashes to the root-only-readable `/etc/shadow`, this attack is prevented for non-privileged users.

During a pentest, if you can read `/etc/shadow` (either as root or through a misconfiguration), you can extract the hashes and crack them offline using `john` or `hashcat`.
</details>

<details>
<summary><strong>Question 4:</strong> Write a one-liner that extracts all unique IP addresses from a log file and sorts them by frequency.</summary>

**Answer:**

```bash
grep -oE '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' access.log | sort | uniq -c | sort -rn
```

Breakdown:
- `grep -oE '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}'` — Extract only IP-like patterns (extended regex, only matching part)
- `sort` — Sort alphabetically (required for `uniq` to work correctly)
- `uniq -c` — Deduplicate and prepend count of occurrences
- `sort -rn` — Sort numerically in reverse order (most frequent first)

This is invaluable for identifying the most active IP addresses in web server logs, authentication logs, or packet captures.
</details>

<details>
<summary><strong>Question 5:</strong> What is the significance of the <code>/tmp</code> directory's sticky bit, and how does it affect security?</summary>

**Answer:**

The `/tmp` directory has permissions `drwxrwxrwt` — the `t` at the end indicates the **sticky bit** is set. This means:

- Anyone can **create** files in `/tmp` (world-writable)
- Anyone can **read** files in `/tmp` (world-readable)
- Only the **file owner** (or root) can **delete or rename** their own files

Without the sticky bit, any user could delete any other user's files in a world-writable directory, enabling denial-of-service attacks or race conditions.

**Security implications for pentesting:**
- `/tmp` is often used to stage exploits, download tools, and compile payloads because it is world-writable
- Files in `/tmp` are typically cleared on reboot, so persistence mechanisms should use other locations
- Some systems mount `/tmp` with `noexec`, preventing execution of binaries (bypass: use interpreted scripts like `python` or `bash`)
- Cron jobs that operate on files in `/tmp` may be vulnerable to race conditions or symlink attacks
</details>

<details>
<summary><strong>Question 6:</strong> How would you use <code>awk</code> to extract all usernames from <code>/etc/passwd</code> that have a UID greater than or equal to 1000 and have <code>/bin/bash</code> as their shell?</summary>

**Answer:**

```bash
awk -F: '$3 >= 1000 && $7 == "/bin/bash" {print $1}' /etc/passwd
```

Breakdown:
- `-F:` — Use colon as field separator (passwd format is colon-delimited)
- `$3 >= 1000` — Third field (UID) is greater than or equal to 1000 (regular users)
- `$7 == "/bin/bash"` — Seventh field (shell) is exactly `/bin/bash`
- `&&` — Both conditions must be true
- `{print $1}` — Print the first field (username)

This identifies real user accounts with interactive shells — exactly the accounts you want to target for password attacks during a pentest.
</details>

<details>
<summary><strong>Question 7:</strong> Explain the purpose of tmux and describe a scenario where it saves your pentest.</summary>

**Answer:**

tmux (Terminal Multiplexer) allows you to create persistent terminal sessions with multiple windows and panes that survive connection drops.

**Scenario that saves your pentest:**

You are connected via SSH to your Kali machine on a VPN. You have:
- A Metasploit listener waiting for a reverse shell on port 4444
- A directory brute force running against the web server (2 hours in, 60% complete)
- An active reverse shell on the target machine where you are running LinPEAS

Your home internet drops for 3 minutes. Without tmux, **all three processes die** — your listener is gone, your brute force needs to restart from scratch, and your shell on the target is lost.

With tmux, all three processes continue running inside the tmux session on the server. When your internet comes back, you simply `tmux attach -t pentest` and everything is exactly where you left it.

This is not a hypothetical — it happens constantly on real engagements, especially when working over VPNs to remote labs or client networks.
</details>

<details>
<summary><strong>Question 8:</strong> What is the difference between <code>kill</code>, <code>kill -9</code>, and <code>kill -15</code>? When should you use each?</summary>

**Answer:**

| Command | Signal | Name | Behavior |
|:--------|:-------|:-----|:---------|
| `kill PID` | SIGTERM (15) | Terminate | Politely asks the process to exit. The process can catch this signal, clean up resources, close files, and exit gracefully. This is the **default** and should be tried first. |
| `kill -15 PID` | SIGTERM (15) | Terminate | Identical to `kill PID` — explicit form of the default. |
| `kill -9 PID` | SIGKILL (9) | Kill | **Forcefully terminates** the process immediately. The process **cannot catch, ignore, or handle** this signal. The kernel terminates it directly. Can leave behind lock files, temporary files, corrupted data, or zombie processes. |

**When to use each:**

1. Always try `kill PID` (SIGTERM) first — give the process a chance to clean up
2. Wait a few seconds
3. If the process is still running, use `kill -9 PID` (SIGKILL) as a last resort

Other useful signals:
- `kill -1 PID` (SIGHUP) — Often causes daemons to reload configuration
- `kill -2 PID` (SIGINT) — Same as pressing Ctrl+C
- `kill -19 PID` (SIGSTOP) — Pause process (cannot be caught)
- `kill -18 PID` (SIGCONT) — Resume paused process
</details>

<details>
<summary><strong>Question 9:</strong> You have gained a low-privilege shell on a Linux target. List the first five commands you would run and explain why.</summary>

**Answer:**

```bash
# 1. Who am I and what are my privileges?
id
# Shows UID, GID, and group memberships. Tells you if you're in
# interesting groups (sudo, docker, lxd, disk, etc.)

# 2. What can I run with sudo?
sudo -l
# Lists commands you can run as root without knowing root's password.
# Misconfigured sudo entries are one of the most common privesc vectors.
# Example: (root) NOPASSWD: /usr/bin/vim → instant root

# 3. What SUID binaries exist?
find / -perm -4000 -type f 2>/dev/null
# SUID binaries run as their owner (often root). Exploitable ones
# give you root access. Cross-reference with GTFOBins.

# 4. What system am I on?
uname -a && cat /etc/os-release
# Kernel version reveals potential kernel exploits.
# OS version reveals specific vulnerabilities and exploit compatibility.

# 5. Who else is on this system and what's running?
w && ps aux
# 'w' shows logged-in users and what they're doing.
# 'ps aux' shows all running processes — look for processes running
# as root that you can interact with, cron jobs, database servers
# with credentials in the command line, etc.
```

These five commands give you the most critical information in seconds: your identity, your privileges, potential escalation paths, the system landscape, and who else is present.
</details>

<details>
<summary><strong>Question 10:</strong> What is the difference between <code>ss -tulnp</code> and <code>netstat -tulnp</code>? Parse the flags and explain when you would use each.</summary>

**Answer:**

Both commands show the same information — listening network ports and their associated processes. The flags are identical in meaning:

| Flag | Meaning |
|:-----|:--------|
| `-t` | Show TCP sockets |
| `-u` | Show UDP sockets |
| `-l` | Show only **listening** sockets (servers waiting for connections) |
| `-n` | Show **numeric** addresses and ports (don't resolve DNS or service names) |
| `-p` | Show the **process** using each socket (requires root for all processes) |

**Differences:**

| Aspect | `ss` | `netstat` |
|:-------|:-----|:---------|
| Speed | **Faster** (reads directly from kernel via netlink) | Slower (parses `/proc/net/`) |
| Modern | Part of `iproute2` (actively maintained) | Part of `net-tools` (deprecated) |
| Availability | Available on all modern Linux | May not be installed on minimal systems |
| Output format | Slightly different column layout | Traditional format |
| Filtering | More powerful built-in filters (`ss state established`) | Requires piping to grep |

**When to use each:**
- Use `ss` by default on modern systems — it is faster and more capable
- Use `netstat` on older systems where `ss` is not available
- Know both because you will encounter both in tutorials, writeups, and on target systems with different tools installed
</details>

---

## Summary

This module covered the essential skills you need to operate effectively in Kali Linux:

- **Command line mastery** — navigation, file operations, text processing, I/O redirection, and process management
- **Filesystem knowledge** — where to find credentials, logs, configurations, and sensitive data
- **User and permission management** — understanding the permission model and how to exploit SUID/SGID misconfigurations
- **Networking** — interfaces, routing, DNS, firewalls, packet capture, and the ports you need to know by heart
- **Package management** — keeping your tools up to date and installing new ones
- **Tool awareness** — a categorized overview of Kali's most important security tools
- **Bash scripting** — automating reconnaissance, scanning, and analysis
- **tmux** — organizing your terminal workflow for real-world engagements

> **Practice, practice, practice.** Open a terminal right now and try every command in this module. Build muscle memory. The difference between a beginner and a professional is not knowledge — it is speed. When you can fly through these commands without thinking, you are ready for the modules ahead.
{: .tip }

---

**Next Module:** [Module 04 — Passive Reconnaissance (OSINT)](04-passive-recon.html) — gathering intelligence without touching the target.
