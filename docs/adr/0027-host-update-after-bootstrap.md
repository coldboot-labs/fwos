# Host update is refused until bootstrap

Host update is an admin operation. The Host update program's socket does not accept a Host update until bootstrap has created an admin. A published box has no admin (ADR-0024). Dev guests bootstrap (or the test creates that admin) before they call the socket. Pulling a newer Release on an unclassified box is not a First-install path.
