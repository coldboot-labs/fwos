# Desired state exports include secrets and support optional encryption

An explicit authenticated admin export includes the network secrets needed to restore full Desired state, such as WireGuard private keys; ordinary status responses omit those secrets. Unencrypted export remains available, clearly identified as sensitive, with an option to encrypt the export using an operator-supplied passphrase (ADR-0053). Encryption is an export option rather than a prerequisite for configuration import/export.

Extends ADR-0045.
