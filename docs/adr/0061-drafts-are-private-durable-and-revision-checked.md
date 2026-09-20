# Drafts are private, durable, and revision-checked

Each administrator has a private saved Draft Desired state based on an Accepted Desired state revision; it survives logout and reboot but never applies automatically. If the accepted configuration changes underneath a draft, applying it requires reconciliation and another review rather than a silent overwrite or automatic merge in v1. The standalone Save and apply shortcut is available only when that administrator has no other pending edits and uses the same revision checks.

Extends ADR-0059. Draft ownership follows the source-qualified administrator identity from ADR-0056, so matching usernames from different Authentication sources do not share pending edits.
