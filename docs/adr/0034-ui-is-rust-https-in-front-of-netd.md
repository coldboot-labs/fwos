# The UI is a Rust HTTPS daemon in front of netd

The UI built-in addon is a Rust daemon: HTTPS to the browser (JSON API plus a static JS SPA, React or similar) and a unix-socket client of `netd`. The browser never talks to `netd`. There is no second Desired-state API. `netd` does not grow HTTP. Cleartext HTTP stays out (ADR-0031).

Considered: `netd` exposing HTTPS; a third API daemon; Python UI. Rejected — `netd` stays the one `CAP_NET_ADMIN` Desired-state API on a unix socket (ADR-0011); the UI is a client, rewritten in Rust.