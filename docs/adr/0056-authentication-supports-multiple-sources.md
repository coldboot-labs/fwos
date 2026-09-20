# Authentication supports multiple sources; v1 implements local passwords

v1 authenticates operators against local username and password records persisted as Identity configuration on `/var`, storing salted password hashes rather than plaintext passwords. Authentication is designed to support multiple configured Authentication sources and source-qualified identities, with room for interactive challenges and required authentication assurance instead of assuming every login is one username/password check. Only the local password source is implemented in v1; PAM-backed OS accounts, OpenID Connect federated login, and MFA are future implementations.

Authentication establishes identity; appliance authorization decides what that identity may do. Future MFA requirements must be enforceable across allowed sources, rather than being bypassed by selecting another login method.

Complements the UI-first operator interface in ADR-0055. Multiple Authentication sources are an architectural requirement, not a promise to implement the additional sources in v1.

ADR-0057 specifies multiple local administrator accounts in v1, with fine-grained roles deferred.

ADR-0063 separates Identity configuration from network Desired state so network recovery preserves current administrator access settings.
