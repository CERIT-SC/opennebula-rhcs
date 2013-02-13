# OpenNebula RHCS driver for KVM machines

*Red Hat® Cluster Suite* (RHCS) is set of components for building scalable,
load balancing and highly-available (HA) clusters. RHCS is part of
[High Availability Add-On][HA] in the Red Hat® Enterprise Linux®, but is
also freely available in RHEL-like systems (CentOS, SL, Oracle Linux). Other
distributions may work as well, but not all required RHCS components
in current versions should be present.

Following scripts add support into OpenNebula for managing **truly HA virtual
machines** via RHCS.

### Supported features

* hypervisor:
 * **KVM** via libvirt
* virtual machine actions:
 * deploy, cancel, power off
 * reboot, reset (same behaviour)
 * live migration

### Unsupported features

* hypervisor:
 * Xen
* virtual machine actions:
  * suspend, resume, stop
  * offline migration
  * disk attach/detach

## Cluster modes

* single-node cluster
* cluster

## Physical host

### Requirements

* latest RHEL 6.x (or CentOS, SL, Oracle Linux, ...)
* shared storage on all cluster nodes
* working RHCS cluster
 * running **ricci** daemon on all nodes, working `ccs` command
 * same RHCS cluster node and OpenNebula names
* [hypervisor][KVM] configured for OpenNebula
* cluster management commands enabled via `sudo`

#### Cluster configuration management

Cluster configuration is managed by `ccs` command. You must have `ricci`
daemon konfigured and running on all nodes and all nodes must by able
to synchronize configuration on all other cluster nodes password-less.

Example:

    $ ccs -h localhost --sync --activate

#### Node/host names

Management scripts can create failover domain with list of preferred
cluster node for each virtual machine. To enable this functionality,
RHCS cluster nodes must have same name as OpenNebula hosts. This binding
is also required for virtual machine migration.

Example - RHCS `/etc/cluster/cluster.conf`:

    <?xml version="1.0" ?>
    <cluster config_version="1" name="cluster">
        <clusternode name="node1.example.com" nodeid="1" />
        <clusternode name="node2.example.com" nodeid="2" />
    </cluster>

Example - OpenNebula hosts:

    $ onehost list
     ID   NAME                CLUSTER   RVM   ALLOCATED_CPU     ALLOCATED_MEM   STAT
    666   node1.example.com   -           0   0 / 1200 (0%)   0K / 62.9G (0%)   on    
    667   node2.example.com   -           0   0 / 1200 (0%)   0K / 62.9G (0%)   on    

#### Priviledged commands execution via `sudo`

**oneadmin** must be able to mange RHCS cluster. Following
configuration must be present in your `/etc/sudoers` (use `visudo`
for change):

    Defaults:oneadmin !requiretty
    %oneadmin ALL=(ALL) NOPASSWD: /usr/sbin/cman_tool *
    %oneadmin ALL=(ALL) NOPASSWD: /usr/sbin/clustat *
    %oneadmin ALL=(ALL) NOPASSWD: /usr/sbin/clusvcadm *
    %oneadmin ALL=(ALL) NOPASSWD: /usr/sbin/ccs *

## OpenNebula server (driver configuration)

Copy remote scripts into ??? .

Add new virtualization driver configuration into `oned.conf`:

    VM_MAD = [
        name       = "kvm_rhcs",
        executable = "one_vmm_exec",
        arguments  = "-t 0 -r 0 rhcs",
        default    = "vmm_exec/vmm_exec_kvm.conf",
        type       = "kvm" ]

Currently only the KVM virtual machines are supported.

## OpenNebula host configuration

For OpenNebula hosts only following combination of drivers is supported:

<table>
	<tr>
		<th>Information driver:</th>
		<td>kvm</td>
	</tr>

	<tr>
		<th>Virtualization driver:</th>
		<td>kvm_rhcs</td>
	</tr>

	<tr>
		<th>Network driver:</th>
		<td>dummy</td>
	</tr>
</table>

Example:

    $ onehost create node1.example.com --im kvm --vm kvm_rhcs --net dummy

[KVM]: http://opennebula.org/documentation:documentation:kvmg "OpenNebula KVM driver"
[HA]:  http://www.redhat.com/products/enterprise-linux-add-ons/high-availability/ "Red Hat® Enterprise Linux® High Availability Add-On"
