# v1 supports multiple local administrators

Bootstrap creates the first local administrator, and the authenticated UI can manage additional local administrator accounts in v1. These accounts have the same appliance-administrator permissions; fine-grained roles are deferred rather than requiring operators to share one login.

Extends the local Authentication source in ADR-0056 without bringing external Authentication sources or MFA into v1.
