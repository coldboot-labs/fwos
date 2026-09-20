# Identity configuration is separate from network rollback

On a bootstrapped appliance, Identity configuration—administrator accounts, passwords, and Authentication source settings and requirements—is separate from network Desired state. Restoring an earlier network revision preserves current Identity configuration, so recovery cannot resurrect deleted administrators, restore old passwords, or weaken current authentication requirements. Account and authentication changes are explicit, separate operations rather than side effects of network application or rollback.

Clarifies the persisted configuration boundary in ADR-0056 and the recovery guarantees in ADRs 0043, 0047, 0048, and 0058. A Host image rollback may restore the pre-update network configuration under ADR-0046, but must preserve current Identity configuration in shared `/var`; the previous Release must remain able to read it. Discarding tentative credentials before durable Bootstrap completion remains governed by ADR-0051.
