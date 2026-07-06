As a DevOps engineer, almost everything you touch is a file:

* Nginx configuration
* Kubernetes manifests
* Dockerfiles
* Terraform state
* SSH keys
* Certificates
* Logs
* Systemd unit files
* Environment files

Understanding how Linux manages files makes troubleshooting much easier.

---

# 1. stat

## What is this?

`stat` displays detailed information (metadata) about a file or directory.

Example:

```bash
stat nginx.conf
```

Output

```
File: nginx.conf
Size: 2048
Access: rw-r--r--
Uid: root
Gid: root
Access Time
Modify Time
Change Time
Inode: 563291
```

---

## Why is it required?

Sometimes two files have

* same name
* same content

But are actually different files.

Sometimes permissions changed.

Sometimes owner changed.

Sometimes timestamp matters.

`stat` shows all this information.

---

## DevOps use

Suppose

```
/etc/nginx/nginx.conf
```

is not loading.

You run

```bash
stat /etc/nginx/nginx.conf
```

Immediately you see

```
Owner: ubuntu
```

instead of

```
root
```

You found the issue.

---

## Why recruiters care

They want someone who investigates instead of guessing.

Production engineers inspect metadata constantly.

---

## Production problem

Example

```
SSL certificate renewed

But nginx still serves old certificate.
```

You compare

```bash
stat old.crt
stat new.crt
```

Modification time tells you nginx is still using old file.

---

## Troubleshooting

```
Service cannot read config.
```

Check

```
Owner
Permissions
Size
Modification time
```

All from stat.

---

## Dependencies

Affects

* systemd
* nginx
* apache
* docker
* ssh
* kubernetes secrets

Because they all read files.

---

## Analogy

Imagine a passport.

The passport is not the person.

It contains information about the person.

Similarly

`stat` doesn't show file content.

It shows file information.

---

## Test yourself

1. Does stat show file content?
2. Why is modification time useful?
3. When would owner information help?

---

# 2. file

## What is this?

Determines actual file type.

Example

```bash
file backup
```

Output

```
gzip compressed data
```

Even if there is no extension.

---

## Why required?

Linux doesn't trust extensions.

Windows:

```
photo.jpg
```

Linux checks actual content.

---

## DevOps use

Downloaded backup.

Named

```
backup.sql
```

Actually

```
backup.sql.gz
```

You know instantly.

---

## Recruiters care

Many production issues happen because people assume file type.

---

## Production problem

Application says

```
invalid certificate
```

Run

```bash
file server.crt
```

Output

```
ASCII text
```

Oops.

Someone copied wrong file.

---

## Troubleshooting

```
Docker image won't build.
```

Run

```bash
file script.sh
```

Output

```
CRLF line endings
```

Now you know Windows created it.

---

## Dependencies

* Docker
* Bash
* SSL
* Backup systems

---

## Analogy

Looking at someone's ID card instead of guessing from clothes.

---

## Test

1. Does Linux rely on extension?
2. Why use file instead of ls?
3. Can file identify binary?

---

# 3. basename

## What is this?

Returns filename only.

```
/etc/nginx/nginx.conf
```

↓

```
nginx.conf
```

Example

```bash
basename /etc/nginx/nginx.conf
```

---

## Why required?

Useful inside shell scripts.

---

## DevOps use

Loop through many files.

Need only filenames.

---

## Recruiters care

Automation.

Every DevOps engineer writes scripts.

---

## Production

Generate backup names.

```
config.yaml

↓

config.yaml.backup
```

---

## Troubleshooting

Logging.

Instead of printing long path.

Print

```
config.yaml
```

---

## Dependencies

Mostly

* Bash
* Cron
* CI/CD

---

## Analogy

Your full address vs only your name.

---

## Test

1. What does basename remove?
2. Why useful in scripting?
3. Does basename touch the file?

---

# 4. dirname

## What is this?

Returns directory path.

```
/etc/nginx/nginx.conf
```

↓

```
/etc/nginx
```

---

## Why required?

Scripts often need parent folder.

---

## DevOps

Suppose

```
config.yaml
```

Need log folder beside it.

```
dirname
```

helps.

---

## Recruiters care

Portable scripts.

---

## Production

Deployment script.

Automatically create backup in same directory.

---

## Troubleshooting

Need to verify directory exists.

---

## Dependencies

Shell

Automation

CI/CD

---

## Analogy

Home address.

Basename = your name.

Dirname = your house.

---

## Test

1. Difference between dirname and basename?
2. Does dirname return filename?
3. Why useful in scripts?

---

# 5. realpath

## What is this?

Returns absolute canonical path.

Example

```
../config/app.yaml
```

↓

```
/home/sonu/project/config/app.yaml
```

---

## Why required?

Applications should know exact path.

---

## DevOps

Docker volume.

Need absolute path.

---

## Recruiters care

Avoid relative-path bugs.

---

## Production

Application started from another directory.

Relative path fails.

Use

```
realpath
```

---

## Troubleshooting

Find actual location of symbolic links.

---

## Dependencies

Docker

Terraform

Ansible

Kubernetes mounts

---

## Analogy

Instead of saying

"Near the market"

You provide complete GPS location.

---

## Test

1. Difference between pwd and realpath?
2. Why use absolute paths?
3. Does realpath resolve symlinks?

---

# 6. pwd

## What is this?

Shows current working directory.

```
pwd
```

Output

```
/home/ubuntu/project
```

---

## Why required?

Linux commands often depend on current directory.

---

## DevOps

Deployment script fails.

First command

```
pwd
```

Maybe running from wrong directory.

---

## Recruiters care

Many beginners execute commands from wrong location.

---

## Production

CI pipeline

```
terraform apply
```

fails.

Reason

Wrong directory.

---

## Troubleshooting

Verify

```
pwd
```

before destructive commands.

---

## Dependencies

Everything using relative paths.

---

## Analogy

Google Maps asking

"Your current location."

---

## Test

1. What does pwd show?
2. Why can wrong directory cause failures?
3. Difference between pwd and ls?

---

# Understanding Concepts

# 7. Inode

## What is this?

An **inode** is Linux's internal record for a file. It stores almost everything *about* the file except its name and contents.

An inode contains:

* File owner (UID/GID)
* Permissions
* File size
* Timestamps
* Number of hard links
* Pointers to the data blocks on disk

Every file has a unique inode number within its filesystem.

Example:

```bash
ls -i file.txt
```

```
231456 file.txt
```

---

## Why is it required?

Linux identifies files by inode, not by filename. A filename is simply a directory entry that points to an inode.

---

## DevOps use

Knowing inodes explains why deleting a file doesn't always free disk space immediately if a process still has it open.

---

## Why recruiters care

Many production issues—disk usage, hard links, log rotation—require understanding inodes.

---

## Production problem

A log file is deleted:

```bash
rm /var/log/app.log
```

But disk usage doesn't decrease because the application still has the inode open. Restarting the service releases it.

---

## Troubleshooting

Use:

```bash
lsof | grep deleted
```

to find deleted-but-open files consuming space.

---

## Dependencies

* Filesystems
* Logging systems
* Backup tools
* Log rotation

---

## Analogy

Think of an inode as a person's government ID number. People may change their name, but the government tracks them by their ID.

---

## Test

1. Does an inode store the filename?
2. Why doesn't deleting a file always free disk space?
3. How are hard links related to inodes?

---

# 8. Metadata

## What is this?

Metadata means "data about data." For a file, it includes information such as:

* Owner
* Permissions
* Size
* Timestamps
* Inode number

---

## Why is it required?

The operating system needs metadata to decide who can access the file, when it changed, and how to locate it.

---

## DevOps use

Permissions or ownership issues are metadata problems, not content problems.

---

## Why recruiters care

Production debugging often starts by inspecting metadata.

---

## Production problem

An application can't read its configuration because the file permissions are incorrect, even though the file content is valid.

---

## Troubleshooting

```bash
stat config.yaml
ls -l config.yaml
```

---

## Dependencies

All applications reading files rely on correct metadata.

---

## Analogy

Metadata is like the label on a shipping box: sender, receiver, weight, and date—not the items inside.

---

## Test

1. Is file content metadata?
2. Which command displays metadata?
3. Why are permissions considered metadata?

---

# 9. Hard Links

## What is this?

A hard link is another filename pointing to the **same inode**.

```bash
ln file.txt file2.txt
```

Both names reference the same underlying file.

---

## Why is it required?

It allows multiple directory entries to reference identical data without copying it.

---

## DevOps use

Useful in backups and package management to save disk space.

---

## Why recruiters care

Understanding hard links helps explain disk usage and filesystem behavior.

---

## Production problem

Editing `file.txt` also changes `file2.txt` because they're the same file.

---

## Troubleshooting

Compare inode numbers:

```bash
ls -li
```

Same inode means they're hard links.

---

## Dependencies

* Filesystems
* Backup software
* Package managers

---

## Analogy

Two different nicknames referring to the same person.

---

## Test

1. Do hard links share an inode?
2. What happens if one hard link is deleted?
3. Can hard links span different filesystems?

---

# 10. Soft Links (Symbolic Links)

## What is this?

A symbolic (soft) link is a special file that stores the path to another file.

```bash
ln -s /etc/nginx/nginx.conf nginx.conf
```

---

## Why is it required?

It provides flexible references across directories or even different filesystems.

---

## DevOps use

Commonly used for:

* Current application releases (`current -> releases/v12`)
* Configuration management
* Shared libraries

---

## Why recruiters care

Symbolic links are widely used in deployments and Linux administration.

---

## Production problem

A deployment updates the `current` symlink to point to a new release, allowing near-instant rollbacks.

---

## Troubleshooting

```bash
ls -l
```

or

```bash
realpath current
```

to verify where the link points.

---

## Dependencies

* Web servers
* Shared libraries
* Deployment tools
* Package managers

---

## Analogy

A shortcut on your desktop pointing to the real file.

---

## Test

1. Does a symbolic link have its own inode?
2. What happens if the target file is deleted?
3. Can symbolic links point across filesystems?

---

# 11. Hidden Files

## What is this?

Hidden files are files whose names begin with a dot (`.`).

Examples:

```text
.bashrc
.gitignore
.env
.ssh/
```

Linux doesn't treat them specially—they're simply hidden by convention in normal directory listings.

---

## Why is it required?

To keep configuration files separate from everyday working files.

---

## DevOps use

Many critical configurations are stored in hidden files:

* `.bashrc`
* `.profile`
* `.gitconfig`
* `.env`
* `.ssh`

---

## Why recruiters care

A DevOps engineer regularly edits and manages hidden configuration files.

---

## Production problem

A deployment fails because the `.env` file containing environment variables wasn't copied to the server.

---

## Troubleshooting

Use:

```bash
ls -la
```

to reveal hidden files and confirm required configuration exists.

---

## Dependencies

* Shell initialization
* Git
* SSH
* Docker Compose
* Many application frameworks

---

## Analogy

Hidden files are like the service panel behind a machine: users don't normally need it, but technicians rely on it.

---

## Test

1. How do you list hidden files?
2. Are hidden files encrypted or protected?
3. Why do Linux applications commonly use hidden configuration files?

---

# What a Mid-Level DevOps Engineer Should Remember

These topics form the foundation of Linux file management. In interviews, you should be able to explain not only **what each command or concept does**, but also **how it helps diagnose real production issues**. For example:

* `stat` → Inspect permissions, ownership, timestamps, and other metadata.
* `file` → Verify the actual file type instead of trusting the extension.
* `basename` / `dirname` → Write robust, reusable shell scripts.
* `realpath` / `pwd` → Eliminate path-related deployment and automation errors.
* **Inodes** → Understand file identity, hard links, and disk-space anomalies.
* **Metadata** → Diagnose permission, ownership, and timestamp issues.
* **Hard links** → Understand shared file identity and storage behavior.
* **Soft links** → Manage versioned deployments and configuration references.
* **Hidden files** → Locate and maintain critical application and user configuration.
