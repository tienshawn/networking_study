#

## Conntrack dump command:

### Kernel/netlink
- ```ovs-dpctl dump-conntrack``` = dump current connections

- ```conntrack -L``` = list connections (similar to above)

- ```conntrack -F``` = flush conntrack

- ```conntrack -E``` = listen for events

### Userspace:
- ```ovs-appctl dpctl/dump-conntrack user@hostname``` = dump current connections