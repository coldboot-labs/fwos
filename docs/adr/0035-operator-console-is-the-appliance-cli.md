# Operator VGA/serial is the Appliance CLI, not a kernel log

VGA and serial are the Appliance CLI (Bootstrap console, then admin CLI). After GRUB, the kernel is quiet and systemd does not print status; panic and emergency may still appear. The CLI clears the tty when it starts. Considered: a noisy Fedora serial as the product console; a second debug serial. Rejected — operators read the CLI, not dmesg; journal is for debug. GRUB stays (rescue/edit).
