# The Installer embeds the Host image

The Anaconda ISO contains the Host image. First install works with no registry. Registry pull is Host update of an already-installed box (ADR-0019), not First install. A thin ISO that pulls during install would undo “First install without a registry.”
