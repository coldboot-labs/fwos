# The UI is HTTPS with a self-signed certificate

The UI is HTTPS in both the Host-netns phase and in `mgmt`. There is no HTTP, including on link-local. v1 has no PKI or ACME; the browser warning is accepted. First-boot admin must not ride HTTP on an RFC1918 address that is often the upstream LAN.
