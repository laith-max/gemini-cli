# Reporting Security Issues

To report a security issue, please use [https://g.co/vulnz](https://g.co/vulnz).
We use g.co/vulnz for our intake, and do coordination and disclosure here on
GitHub (including using GitHub Security Advisory). The Google Security Team will
respond within 5 working days of your report on g.co/vulnz.

[GitHub Security Advisory]:
  https://github.com/google-gemini/gemini-cli/security/advisories

## Shared Responsibility Model

Gemini CLI is designed as a developer tool for single-user environments. It does
not enforce a security boundary between multiple user accounts operating on the
same device or environment.

### Best practices for multi-user environments

If you use Gemini CLI in an environment shared with other users, we recommend
the following practices to prevent cross-user leakage and privilege escalation:

- **Restrict directory permissions:** Ensure your `~/.gemini` configuration
  directory is readable and writable only by your user account (for example,
  `chmod 700 ~/.gemini`). Gemini CLI requires write permissions to this
  directory.
- **Avoid global installations:** Install Gemini CLI locally per-user rather
  than globally to minimize the risk of shared dependencies being compromised.
- **Isolate execution paths:** Don't run Gemini CLI from shared directories
  (such as `C:\` on Windows) where other users have write access. This prevents
  attackers from hijacking the dependency resolution process (for example, via
  malicious `node_modules` folders) and executing code in your context.
