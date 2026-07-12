This is one of the **most important Linux topics** for DevOps. If Bash scripting is the language, then **pipes and redirection are how Linux programs communicate with each other.**

Almost every production script, CI/CD pipeline, troubleshooting session, log collection process, and monitoring tool uses these concepts.

---

# First understand the philosophy

Linux follows a simple philosophy:

> **Do one thing well.**

Instead of making one giant command, Linux provides many small commands.

Example:

* `ps` → shows processes
* `grep` → filters text
* `sort` → sorts
* `awk` → processes columns
* `wc` → counts

The magic is combining them.

Example:

```bash
ps aux | grep nginx | wc -l
```

This creates a workflow:

```
ps
 │
 ▼
grep
 │
 ▼
wc
```

This communication is possible because of **stdin, stdout, stderr and pipes.**

---

# Data Streams in Linux

Every Linux process automatically gets three data streams.

```
           Terminal

Keyboard ----------------------> stdin (0)

Program -----------------------> stdout (1)

Errors ------------------------> stderr (2)
```

Every program has these.

| Stream | File Descriptor | Purpose       |
| ------ | --------------- | ------------- |
| stdin  | 0               | Input         |
| stdout | 1               | Normal output |
| stderr | 2               | Errors        |

Everything else (pipes, redirection) is built on top of these.

---

# 1. stdin

## What is this?

Standard Input.

It is where a program receives input.

Usually:

```
Keyboard
```

Example

```bash
cat
```

Now type

```
Hello
```

Output

```
Hello
```

`cat` is reading from stdin.

---

## Why required?

Programs need input.

Without stdin, every program would need its own input method.

Linux standardized it.

---

## Mid-Level DevOps usage

Examples

```
mysql < backup.sql

kubectl apply -f -

docker login

openssl
```

Many tools accept stdin.

---

## Production problem solved

Instead of creating temporary files,

Bad

```
download
save
read
delete
```

Good

```
curl ... | jq
```

Everything happens in memory.

---

## Troubleshooting

Example

```bash
kubectl get pods | grep CrashLoop
```

grep reads stdin.

---

## Dependencies

Anything reading input

* grep
* sort
* awk
* sed
* jq
* mysql
* docker
* kubectl

---

## Analogy

stdin is like

**Restaurant waiter taking your order.**

You provide input.

---

## Test yourself

1.

What file descriptor is stdin?

Answer:

```
0
```

---

2.

Can stdin come from a file instead of keyboard?

Yes.

---

3.

Can stdin come from another program?

Yes using pipe.

---

# 2. stdout

## What is this?

Normal output.

Example

```bash
ls
```

prints to stdout.

---

## Why required?

Programs need a standard output destination.

---

## DevOps usage

Redirect output

```
terraform output > output.txt

kubectl get pods > pods.txt
```

---

## Production example

Daily backup

```
mysqldump > backup.sql
```

---

## Troubleshooting

Save logs

```
kubectl describe pod nginx > pod.log
```

---

## Dependencies

Almost every Linux program.

---

## Analogy

stdout is like

Teacher speaking to students.

---

## Questions

1.

stdout descriptor?

```
1
```

---

2.

Can stdout be redirected?

Yes.

---

3.

Does stdout include errors?

No.

---

# 3. stderr

## What is this?

Error output.

Separate stream.

Example

```
ls abc
```

```
No such file
```

This is stderr.

---

## Why required?

Errors shouldn't mix with normal output.

---

## DevOps usage

Save only errors

```
terraform apply 2> errors.log
```

---

## Production example

CI pipeline

```
Build logs

Error logs
```

stored separately.

---

## Troubleshooting

```
systemctl restart nginx 2> restart-errors.log
```

---

## Dependencies

Every CLI program.

---

## Analogy

stdout = Teacher teaching.

stderr = Principal announcing emergency.

Different channels.

---

## Questions

1.

stderr descriptor?

```
2
```

---

2.

Can stdout and stderr be redirected separately?

Yes.

---

3.

Why keep them separate?

Easy troubleshooting.

---

# 4. >

## What is this?

Redirect stdout to file.

Example

```bash
ls > files.txt
```

Instead of terminal

↓

file.

---

## Why required?

Save results.

---

## DevOps usage

```
kubectl get pods > report.txt
```

---

## Production problem

Nightly inventory.

---

## Troubleshooting

Save command output.

---

## Affects

stdout only.

---

## Analogy

Instead of speaking,

write into notebook.

---

## Questions

1.

Overwrite or append?

Overwrite.

---

2.

stderr redirected?

No.

---

3.

Can file exist?

Yes. It gets overwritten.

---

# 5. >>

## What is this?

Append stdout.

Example

```bash
date >> log.txt
```

---

## Difference

```
>
```

Replace

```
>>
```

Append

---

## Production

Logs.

---

## Troubleshooting

Collect outputs.

---

## Analogy

Diary.

Every day add new page.

---

## Questions

1.

Overwrite?

No.

---

2.

Creates file?

Yes.

---

3.

Used for logs?

Yes.

---

# 6. <

## What is this?

Redirect file to stdin.

Example

```bash
sort < numbers.txt
```

---

Instead of keyboard,

stdin comes from file.

---

## Production

```
mysql < backup.sql
```

Restore database.

---

## Troubleshooting

Replay stored inputs.

---

## Analogy

Instead of asking customer,

read written form.

---

## Questions

1.

stdin source?

File.

---

2.

Used for restore?

Yes.

---

3.

Alternative?

Many commands accept filename directly.

---

# 7. << (Here Document)

## What is this?

Provide multiline input.

Example

```bash
cat << EOF
Hello
Linux
EOF
```

Output

```
Hello
Linux
```

---

## Why required?

Avoid creating temporary files.

---

## DevOps usage

Create config.

```bash
cat <<EOF > config.yaml
apiVersion: v1
kind: Pod
EOF
```

---

Production

Generate YAML dynamically.

---

Troubleshooting

Quick scripts.

---

Analogy

Writing letter directly.

---

Questions

1.

Need temp file?

No.

---

2.

Ends with?

Delimiter.

---

3.

Useful in automation?

Very.

---

# 8. |

## What is this?

Pipe.

Output of one command

↓

input of another.

```
Command A

↓

Command B
```

---

Example

```bash
ps aux | grep nginx
```

---

## Why required?

Combine tools.

---

## DevOps

```
kubectl get pods

↓

grep

↓

awk

↓

sort
```

---

Production

Monitoring

Filtering

Automation

---

Troubleshooting

```
journalctl -u nginx | grep error
```

---

Dependencies

stdin/stdout.

---

Analogy

Factory conveyor belt.

---

Questions

1.

Pipe transfers?

stdout → stdin.

---

2.

stderr piped?

No.

---

3.

Can multiple pipes exist?

Unlimited.

---

# 9. tee

## What is this?

Displays output AND saves to file.

Example

```bash
ls | tee files.txt
```

Output

```
Screen

AND

files.txt
```

---

## Why required?

Normally

```
>
```

hides output.

tee shows both.

---

## DevOps

```
terraform apply | tee terraform.log
```

---

Production

Keep audit logs.

---

Troubleshooting

See live output.

Save later.

---

Analogy

Teacher speaking while recorder records.

---

Questions

1.

Difference from > ?

Shows output.

---

2.

Can append?

```
tee -a
```

---

3.

Useful in CI?

Extremely.

---

# Production Examples Every DevOps Engineer Uses

## Save Terraform logs

```bash
terraform apply | tee apply.log
```

---

## Find failed Pods

```bash
kubectl get pods -A | grep CrashLoop
```

---

## Restart service and save errors

```bash
systemctl restart nginx 2> errors.log
```

---

## Count failed login attempts

```bash
grep "Failed password" auth.log | wc -l
```

---

## Create YAML quickly

```bash
cat <<EOF > app.yaml
apiVersion: v1
kind: ConfigMap
EOF
```

---

# Services and Dependencies

| Topic  | Affects             | Why                       |
| ------ | ------------------- | ------------------------- |
| stdin  | Every CLI program   | Receives input            |
| stdout | Every CLI program   | Produces normal output    |
| stderr | Every CLI program   | Produces errors           |
| >      | Filesystem          | Writes files              |
| >>     | Filesystem          | Appends files             |
| <      | Filesystem          | Reads files               |
| <<     | Shell               | Supplies inline input     |
| |      | Shell + processes   | Connects commands         |
| tee    | Filesystem + stdout | Saves and displays output |

---

# 15 Mid-Level DevOps Interview Questions (with Answers)

### 1. What are stdin, stdout, and stderr?

They are the three standard I/O streams available to every Linux process. `stdin` (file descriptor 0) is the input stream, `stdout` (1) carries normal output, and `stderr` (2) carries error messages. Keeping output and errors separate allows automation and troubleshooting to work reliably.

---

### 2. What is the difference between `>` and `>>`?

`>` redirects `stdout` to a file and **overwrites** any existing content. `>>` redirects `stdout` and **appends** to the end of the file, preserving existing content.

---

### 3. Why is `stderr` separated from `stdout`?

Programs often need to process only successful output. If errors were mixed with normal output, scripts and pipelines could fail or parse incorrect data. Separate streams also make troubleshooting easier.

---

### 4. How do you redirect only error messages to a file?

```bash
command 2> errors.log
```

The `2` refers to the `stderr` file descriptor.

---

### 5. How do you redirect both stdout and stderr into the same file?

```bash
command > output.log 2>&1
```

or in Bash:

```bash
command &> output.log
```

This captures both normal output and errors in one file.

---

### 6. What does a pipe (`|`) do?

A pipe connects the `stdout` of one command directly to the `stdin` of another, allowing multiple small commands to work together without temporary files.

Example:

```bash
ps aux | grep nginx
```

---

### 7. Does a pipe send `stderr` to the next command?

No. A normal pipe transfers only `stdout`. If you also want errors in the pipeline, redirect `stderr` first:

```bash
command 2>&1 | grep error
```

---

### 8. What is the purpose of `tee`?

`tee` writes output to both the terminal and one or more files at the same time.

Example:

```bash
terraform apply | tee apply.log
```

This lets you monitor the command live while keeping a log.

---

### 9. What is a here-document (`<<`)?

A here-document provides multiline input directly in the shell script without creating a temporary file.

Example:

```bash
cat <<EOF
Hello
World
EOF
```

---

### 10. When would you use input redirection (`<`)?

When a program should read input from a file instead of the keyboard.

Example:

```bash
mysql < backup.sql
```

---

### 11. How would you find all failed SSH login attempts?

```bash
grep "Failed password" /var/log/auth.log | wc -l
```

This filters the log and counts matching lines.

---

### 12. Why are pipes preferred over temporary files?

Pipes:

* reduce disk I/O,
* improve performance,
* avoid cleanup of temporary files,
* simplify command chaining,
* are less error-prone in automation.

---

### 13. Why is `tee` commonly used in CI/CD pipelines?

CI/CD jobs need real-time console output for visibility while also saving logs for later review, auditing, or debugging. `tee` provides both simultaneously.

---

### 14. A script runs successfully but creates an empty output file. What would you check?

Check whether:

* the command is writing to `stderr` instead of `stdout`,
* the command actually produced output,
* the redirection is correct,
* permissions allow writing to the file,
* an earlier command in a pipeline failed.

---

### 15. Explain the difference between these two commands:

```bash
grep error app.log | wc -l
```

and

```bash
wc -l app.log
```

The first counts only lines containing `"error"`. The second counts **all** lines in the file, regardless of content.

---

## Interview Tip

For a **mid-level DevOps engineer**, interviewers usually expect you to go beyond syntax. You should be able to explain **why** these features exist and how they are used in production.

For example, instead of saying:

> "`tee` writes to a file and the terminal."

Aim for:

> "I use `tee` in CI/CD pipelines—for example, `terraform apply | tee apply.log`—so engineers can monitor progress in real time while preserving the complete output for auditing and post-failure analysis. This avoids rerunning long deployments just to recover logs."

That production-oriented explanation demonstrates both technical knowledge and practical experience.
