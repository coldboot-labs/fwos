# Incomplete Bootstrap can be discarded and repeated

After failure or interruption before Bootstrap completion is durably recorded, FWOS discards the attempt's tentative admin credentials and partial Desired state and restores the console-opted Bootstrap environment before unauthenticated setup resumes. Completion is recorded only once the admin exists and interface roles plus UI exposure have been applied; merely choosing credentials does not establish permanent ownership of an unfinished appliance. Once Bootstrap has completed, failures use authenticated recovery and never automatically reopen unauthenticated Bootstrap.

Amends ADR-0028 and ADR-0039. The console modes in ADR-0033 and post-bootstrap recovery rule in ADR-0048 stand.
