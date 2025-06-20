![Jamf Now Live 2025 Event Banner](https://media.jamf.com/images/events/JNL25-lp-5-1680x1050px.webp?q=80&w=800)

# 🚀 Zero to Hero: A Practical Guide to Getting Started with Blueprints

Hey folks,

This README is your quick-reference from our Jamf Pro session on configuring services and blueprints for macOS. Below you’ll find the exact configurations and breakdowns we demoed, with clear explanations and a focus on security and usability. Use these as a base for your own Jamf blueprints or as a learning resource!

---

## 🔐 SSH Service Hardening

We started by locking down SSH to keep remote access tight and secure. Here’s the config we used:

```
KbdInteractiveAuthentication False
PermitRootLogin False
UnusedConnectionTimeout 900
```

### What does this do?

| Option                        | Value  | What it Means                                                                                   |
|-------------------------------|--------|-----------------------------------------------------------------------------------------------|
| `KbdInteractiveAuthentication`| False  | Disables keyboard-interactive (password) prompts—SSH keys only, no password guessing!          |
| `PermitRootLogin`             | False  | Blocks direct root login over SSH—no more “root” brute force attacks.                          |
| `UnusedConnectionTimeout`     | 900    | Boots idle SSH connections after 15 minutes—keeps things tidy and safe.                        |

**Why?**  
- 🔒 Only allows key-based authentication, which is much harder to attack.
- 🚫 No direct root access means attackers can’t just guess a root password.
- ⏳ Idle sessions get cleaned up automatically, reducing risk from forgotten logins.

---

## 🛡️ Sudoers: Safe Privileges for Admins

Next, we made sure admins can do their jobs—but not blow up the system by accident or run risky Jamf commands.

```
# Allow all admin users to run most things (with password)
%admin ALL=(ALL) ALL

# Allow these jamf commands without a password
Cmnd_Alias JAMF_NOPASS = /usr/local/bin/jamf flushDocks, \
                         /usr/local/bin/jamf flushCaches, \
                         /usr/local/bin/jamf recon, \
                         /usr/local/bin/jamf policy, \
                         /usr/local/bin/jamf reboot
%admin ALL=(root) NOPASSWD: JAMF_NOPASS

# Block these dangerous system commands (even with sudo)
Cmnd_Alias BLOCKED_SYSTEM_COMMANDS = /bin/rm, \
                                     /usr/sbin/systemsetup
%admin ALL=(ALL) NOPASSWD: !BLOCKED_SYSTEM_COMMANDS

# Block these risky jamf commands (even if jamf is generally allowed)
Cmnd_Alias BLOCKED_JAMF = /usr/local/bin/jamf removeFramework, \
                          /usr/local/bin/jamf removeframework, \
                          /usr/local/bin/jamf flushPolicyHistory, \
                          /usr/local/bin/jamf policy -event *
%admin ALL=(root) NOPASSWD: !BLOCKED_JAMF
```

### How does this work?

| Rule / Alias                        | What It Covers                                                                                                  | Effect / Why We Did It                                              |
|-------------------------------------|----------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------|
| `%admin ALL=(ALL) ALL`              | All commands for admin group (with password)                                                                   | Standard admin access, still needs password.                        |
| `JAMF_NOPASS`                       | Safe Jamf commands (`flushDocks`, `flushCaches`, `recon`, `policy`, `reboot`)                                  | Lets admins run these Jamf tasks with a password—smooth workflow.|
| `%admin ALL=(root) NOPASSWD: JAMF_NOPASS` | Applies passwordless sudo for those Jamf commands only.                                                        | Convenience for day-to-day management, no security risk.            |
| `BLOCKED_SYSTEM_COMMANDS`           | `/bin/rm`, `/usr/sbin/systemsetup`                                                                             | Prevents accidental or malicious system destruction.                |
| `%admin ALL=(ALL) NOPASSWD: !BLOCKED_SYSTEM_COMMANDS` | Explicitly blocks those commands, even if sudo would normally allow.                                           | Safety net—these can’t be run, period.                             |
| `BLOCKED_JAMF`                      | Risky Jamf commands (`removeFramework`, `flushPolicyHistory`, `policy -event *`)                               | Blocks dangerous Jamf actions, even if general Jamf access is allowed.|
| `%admin ALL=(root) NOPASSWD: !BLOCKED_JAMF` | Ensures these Jamf commands are always blocked, even for admins.                                               | Stops mistakes and protects device management.                      |

---

## 📝 Key Takeaways

- **Principle of Least Privilege:** Admins get what they need, nothing more.
- **Safety First:** Destructive commands are blocked at the source.
- **Smooth Experience:** Passwordless sudo for safe Jamf actions keeps workflows fast and frustration-free.
- **Security by Default:** SSH and sudoers are hardened out of the box.

---

If you want to dig deeper or need a hand with your own Jamf blueprints, just reach out!  
Happy configuring! 🚀
