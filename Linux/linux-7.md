These four topics are **boot-time troubleshooting topics**. Junior engineers often stop at `systemctl`. Mid-level DevOps engineers are expected to understand **what happens before systemd even starts**, because production servers sometimes fail to boot.

---

# 1. Bootloader

## 1. What is this?

A **bootloader** is the first software that runs after the BIOS/UEFI firmware finishes hardware initialization.

Its job is to:

* Find the Linux kernel
* Load the kernel into RAM
* Pass kernel parameters
* Hand over control to the kernel

On most Linux systems, the bootloader is **GRUB2**.

Boot sequence:

```
Power ON
      │
      ▼
BIOS / UEFI
      │
      ▼
Bootloader (GRUB2)
      │
      ▼
Linux Kernel
      │
      ▼
systemd
      │
      ▼
Services
      │
      ▼
Login
```

---

## 2. Why is it required?

Without a bootloader,

the CPU has no idea

* where Linux is
* which kernel to load
* which filesystem contains it

The bootloader acts as the bridge between firmware and Linux.

---

## 3. How does this help in a Mid-level DevOps role?

You'll use it when:

* Kernel upgrade fails
* Multiple kernels exist
* Server won't boot
* Need to boot an older kernel
* Need recovery mode
* Need emergency shell

Very common after OS updates.

---

## 4. Why recruiters care?

Because production servers occasionally fail after:

* kernel update
* package update
* filesystem corruption
* wrong kernel module
* incorrect GRUB configuration

A DevOps engineer should know how to recover instead of rebuilding the server.

---

## 5. Production problem example

Suppose

```
yum update
```

updates the kernel.

After reboot:

```
Kernel Panic
```

Server is down.

Solution:

At GRUB menu:

```
Select previous kernel
```

Boot succeeds.

Production restored within minutes.

---

## 6. Troubleshooting example

Symptoms

```
Server doesn't boot
```

Check:

GRUB menu appears?

If yes

Problem probably isn't BIOS.

Boot previous kernel.

If works,

latest kernel is broken.

---

## 7. Dependencies affected

Bootloader loads

* Kernel
* initramfs
* kernel parameters

If bootloader breaks

Nothing else starts.

No systemd.

No Docker.

No Kubernetes.

No SSH.

Nothing.

---

## 8. Analogy

Imagine an airplane.

Pilot (BIOS)

asks

Ground crew (GRUB)

```
Which runway?
Which aircraft?
```

Ground crew prepares everything.

Then pilot starts flying.

GRUB prepares Linux for execution.

---

## 9. Three questions you should answer

**Q1**

What does GRUB actually load?

Answer:

Kernel + initramfs + kernel parameters.

---

**Q2**

Can Linux boot without a bootloader?

Only in special embedded systems.

Normal Linux servers require one.

---

**Q3**

What is the first thing GRUB executes?

It loads the selected Linux kernel into memory.

---

# 2. Kernel Parameters

---

## 1. What is this?

Kernel parameters are options passed by GRUB to Linux during boot.

Example:

```
quiet
```

```
ro
```

```
single
```

```
systemd.unit=rescue.target
```

```
init=/bin/bash
```

View them:

```
cat /proc/cmdline
```

Example

```
BOOT_IMAGE=/vmlinuz root=/dev/sda2 ro quiet
```

---

## 2. Why required?

They control

* boot behavior
* debugging
* hardware drivers
* filesystem mounting
* logging
* rescue modes

Without recompiling the kernel.

---

## 3. DevOps benefit

Useful for

* boot debugging
* storage issues
* networking issues
* cloud VM recovery
* kernel tuning

---

## 4. Recruiters care because

Production debugging often requires temporary kernel parameters.

Examples

```
systemd.unit=rescue.target
```

```
init=/bin/bash
```

```
nomodeset
```

---

## 5. Production problem

Example

GPU driver crashes.

Server freezes during boot.

Temporarily add

```
nomodeset
```

Boot succeeds.

Later fix driver.

---

## 6. Troubleshooting

Filesystem won't mount.

Add

```
init=/bin/bash
```

Boot into shell.

Repair filesystem.

Reboot.

---

## 7. Dependencies

Kernel parameters affect

* initramfs
* filesystem
* networking
* drivers
* logging
* SELinux
* systemd

They influence the system before most services start.

---

## 8. Analogy

Imagine giving instructions to a chef.

Instead of

```
Cook normally
```

You say

```
Less oil
More salt
No sugar
```

Kernel parameters are startup instructions.

---

## 9. Questions

What command shows kernel parameters?

```
cat /proc/cmdline
```

---

Who passes kernel parameters?

GRUB.

---

Can kernel parameters change boot behavior?

Absolutely.

---

# 3. Rescue Mode

---

## 1. What is this?

Rescue mode starts Linux with

very few services.

Only essential services start.

Networking may or may not.

Usually:

single-user maintenance mode.

---

## 2. Why required?

To repair

* filesystem
* boot issues
* passwords
* services
* fstab
* permissions

without loading the entire OS.

---

## 3. DevOps benefit

Very useful when

Production server won't boot fully.

You can repair

instead of rebuilding.

---

## 4. Recruiters care

Every Linux administrator eventually faces

```
Server boots halfway
```

Rescue mode is standard recovery.

---

## 5. Production problem

Wrong entry inside

```
/etc/fstab
```

System hangs during boot.

Boot into rescue mode.

Edit

```
/etc/fstab
```

Remove bad entry.

Reboot.

Problem solved.

---

## 6. Troubleshooting example

Forgot root password.

Boot

```
systemd.unit=rescue.target
```

Reset password.

Done.

---

## 7. Dependencies

Starts

* root filesystem
* systemd
* basic shell

Does NOT start

most user services

Docker

Kubernetes

Databases

Apache

Nginx

This minimizes interference during repair.

---

## 8. Analogy

Imagine repairing a factory.

Instead of shutting down electricity,

you stop all production lines

while keeping maintenance lights on.

That's rescue mode.

---

## 9. Questions

Can multiple users log in?

Normally no.

---

Why fewer services?

Safer maintenance.

---

Can filesystem be repaired?

Yes.

---

# 4. Emergency Mode

---

## 1. What is this?

Emergency mode is even more minimal than rescue mode.

Only

```
root filesystem

basic shell
```

No networking.

Almost nothing starts.

---

## 2. Why required?

When even rescue mode cannot start.

For severe failures like

* corrupted filesystem
* broken fstab
* missing root device

---

## 3. DevOps benefit

Last recovery option before reinstalling Linux.

---

## 4. Recruiters care

Shows deep Linux troubleshooting skills.

Anyone can restart services.

Few engineers recover an unbootable server.

---

## 5. Production problem

Filesystem corruption.

System cannot mount root filesystem.

Emergency shell opens.

Run

```
fsck
```

Repair filesystem.

Reboot.

Server recovered.

---

## 6. Troubleshooting

Commands

```
mount
```

```
fsck
```

```
journalctl
```

```
vi /etc/fstab
```

Fix issue.

Reboot.

---

## 7. Dependencies

Starts almost nothing.

No

* networking
* Docker
* SSH
* databases
* cron
* Kubernetes
* monitoring agents

Only enough to repair the system.

---

## 8. Analogy

Imagine a hospital.

Normal mode

Entire hospital works.

Rescue mode

Only emergency ward works.

Emergency mode

Only one doctor with a flashlight.

Enough to save the patient.

---

## 9. Questions

Which mode starts fewer services?

Emergency mode.

---

Can SSH work?

Usually no, because networking is not started.

---

When should emergency mode be used?

When rescue mode cannot boot or the system cannot reach a usable state.

---

# Rescue Mode vs Emergency Mode

| Feature           | Rescue Mode                        | Emergency Mode                                                   |
| ----------------- | ---------------------------------- | ---------------------------------------------------------------- |
| Root filesystem   | Mounted                            | Mounted (minimal; may require manual actions depending on issue) |
| systemd           | Yes                                | Minimal                                                          |
| Networking        | Sometimes                          | No                                                               |
| Services          | Few                                | Almost none                                                      |
| Maintenance shell | Yes                                | Yes                                                              |
| Multi-user        | No                                 | No                                                               |
| Purpose           | Repair a partially bootable system | Recover from severe boot failures                                |

---

# 10 Important Interview Questions

### 1. Explain the Linux boot sequence.

**Answer**

BIOS/UEFI → Bootloader (GRUB) → Kernel → initramfs → systemd → Targets → Services → Login.

---

### 2. What is GRUB?

A bootloader that loads the Linux kernel, initramfs, and passes kernel parameters before transferring control to the kernel.

---

### 3. What are kernel parameters?

Arguments passed by the bootloader to customize kernel behavior during startup, such as selecting boot targets, enabling debugging, or changing hardware initialization.

---

### 4. How do you see current kernel parameters?

```bash
cat /proc/cmdline
```

---

### 5. Server stopped booting after a kernel update. What would you do?

* Access the GRUB menu.
* Boot the previous working kernel.
* Investigate logs and the failed kernel.
* Fix or reinstall the problematic kernel before switching back.

---

### 6. Rescue mode vs emergency mode?

Rescue mode starts a minimal system with essential services for maintenance. Emergency mode starts an even smaller environment with almost no services, intended for severe boot failures.

---

### 7. A wrong `/etc/fstab` entry prevents the server from booting. How do you recover?

Boot into rescue or emergency mode, edit `/etc/fstab` to correct or comment out the invalid entry, then reboot.

---

### 8. What is `initramfs`, and why is it needed?

`initramfs` is a temporary root filesystem loaded by the bootloader along with the kernel. It contains the drivers and scripts needed to locate and mount the real root filesystem before handing control to the normal system.

---

### 9. How do kernel parameters help during troubleshooting?

They allow you to temporarily change boot behavior—for example, boot into rescue mode, disable problematic hardware features, or start a shell early—without modifying the installed operating system.

---

### 10. Why should a DevOps engineer understand the boot process?

Because failures can occur before application services or even `systemd` start. Understanding the boot process enables faster recovery from kernel issues, filesystem problems, bootloader failures, and configuration mistakes, reducing production downtime.
