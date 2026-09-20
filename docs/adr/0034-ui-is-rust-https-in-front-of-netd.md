# The UI is a Rust HTTPS daemon in front of netd

The UI built-in addon is a Rust daemon: HTTPS to the browser (JSON API plus a static JS SPA, React or similar) and a unix-socket client of `netd`. It is also a v1 client of the separate Host update program for update operations (ADR-0055). The browser never talks to `netd` directly, and the UI does not implement a second Desired-state authority. `netd` does not grow HTTP. Cleartext HTTP stays out (ADR-0031).

Considered: `netd` exposing HTTPS; a third API daemon; Python UI. Rejected — `netd` stays the Desired-state authority on a unix socket (ADR-0011); the UI is a client, rewritten in Rust.
