# 10G drives architecture; KVM virtio is first-class

10G is a veto on extra hops and userspace packet paths, not a published SLA. The first-class appliance is a KVM VM (QEMU/Proxmox) with virtio-net as a Traffic NIC in `fwd`. Metal is the same image, second. VF/SR-IOV is optional acceleration, third, and must not be required. VMware/Hyper-V may boot; not a v1 claim. XDP_DROP/PASS prefilter is allowed and not required to make virtio fast.
