Hasan, welcome to your next Linux lesson! Today, we're going to study **Linux File Management from a professional, real-world perspective**. I'll teach you as a Linux instructor would teach an aspiring system administrator, DevOps engineer, or security engineer.

Our goal is not simply to memorise commands. You should understand **what each command does, when to use it, how to use it safely, and how professionals apply it in real working environments.**

# Linux File Management: Professional Training

**Level:** Beginner to Advanced  
**Learning approach:** Concepts → Commands → Practical examples → Real-world scenarios → Safety rules → Assessment

## 1. Our learning roadmap

We'll divide the subject into seven modules.

| Module | Topic | What you'll learn |
|---|---|---|
| 1 | File Deletion & Safety | `rm`, `rmdir`, `unlink`, `shred`, safe deletion and recovery limitations |
| 2 | File Editing | `nano`, `vim`, `cat`, `less`, `tee`, redirection and editing configuration files |
| 3 | File Permissions | `chmod`, `chown`, `chgrp`, `umask`, ACLs and special permissions |
| 4 | Finding Files | `find`, `locate`, `which`, `whereis`, `grep` and file searches |
| 5 | Copying Files | `cp`, `rsync`, recursive copying, metadata preservation and backups |
| 6 | Moving & Renaming | `mv`, safe file organisation, cross-filesystem moves and atomic replacement |
| 7 | Linking Files | Hard links, symbolic links, inode behaviour and practical applications |

We'll also learn professional techniques such as command chaining, quoting filenames, dry runs, permission troubleshooting, and safe file operations over SSH.

**Important:** These modules are interconnected. For example, copying a file is easy, but copying it while preserving ownership, permissions and timestamps requires a deeper understanding.

---

# Module 1: File Deletion & Safety

## 2. Understand deletion before using commands

Linux provides several ways to remove files and directories. However, deleting a file is not always equivalent to immediately erasing its contents from a storage device.

When you delete a file, the filesystem generally removes its directory entry or link. The underlying data may remain on the storage device until it is overwritten or otherwise reclaimed.

This distinction matters when managing servers, databases, backups and sensitive information.

### 2.1 The essential deletion commands

| Command | Purpose | Example |
|---|---|---|
| `rm` | Remove files and, with suitable options, directories | `rm report.txt` |
| `rmdir` | Remove empty directories | `rmdir old_folder` |
| `unlink` | Remove one filesystem directory entry | `unlink report.txt` |
| `rm -r` | Remove a directory and its contents recursively | `rm -r old_folder` |
| `rm -i` | Ask for confirmation before removing each file | `rm -i report.txt` |
| `rm -I` | Ask for confirmation before a large removal or recursive operation | `rm -I *.log` |
| `shred` | Attempt to overwrite file contents before removing them | `shred -u secret.txt` |

**Professional distinction:** `rm` is the standard general-purpose removal command. `rmdir` is useful when you want to ensure a directory is empty before removing it. `shred` is not a universal secure-erasure solution.

### 2.2 Delete a single file

Suppose your current directory contains:

```text
project/
├── report.txt
├── notes.txt
└── backup.txt
```

To remove `report.txt`:

```bash
rm report.txt
```

To remove it with an interactive confirmation:

```bash
rm -i report.txt
```

The second command is useful when you want an extra opportunity to catch mistakes.

To remove several specifically named files:

```bash
rm report.txt notes.txt backup.txt
```

**Remember:** Linux commands are case-sensitive. `Report.txt` and `report.txt` can be two different files.

### 2.3 Remove an empty directory

```bash
rmdir old_folder
```

This works only if `old_folder` is empty.

Why is this useful?

Because the command refuses to remove a non-empty directory. That restriction provides a small but valuable safety check.

### 2.4 Remove a directory containing files

The recursive option `-r` allows `rm` to traverse a directory and remove its contents.

```bash
rm -r old_folder
```

For an interactive version:

```bash
rm -ri old_folder
```

The command can ask for confirmation as it processes the directory tree.

**Warning:** Recursive deletion can remove many files in one operation. Always inspect the target before running it.

### 2.5 Understand `rm -f` and `rm -rf`

You'll frequently encounter these commands in shell scripts and deployment instructions:

```bash
rm -f temporary.txt
```

The `-f` option suppresses certain prompts and ignores missing files.

```bash
rm -rf old_build/
```

Here:

- `-r` means recursive.
- `-f` means force.
- Together, they remove the target directory tree without the usual interactive safeguards.

These commands are not inherently malicious, but **a mistaken path can turn an ordinary cleanup operation into a serious incident**.

For example, do not casually run:

```bash
rm -rf *
```

This removes non-hidden entries in the current directory, subject to shell expansion and permissions. It does not mean "delete only temporary files."

A particularly dangerous situation occurs when variables or paths are constructed incorrectly in scripts.

### Professional technique: inspect before deleting

Use:

```bash
pwd
ls -la
```

Then inspect the specific target:

```bash
ls -ld -- old_folder
```

Only after confirming your current directory and intended target should you execute a deletion command.

For important data, take a backup or snapshot first.

## 3. Advanced deletion techniques

### 3.1 Delete files matching a pattern

Imagine a server contains these files:

```text
app.log
app.log.1
app.log.2
error.log
access.log
notes.txt
```

To remove files ending in `.log`:

```bash
rm -- *.log
```

This removes matching files in the current directory, such as `app.log`, `error.log` and `access.log`. It does not automatically search subdirectories.

Before deleting, preview the matches:

```bash
printf '%s\n' *.log
```

If there are no matching files, some shells may print the literal pattern instead. For more reliable matching in Bash:

```bash
shopt -s nullglob
files=( *.log )
printf '%s\n' "${files[@]}"
```

This creates an array containing the matching filenames. If the array is empty, no files matched.

### 3.2 Delete files older than a specified period

A real-world administrator may need to clean up old log files.

First, preview files in `/var/log/myapp` that haven't been modified in the last 30 days:

```bash
find /var/log/myapp -type f -mtime +30 -print
```

After verifying the results, you could delete the matching files:

```bash
find /var/log/myapp -type f -mtime +30 -delete
```

Understand the options:

- `find`: searches a directory tree.
- `-type f`: matches regular files.
- `-mtime +30`: selects files whose modification time falls beyond the specified age threshold, using `find`'s day-based rounding rules.
- `-print`: displays matching paths.
- `-delete`: deletes matching entries.

**Professional rule:** Run the `-print` version first. Confirm that the search directory and results are correct before using `-delete`.

Also, don't assume a file is safe to delete just because it is old. Logs may be required for audits, security investigations or troubleshooting.

### 3.3 Delete files with unusual names

Suppose a file is named:

```text
-report.txt
```

Because the name begins with a hyphen, a command-line program might interpret it as an option.

Use:

```bash
rm -- -report.txt
```

The `--` marks the end of command options for programs that support this convention.

Alternatively:

```bash
rm ./-report.txt
```

The second form makes the filename look like a relative path rather than an option.

For filenames containing spaces:

```bash
rm "my report.txt"
```

For filenames containing shell wildcard characters, quoting is especially important:

```bash
rm -- 'report[old].txt'
```

Without the quotes, the shell might interpret the square brackets as a pattern.

### 3.4 Can you securely delete a file?

You may encounter:

```bash
shred -u secret.txt
```

`shred` attempts to overwrite the file's contents before unlinking it. However, this does **not** guarantee secure erasure on every storage system.

For example:

- SSD wear levelling can leave old data in physical storage cells.
- Copy-on-write filesystems can preserve previous data blocks.
- Snapshots and backups may retain earlier versions.
- Cloud storage and network filesystems can have different storage behaviour.

For sensitive information, use an appropriate storage-security strategy, such as encryption, managed retention and deletion of encryption keys where appropriate.

### 3.5 Can deleted files be recovered?

Sometimes, yes.

Recovery depends on the filesystem, whether the data has been overwritten or reclaimed, the storage hardware, and whether snapshots or backups exist.

If you accidentally delete an important file:

1. Stop unnecessary writes to the affected filesystem.
2. Avoid installing recovery software onto the affected storage device.
3. Check backups, snapshots and version history.
4. For critical data, consider a professional recovery process.

**Golden rule:** Treat `rm` as a potentially irreversible operation. Linux does not provide a universal recycle bin for files deleted through the command line.

---

# Module 2: File Editing

## 4. The different ways to read and edit files

Linux offers several tools for viewing, creating and modifying files. Choosing the right one depends on whether you want to inspect a file, edit it interactively, automate a change or write command output into a file.

### 4.1 Read a file without editing it

| Command | Purpose | Example |
|---|---|---|
| `cat` | Print a file's contents | `cat notes.txt` |
| `less` | Browse a file interactively | `less notes.txt` |
| `head` | Display the beginning of a file | `head -n 20 app.log` |
| `tail` | Display the end of a file | `tail -n 20 app.log` |
| `tail -f` | Follow new content as it is written | `tail -f app.log` |
| `nl` | Display lines with line numbers | `nl -ba notes.txt` |

**Professional tip:** Use `less` for large files. It allows you to navigate through content without printing the entire file to your terminal.

```bash
less /var/log/syslog
```

Your system might use a different logging arrangement. On systemd-based systems, you can also inspect the system journal with:

```bash
journalctl
```

Useful shortcuts in `less` include:

- `Space`: move forward one screen.
- `b`: move backward one screen.
- `/error`: search for `error`.
- `n`: move to the next match.
- `q`: quit.

To follow a growing log file:

```bash
tail -f app.log
```

Press `Ctrl+C` to stop following it.

## 5. Create and edit files

### 5.1 Use `touch` to create an empty file

```bash
touch notes.txt
```

If the file already exists, `touch` normally updates its timestamps instead of clearing its contents.

### 5.2 Edit with Nano

Nano is beginner-friendly and useful for quick configuration changes.

```bash
nano notes.txt
```

Common shortcuts:

| Shortcut | Action |
|---|---|
| `Ctrl+O` | Write (save) the file |
| `Enter` | Confirm the filename when saving |
| `Ctrl+X` | Exit |
| `Ctrl+W` | Search |
| `Ctrl+K` | Cut the current line |
| `Ctrl+U` | Paste the cut text |

The exact interface can vary slightly by version.

### 5.3 Edit with Vim

Vim is widely used in server administration because it is available in many environments and supports efficient editing.

```bash
vim notes.txt
```

The essential operations are:

| Operation | Command |
|---|---|
| Enter insert mode | `i` |
| Return to normal mode | `Esc` |
| Save | `:w` |
| Quit | `:q` |
| Save and quit | `:wq` |
| Quit without saving changes | `:q!` |
| Search for a word | `/word` |

**Important:** Vim has different modes. If you are unsure what mode you're in, press `Esc` before entering a command such as `:wq`.

## 6. Understand Linux redirection

This is one of the most important concepts for professional Linux work.

### 6.1 Write output into a file

```bash
echo "Hello Linux" > notes.txt
```

The `>` operator redirects standard output into the file, replacing its previous contents.

**Be careful:** If `notes.txt` already contains important data, this command overwrites it.

### 6.2 Append instead of overwriting

```bash
echo "Second line" >> notes.txt
```

The `>>` operator appends output to the end of the file.

This is useful for logs, reports and simple records.

### 6.3 Redirect errors separately

Linux programs commonly use three standard streams:

- Standard input: file descriptor `0`.
- Standard output: file descriptor `1`.
- Standard error: file descriptor `2`.

For example:

```bash
find / -name "notes.txt" > results.txt 2> errors.txt
```

This attempts to search from the filesystem root, writing normal results to `results.txt` and error messages to `errors.txt`.

Some directories will normally produce permission errors unless you have sufficient privileges.

To combine standard output and standard error into one file:

```bash
find / -name "notes.txt" > search.log 2>&1
```

This redirects standard output first, then redirects standard error to the same destination.

## 7. Edit files using command-line text processors

Interactive editors are not always the best choice. Sometimes you need to perform a repeatable change across many files.

### 7.1 Use `sed` for text replacement

Suppose `config.txt` contains:

```text
environment=development
```

You want to replace `development` with `production`.

First, preview the change:

```bash
sed 's/development/production/g' config.txt
```

This prints the modified text to the terminal without changing the original file.

To save the output to another file:

```bash
sed 's/development/production/g' config.txt > config.new
```

Review the new file before replacing the original.

### 7.2 Use `tee` to write and display output

```bash
echo "Deployment started" | tee deployment.log
```

This displays the text and writes it to `deployment.log`.

To append instead:

```bash
echo "Deployment completed" | tee -a deployment.log
```

`tee` is especially useful when you want to observe output while recording it.

### 7.3 Edit a privileged configuration file safely

Suppose you need to edit a system configuration file.

A safer workflow is:

1. Inspect the existing file.
2. Back it up.
3. Edit it with an appropriate privileged editor.
4. Validate the configuration.
5. Apply the change.
6. Verify that the service still works.

For example, to back up the SSH server configuration before editing:

```bash
sudo cp -a /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
```

Then edit it:

```bash
sudoedit /etc/ssh/sshd_config
```

Before reloading the SSH service, validate the configuration:

```bash
sudo sshd -t
```

If validation succeeds, you can reload the service using the appropriate service manager and service name for your system.

**Critical safety tip:** When changing SSH settings on a remote server, keep your existing session open until you have verified that a new connection works. An incorrect configuration can lock you out.

---

# Module 3: Linux File Permissions

## 8. Why do Linux permissions matter?

Imagine you are managing a production server with three types of users:

- A system administrator who manages the server.
- A developer who updates application code.
- A web server process that reads application files and writes logs.

Should every user have permission to modify every file?

Absolutely not.

Linux permissions help control who can read, modify and execute files or access directories. Proper permission management is fundamental to security, system administration and DevOps.

## 9. Understand read, write and execute

Linux uses three basic permission types:

| Permission | Symbol | For a regular file | For a directory |
|---|---|---|---|
| Read | `r` | Read the contents | List directory entries |
| Write | `w` | Modify the contents | Create, delete or rename entries, subject to other restrictions |
| Execute | `x` | Execute the file if it is executable | Traverse or access entries through the directory |

The distinction between file and directory permissions is important.

For example, a user might have permission to read a file but lack permission to access it because one of its parent directories does not allow traversal.

Similarly, deleting a file usually depends on permissions on its containing directory, not merely on whether the file itself is writable.

## 10. Understand the output of `ls -l`

Run:

```bash
ls -l
```

You might see:

```text
-rw-r--r-- 1 hasan developers 1200 Oct 24 18:30 notes.txt
```

Let's break this down:

```text
- rw- r-- r--
│  │   │   │
│  │   │   └── Permissions for others
│  │   └────── Permissions for the group
│  └────────── Permissions for the owner
└───────────── File type
```

The first character indicates the file type:

- `-`: regular file.
- `d`: directory.
- `l`: symbolic link.

The next nine characters represent the permissions for the owner, group and others.

In our example:

- Owner: `rw-` — can read and write.
- Group: `r--` — can read.
- Others: `r--` — can read.

The output also shows ownership, file size, modification time and filename.

## 11. Change permissions with `chmod`

### 11.1 Symbolic permission notation

Suppose you have a private script:

```bash
chmod u+x script.sh
```

This adds execute permission for the owner.

The symbolic notation uses:

| Symbol | Meaning |
|---|---|
| `u` | Owner |
| `g` | Group |
| `o` | Others |
| `a` | All three categories |
| `+` | Add permission |
| `-` | Remove permission |
| `=` | Set the specified permissions |

Examples:

```bash
chmod u+w notes.txt
```

Add write permission for the owner.

```bash
chmod g-w notes.txt
```

Remove write permission from the group.

```bash
chmod o-r notes.txt
```

Remove read permission from others.

```bash
chmod a+r notes.txt
```

Add read permission for everyone.

**Professional tip:** When changing permissions, prefer the smallest necessary change rather than granting broad permissions without a clear reason.

### 11.2 Numeric permission notation

Linux also supports numeric permissions.

Each permission has a value:

- Read = `4`
- Write = `2`
- Execute = `1`

Add the values to calculate the desired permission for each category.

| Number | Permission | Calculation |
|---:|---|---|
| `7` | `rwx` | 4 + 2 + 1 |
| `6` | `rw-` | 4 + 2 |
| `5` | `r-x` | 4 + 1 |
| `4` | `r--` | 4 |
| `0` | `---` | 0 |

Now consider these examples.

**Example 1: `chmod 644 notes.txt`**

```text
Owner: rw-
Group: r--
Others: r--
```

This is a common setting for non-sensitive, readable text files.

**Example 2: `chmod 600 credentials.txt`**

```text
Owner: rw-
Group: ---
Others: ---
```

This restricts ordinary permission-based access to the owner.

**Example 3: `chmod 700 private_script.sh`**

```text
Owner: rwx
Group: ---
Others: ---
```

This gives the owner full permissions while denying group and others these permissions.

**Example 4: `chmod 755 script.sh`**

```text
Owner: rwx
Group: r-x
Others: r-x
```

This is common for executable scripts and programs that are intended to be accessible to other users.

Remember that `chmod` does not automatically make every file safe. Access-control lists, parent-directory permissions, mount options, security policies and the executing process's privileges can also matter.

## 12. Apply permissions recursively

To apply permissions to a directory and its contents:

```bash
chmod -R 755 project/
```

However, this is often inappropriate for a mixed directory containing both scripts and data files. It can give execute permission to ordinary files and may expose sensitive content.

A more deliberate approach is:

```bash
find project/ -type d -exec chmod 755 {} +
```

This sets directory permissions to `755`.

For regular files:

```bash
find project/ -type f -exec chmod 644 {} +
```

This sets regular-file permissions to `644`.

**Warning:** These examples are suitable only when those permissions fit your security requirements. They are not universal defaults for every project, and applying them recursively can disrupt existing access controls.

## 13. Change ownership with `chown`

Imagine that a deployment process has created files owned by `root`, but your application needs to read files owned by a dedicated application account.

You can change ownership using:

```bash
sudo chown appuser notes.txt
```

Change both owner and group:

```bash
sudo chown appuser:appgroup notes.txt
```

Change group ownership only:

```bash
sudo chgrp developers notes.txt
```

To apply ownership recursively:

```bash
sudo chown -R appuser:appgroup project/
```

Use recursive ownership changes carefully, particularly under system directories.

**Professional security rule:** Do not solve every permission error by running the application as root. Identify the required access and grant only that access.

## 14. Understand `umask`

The `umask` setting controls which permission bits are removed from the default permissions when new files and directories are created.

Inspect the current value:

```bash
umask
```

For example, a common setting is:

```text
0022
```

For ordinary file creation, the usual starting permission mode is `666`; for directories, it is `777`. The umask removes applicable permission bits.

With `umask 022`:

- New regular files typically receive `644`.
- New directories typically receive `755`.

With `umask 077`:

- New regular files typically receive `600`.
- New directories typically receive `700`.

The actual permissions can also be affected by the application creating the file and the filesystem.

To set a restrictive umask for your current shell:

```bash
umask 077
```

This is particularly useful when working with private files, credentials and temporary data.

## 15. Advanced permissions: ACLs and special bits

### 15.1 Access Control Lists (ACLs)

Suppose a file belongs to `hasan`, but you also want to grant read access to a specific user named `developer1` without changing the file's owning group.

Where ACL support is available:

```bash
setfacl -m u:developer1:r notes.txt
```

Inspect the ACL:

```bash
getfacl notes.txt
```

Remove that user's ACL entry:

```bash
setfacl -x u:developer1 notes.txt
```

ACLs allow more granular access control than the basic owner-group-others permission model.

Be aware that an ACL's effective permissions may be limited by its ACL mask.

### 15.2 Special permission bits

Three additional permission mechanisms are worth learning.

| Bit | Numeric value | Purpose |
|---|---:|---|
| setuid | `4000` | On applicable executables, runs the program with the file owner's effective user ID |
| setgid | `2000` | On applicable executables, runs with the file's group ID; on directories, commonly causes new entries to inherit the directory's group |
| Sticky bit | `1000` | On directories, restricts who can remove or rename entries |

For example, the sticky bit is commonly used on shared temporary directories such as `/tmp`.

A directory with mode `1777` is writable and traversable by everyone, but the sticky bit restricts deletion or renaming of entries to the file owner, directory owner or a privileged process, subject to system rules.

Never add setuid or setgid bits casually. Incorrect use can create serious security vulnerabilities.

---

# Module 4: Finding Files and Searching Their Contents

## 16. Choose the right search tool

Linux provides several commands for locating files, identifying programs and searching text.

| Command | Purpose | Example |
|---|---|---|
| `find` | Search filesystem entries using criteria | `find . -name "*.txt"` |
| `locate` | Search an indexed database of file paths | `locate notes.txt` |
| `which` | Find an executable through `PATH` | `which python3` |
| `command -v` | Identify how the shell resolves a command | `command -v python3` |
| `whereis` | Locate certain binary, source and manual-file paths | `whereis python3` |
| `grep` | Search file contents for text patterns | `grep "error" app.log` |

The key distinction is that **`find` searches filesystem entries, while `grep` searches file contents**.

## 17. Find files by name

Search the current directory and its subdirectories:

```bash
find . -name "notes.txt"
```

Find files ending in `.log`:

```bash
find . -type f -name "*.log"
```

Find names without case sensitivity:

```bash
find . -iname "notes.txt"
```

Find directories named `backup`:

```bash
find . -type d -name "backup"
```

Useful options:

- `.`: start from the current directory.
- `/var/log`: start from a specified directory.
- `-type f`: regular files.
- `-type d`: directories.
- `-name`: case-sensitive filename pattern.
- `-iname`: case-insensitive filename pattern.

## 18. Find files by size, age and ownership

Find regular files larger than 100 MB:

```bash
find . -type f -size +100M -print
```

Find files modified within the last day:

```bash
find . -type f -mtime -1 -print
```

Find files belonging to a particular user:

```bash
find . -user hasan -print
```

Find empty files:

```bash
find . -type f -empty -print
```

These searches are useful for troubleshooting disk usage, locating old files and investigating ownership problems.

**Important distinction:** `-mtime` measures modification time in whole-day units with `find`'s rounding rules. It does not mean "exactly the last 24 hours" in every use case. For more precise time windows, consider `-mmin` or explicit timestamp comparisons.

## 19. Search inside files using `grep`

Search for the word `error` in a log:

```bash
grep "error" app.log
```

Ignore case:

```bash
grep -i "error" app.log
```

Show line numbers:

```bash
grep -n "error" app.log
```

Search recursively:

```bash
grep -Rni "error" ./logs/
```

The last command searches files under `./logs/`, ignoring case and displaying line numbers.

To show context around a match:

```bash
grep -C 3 "error" app.log
```

This displays three lines before and after each matching line.

### Real-world example: find SSH authentication errors

If the relevant system log is available:

```bash
sudo grep -i "failed password" /var/log/auth.log
```

Some distributions use different log paths or rely on the system journal instead. On systems using systemd, this may be useful:

```bash
sudo journalctl -u ssh
```

The service might be named `sshd` instead of `ssh`, depending on the distribution.

### Search for files and then search their contents

Suppose you want to find Python files containing `DATABASE_URL`:

```bash
find . -type f -name "*.py" -exec grep -nH "DATABASE_URL" {} +
```

This searches Python files under the current directory and displays matching line numbers and filenames.

**Professional tip:** Avoid indiscriminate recursive searches through huge filesystems. Start with the narrowest useful directory, such as your project folder.

---

# Module 5: Copying Files

## 20. Learn the different ways to copy files

Copying is a fundamental operation in Linux administration. You'll use it for backups, application deployments, configuration changes, transferring files and preparing recovery procedures.

The main commands are:

| Command | Purpose |
|---|---|
| `cp` | Copy files and directories |
| `rsync` | Synchronise files and directory trees |
| `dd` | Copy and convert raw data at a lower level |
| `install` | Copy files while setting specified permissions and ownership, where supported |

For ordinary file management, start with `cp` and `rsync`. The `dd` command is powerful but can destroy data if you specify the wrong input or output device.

## 21. Copy a single file

Suppose your directory contains:

```text
project/
├── report.txt
└── backup/
```

Copy the file into the backup directory:

```bash
cp report.txt backup/
```

The resulting structure is:

```text
project/
├── report.txt
└── backup/
    └── report.txt
```

Copy the file under a different name:

```bash
cp report.txt backup/report_copy.txt
```

Copy multiple files into one directory:

```bash
cp report.txt notes.txt backup/
```

This assumes the source files exist and `backup/` is an appropriate destination directory.

### 21.1 Avoid accidental overwrites

By default, `cp` can overwrite an existing destination file.

For an interactive prompt before overwriting:

```bash
cp -i report.txt backup/
```

You can also use:

```bash
cp -n report.txt backup/
```

The `-n` option requests that existing destination files not be overwritten. Behaviour can vary with the implementation and version, so check your system's manual if you rely on it in automation.

Another option is:

```bash
cp -v report.txt backup/
```

The `-v` option displays the operation.

**Professional tip:** `-v` is useful for visibility, but it is not a safety mechanism by itself.

## 22. Copy directories recursively

To copy a directory and its contents:

```bash
cp -r project/ project_backup/
```

This creates a copy of the directory tree.

When preserving metadata matters, consider:

```bash
cp -a project/ project_backup/
```

The archive option attempts to preserve important attributes, including permissions, timestamps, symbolic links and ownership, subject to system permissions and filesystem capabilities.

For a system administrator, the distinction between `cp -r` and `cp -a` is important.

- `cp -r` is suitable for many straightforward directory copies.
- `cp -a` is often preferable for backups or copying directory trees where metadata matters.

However, neither command automatically creates a complete, application-consistent backup of a live database or running service.

## 23. Copy files with `rsync`

`rsync` is widely used for efficient file transfers and directory synchronisation.

For example:

```bash
rsync -av project/ project_backup/
```

This synchronises the contents of `project/` into `project_backup/`.

The options mean:

- `-a`: archive mode, preserving a range of attributes and recursively processing directories.
- `-v`: verbose output.

### 23.1 Understand the trailing slash

Compare:

```bash
rsync -av project/ project_backup/
```

with:

```bash
rsync -av project project_backup/
```

The first copies the *contents* of `project/` into `project_backup/`.

The second copies the `project` directory itself into the destination, typically resulting in `project_backup/project/`.

Understanding the trailing slash prevents a common source of deployment and backup mistakes.

### 23.2 Preview an `rsync` operation

Before synchronising important data:

```bash
rsync -av --dry-run project/ project_backup/
```

The `--dry-run` option previews the intended operation without making the normal file changes.

This is particularly useful when deploying applications or synchronising large directory trees.

### 23.3 Transfer files to another machine

For example:

```bash
rsync -av project/ user@server:/srv/project/
```

This sends the local directory's contents to a remote machine using a supported remote-shell transport, commonly SSH.

You need suitable network access, authentication and permissions on the destination.

**Critical warning:** Adding `--delete` changes the risk profile:

```bash
rsync -av --delete source/ destination/
```

This can remove files from `destination/` that are absent from `source/`.

Always preview destructive synchronisation operations with `--dry-run`, and confirm that the source and destination are correct.

---

# Module 6: Moving and Renaming Files

## 24. Understand the `mv` command

The `mv` command moves files and directories. It is also used to rename them.

### 24.1 Rename a file

```bash
mv old_name.txt new_name.txt
```

This changes the filename from `old_name.txt` to `new_name.txt`.

### 24.2 Move a file to another directory

```bash
mv report.txt documents/
```

The file is moved into `documents/`.

### 24.3 Move and rename simultaneously

```bash
mv report.txt documents/final_report.txt
```

This moves the file and gives it a new name.

### 24.4 Move multiple files

```bash
mv report.txt notes.txt documents/
```

This moves both files into the destination directory.

### 24.5 Protect against accidental overwrites

Use:

```bash
mv -i report.txt documents/
```

The command prompts before overwriting an existing destination file.

You can also use:

```bash
mv -n report.txt documents/
```

This requests that existing destination files not be overwritten, subject to the implementation's behaviour.

For a more explicit warning, use:

```bash
mv -v report.txt documents/
```

Again, verbose output is for visibility, not protection.

## 25. What happens when you move a file between filesystems?

This is a useful advanced concept.

When the source and destination are on the same filesystem, a move can usually be completed by changing directory entries. The file's data does not need to be copied in the ordinary way.

When the source and destination are on different filesystems, the operation generally requires copying the data and then removing the original.

This difference affects performance and failure handling.

For example, moving a large file between two mounted drives can take considerably longer than renaming it within one directory.

### Professional tip: avoid assuming every move is atomic

A same-filesystem rename can generally provide atomic visibility of the name change, subject to the operation and filesystem conditions. A cross-filesystem move is not a single atomic rename.

If you're deploying production configuration files, a common safer pattern is to:

1. Prepare the replacement file separately.
2. Validate its contents.
3. Place it on the same filesystem as the target.
4. Set the correct ownership and permissions.
5. Rename it into place.
6. Verify the service.

For a replacement such as a configuration file, the final rename may be atomic when the relevant filesystem and operation support it.

This technique helps reduce the risk of applications reading a partially written replacement file.

---

# Module 7: Linking Files

## 26. What is a link in Linux?

A link gives you another way to refer to filesystem content.

Linux supports two major types:

1. Hard links.
2. Symbolic links, commonly called symlinks.

They may look similar when you list files, but they behave differently.

## 27. Hard links

A hard link is another directory entry referring to the same filesystem object, generally identified by the same inode.

Create one with:

```bash
ln original.txt hardlink.txt
```

Now inspect both files:

```bash
ls -li original.txt hardlink.txt
```

On a typical Linux filesystem, both entries will display the same inode number.

### 27.1 What happens when you modify a hard link?

Suppose `original.txt` contains:

```text
Hello Linux
```

You append text through the hard link:

```bash
echo "Hard link example" >> hardlink.txt
```

Reading `original.txt` will show the appended text too.

Why?

Because the two directory entries refer to the same underlying file object.

### 27.2 What happens when you delete the original?

```bash
rm original.txt
```

The data remains accessible through `hardlink.txt`, provided that the file is still linked and accessible.

The content is not removed simply because one of its names has been deleted.

### 27.3 Hard-link limitations

Hard links generally:

- Cannot cross filesystem boundaries.
- Cannot normally be created for directories by ordinary users.
- Refer to the same underlying inode rather than a separate copy.
- Can make storage organisation and deletion behaviour less obvious if you're unfamiliar with them.

**Real-world use:** Hard links can be useful in specialised storage layouts and some backup implementations. They are less common than symbolic links in everyday application configuration.

## 28. Symbolic links

A symbolic link stores a pathname that points to another file or directory.

Create one:

```bash
ln -s original.txt shortcut.txt
```

Inspect it:

```bash
ls -l shortcut.txt
```

You should see that `shortcut.txt` points to `original.txt`.

### 28.1 Use symbolic links for directories

Suppose your application is stored at:

```text
/opt/myapp/releases/v2/
```

You want a convenient path that refers to the current release.

You could create a symlink:

```bash
ln -s /opt/myapp/releases/v2 /opt/myapp/current
```

Applications and scripts can then refer to:

```text
/opt/myapp/current/
```

rather than hard-coding the versioned release directory everywhere.

This approach is widely used in deployment layouts, although the exact deployment process must account for service behaviour and permissions.

### 28.2 What happens if the target is deleted?

If you remove `original.txt`:

```bash
rm original.txt
```

the symbolic link `shortcut.txt` normally remains, but it becomes a dangling symlink because its target no longer exists.

Check the link with:

```bash
ls -l shortcut.txt
```

To inspect the target path:

```bash
readlink shortcut.txt
```

To check whether the target currently resolves to an existing file or directory:

```bash
test -e shortcut.txt && echo "Target exists" || echo "Target missing"
```

Note that `test -e` follows the link. It returns false for a dangling symlink.

### 28.3 Absolute versus relative symbolic links

An absolute link specifies a full path:

```bash
ln -s /opt/myapp/config/settings.conf settings.conf
```

A relative link uses a path relative to the directory containing the link:

```bash
ln -s ../shared/settings.conf settings.conf
```

Relative links can be useful when an entire directory tree needs to be relocated. However, they can break if the relative directory structure changes.

### 28.4 Replace a symbolic link safely

Suppose `/opt/myapp/current` points to an old release and you want to point it to a new one.

For a controlled deployment, you can create a new link under a temporary name and then rename it into place on the same filesystem:

```bash
ln -s /opt/myapp/releases/v3 /opt/myapp/current.new
mv -Tf /opt/myapp/current.new /opt/myapp/current
```

**Use this carefully:** On GNU/Linux, `-T` treats the destination as a normal path rather than a directory. The replacement behaviour depends on the exact paths, filesystem and `mv` implementation. Ensure the destination is the intended symlink, not a real directory, and validate the procedure in a test environment first.

For production systems, you should also understand how the application handles existing file descriptors, cached paths and service reloads. Changing a symlink does not necessarily cause a running process to start using the new release immediately.

## 29. Hard links versus symbolic links

| Feature | Hard link | Symbolic link |
|---|---|---|
| Refers to | The same filesystem object | A pathname |
| Inode | Same inode as the linked file | Has its own inode |
| Can point to a directory | Generally prohibited for ordinary users | Yes |
| Can cross filesystems | No | Yes |
| Survives deletion of the original name | Yes, if another hard link remains | The link remains, but may become dangling |
| Can point to a nonexistent target | No | Yes |
| Common use | Specialised storage and backup layouts | Application paths, shared resources and deployment layouts |

**The simplest way to remember the difference:** A hard link is another name for the same filesystem object; a symbolic link is a separate filesystem object that refers to a pathname.

---

# Module 8: Professional Linux Tips and Tricks

Hasan, the previous seven modules taught you the commands. Now let's focus on the techniques that distinguish someone who knows Linux commands from someone who can safely manage Linux systems.

## 30. Use a safe workflow before changing files

Develop this habit whenever you work with important files:

**Inspect → Verify → Back up → Execute → Validate**

For example, suppose you need to modify a configuration file.

```bash
# 1. Check your current location
pwd

# 2. Inspect the file
ls -l config.conf
cat config.conf

# 3. Create a backup, preserving metadata
cp -a config.conf config.conf.bak

# 4. Edit the file
nano config.conf

# 5. Compare the original and modified versions
diff -u config.conf.bak config.conf
```

This workflow lets you review the changes before deciding whether to apply them.

The exact validation step depends on the file. For a service configuration, use the service's configuration-validation command before restarting it.

**Professional principle:** Never assume a command succeeded just because you executed it.

## 31. Use quoting to prevent filename errors

Consider this filename:

```text
quarterly financial report.txt
```

This command is incorrect for that filename:

```bash
cp quarterly financial report.txt backup/
```

The shell interprets the spaces as argument separators.

Instead, use:

```bash
cp "quarterly financial report.txt" backup/
```

For a filename stored in a shell variable:

```bash
filename="quarterly financial report.txt"
cp -- "$filename" backup/
```

The quotation marks preserve the filename as a single argument.

The `--` protects against filenames beginning with a hyphen when the command supports that convention.

**Rule:** When working with filenames stored in variables, quote the variables unless you have a specific reason not to.

## 32. Use tab completion instead of typing everything

Suppose you want to work with:

```text
application_configuration_backup.txt
```

You can start typing:

```bash
cat application_
```

Then press `Tab`.

Your shell may complete the filename automatically. If several filenames match, pressing `Tab` again may display the available options.

Tab completion reduces typing and helps prevent spelling errors.

## 33. Understand the difference between `>` and `>>`

These two operators look similar but behave differently.

```bash
echo "First line" > notes.txt
```

This replaces the existing contents of `notes.txt`.

```bash
echo "Second line" >> notes.txt
```

This appends to the file.

Consider this dangerous command:

```bash
command > important_file.txt
```

Even if `command` fails, the shell normally opens and truncates the output file before running the command.

If the file contains valuable data, that data can be lost.

For important operations, write output to a new file and inspect it before replacing the original.

## 34. Understand `sudo` and redirection

A common Linux mistake is:

```bash
sudo echo "new setting" > /etc/example.conf
```

This may fail with a permission error.

Why?

The shell processes `>` before executing `sudo`. The shell itself attempts to open `/etc/example.conf`, and your ordinary user may not have permission to write there.

One alternative is:

```bash
echo "new setting" | sudo tee /etc/example.conf
```

This allows `tee`, running with elevated privileges, to write the file.

However, this replaces the file's contents. For an existing configuration file, a privileged editor or a carefully validated temporary-file workflow is usually safer.

To append instead:

```bash
echo "additional setting" | sudo tee -a /etc/example.conf
```

Be cautious when appending configuration directives: duplicate or conflicting settings can change how a service behaves.

## 35. Check your commands' exit status

Linux commands usually communicate success or failure through an exit status.

Immediately after a command, run:

```bash
echo $?
```

A status of `0` normally indicates success. A non-zero status indicates some kind of failure or other condition.

For example:

```bash
cp report.txt backup/
echo $?
```

If the copy succeeded, the exit status will ordinarily be `0`.

**Important:** Check `$?` immediately. Running another command first will generally replace the previous command's exit status.

In shell scripts, you can also use conditional execution:

```bash
cp report.txt backup/ && echo "Copy completed successfully"
```

The message is printed only if the copy command succeeds.

For more advanced scripts, learn proper error handling, traps and reliable status propagation.

## 36. Use `stat` to inspect file metadata

The `stat` command displays information about a file or directory.

```bash
stat report.txt
```

Depending on your system, the output includes:

- File size.
- Permissions.
- Owner and group.
- Inode number.
- Access time.
- Modification time.
- Metadata-change time.

These timestamps have different meanings:

| Timestamp | Meaning |
|---|---|
| Access time (`atime`) | When file content was last accessed, subject to filesystem settings |
| Modification time (`mtime`) | When file contents were last modified |
| Change time (`ctime`) | When filesystem metadata last changed |

**Important:** `ctime` does not mean creation time. Creation or birth time is a separate attribute and is not available on every filesystem or through every tool.

These distinctions matter when investigating changes, debugging file synchronisation and auditing filesystem activity.

---

# Module 9: Real-World Practice Lab

Let's combine the concepts into one realistic exercise.

## Scenario: Manage a small application directory

You have a directory containing application files, configuration files and logs.

Your task is to organise the project, create a backup, adjust permissions, locate errors and prepare the application directory for deployment.

### Step 1: Create the practice environment

Run these commands in your home directory:

```bash
mkdir -p ~/linux-lab/project/logs
mkdir -p ~/linux-lab/backups

cd ~/linux-lab/project

touch app.py config.txt logs/app.log

echo "environment=development" > config.txt
echo "Application started" > logs/app.log
echo "ERROR: Database connection failed" >> logs/app.log
```

Verify the structure:

```bash
find ~/linux-lab -maxdepth 3 -print
```

You should see the project directory, its files and the backup directory.

### Step 2: Inspect the files

```bash
pwd
ls -lah
ls -lah logs/
stat config.txt
```

Your objective is to identify the current directory, inspect the files and understand their metadata.

### Step 3: Back up the project

From the project directory, run:

```bash
cp -a . ~/linux-lab/backups/project-backup/
```

Inspect the backup:

```bash
find ~/linux-lab/backups/project-backup -maxdepth 3 -print
```

This copies the contents of the current directory, including hidden entries, into the backup directory.

**Real-world consideration:** A simple filesystem copy is not necessarily a consistent backup if applications are actively modifying files. Production databases and services may require application-aware backup procedures.

### Step 4: Change file permissions

Make the configuration file readable and writable only by its owner:

```bash
chmod 600 config.txt
```

Give the application script owner-only write access and allow the owner to execute it:

```bash
chmod 700 app.py
```

Verify the permissions:

```bash
ls -l config.txt app.py
```

Ask yourself:

1. Why is `600` appropriate for some private configuration files?
2. Why might `700` be inappropriate for a script that needs to be readable by a dedicated application group?
3. Does changing permissions make the files inaccessible to root or every privileged process?

The answer to the third question is no. Standard discretionary permissions are not an absolute barrier against privileged access.

### Step 5: Find the error

Search the log:

```bash
grep -n "ERROR" logs/app.log
```

Find all log files under the project:

```bash
find . -type f -name "*.log"
```

You have now used two different search methods: one for file contents and one for filesystem paths.

### Step 6: Create a symbolic link

From the project directory:

```bash
ln -s config.txt current-config
```

Inspect the link:

```bash
ls -l current-config
readlink current-config
```

Check whether the target exists:

```bash
test -e current-config && echo "Target exists"
```

Notice that the symbolic link refers to `config.txt`; it does not contain an independent copy of the configuration.

### Step 7: Move a log file

Create an archive directory:

```bash
mkdir -p ~/linux-lab/project/archive
```

Move the log file:

```bash
mv ~/linux-lab/project/logs/app.log \
   ~/linux-lab/project/archive/app.log
```

Verify the result:

```bash
find ~/linux-lab/project -type f -print
```

This demonstrates how file movement changes the directory structure.

### Step 8: Clean up safely

Before deleting anything, inspect the files in the lab:

```bash
find ~/linux-lab/project -maxdepth 4 -print
```

If you want to remove only the archived log file:

```bash
rm -i ~/linux-lab/project/archive/app.log
```

Confirm the prompt only after checking that the file is the intended target.

**Never run broad recursive deletion commands against a directory until you have verified its contents and path.**

---

# Module 10: Your Professional Linux Command Reference

Use this table for revision.

| Task | Command |
|---|---|
| Display current directory | `pwd` |
| List files with details | `ls -lah` |
| Create an empty file | `touch file.txt` |
| Create directories | `mkdir -p parent/child` |
| Read a file | `cat file.txt` |
| Browse a large file | `less file.txt` |
| Edit a file | `nano file.txt` |
| Delete a file | `rm file.txt` |
| Delete an empty directory | `rmdir directory/` |
| Remove a directory recursively | `rm -r directory/` |
| Find files by name | `find . -name "file.txt"` |
| Search file contents | `grep -n "word" file.txt` |
| Copy a file | `cp source.txt destination.txt` |
| Copy a directory | `cp -a source/ destination/` |
| Synchronise directories | `rsync -av source/ destination/` |
| Rename a file | `mv old.txt new.txt` |
| Add execute permission for owner | `chmod u+x script.sh` |
| Set owner-only file permissions | `chmod 600 file.txt` |
| Change file owner | `sudo chown user file.txt` |
| Inspect metadata | `stat file.txt` |
| Create a hard link | `ln original.txt hardlink.txt` |
| Create a symbolic link | `ln -s original.txt symlink.txt` |
| Inspect a symbolic link | `readlink symlink.txt` |
| Compare two files | `diff -u old.txt new.txt` |
| Check command success | `echo $?` |

Keep this table as a quick reference, but practise the commands rather than relying on memorisation alone.

---

# Module 11: Assessment — Test Your Understanding

Now I want you to think like a Linux engineer.

**Don't look up the answers yet.** Try each task in your terminal or explain which command you would use and why.

### Question 1: File deletion and safety

You need to delete all `.tmp` files inside `/home/hasan/project` and its subdirectories.

Before deleting anything, you must display the matching files.

**Your task:** Write the two commands: one to preview the matches and one to delete them.

### Question 2: File permissions

You have a file called `database.conf`.

The requirements are:

- The owner can read and modify it.
- Group members cannot access it through ordinary file permissions.
- Other users cannot access it through ordinary file permissions.

**Your task:** Which `chmod` command would you use? Explain the numeric calculation.

### Question 3: Copying and backups

You need to back up an application directory to another directory while preserving important file metadata.

**Your task:** Write the command you would use and explain why you chose it over a basic recursive copy.

### Question 4: Finding and searching

You need to find every `.log` file under the current directory and identify which ones contain the word `ERROR`, ignoring case.

**Your task:** Write a command or a short sequence of commands that achieves this.

### Question 5: Hard links versus symbolic links

You create a hard link and a symbolic link to the same file.

Then you delete the original filename.

**Your task:** Explain what happens to each link and why.

---

## How I'll evaluate your answers

I'll assess each answer based on:

- **Technical accuracy:** Does the command achieve the intended result?
- **Safety:** Have you considered data loss and unintended side effects?
- **Understanding:** Can you explain why the command works?
- **Professional practice:** Would your approach be appropriate on a real server?

I'll give you feedback, correct any misconceptions and identify which concepts you should practise further.

**Your next step:** Answer Questions 1–5 in your own words. You don't need to get everything right. The objective is to develop the judgment required to use Linux confidently and safely in real-world environments.
