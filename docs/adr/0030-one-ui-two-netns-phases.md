# One UI, two netns phases

The bootstrap web UI is the same built-in addon that later runs in `mgmt`. It starts in the Host netns (ADR-0028) and is restarted into `mgmt` when bootstrap completes. There is no separate first-boot HTTP program and no Host-netns proxy into an empty `mgmt`. It talks to `netd` in both phases.
