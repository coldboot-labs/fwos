# v1 console provides limited authenticated recovery

After Bootstrap, the Appliance console provides an authenticated recovery menu for status, restoring the previous Accepted Desired state, rolling back the Host image, and rebooting. Keep the previous accepted configuration revision even after its replacement is accepted, so an administrator can undo a successfully applied but unsuitable configuration without the UI. This is a limited recovery interface, not the full Appliance CLI or a Host shell.

Extends ADR-0048's interrupted-apply recovery and defines the v1 console scope retained by ADR-0055. Configuration restoration and Host image rollback are distinct operations; Host image rollback keeps the persistent-state compatibility requirements in ADR-0046, and neither recovery operation reopens unauthenticated Bootstrap.

Network restoration preserves current Identity configuration under ADR-0063; recovery does not restore old administrator passwords or deleted accounts.
