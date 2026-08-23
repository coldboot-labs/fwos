# Desired state is TOML on disk and JSON on the socket

One serde model: TOML files on `/var` (comments, human rescue), JSON on the unix socket (UI, Appliance CLI, other clients). JSON-everywhere was considered and rejected.
