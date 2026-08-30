# v1 has no network SSH

v1 operators reach the appliance via the UI and console, not SSH. sshd is not a v1 product; reopen later. There is no injected-key Disk image: QEMU tests drive serial (Appliance CLI / Bootstrap console) and HTTPS (UI). This drops sshd-in-`mgmt`, stick DNAT of port 22, Host-netns sshd, and “SSH works in mgmt after Bootstrap” as pass/fail.

Considered: keep network SSH after Bootstrap; keep injected-key SSH as a test-only adapter. Rejected — v1 does not need SSH as a product, and the test adapter is what dominated the work.

If SSH is reopened later, it presents the Appliance CLI (admin), not a Host shell.

Supersedes the SSH clauses of ADR-0006, ADR-0024, and ADR-0028. The UI still never joins `fwd`.
