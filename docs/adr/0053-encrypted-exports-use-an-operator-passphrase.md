# Encrypted exports use an operator-supplied passphrase

v1's optional encrypted Desired state export uses an operator-supplied passphrase, with the same passphrase required to restore the export. FWOS does not retain that passphrase, and unencrypted export remains available. Use an established encrypted-file format such as `age` instead of a bespoke encryption scheme; public/private-key recipient workflows are outside this v1 option.

Refines ADR-0050.
