# The UI is HTTPS with a self-signed certificate

The UI uses HTTPS during Bootstrap and steady state, always in `mgmt` (ADR-0039). There is no HTTP. v1 has no PKI or ACME; the browser warning is accepted. First-boot admin must not ride HTTP on an RFC1918 address that is often the upstream LAN.
