# What Does "/bin/bash -p" Do?

* **Command Breakdown**:
  * `/bin/bash` is the path to the Bash shell executable.
  * `-p` flag stands for "privileged mode."
  * In many privilege escalation scenarios, an attacker might gain the ability to execute a command as root (or with elevated privileges), but only for that single command
* **Behavior**:
  * Launches Bash without resetting the effective user ID (EUID) to match the real user ID (RUID).
  * Useful in privilege escalation contexts.

**Why Is This Important in Privilege Escalation?**

1. **Maintaining Elevated Privileges**:
   * Helps turn a one-time privileged command into a persistent privileged session.
2. **Exploiting SUID Binaries**:
   * Prevents privilege-dropping behaviour when dealing with SUID (Set User ID) binaries.
3. **Turning Limited Access into Full Control**:
   * Can transform limited privilege escalation into full root access.

**Practical Example**

* **Scenario**:
  * Attacker exploits a vulnerable SUID binary to execute a command as root.

**Security Implications**

* **Offensive Security**:
  * Valuable for penetration testers in privilege escalation.
* **Defensive Security**:
  * Importance of managing SUID binaries carefully.
  * Monitor for unexpected privilege escalations.
  * Implement proper access controls and the principle of least privilege.
