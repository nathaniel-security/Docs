# SELinux: A Practical Guide for Real-World Linux Security

If you've ever found yourself staring at cryptic SELinux denial logs or wondering what the heck `httpd_t` even means, you're not alone. SELinux is a powerful beast, and when tamed, it can lock down your system tighter than Fort Knox. Let's break it all down.

***

### 🚦 What Is SELinux?

SELinux stands for **Security-Enhanced Linux**. It's a kernel-level security system that enforces _Mandatory Access Control (MAC)_. Unlike normal Linux permissions, SELinux doesn’t care if you’re root. If the rules say “no,” it’s a hard no.

**In simple terms,** SELinux acts like an ultra-paranoid bodyguard between every process and file on your system.

***

### 🛠️ SELinux Modes: How Paranoid Is Your System?

#### 🧩 What is `SELINUXTYPE` in `/etc/selinux/config`?

When you open `/etc/selinux/config`, you'll see a line like:

```
SELINUXTYPE=targeted
```

This defines **which policy module SELinux should use**.

Common values:

* `targeted`: Only confines specific targeted services (e.g., `httpd`, `sshd`). Most commonly used.
* `mls`: Multi-Level Security. Used in environments needing high-level clearance enforcement.

Think of this as the "flavor" of SELinux. The `targeted` policy is practical for most use cases.

* **Disabled**: SELinux is off. You're flying blind.
* **Permissive**: Logs policy violations but doesn't block anything. Great for testing.
* **Enforcing**: Fully active. Blocks anything not explicitly allowed.

```
getenforce           # check mode
setenforce 0         # permissive
setenforce 1         # enforcing
```

To make it permanent:

```
sudo nano /etc/selinux/config
```

Set:

```
SELINUX=enforcing
SELINUXTYPE=targeted
```

***

### 🧠 SELinux Architecture (Simplified)

Here’s what happens when a process tries to access a file:

1. **Subject**: A process (like Apache or PHP).
2. **Object Manager**: Kernel service that intercepts the action.
3. **Security Server**: Checks the policy and decides “yes” or “no.”
4. **AVC (Access Vector Cache)**: Remembers the last answer to save time.

**Analogy:**

* Object Manager = bouncer
* Security Server = manager
* AVC = notebook with past decisions

***

### 🏷️ What Are Labels?

SELinux uses labels to make access decisions. These labels are made up of four fields:

```
user:role:type:level
```

Example:

```
system_u:system_r:httpd_t:s0
```

#### 📌 Explanation:

* **user**: SELinux user (not the same as Linux user). Defines identity within SELinux. E.g., `system_u`, `user_u`, `staff_u`.
* **role**: Groups of actions that a user is allowed to perform. E.g., `system_r`, `user_r`.
* **type**: The most critical field. Determines what can access what. This is where the bulk of access control happens. E.g., `httpd_t`, `ssh_t`, `user_home_t`.
* **level**: Optional. This is where MLS/MCS labels like `s0`, `s0:c1,c2` live. It’s used for additional access separation in sensitive or containerized environments.

Every file and process has a context label:

```
user:role:type:level
```

Example:

```
system_u:system_r:httpd_t:s0
```

Most important field? **Type**. That’s what defines what can talk to what.

***

### 👤 SELinux Users vs Linux Users (Simple Explanation)

#### Linux Users

* Regular users like `nathaniel`, `root`, `admin`.
* Defined in `/etc/passwd`.

#### SELinux Users

* Security contexts like `user_u`, `staff_u`, `sysadm_u`, `system_u`.
* Mapped from Linux users using `semanage login -l`.

#### Why Different?

* SELinux users define what **roles and types** a user can operate in.
* Linux users define what files and commands a user can run — **but SELinux overrides that**.

#### Example Mapping:

```
$ semanage login -l

Login Name    SELinux User     MLS/MCS Range
__default__   user_u           s0
root          root             s0-s0:c0.c1023
```

So Linux user `root` is mapped to SELinux user `root`, who has broader access. A normal user might be mapped to `user_u`, who has limited capabilities,containerised even if they `sudo`.

***

### 🧱 MLS vs MCS: Why It Matters and What `:s0` Means

When you look at SELinux labels, the last part often says `:s0` - That's the **level** part of the label: `user:role:type:level`.

#### So why are we talking about it?

Because that `:level` is where **MLS (Multi-Level Security)** or **MCS (Multi-Category Security)** kicks in.

* It’s **part of every label**, even if you don’t configure it.
* It influences how data is separated, especially in high-security or containerized environments.

#### 🧱 MLS (Multi-Level Security)

* Used in military/gov settings.
* Implements clearance levels (Unclassified, Secret, Top Secret).
* Enforces **No Read Up, No Write Down** — you can't read more secret stuff than your level, and you can't write to something below your level to avoid data leaks.
* Not commonly used unless you're in a high-compliance scenario.

#### 🗂 MCS (Multi-Category Security)

* Used in containers (like Docker, Podman).
* Easier to manage — uses categories instead of levels.
* Isolates containers by giving each one a different category (`c0,c1`, etc.).

#### 🔗 How it ties in:

Even if your system only uses the default `:s0`, understanding it tells you:

* What **kind** of isolation is SELinux enforcing?
* Why your PHP process might be able to see File A but not File B (different MCS categories).
* How to troubleshoot cross-container access problems.

> TL;DR — You can’t understand labels without understanding `:s0`, and you can’t enforce multi-tenant/container security without knowing how MCS works.

***

### 📜 Writing Your First SELinux Policy (PHP Example)

Let’s say your PHP app needs access to some new directory. Here’s the process:

#### 1. Install SELinux Tools

```
sudo dnf install selinux-policy-devel policycoreutils-python-utils setools-console
```

#### 2. Switch to Permissive Mode for Testing

```
sudo setenforce 0
```

Run your app. SELinux will log everything it _would_ block.

#### 3. Collect Denials

```
sudo ausearch -m AVC,USER_AVC -ts recent > php.audits
```

#### 4. Generate & Install Policy

```
grep php-fpm php.audits | audit2allow -M php_local
sudo semodule -i php_local.pp
```

#### 5. Test in Enforcing Mode

```
sudo setenforce 1
```

#### 6. Fix File Contexts

```
sudo semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/html/uploads(/.*)?"
sudo restorecon -Rv /var/www/html/uploads
```

***

### 🔍 Inspecting and Debugging

* `semodule -l`: List loaded modules
* `sesearch`: Search current policy
* `semanage boolean -l`: List toggleable options
* `sealert -a /var/log/audit/audit.log`: Fancy GUI-style analysis

***

### 🔚 Wrapping Up

I'm still learning SELinux myself, and this post is a collection of the core concepts that helped me understand it better. SELinux can be intimidating at first — but breaking it down into modes, labels, policies, and logs makes it a lot more approachable.

Start small, use permissive mode to learn from the logs, and don’t hesitate to make and test your own policies. The more you experiment, the more sense it starts to make.

**Your servers deserve better than `chmod 777`. Let’s give them the SELinux treatment — together.**\*\*
