---
**EXAMPLE DOCUMENTATION - FOR PORTFOLIO PURPOSES ONLY**

*This is a sample technical documentation guide created to demonstrate documentation skills. The product, system names, commands, and specifications are fictional and for illustrative purposes only.*

---

# Planning and Configuring Platform Virtualization Zones

This guide covers how to plan and configure virtualization partitions in the CloudPlatform environment. It provides procedures for verifying hardware support, partition configuration, and applying resource constraints specific to virtualization partitions.

## Topics Covered

- [About Platform Virtualization Zones](#about-platform-virtualization-zones)
- [Virtualization and Standard Zones Concepts](#virtualization-and-standard-zones-concepts)
- [Hardware and Software Requirements](#hardware-and-software-requirements)
- [Verifying Hardware and Software Support](#verifying-hardware-and-software-support)
- [Tuning Host Memory Management](#tuning-host-memory-management)
- [Configuring a Virtualization Zone](#configuring-a-virtualization-zone)
- [Configuring and Customizing Zone Resources](#configuring-and-customizing-zone-resources)

---

## About Platform Virtualization Zones

A platform virtualization partition runs with a separate kernel and operating system installation from the host system. The separate kernel and OS installation provide for greater independence and enhanced security of operating system instances and applications.

The administrative and structural content of a virtualization partition is entirely independent from that of the host system. For example, a virtualization partition does not share system packages with the host, or the host kernel. Updates on the host are not propagated to zones and do not affect virtualization partitions. Similarly, package management commands are fully functional and isolated within a virtualization partition.

System processes are handled in the zone's separate process ID table and are not shared with the host system. Resource management in virtualization partitions is also different. Resource controls such as `max-processes` are not available when configuring a virtualization partition.

Use the existing `sysconnect`, `sysconfig`, and `sysadmin` commands to manage and administer virtualization partitions on the host system.

### Related Concepts

For more information about the virtualization framework, see the relevant platform documentation.

**Caution:** Running both hypervisor and virtualization partitions on the same system might cause system instability.

---

## Virtualization and Standard Zones Concepts

This documentation assumes you are familiar with the following resource management and partition concepts:

- Resource controls that determine how applications use available system resources
- Commands used to configure, install, and administer zones, primarily `sysconfig`, `sysadmin`, and `sysconnect`
- System configuration resources and property types
- Host zones and guest zones
- The whole-root guest zone model
- Authorizations granted through the `vmcfg` utility
- Host administrator and partition administrator
- The partition state model
- Partition isolation characteristics
- Network concepts and configuration
- Partition shared-IP and exclusive-IP types

For more information about these concepts, see the platform's virtualization documentation.

---

## Hardware and Software Requirements for Platform Virtualization Zones

To use virtualization partitions, you must be running CloudPlatform version 8.0 or later on your host operating system.

The physical host system must have the following components:

### Class-A (High-Performance) Systems

- A Class-A HS-series processor system with at least System Firmware 7.1 or later
- A Class-A HM-series processor system with at least System Firmware 7.4 or later
- A Class-A XL-series processor system with at least Hardware Controller 3.0 or later
- You can download the latest firmware from your platform vendor's support portal.

### Class-B (Standard) Systems

- A standard multi-core processor-based system with hardware virtualization capabilities enabled in system settings. Virtualization zones require support for advanced memory management features including nested paging or equivalent memory translation technologies.

### Both System Types Require

- A minimum of 8 GB of physical RAM
- The virtualization partition support package, `platform/virt/virtualization-support`

  For information on obtaining and installing software packages, see the platform's package management documentation.

- Sufficient tuning of the system memory management to prevent resource exhaustion. See [Tuning Host Memory Management](#tuning-host-memory-management).

Virtualization zones can run in guest instances on the platform hypervisor for Class-A systems. Each hypervisor instance for Class-A has an independent limit for the number of virtualization partitions that can run. The limit is 1024 for HS-series systems, and 512 for HM-series systems.

Virtualization zones cannot run in virtual machine guest environments or in containerized host systems.

---

## Verifying Hardware and Software Support on Virtualization Zone Hosts

Before planning and deploying a virtualization partition, you must verify that the host has the hardware and software requirements as described in [Hardware and Software Requirements](#hardware-and-software-requirements). You can use the `hostsysinfo` command to verify the hardware requirements, firmware or BIOS requirements, and virtualization partition brand package requirements on the host.

### How to Verify Virtualization Zone Support on a Host

1. **On the host, become a system administrator.**

   For more information, see your platform's documentation on administrative privileges.

2. **Verify that the CloudPlatform version is at least 8.0.**

   ```bash
   # sysinfo --version
   ```

   For example, on the host `primary`:

   ```bash
   primary# sysinfo --version
   CloudPlatform version 8.0 kernel 9.2
   ```

3. **Verify the installation of the virtualization support package, `platform/virt/virtualization-support`.**

   ```bash
   # softwares check platform/virt/virtualization-support
   ```

   The following example shows that the virtualization support package is installed on the host `primary`:

   ```bash
   primary# softwares check platform/virt/virtualization-support
   PACKAGE                                    VERSION           STATUS
   platform/virt/virtualization-support       8.0-2024.1        installed
   ```

4. **Run the `platforminfo` command.**

   ```bash
   # platforminfo --capabilities
   ```

   The following example output shows that virtualization partitions are supported on the host `primary`:

   ```bash
   primary# platforminfo --capabilities
   FEATURE                  SUPPORT
   host-domains             enabled
   standard-zones           available
   virtualization-zones     available
   ```

   See the `platforminfo(1M)` man page for further information.

---

## Tuning Host Memory Management

To ensure efficient performance, you must configure the host to limit system page cache usage. This value needs to be set only once on the host when you are planning your virtualization partition configuration.

**Caution:** Failure to limit host page cache can lead to out-of-memory failures.

To limit the page cache on the host, as a system administrator, set the `cache_max_size` parameter to the cache value in megabytes. The suggested value is one-half of what you would like the host memory pool to allocate to caching. For example, if you want the cache to use less than 2 GB of memory, set this parameter to 1024 MB, or `0x40000000`.

See your platform's memory management reference manual and caching documentation for further information.

You must reboot the host to have the changes take effect.

---

## Configuring a Virtualization Zone

This section describes how to configure a virtualization partition.

### How to Configure a Virtualization Zone

This procedure describes how to configure a virtualization partition using the virtualization partition template, `SYSvirt-zone`. For an overview of partition template properties, see your platform's partition template documentation. For general information regarding partition configuration, see your platform's zone creation and management documentation.

**Before You Begin**

Before you begin to configure a virtualization partition, you must confirm virtualization partition hardware support, software support, and memory configuration on your host system. See [Verifying Hardware and Software Support](#verifying-hardware-and-software-support) and [Tuning Host Memory Management](#tuning-host-memory-management).

**Steps**

1. **Become a system administrator.**

   For more information, see your platform's documentation on administrative privileges.

2. **Create a new virtualization partition configuration.**

   The virtualization partition template is `virt-standard`. For example, on the host `primary`, to create a new virtualization partition configuration for the partition named `vm1`:

   ```bash
   primary# sysconfig create vm1 --template virt-standard
   Zone configuration created for vm1
   ```

   The remaining configuration steps in this procedure use the virtualization partition `vm1`.

3. **Add any additional virtualization partition resources.**

   You can set some virtualization partition resources now or after the zone is configured. For more information, see [Configuring and Customizing Zone Resources](#configuring-and-customizing-zone-resources).

4. **Apply the partition configuration.**

   ```bash
   primary# sysconfig apply vm1
   ```

5. **Exit the configuration tool.**

   ```bash
   primary# exit
   ```

6. **(Optional) Verify the partition configuration.**

   You can verify a zone prior to installation. If you skip this step, the verification is performed automatically when you install the zone.

   ```bash
   # sysadmin verify zone vm1
   ```

   For example, to verify the virtualization partition `vm1` on the host `primary`:

   ```bash
   primary# sysadmin verify zone vm1
   Configuration verified successfully
   ```

   If you see an error message and the zone fails to verify, make the corrections specified in the message and try the command again. If no error messages are displayed, you can proceed with installation.

---

## Configuring and Customizing Zone Resources

Zone resources are mechanisms for managing machine, system, and CPU resources. Resources are set when planning a partition configuration. Note that some resources on virtualization partitions differ from what is available in other zone types. For example, there is no support for the `max-processes`, `fs-allowed`, and `network-type` resources in virtualization partitions.

This section describes how to configure resources to add additional support for the following components:

- Virtualization partition CPUs. See [Managing Zone CPUs](#managing-zone-cpus).
- Virtualization partition memory. See [Managing Zone Memory](#managing-zone-memory).
- Virtualization partition storage devices. See [Managing Zone Storage Devices](#managing-zone-storage-devices).
- Virtualization partition network devices and configuration. See [Managing Zone Network Devices](#managing-zone-network-devices).

You use the `sysconfig` command on the host system to set or modify virtualization partition resources.

**Note:** You must be the host administrator or a user with appropriate authorization on the host to use the `sysconfig` command.

See your platform's virtualization documentation and the `virt-zone(5)` man page for additional information about virtualization partition resources.

---

## Managing Zone CPUs

By default, a virtualization partition is given one virtual CPU upon creation. You can alter the number of virtual CPUs by adding and modifying the `vcp-allocation` resource.

Use the `cpu-affinity` system configuration property to pin a specific host CPU to the virtualization partition.

Note that if you have already defined the `cpu-affinity` property, the default number of virtual CPUs configured in the virtual platform matches the lower value of the range specified in the `cpu-affinity` setting. If both `vcp-allocation` and `cpu-affinity` are configured, they are cross-checked for consistency.

See your platform's documentation on partition resources for general information on how to set the `cpu-count` and `cpu-pinning` partition resources.

### Example 1-1: Adding Additional Virtual CPUs to a Zone

This example shows how to add additional virtual CPUs to the virtualization partition `vm1`.

```bash
primary# sysconfig edit vm1 --vcpus 8
Zone vm1 updated with 8 virtual CPUs
primary# sysconfig show vm1 --vcpus
vcp-allocation: 8
```

### Example 1-2: Adding CPU Affinity to a Zone

This example shows how to set CPU affinity for the virtualization partition `vm1`.

```bash
primary# sysconfig set vm1 cpu-affinity=0-7
CPU affinity set for zone vm1
primary# sysconfig show vm1 cpu-affinity
cpu-affinity: cores 0-7
```

---

## Managing Zone Memory

You must allocate a fixed amount of physical RAM to the virtualization partition virtual platform. You can define this amount by setting the `mem-pool` parameter.

The physical memory assigned to a virtualization partition is allocated in its entirety when it is configured. The memory allocated is only for the exclusive use of the virtualization partition. For example, once a virtualization partition is booted, all of the memory as specified in the `mem-pool` parameter appears to be in use to the host operating system.

On Class-B (Standard) systems, the `mem-pool` parameter must be set in increments of 4 megabytes (MB).

On Class-A (High-Performance) systems, the `mem-pool` parameter must be set in increments of 256 megabytes (MB).

The zone allocates the `memory-allocation` resource when the partition boots. This amount remains fixed while the partition is running.

See your platform's documentation on partition resources for general information on how to set the `memory-allocation` zone resource.

### Example 1-3: Setting Memory Allocation on a Class-A System

This example shows how to set the `mem-pool` parameter on a Class-A system.

```bash
primary# sysconfig set vm1 mem-pool=2048M
Memory pool set for zone vm1
primary# sysconfig show vm1 mem-pool
mem-pool: 2048M
```

### Example 1-4: Setting Memory Allocation on a Class-B System

This example shows how to set the `mem-pool` parameter on a Class-B system.

```bash
primary# sysconfig set vm1 mem-pool=16G
Memory pool set for zone vm1
primary# sysconfig show vm1 mem-pool
mem-pool: 16G
```

**Memory Size Considerations**

If partition memory size is increased prior to installation, you must also increase the zone root disk size to account for the larger swap and dump devices. If a virtualization partition does not have a disk explicitly added, a virtual disk is created and used as the root disk. By default, the virtual disk is 16GB in size. If a different root disk size is required, use the `sysadmin install vm1 --root-disk-size` option to modify the disk size. For example, to specify a 32GB root disk size on the virtualization partition `vm1`:

```bash
primary# sysadmin install vm1 --root-disk-size 32G
```

For additional information on setting the `mem-pool` parameter, see your platform's documentation on partition resources. For information on modifying the disk size using the `sysadmin` command, see the `sysadmin(1M)` man page.

---

## Managing Zone Storage Devices

A virtualization partition root storage is always accessible. You can add additional storage devices to a virtualization partition by using the `add storage` resource. Additional virtualization partition storage devices have the following requirements:

- The full storage device path (for example, `/dev/storage/disk9`) must be specified.

- The storage device must be defined by only one of the following:
  - The `add storage match` resource property. If you specify a storage device for the `add storage match` resource property, you must specify a device that is present in `/dev/storage`, `/dev/virtual/storage`, or `/dev/storage-id/`.
  - A valid storage URI.

- The storage device must be a whole disk or LUN.

Use the `boot-sequence` resource property to specify the boot order of each storage device. The `boot-sequence` resource property must be set to any positive integer value.

**Caution:** The `boot-sequence` resource property must be set only if the device is to be used as a boot device. If the `boot-sequence` property is set on devices other than boot devices, data corruption might result.

To unset the `boot-sequence` property, use the `clear boot-sequence` command.

If multiple bootable devices are present during installation, the devices will be used for a mirrored storage pool in the zone.

The default boot order of each device is determined by sorting devices first by `boot-order`, then by device ID if multiple devices have the same boot-order.

### Example 1-5: Adding Storage Devices to a Zone

This example shows how to add the storage device `/dev/storage/disk9` to the virtualization partition `vm1`.

```bash
primary# sysconfig attach vm1 storage /dev/storage/disk9
primary# sysconfig set vm1 storage.0.boot-sequence=4
Storage device attached to zone vm1
```

### Example 1-6: Changing the Zone Default Boot Device to Use a Remote Storage URI

This example shows how to change the default boot device on the virtualization partition `vm1` to use a remote storage URI located at `rstore://array-01/volume-a2f8c0x`.

```bash
primary# sysconfig set vm1 storage.primary.address=rstore://array-01/volume-a2f8c0x
primary# sysconfig show vm1 storage
storage:
  primary: rstore://array-01/volume-a2f8c0x
  id: 0
  boot-sequence: 0
```

---

## Managing Zone Network Devices

Virtualization zones provide network access by adding `net` or `anet` resources. See your platform's virtualization documentation for further information about these two resource types.

Exclusive-IP zones must be used for virtualization partitions. See your platform's zone documentation on exclusive-IP partition network configuration for more information about exclusive-IP zones.

You can supply additional MAC addresses to support nested zones, or zones where a virtualization partition hosts non-global child zones. See your platform's documentation on nested partition configuration for more information.

You can optionally specify a network device ID to identify the virtual network interface address from inside the zone and determine the order in which the network interfaces are presented to the virtualization partition. This process is similar to moving a NIC from one physical slot to another.

See your platform's documentation on partition resources for general information on how to set network partition resources.

### Example 1-7: Adding Network Resources to a Zone

This example shows how to add a network resource to the virtualization partition `vm1`. The interface ID is set to 3 to determine the order in which the new network interface is presented to the zone.

```bash
primary# sysconfig attach vm1 network --interface-id 3
Network interface attached to zone vm1
primary# sysconfig show vm1 network
interface-id: 3
```

### Example 1-8: Removing Network Devices From a Zone

This example shows how to remove a network device from the virtualization partition `vm1`. The information on the existing network interfaces is listed and interface ID 1 is detached.

```bash
primary# sysconfig show vm1 network --all
network:
  interface-id: 0
  link: auto
  mac-address: generated
  protection: enabled
  mtu: default

network:
  interface-id: 1
  link: auto
  mac-address: generated
  protection: enabled
  mtu: default

primary# sysconfig detach vm1 network --interface-id 1
Network interface detached from zone vm1
```

---

## Related Topics

- CloudPlatform virtualization documentation
- CloudPlatform zone administration guide
- CloudPlatform system administration reference manual

---

## Support

For questions about this documentation or technical issues, contact your CloudPlatform support team.

Last updated: Current

---

**NOTE:** This is example documentation created for portfolio demonstration purposes. All product names, commands, system specifications, and technical details are fictional and created to showcase technical writing skills, structure, and clarity. This documentation is not associated with any real product or system.
