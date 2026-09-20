# Host update is refused until bootstrap

Host update is an admin operation. The Host update program's socket does not accept a Host update until Bootstrap completion is durably recorded (ADR-0051); tentative admin credentials in an unfinished attempt do not satisfy this gate. A published box has no admin (ADR-0024). Dev guests and tests complete Bootstrap before they call the socket. Pulling a newer Release on an unclassified box is not a First-install path.
