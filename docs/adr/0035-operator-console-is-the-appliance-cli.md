# Operator VGA/serial displays the Appliance console, not a kernel log

VGA and serial display the Appliance console, including Bootstrap and post-bootstrap recovery; full configuration commands are deferred with the Appliance CLI to v2 (ADR-0055). After GRUB, the kernel is quiet and systemd does not print status; panic and emergency may still appear. The Appliance console clears the tty when it starts. Considered: a noisy Fedora serial as the product console; a second debug serial. Rejected — operators use the Appliance console; journal is for debug. GRUB stays (rescue/edit).
