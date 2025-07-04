---
coverY: 0
---

# Falco

<figure><img src="../.gitbook/assets/ChatGPT Image Jul 5, 2025, 01_00_35 AM.png" alt="" width="375"><figcaption></figcaption></figure>

### Install on a host

* This is for Debian with modern ebpf

```
curl -fsSL https://falco.org/repo/falcosecurity-packages.asc | \
sudo gpg --dearmor -o /usr/share/keyrings/falco-archive-keyring.gpg

```

```
echo "deb [signed-by=/usr/share/keyrings/falco-archive-keyring.gpg] https://download.falco.org/packages/deb stable main" | \
sudo tee -a /etc/apt/sources.list.d/falcosecurity.list
```

```
sudo apt-get install apt-transport-https
sudo apt-get update -y
sudo apt install -y dkms make linux-headers-$(uname -r)
# If you use falcoctl driver loader to build the eBPF probe locally you need also clang toolchain
sudo apt install -y clang llvm
# You can install also the dialog package if you want it
sudo apt install -y dialog
sudo apt-get install -y falco
```

### Breakdown: How Falco Talks to the Kernel

* `libscap` — _The Kernel Whisperer_
* Think of `libscap` as Falco’s **ears to the kernel**.
* It sits in **userspace** and reads **syscall events** (or eBPF events) pushed into a **ring buffer**.
* It doesn’t care _how_ the events got there — could be from:
  * Old-school **kernel module**
  * **Modern eBPF** magic

#### 🧠 “System CAPture” = `libscap`

* CAP = "Capture"
* It pulls events like:
  * `open()`, `execve()`, `socket()`, `mkdir()`, etc.
  * And hands it over to Falco’s engine to evaluate against security rules.

### Concepts

#### Event Sources

* by default falco consumes events from the Linux Kernel via drivers
  * This can be extended with plugins
* Rules are partitioned by event source, meaning each rule applies to a specific source and is triggered exclusively by events from that source.

> NOTE:- Falco does not support correlating events from different sources.

* Falco uses different instrumentations to analyse the system workload and pass security events to userspace.
  * We usually refer to these instrumentations as drivers, since a driver runs in kernel space.
  * The driver provides the syscall event source since the monitored events are strictly related to the syscall context.
* **Modern eBPF in Falco** means:
  * You **don’t need to compile a kernel module** or deal with the “which kernel headers?” nightmare.
  * Falco uses **CO-RE** (Compile Once, Run Everywhere) to run on different kernels **without recompilation**.
  * It's way more **portable** and works even on funky, custom or bleeding-edge kernels.
* The minimal set of capabilities required by Falco to run the modern eBPF probe is the following:
  * CAP\_SYS\_BPF
  * CAP\_SYS\_PERFMON
  * CAP\_SYS\_RESOURCE
  * CAP\_SYS\_PTRACE

**Need to implement for mordern embf probe**

* the 2 main required features are:
  * BPF ring buffer support. (https://www.kernel.org/doc/html/next/bpf/ringbuf.html)
  * A kernel that exposes BTF. (https://docs.kernel.org/bpf/btf.html)

**Using --disable-source & --enable-source**

```
falco --disable-source=syscall --disable-source=k8s_audit
```

* Disables specific event sources while keeping others active. This is useful if you want to prevent Falco from consuming events from certain sources.

```
falco --enable-source=syscall --enable-source=k8s_audit
```

**List of ignored sys calls**

```
falco -i
```

**🔁 How Falco Handles Dropped Events**

* Every second, Falco checks how many system call events the kernel has sent.
  * It looks at:
    * Total syscalls processed
    * How many failed because the shared buffer (between kernel and Falco) was full
* These failures = dropped events (Falco missed something!)
* What Falco Does If Events Are Dropped
  * You can configure how Falco reacts:

| Action   | What it Does                                        |
| -------- | --------------------------------------------------- |
| `ignore` | Do nothing (default)                                |
| `log`    | Write a **CRITICAL** log saying the buffer was full |
| `alert`  | Raise a **Falco alert** about the dropped event     |
| `exit`   | **Stop Falco** with an error code (non-zero)        |

* example log

```
Wed Mar 27 15:28:22 2019: Falco initialized with configuration file /etc/falco/falco.yaml
Wed Mar 27 15:28:22 2019: Loading rules from file /etc/falco/falco_rules.yaml:
Wed Mar 27 15:28:24 2019: Falco internal: syscall event drop. 1 system calls dropped in last second.
15:28:24.000207862: Critical Falco internal: syscall event drop. 1 system calls dropped in last second.(n_drops=1 n_evts=1181)
Wed Mar 27 15:28:24 2019: Falco internal: syscall event drop. 1 system calls dropped in last second.
Wed Mar 27 15:28:24 2019: Exiting.
```

Falco can **limit how many alerts/logs it sends in a short time**—to avoid flooding your logs or inbox.

* 🔕 **Disabled by default**
* 📦 Can be turned on in the `falco.yaml` config under the `outputs` section
  * https://falco.org/docs/reference/daemon/config-options/

#### Generating sample events

* To quickly check if Falco’s rules are firing correctly, use the built-in event-generator tool.
  * https://github.com/falcosecurity/event-generator

```
event-generator run [regexp]
```

* list all tools

```
event-generator list --all
```

### Falco Rules

* Falco rules are written in **YAML** and consist of **three main building blocks**:

| Element    | What It Is              | What It Does                                                                                                                                    |
| ---------- | ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| **Rules**  | The main logic          | Define **when to trigger an alert** (e.g., someone runs `chmod 777` on `/etc/shadow`). Includes a condition and a message.                      |
| **Macros** | Reusable logic snippets | Think of them like **shortcuts** for common conditions. You can use them inside rules or other macros.                                          |
| **Lists**  | Groups of items         | A bunch of values (like suspicious binaries, paths, usernames) you want to reuse. Can be used in macros/rules, but **not directly as filters**. |

* Falco rules files can also include **version checks** to prevent breakage.

| Element                        | What It Does                                                                                                            |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------------- |
| **`required_engine_version`**  | Makes sure the rule set only runs with a compatible **Falco engine**. Prevents crashes due to unsupported features.     |
| **`required_plugin_versions`** | Ensures rules that rely on specific **plugins** (like cloud, k8s, etc.) only run when the correct versions are present. |

#### Basic Elements of Falco Rules

```
- rule: shell_in_container
  desc: notice shell activity within a container
  condition: >
    evt.type = execve and 
    evt.dir = < and 
    container.id != host and 
    (proc.name = bash or
     proc.name = ksh)    
  output: >
    shell in a container |
    user=%user.name container_id=%container.id container_name=%container.name 
    shell=%proc.name parent=%proc.pname cmdline=%proc.cmdline    
  priority: WARNING

```

**🧠 1. Rule**

* A YAML object defining **when an alert should fire**.
* Key fields:
  * `rule`: Unique name.
  * `desc`: What it does.
  * `condition`: Boolean logic based on event fields.
  * `output`: Message, with `%field` placeholders.
  * `priority`: Severity (e.g., EMERGENCY → DEBUG).
* Optional advanced fields: `exceptions`, `enabled`, `tags`, `warn_evttypes`, `skip-if-unknown-filter`
* Why it matters: Defines exactly what behavior Falco considers suspicious and how to report it.

**🔍 2. Condition**

* Boolean expression triggering rule when **true**.
* Can reference any supported event fields (syscall or metadata).
* Example:

```
condition: evt.type = execve and evt.dir = < and container.id != host and proc.name = bash
```

**💬 3. Output**

* A templated message, interpolating event data: `%proc.cmdline`, `%user.name`, etc.
* Supports transformations like `%toupper(user.name)`

**⚠️ 4. Priority**

* Indicates severity of alerts; not evaluation order.
* Common levels: `EMERGENCY`, `ALERT`, `CRITICAL`, `ERROR`, `WARNING`, `NOTICE`, `INFORMATIONAL`, `DEBUG`
* Guidelines:
  * Writing state = ERROR
  * Reading sensitive files = WARNING
  * Unexpected behavior = NOTICE
  * Policy violations = INFO

**♻️ 5. Macros**

* Named, reusable **condition snippets**.
* Can call one macro from another (if already defined).
* Example:

```
- macro: container
  condition: container.id != host

- macro: spawned_process
  condition: evt.type = execve and evt.dir = <
```

**🧭 What does `evt.dir = <` mean?**

* **`evt.dir`** is short for **"event direction"**
* It indicates **which direction** the syscall is going:
  * `**<**` = **entering the syscall** (i.e., the syscall was just _called_)
  * `**>**` = **exiting the syscall** (i.e., the syscall just _finished_)
  * `**<>**` = either direction

**📋 6. Lists**

* Named collections of values used inside conditions or other lists.

```
- list: shell_binaries
  items: [bash, sh, zsh]
```

**👁️ Visibility**

* Ordering matters:
  * Lists reference only **earlier lists**.
  * Macros reference only **earlier macros**, but **any lists**.
  * Rules can reference **any macros**, even ones defined later

**TL;DR**

* **Rules** = core detection + metadata
* **Conditions** = event-matching logic
* **Outputs** = alert templates
* **Priority** = severity classification
* **Macros** = reusable bits of logic
* **Lists** = reusable sets of values

#### Default and Local Rules Files

By default, Falco loads rules in this order (from `falco.yaml`):

1. **`/etc/falco/falco_rules.yaml`** – the official default rules
2. **`/etc/falco/falco_rules.local.yaml`** – for local overrides
3. **`/etc/falco/rules.d/`** – any extra rule files you drop here

**⚙️ 2. Customize via CLI**

Running Falco manually? Use the `-r` flag:

```
falco -r /path/to/rules1.yaml -r /path/to/project/rules/
```

* Load multiple files or directories
* If you use `-r`, Falco **ignores** the default `rules_files` list

**🧩 3. Managing with falcoctl**

The `falcoctl` tool (CLI or Helm-based):

* Can **download** specific rule versions (via OCI artifacts)
* Puts them in `/etc/falco` by default
* Ensures your rules match your Falco engine version

**📦 What Are OCI Artifacts?**

**OCI = Open Container Initiative** Originally, OCI defined a standard for **container images**.\
Now, it also supports **generic files and packages** (not just containers)—called **OCI artifacts**.

**🔧 In the context of Falco:**

Falco uses **OCI artifacts** to package and distribute:

* Default rules
* Plugins
* Rulesets
* Configs All of this is pushed to and pulled from **OCI-compatible registries** (like Docker Hub, GitHub Container Registry, or your own Harbor).

#### Condition Syntax

**🔍 What Is a Condition?**

* A condition is a Boolean expression that filters events—if it evaluates to true, the rule fires, producing an alert
* Typically, it combines:
  * Event fields (like evt.type, proc.name, fd.name)
  * Comparison operators (=, !=, <, >, contains, in, glob, pmatch, etc.)
  * Logical operators (and, or, not)
  * Transformers (e.g., toupper(field), or % placeholders in outputs)

**🛠️ Operators Explained**

* Logical operators
  * and, or, not: Combine or negate comparisons

**Comparison operators**

* `=`, `!=`, `<`, `>`, `<=`, `>=`: Numerics or strict equality
* `contains`, `icontains` (case-insensitive), `bcontains` (byte-level)
* `endswith`, `glob`, `pmatch` (prefix matching)
* `exists`: checks presence of a field
* `in`, `intersects`: for sets or lists

**🧠 Example Condition**

```
evt.type = execve and evt.dir = < and (proc.name = cat or proc.name = grep)
```

* Triggers on **shell execs** entering via `execve`, specifically for `cat` or `grep`
  * You can also use macros and lists to simplify and modularize your logic.

**⚙️ Best Practices for Condition Writing**

1. **Start with `evt.type` early** in the condition to filter by event, improving performance
2. **Use positive matches** (`evt.type = execve`, not `!=`) to avoid unnecessary overhead
3. **Isolate event types** — don’t mix unrelated syscalls in a single rule for clarity and speed

**TL;DR**

* ✔️ Conditions are Boolean filters deciding when alerts fire.
* ✔️ Use logical, comparison, and set operators smartly.
* ✔️ Pre-filter with `evt.type` at the start.
* ✔️ Prefer positive logic and keep conditions focused.
* https://falco.org/docs/concepts/rules/conditions/

#### Overriding Rules

**🔄 1. What Override Means**

You can **customize or tweak** default Falco rules without touching them directly by either:

1. **Splitting files** — put your custom logic in a separate YAML loaded after the defaults.
2. **Using an `override` block** within that custom file to enlarge or replace parts of rules, macros, or lists.

***

**⚙️ 2. How Overriding Works**

In your custom rule file (e.g. `falco_rules.local.yaml`), you target an existing object using its name and specify changes via `override`:

* **`append`** → adds onto an existing element
* **`replace`** → completely substitutes the element

You can’t mix `append` and `replace` on the same item.

**What you can override:**

| Element   | appendable fields                                   | replaceable fields                                                                                                                                                                                          |
| --------- | --------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **List**  | `items`                                             | `items`                                                                                                                                                                                                     |
| **Macro** | `condition`                                         | `condition`                                                                                                                                                                                                 |
| **Rule**  | `condition`, `output`, `desc`, `tags`, `exceptions` | `condition`, `output`, `desc`, `priority`, `tags`, `exceptions`, `enabled`, `warn_evttypes`, `skip-if-unknown-filter` [falco.org](https://falco.org/docs/concepts/rules/overriding/?utm_source=chatgpt.com) |

**🔄 1. What Override Means**

You can **customize or tweak** default Falco rules without touching them directly by either:

1. **Splitting files** — put your custom logic in a separate YAML loaded after the defaults.
2. **Using an `override` block** within that custom file to enlarge or replace parts of rules, macros, or lists.

**⚙️ 2. How Overriding Works**

In your custom rule file (e.g. `falco_rules.local.yaml`), you target an existing object using its name and specify changes via `override`:

* **`append`** → adds onto an existing element
* **`replace`** → completely substitutes the element

You can’t mix `append` and `replace` on the same item.

**What you can override:**

| Element   | appendable fields                                   | replaceable fields                                                                                                                                                                                          |
| --------- | --------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **List**  | `items`                                             | `items`                                                                                                                                                                                                     |
| **Macro** | `condition`                                         | `condition`                                                                                                                                                                                                 |
| **Rule**  | `condition`, `output`, `desc`, `tags`, `exceptions` | `condition`, `output`, `desc`, `priority`, `tags`, `exceptions`, `enabled`, `warn_evttypes`, `skip-if-unknown-filter` [falco.org](https://falco.org/docs/concepts/rules/overriding/?utm_source=chatgpt.com) |

**🛠️ 3. Real Examples**

**🧩 Append to a list**

* Default:

```
- list: my_programs
  items: [ls, cat, pwd]
```

* Your file:

```
- list: my_programs
  items: [cp]
  override:
    items: append
```

* Result: cp is added, so the list becomes \[ls, cat, pwd, cp].

**🧩 Replace a list**

```
- list: my_programs
  items: [vi, vim, nano]
  override:
    items: replace

```

* Result: Old list replaced entirely with \[vi, vim, nano].

**🧩 Append to a macro**

* Default:

```
- macro: access_file
  condition: evt.type = open
```

* Your file:

```
- macro: access_file
  condition: or evt.type = openat
  override:
    condition: append
```

* Result: Macro now matches both evt.type = open and openat.

**🧩 Modify a rule**

```
- rule: program_accesses_file
  condition: evt.type = open and proc.name in (cat, ls)
  output: ...original msg...
  override:
    condition: append
    output: replace

```

* Result:
  * Condition gets more checks appended (e.g., and not user.name=root)
  * Output is completely replaced by yours.

#### ❗ Rule Exceptions

* Let you whitelist specific combos that should _not_ trigger alerts.

```
exceptions:
  - name: allow_nginx_etc
    fields: [proc.name, fd.name]
    comps: [=, startswith]
    values:
      - ["nginx", "/etc/nginx"]
```

* This allows nginx to write under /etc/nginx without raising an alert
* Best Practices:
  * Use **2+ fields** (e.g., `proc.name` + `fd.name`)
  * Avoid overly broad exceptions
  * You can override exceptions using `append` or `replace`

#### 🛑 Disabling or Enabling Rules via Config/CLI

Since Falco 0.38.0, you can selectively enable or disable rules/tags in your falco.yaml under the rules: section, or via CLI/Helm:

```
rules:
	- disable:
	  rule: "*"                 # turn off all rules
	- enable:
		rule: "Netcat Remote Code Execution in Container"
	- enable:
      rule: "Delete or rename shell history"
	- disable:
	    tag: ssh


```

***

## 🔤 Acronym Glossary

* system call
  * Syscalls stands for system calls, a way to request a service from the running kernel.

## Reference

* https://falco.org/docs/
* https://docs.kernel.org/bpf/btf.html
* https://www.kernel.org/doc/html/next/bpf/ringbuf.html
* https://falco.org/docs/reference/daemon/config-options/
* https://github.com/falcosecurity/event-generator
* https://falco.org/blog/adaptive-syscalls-selection/
* https://falco.org/docs/concepts/rules/conditions/
