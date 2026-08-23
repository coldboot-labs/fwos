# /usr is firmware; config and state live on /var

The host image’s `/usr` is immutable firmware (bootc). Appliance config, daemon state, and the `netd` socket directory live on `/var`. Addons do not write the host image. `/etc` is not the product config store.
