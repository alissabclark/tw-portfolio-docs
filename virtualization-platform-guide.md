---
***EXAMPLE DOCUMENTATION - FOR PORTFOLIO PURPOSES ONLY***

*This is a sample technical documentation guide created to demonstrate documentation skills. The product, system names, commands, and specifications are fictional and for illustrative purposes only.*

---

# Planning and Configuring Platform Virtualization Zones

This guide covers how to plan and configure virtualization partitions in the CloudPlatform environment. It provides procedures for verifying hardware support, partition configuration, and applying resource constraints specific to virtualization partitions.

## Topics Covered

- [About Platform Virtualization Zones](#about-platform-virtualization-zones)
- [Hardware and Software Requirements](#hardware-and-software-requirements)
- [Hardware Command Reference by System Class](#hardware-command-reference-by-system-class)
- [Verifying Hardware and Software Support](#verifying-hardware-and-software-support)
- [Tuning Host Memory Management](#tuning-host-memory-management)
- [Configuring a Virtualization Zone](#configuring-a-virtualization-zone)
- [Managing Zone Resources](#managing-zone-resources)
- [Installing the Zone and Adjusting Root Disk Size](#installing-the-zone-and-adjusting-root-disk-size)

---

## About Platform Virtualization Zones

A platform virtualization partition runs with a separate kernel and operating system installation from the host system. The separate kernel and OS installation provide greater independence and enhanced security for operating system instances and applications.



The administrative and structural content of a virtualization partition remains entirely independent from the host system. For example, a virtualization partition does not share system packages or the kernel with the host. Updates on the host do not propagate to zones and do not affect virtualization partitions. Similarly, package management commands remain fully functional and isolated within a virtualization partition.

The separate process ID table for the zone handles system processes, which the system does not share with the host. Resource management in virtualization partitions also follows a different model. Resource controls, such as `max-processes`, are unavailable during the configuration of a virtualization partition.

---

## Hardware and Software Requirements

The host operating system requires **CloudPlatform version 8.0** or later to support virtualization partitions. The physical host system must include the following components:

### Class-A (High-Performance) Systems
* A Class-A HS-series processor system requires **System Firmware 7.1** or later.
* A Class-A HM-series processor system requires **System Firmware 7.4** or later.
* A Class-A XL-series processor system requires **Hardware Controller 3.0** or later.

### Class-B (Standard) Systems
* A standard multi-core processor-based system requires hardware virtualization capabilities enabled in the system settings.
* Virtualization zones require support for advanced memory management features, including **nested paging** or equivalent memory translation technologies.

### General Requirements
* The system requires a minimum of **8 GB of physical RAM**.
* The host requires the virtualization partition support package, **`platform/virt/virtualization-support`**.
* The administrator performs sufficient tuning of the system memory management to prevent resource exhaustion.

---

## Hardware Command Reference by System Class

The following table lists the specific hardware and firmware requirements alongside the commands used to verify the host configuration.

| System Class | Processor Series | Minimum Requirement                     | Verification Command          |
| :----------- | :--------------- | :-------------------------------------- | :---------------------------- |
| **Class-A**  | HS-Series        | System Firmware 7.1                     | `hostsysinfo`                 |
| **Class-A**  | HM-Series        | System Firmware 7.4                     | `hostsysinfo`                 |
| **Class-A**  | XL-Series        | Hardware Controller 3.0                 | `hostsysinfo`                 |
| **Class-B**  | Multi-core       | Hardware Virtualization / Nested Paging | `platforminfo --capabilities` |

---

## Verifying Hardware and Software Support

Before planning and deploying a virtualization partition, the administrator verifies that the host meets all hardware and software requirements. The **`hostsysinfo`** command verifies the hardware, firmware, or BIOS requirements, as well as the virtualization partition brand package requirements on the host.

### Steps to Verify Support
1. **Become a system administrator** on the host.
2. **Verify** that the CloudPlatform version is at least 8.0 by running `sysinfo --version`.
3. **Check** for the installation of the virtualization support package by using `softwares check platform/virt/virtualization-support`.
4. **Run** the `platforminfo --capabilities` command to confirm that virtualization partitions are "available".

---

## Tuning Host Memory Management

To ensure efficient performance, the administrator configures the host to limit system page cache usage.

### Steps to Tune Memory
1. **Set** the `cache_max_size` parameter to the desired cache value in megabytes.
2. **Reboot** the host to allow the changes to take effect.

---

## Configuring a Virtualization Zone

The configuration process uses the `sysconfig` utility and the `virt-standard` template.

### Steps to Configure a Zone
1. **Become a system administrator**.
2. **Create** a new virtualization partition configuration through the `sysconfig create [zone-name] --template virt-standard` command.
3. **Add** any additional virtualization partition resources as required for the deployment.
4. **Apply** the partition configuration using `sysconfig apply [zone-name]`.
5. **Exit** the configuration tool.
6. **Verify** the partition configuration using `sysadmin verify zone [zone-name]` before installation.

---

## Managing Zone Resources

The administrator uses the `sysconfig` command on the host system to set or modify virtualization partition resources.

### CPUs and Memory
* **CPUs**: By default, the system provides a virtualization partition with one virtual CPU. The administrator alters the number of virtual CPUs by modifying the **`vcp-allocation`** resource.
* **Memory**: The administrator allocates a fixed amount of physical RAM to the virtualization partition by setting the **`mem-pool`** parameter. On **Class-B systems**, the administrator sets the parameter in increments of **4 MB**. On **Class-A systems**, the administrator uses increments of **256 MB**.

### Storage Management
The administrator adds storage devices through the **`attach storage`** command. If the partition requires multiple bootable devices, the system creates a mirrored storage pool during installation.

**Steps to Add a Storage Device**
1. **Identify** the full path of the storage device, such as `/dev/storage/disk9`.
2. **Attach** the device to the zone by using `sysconfig attach [zone-name] storage [device-path]`.
3. **Set** the boot priority if necessary through the `sysconfig set [zone-name] storage.[ID].boot-sequence=[value]` command.

### Network Management
Virtualization partitions require **exclusive-IP** network configurations. The administrator determines the order of network interfaces by assigning a specific network interface ID.

**Steps to Add a Network Interface**
1. **Attach** a network resource by using `sysconfig attach [zone-name] network --interface-id [number]`.
2. **Verify** the network settings through the `sysconfig show [zone-name] network` command.

---

## Installing the Zone and Adjusting Root Disk Size

The installation process creates the virtual environment and installs the operating system kernel. The default configuration allocates a standard size for the root disk, but the administrator can modify this during the initial installation.

**Steps to Install the Zone**
1. **Start** the installation by using the `sysadmin install [zone-name]` command.
2. **Specify** a custom root disk size by adding the `--disk-size` flag followed by the value in gigabytes.
3. **Monitor** the installation progress through the system console.

**Example Command:**
`sysadmin install [zone-name] --disk-size 40G`
