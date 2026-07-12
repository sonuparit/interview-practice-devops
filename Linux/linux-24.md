This is one of the **highest ROI topics** for a Mid-level DevOps Engineer.

Many companies don't hire DevOps engineers to manually click buttons—they hire them to **automate repetitive work safely**. Bash is the glue that connects Linux, Docker, Kubernetes, Terraform, AWS CLI, Git, systemctl, SSH, cron, APIs, and monitoring tools.

A typical production automation script may easily contain:

* loops
* functions
* arrays
* conditions
* exit codes
* redirections
* traps
* strict mode (`set -euo pipefail`)

So interviewers expect you to understand these extremely well.

---

# 1. Loops

## What is this?

A loop repeatedly executes commands.

Types:

```bash
for
while
until
select
```

Example

```bash
for server in web1 web2 web3
do
    ssh "$server" uptime
done
```

---

## Why is it required?

Because infrastructure almost always contains multiple resources.

Instead of

```
Restart server1
Restart server2
Restart server3
...
```

You automate.

---

## DevOps usage

Examples

* Restart 100 services
* Backup multiple databases
* Delete old logs
* Check disk usage on every server
* Pull Docker images
* Deploy application to multiple hosts

---

## Why recruiters care?

Because automation saves hundreds of engineering hours.

Anyone can run one command.

A DevOps engineer automates thousands.

---

## Production example

Need to rotate logs on 500 servers.

Without loop:

Impossible.

With loop:

```bash
for host in $(cat servers.txt)
do
    ssh "$host" sudo logrotate
done
```

---

## Troubleshooting

Suppose deployment failed on server 27.

Print current server.

```bash
echo "Deploying $host"
```

Now you know where it stopped.

---

## Dependencies affected

Usually

* SSH
* Kubernetes
* Docker
* AWS CLI
* APIs
* Files

Because loop repeatedly calls them.

---

## Analogy

Teacher taking attendance.

Instead of remembering every student,

Teacher repeats

```
Roll 1
Roll 2
Roll 3
...
```

---

## Test yourself

1. Difference between for and while?
2. When should while be preferred?
3. What happens if one iteration fails?

---

# 2. Arrays

## What is this?

Store multiple values.

```bash
servers=("web1" "web2" "web3")
```

Access

```bash
echo "${servers[0]}"
```

---

## Why required?

Infrastructure has collections.

* Servers
* Pods
* Files
* IPs

Arrays manage them.

---

## DevOps usage

```bash
pods=(
frontend
backend
payments
)

for pod in "${pods[@]}"
do
kubectl rollout restart deployment "$pod"
done
```

---

## Recruiters care because

Hardcoding is bad.

Arrays make automation reusable.

---

## Production example

Need to restart only selected services.

Instead of editing code everywhere,

Just modify array.

---

## Troubleshooting

Print array

```bash
printf "%s\n" "${servers[@]}"
```

Verify expected values.

---

## Dependencies

Anything receiving list input.

---

## Analogy

Shopping cart.

One basket.

Many items.

---

## Questions

1. Difference between `$array` and `"${array[@]}"`?
2. Why quote arrays?
3. How to loop through arrays?

---

# 3. Functions

## What is this?

Reusable block.

```bash
backup(){

echo "Backing up..."

}
```

---

## Why required?

Avoid duplicate code.

---

## DevOps usage

```bash
restart_service(){

systemctl restart "$1"

}
```

Now

```bash
restart_service nginx
restart_service ssh
```

---

## Recruiters care

Because production scripts become maintainable.

---

## Production example

Instead of repeating

```
check disk

send alert

cleanup

```

inside many places,

Write one function.

---

## Troubleshooting

Debug one function instead of whole script.

---

## Dependencies

Every command inside function.

---

## Analogy

Microwave.

Press one button.

Many operations happen internally.

---

## Questions

1. Why functions improve scripts?
2. How to pass arguments?
3. How to return success?

---

# 4. Arguments

## What is this?

Values passed to script.

```bash
./deploy.sh production
```

Inside

```bash
$1
```

is

```
production
```

---

## Why required?

Reuse same script.

---

## DevOps usage

```
deploy.sh staging

deploy.sh production

deploy.sh testing
```

Same code.

Different environments.

---

## Production

CI/CD pipeline

```
deploy.sh prod
```

No editing.

---

## Troubleshooting

Print received values.

```bash
echo "$@"
```

---

## Dependencies

Pipeline variables

GitHub Actions

Jenkins

Terraform

AWS CLI

---

## Analogy

TV remote.

Same TV.

Different buttons.

---

## Questions

1. Difference between `$@` and `$*`?
2. Difference between `$1` and `$0`?
3. How validate arguments?

---

# 5. Exit Codes

## What is this?

Every command returns

```
0 = success

non-zero = failure
```

Example

```bash
mkdir test

echo $?
```

---

## Why required?

Programs communicate success/failure.

---

## DevOps usage

CI/CD.

If Terraform failed

Pipeline stops.

---

## Recruiters care

Production automation depends on exit codes.

---

## Production example

```bash
systemctl restart nginx

if [ $? -ne 0 ]
then
echo "Restart failed"
fi
```

---

## Troubleshooting

Always inspect

```
echo $?
```

---

## Dependencies

Every Linux command.

---

## Analogy

Traffic signal.

Green = success

Red = failure

---

## Questions

1. Why is 0 success?
2. What does 127 mean?
3. Why shouldn't scripts ignore exit codes?

---

# 6. Conditions

## What is this?

Decision making.

```bash
if
elif
else
```

---

## Why required?

Automation needs logic.

---

## DevOps

```bash
if disk>90%

send alert
```

---

## Production

Only restart if unhealthy.

---

## Troubleshooting

Print evaluated condition.

---

## Dependencies

Monitoring

Services

Files

Network

---

## Analogy

Doctor deciding treatment.

---

## Questions

1. Difference between `=` and `-eq`?
2. Difference between `[[ ]]` and `[ ]`?
3. Why quote variables?

---

# 7. case

## What is this?

Cleaner alternative to many if-else statements.

```bash
case "$1" in

start)
;;

stop)
;;

restart)
;;

esac
```

---

## Why required?

Many options.

---

## DevOps

CLI utilities.

---

## Production

Maintenance script.

---

## Troubleshooting

Easy to verify which branch executed.

---

## Dependencies

Arguments.

---

## Analogy

ATM menu.

Choose one option.

---

## Questions

1. Why prefer case?
2. How wildcard works?
3. Difference from if?

---

# 8. Redirection

## What is this?

Control input/output.

```
>

>>

<

2>

2>>

2>&1
```

---

## Why required?

Logging.

---

## DevOps

```bash
backup.sh >> backup.log 2>&1
```

---

## Production

Keep audit logs.

---

## Troubleshooting

Read logs.

---

## Dependencies

Files.

---

## Analogy

Water pipe.

Direct water to different buckets.

---

## Questions

1. Difference between > and >>?
2. What is stderr?
3. Why use 2>&1?

---

# 9. trap

## What is this?

Runs cleanup when script exits or receives a signal.

```bash
trap cleanup EXIT
```

---

## Why required?

Avoid leftover resources.

---

## DevOps

Delete temp files.

Unlock files.

Remove Kubernetes namespace.

---

## Production

Deployment interrupted.

Cleanup still runs.

---

## Troubleshooting

Add logging inside trap.

---

## Dependencies

Temporary files

Locks

Processes

---

## Analogy

Cleaning hotel room before checkout.

---

## Questions

1. What signals can trap catch?
2. Why use EXIT?
3. Why cleanup important?

---

# 10. set -e

## What is this?

Exit immediately if command fails.

```bash
set -e
```

---

## Why required?

Prevents cascading failures.

---

## Production

Database backup fails.

Script stops immediately.

---

## Troubleshooting

Find first failure quickly.

---

## Dependencies

Every command.

---

## Analogy

Construction stops if foundation cracks.

---

## Questions

1. Why dangerous without set -e?
2. When should it be disabled?
3. Interaction with ||?

---

# 11. set -u

## What is this?

Fail if undefined variable is used.

```bash
set -u
```

---

## Production

```
rm -rf "$DIR"
```

DIR missing.

Without set -u

Very dangerous.

---

## Troubleshooting

Shows typo immediately.

---

## Analogy

Compiler catching misspelled variable.

---

## Questions

1. Why useful?
2. What happens if variable missing?
3. How provide default?

---

# 12. set -o pipefail

## What is this?

Makes pipeline fail if any command fails.

Without

```
A | B | C
```

Only C matters.

With pipefail

Any failure matters.

---

## Production

```bash
curl api | jq
```

If curl fails,

Pipeline fails.

---

## Troubleshooting

Find hidden pipeline failures.

---

## Dependencies

Pipelines.

---

## Analogy

Assembly line.

If first machine breaks,

Whole production should stop.

---

## Questions

1. Why isn't pipeline failure detected by default?
2. What does pipefail change?
3. Why important in CI/CD?

---

# The Golden Rule of Production Bash

Almost every production-grade Bash script should begin with:

```bash
#!/usr/bin/env bash

set -euo pipefail
```

Why?

* `set -e` stops on the first unexpected failure, preventing cascading errors.
* `set -u` catches typos and missing environment variables early.
* `set -o pipefail` ensures failures inside pipelines aren't silently ignored.

Together, these options make scripts far more predictable and are considered a best practice in production automation.

---

# 15 Mid-Level DevOps Interview Questions (with Answers)

### 1. Why do production Bash scripts commonly start with `set -euo pipefail`?

**Answer:** It enables "strict mode." The script exits on command failures (`-e`), treats undefined variables as errors (`-u`), and reports failures anywhere in a pipeline (`pipefail`). This prevents subtle bugs and partial executions.

---

### 2. What is the difference between `$0`, `$1`, `$@`, and `$#`?

**Answer:**

* `$0` → Script name.
* `$1` → First argument.
* `$@` → All arguments as separate quoted values.
* `$#` → Number of arguments passed.

---

### 3. Why should you prefer functions over copying and pasting commands?

**Answer:** Functions improve readability, reduce duplication, centralize logic, simplify maintenance, and make testing and troubleshooting easier.

---

### 4. Why are exit codes important in CI/CD pipelines?

**Answer:** CI/CD systems rely on exit codes to determine whether a step succeeded or failed. A non-zero exit code typically stops the pipeline and prevents faulty deployments.

---

### 5. What is the difference between `>` and `>>`?

**Answer:**

* `>` overwrites the target file.
* `>>` appends to the end of the file.

---

### 6. When would you use a `case` statement instead of multiple `if` statements?

**Answer:** Use `case` when matching one variable against several possible values (for example, `start`, `stop`, `restart`). It is more readable and easier to extend.

---

### 7. Why should variables usually be quoted in Bash?

**Answer:** Quoting prevents word splitting and pathname expansion (globbing), ensuring values containing spaces or special characters are handled correctly.

---

### 8. What problem does `trap` solve?

**Answer:** It guarantees cleanup actions—such as deleting temporary files, releasing lock files, or stopping background processes—even if the script exits unexpectedly.

---

### 9. What happens without `set -o pipefail`?

**Answer:** By default, the pipeline's exit status is the exit status of the last command. Earlier failures may be hidden, causing scripts to appear successful when they are not.

---

### 10. How do you iterate over every element in a Bash array?

**Answer:**

```bash
for item in "${array[@]}"; do
    echo "$item"
done
```

Using `"${array[@]}"` preserves each element correctly, even if it contains spaces.

---

### 11. How would you safely validate that a required argument was provided?

**Answer:**

```bash
if [[ $# -lt 1 ]]; then
    echo "Usage: $0 <environment>"
    exit 1
fi
```

This checks that at least one argument is present before continuing.

---

### 12. Why should a deployment script check exit codes instead of assuming commands succeed?

**Answer:** Infrastructure commands can fail due to network issues, permission problems, missing dependencies, or service failures. Checking exit codes prevents the script from continuing in an inconsistent state.

---

### 13. Give an example of using a loop in production.

**Answer:** Restarting a service across multiple servers:

```bash
for host in "${servers[@]}"; do
    ssh "$host" "sudo systemctl restart nginx"
done
```

This automates repetitive operations and ensures consistency.

---

### 14. How can Bash redirection help during troubleshooting?

**Answer:** Redirecting stdout and stderr to log files preserves execution details for later analysis. For example:

```bash
./deploy.sh >deploy.log 2>&1
```

This captures both normal output and error messages in one file.

---

### 15. A deployment script created temporary files and then failed halfway. How would you ensure cleanup still happens?

**Answer:** Use `trap` to register a cleanup function:

```bash
cleanup() {
    rm -rf "$TMP_DIR"
}

trap cleanup EXIT
```

The cleanup function runs whenever the script exits—whether it succeeds, fails, or is interrupted—helping prevent leftover resources and inconsistent states.

---

## Mid-Level DevOps Interview Tip

For a mid-level DevOps role, don't stop at explaining the Bash syntax. Always connect it to a production scenario. For example:

* **Loops** → Deploy to multiple servers or namespaces.
* **Arrays** → Manage lists of hosts, pods, or services.
* **Functions** → Build reusable deployment or health-check logic.
* **Arguments** → Support different environments (dev, staging, production).
* **Exit codes** → Control CI/CD pipeline success or failure.
* **Conditions** → Make decisions based on service health or system state.
* **Case** → Build operational command-line utilities.
* **Redirection** → Capture logs for auditing and debugging.
* **Trap** → Clean up temporary resources and lock files.
* **`set -euo pipefail`** → Make automation predictable and safe in production.

Interviewers are usually evaluating whether you can design and troubleshoot reliable automation—not just whether you remember Bash syntax.
