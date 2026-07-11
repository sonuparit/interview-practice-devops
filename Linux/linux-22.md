This is one of the highest ROI Linux topics for a DevOps Engineer.

Almost **every CI/CD pipeline, Docker image, Kubernetes container, Terraform execution, Ansible playbook, systemd service, and Bash script** relies on environment variables.

---

# 22. Environment Variables

Environment variables are **key-value pairs stored by the operating system** that programs inherit when they start.

Example:

```bash
HOME=/home/sonu
USER=sonu
PATH=/usr/local/bin:/usr/bin:/bin
SHELL=/bin/bash
```

When you run a program:

```bash
python app.py
```

the program automatically receives these variables.

Think of them as **configuration values passed to every process**.

---

# 1. PATH

## 1. What is this?

PATH is an environment variable containing directories where Linux searches for executable commands.

Example

```bash
echo $PATH
```

Output

```text
/usr/local/bin:/usr/bin:/bin:/usr/sbin
```

Directories are separated by

```
:
```

When you type

```bash
kubectl
```

Linux searches:

```
/usr/local/bin/kubectl

then

/usr/bin/kubectl

then

/bin/kubectl
```

until found.

---

## 2. Why is it required?

Without PATH,

Linux wouldn't know where commands exist.

Instead of

```bash
kubectl
```

you would need

```bash
/usr/local/bin/kubectl
```

every single time.

---

## 3. DevOps Usage

Very important.

CI/CD:

```yaml
terraform apply
```

works because PATH contains Terraform.

Dockerfiles

```dockerfile
ENV PATH="/opt/app/bin:$PATH"
```

Kubernetes containers

Custom applications added into PATH.

---

## 4. Why recruiters care

PATH issues are extremely common.

Many production outages happen because

```
command not found
```

Engineers should immediately know

* PATH
* executable locations
* permissions

---

## 5. Production Problem

Example

Deployment pipeline

```
terraform: command not found
```

Terraform exists

```
/opt/terraform/terraform
```

PATH missing.

Fix

```bash
export PATH=$PATH:/opt/terraform
```

Pipeline works immediately.

---

## 6. Troubleshooting

Suppose

```
docker: command not found
```

Check

```bash
echo $PATH
```

Find binary

```bash
which docker
```

or

```bash
find / -name docker
```

Add correct directory.

---

## 7. Dependencies affected

Almost everything.

Examples

* Git
* Docker
* kubectl
* Terraform
* Helm
* AWS CLI
* Python
* Java

---

## 8. Analogy

Imagine a courier.

PATH is the list of roads the courier checks.

Without roads,

he never reaches the destination.

---

## 9. Three questions

Can you explain how Linux finds commands?

Difference between PATH and absolute path?

Why does "command not found" happen even though software is installed?

---

# 2. HOME

## 1. What is this?

The user's home directory.

Example

```bash
echo $HOME
```

```
/home/sonu
```

---

## 2. Why required?

Applications store user-specific data.

Example

```
~/.ssh

~/.aws

~/.kube

~/.bashrc
```

All depend on HOME.

---

## 3. DevOps Usage

AWS CLI

```
~/.aws/credentials
```

Kubectl

```
~/.kube/config
```

Git

```
~/.gitconfig
```

SSH keys

```
~/.ssh/id_rsa
```

---

## 4. Recruiters care because

Broken HOME causes

* wrong configs
* missing credentials
* login failures

---

## 5. Production Example

CI Runner

```
HOME=/
```

AWS CLI cannot find

```
~/.aws/credentials
```

Deployment fails.

---

## 6. Troubleshooting

Check

```bash
echo $HOME
```

Verify

```bash
ls $HOME
```

---

## 7. Dependencies

* SSH
* Git
* AWS CLI
* Kubectl
* Terraform

---

## 8. Analogy

HOME is your personal locker.

Everyone has their own.

---

## 9. Three questions

Where are SSH keys stored?

How does kubectl know kubeconfig location?

What happens if HOME is wrong?

---

# 3. SHELL

## 1. What is this?

Current shell.

Example

```bash
echo $SHELL
```

```
/bin/bash
```

Could also be

```
/bin/zsh
```

---

## 2. Why required?

Different shells have different syntax.

---

## 3. DevOps Usage

Scripts

```bash
#!/bin/bash
```

Pipeline compatibility.

---

## 4. Recruiters care

Many scripts fail because engineers assume Bash while the system uses `sh` or another shell.

---

## 5. Production Example

Ubuntu

```
/bin/sh -> dash
```

Script

```bash
[[ ]]
```

Fails.

Reason

Not Bash.

---

## 6. Troubleshooting

Check

```bash
echo $SHELL
```

Check script

```bash
head script.sh
```

---

## 7. Dependencies

* Bash scripts
* Login shell
* User profiles
* CI jobs

---

## 8. Analogy

Shell is the translator between you and Linux.

---

## 9. Questions

Difference between bash and sh?

What is a shebang?

Why do shell scripts fail on another server?

---

# 4. env

## 1. What is this?

Displays environment variables.

```bash
env
```

---

## 2. Why required?

Shows current execution environment.

---

## 3. DevOps Usage

Debug pipelines.

Debug containers.

---

## 4. Recruiters care

Quick diagnosis.

---

## 5. Production Example

Application

```
DATABASE_URL missing
```

Run

```bash
env
```

Variable absent.

Problem found.

---

## 6. Troubleshooting

```bash
env | grep AWS
```

---

## 7. Dependencies

Everything reading environment variables.

---

## 8. Analogy

Employee ID card listing all permissions.

---

## 9. Questions

Difference between env and printenv?

How do applications receive variables?

Can env launch a command with temporary variables?

Example:

```bash
env DEBUG=true python app.py
```

---

# 5. export

## 1. What is this?

Makes a shell variable available to child processes.

Example

```bash
export NAME=Sonu
```

---

## 2. Why required?

Without export

Programs cannot access it.

---

## Example

Without export

```bash
NAME=Sonu

python app.py
```

Python won't see it.

With export

```bash
export NAME=Sonu
```

Python receives it.

---

## 3. DevOps Usage

AWS

```bash
export AWS_PROFILE=prod
```

Terraform

```bash
export TF_VAR_region=us-east-1
```

Kubernetes

```bash
export KUBECONFIG=config
```

---

## 4. Recruiters care

Used daily.

---

## 5. Production Example

Terraform using wrong AWS account.

Fix

```bash
export AWS_PROFILE=production
```

---

## 6. Troubleshooting

```bash
echo $AWS_PROFILE
```

---

## 7. Dependencies

Any child process.

---

## 8. Analogy

Giving employees an access badge before entering the office.

---

## 9. Questions

Difference between shell variable and exported variable?

Who inherits exported variables?

Does export survive logout?

---

# 6. printenv

## 1. What is this?

Displays environment variables.

```bash
printenv
```

Specific variable

```bash
printenv PATH
```

---

## 2. Why required?

Cleaner than env.

---

## 3. DevOps Usage

Debugging

```bash
printenv KUBECONFIG
```

---

## 4. Recruiters care

Quick inspection.

---

## 5. Production Example

Wrong Java version.

Check

```bash
printenv JAVA_HOME
```

---

## 6. Troubleshooting

```bash
printenv PATH
```

---

## 7. Dependencies

Applications depending on variables.

---

## 8. Analogy

Looking at one employee record instead of the whole database.

---

## 9. Questions

Difference from env?

Can printenv show one variable?

Why useful in debugging?

---

# 7. source

## 1. What is this?

Runs a script inside the current shell.

Example

```bash
source ~/.bashrc
```

Equivalent

```bash
. ~/.bashrc
```

---

## 2. Why required?

Reload configuration without opening a new terminal.

---

## Example

Edit

```bash
~/.bashrc
```

Add

```bash
export PATH=$PATH:/opt/bin
```

Reload

```bash
source ~/.bashrc
```

Immediately available.

---

## 3. DevOps Usage

Every day.

Activate Python venv

```bash
source venv/bin/activate
```

Load environment

```bash
source env.sh
```

---

## 4. Recruiters care

Most Linux engineers use it daily.

---

## 5. Production Example

New PATH added.

Pipeline still failing.

Forgot

```bash
source ~/.bashrc
```

---

## 6. Troubleshooting

After editing

```bash
~/.bashrc
```

Run

```bash
source ~/.bashrc
```

Verify

```bash
echo $PATH
```

---

## 7. Dependencies

* Bash
* Shell configuration
* Virtual environments
* CI shell sessions

---

## 8. Analogy

Instead of restarting your computer after changing one setting, you press "Reload Settings."

---

## 9. Questions

Difference between running

```bash
./script.sh
```

and

```bash
source script.sh
```

When should source be used?

Why does source modify current shell?

---

# Common Production Problems Involving Environment Variables

| Problem                                     | Root Cause                                                              | Solution                                                                   |
| ------------------------------------------- | ----------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| `terraform: command not found`              | PATH missing Terraform                                                  | Fix PATH                                                                   |
| Git cannot find SSH key                     | HOME incorrect                                                          | Correct HOME                                                               |
| Kubernetes can't connect                    | KUBECONFIG missing                                                      | Export KUBECONFIG                                                          |
| AWS CLI uses wrong account                  | AWS_PROFILE wrong                                                       | Export correct profile                                                     |
| Java app won't start                        | JAVA_HOME incorrect                                                     | Set JAVA_HOME                                                              |
| CI pipeline missing secrets                 | Variables not exported                                                  | Export them or define them in CI                                           |
| Script works interactively but not via cron | Cron has a minimal environment (limited `PATH`, different `HOME`, etc.) | Use absolute paths or explicitly set environment variables in the cron job |
| Changes to `.bashrc` have no effect         | Shell configuration not reloaded                                        | Run `source ~/.bashrc` or start a new shell                                |

---

# 15 Mid-Level DevOps Interview Questions

### 1. What is the difference between a shell variable and an environment variable?

**Answer:** A shell variable exists only in the current shell. An environment variable is exported and inherited by child processes. Use `export` to convert a shell variable into an environment variable.

---

### 2. How does Linux find a command when you type `kubectl`?

**Answer:** The shell searches the directories listed in the `PATH` environment variable from left to right until it finds an executable named `kubectl`.

---

### 3. Why does `command not found` occur even though the software is installed?

**Answer:** Common reasons include:

* The executable's directory is not in `PATH`.
* The binary lacks execute permissions.
* The binary isn't installed where expected.
* The command name is incorrect.

---

### 4. What is the purpose of the `PATH` variable?

**Answer:** It tells the shell where to search for executable programs, allowing commands to be run without specifying their full path.

---

### 5. What is the difference between `env` and `printenv`?

**Answer:** Both display environment variables. `printenv` is commonly used to print specific variables (e.g., `printenv PATH`), while `env` also has the ability to run a command with a modified environment (e.g., `env DEBUG=true ./app`).

---

### 6. What does the `HOME` environment variable do?

**Answer:** It identifies the current user's home directory. Many tools use it to locate user-specific configuration files, such as `~/.ssh`, `~/.aws`, `~/.kube`, and `~/.gitconfig`.

---

### 7. Why is `export` required?

**Answer:** Without `export`, a variable remains local to the current shell and child processes (such as Python, Terraform, or Docker) cannot access it.

---

### 8. What does `source ~/.bashrc` do?

**Answer:** It executes the commands in `.bashrc` within the current shell, immediately applying configuration changes without opening a new terminal.

---

### 9. What is the difference between `source script.sh` and `./script.sh`?

**Answer:** `source` runs the script in the current shell, so environment changes persist. `./script.sh` starts a new shell process; any variable changes disappear when that process exits.

---

### 10. How would you troubleshoot a missing environment variable in production?

**Answer:**

1. Check whether it's present using `printenv` or `env`.
2. Verify where it should be defined (`.bashrc`, systemd service, CI pipeline, container spec, etc.).
3. Confirm it is exported.
4. Restart or reload the process if necessary.

---

### 11. Why do cron jobs often fail while the same command works in your terminal?

**Answer:** Cron runs with a minimal environment. Variables like `PATH`, `HOME`, or application-specific variables may be missing, so absolute paths or explicit environment definitions are often required.

---

### 12. How are environment variables used in containers?

**Answer:** They provide runtime configuration (database URLs, API endpoints, feature flags, credentials references, etc.) without modifying the application image, making deployments portable across environments.

---

### 13. How do systemd services use environment variables?

**Answer:** Environment variables can be defined directly in a unit file (`Environment=`), loaded from an external file (`EnvironmentFile=`), or inherited from the service's execution environment.

---

### 14. Why shouldn't secrets be stored directly in environment variables?

**Answer:** Environment variables are convenient but may be exposed through process listings, debugging tools, logs, or crash dumps. For sensitive production workloads, use dedicated secret-management solutions (such as Kubernetes Secrets with proper controls, cloud secret managers, or vault systems) whenever possible.

---

### 15. How are environment variables used in Kubernetes?

**Answer:** They can be defined directly in a Pod specification, injected from ConfigMaps or Secrets, or populated using the Downward API. This allows the same container image to be configured differently across development, staging, and production environments without rebuilding the image.

---

## Mid-Level DevOps Takeaway

By the interview stage, you should be comfortable explaining not only **what** environment variables are, but also **how they flow through a Linux system**:

* A user logs in and the shell initializes its environment.
* Startup files (such as `.bashrc` or `.profile`) define variables.
* `export` makes selected variables available to child processes.
* Programs inherit those variables when they start.
* Tools like Docker, Kubernetes, systemd, CI/CD platforms, and Terraform rely heavily on these inherited values for configuration.
* When something fails unexpectedly, checking `PATH`, `HOME`, `SHELL`, and the process environment (`printenv`, `env`) is often one of the first and most effective troubleshooting steps.
