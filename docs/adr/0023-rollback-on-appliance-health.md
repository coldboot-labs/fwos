# Automatic rollback is appliance health, not dataplane

A new bootc deployment has failed if the default target is not reached, or `fwd`/`mgmt` do not exist, or `netd` is not running. That rolls back. Desired state that does not apply does not roll back the Host image: that is a config failure, not a failed Release. systemd `boot-complete` alone is too weak for an appliance.
