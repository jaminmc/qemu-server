# Proxmox VE QEMU Server

Proxmox VE QEMU Server (qemu-server) is the backend server component for managing and running KVM/QEMU virtual machines in Proxmox VE. It provides the core functionality for VM lifecycle management, configuration, and hardware emulation.

## Overview

The qemu-server package provides:
- **VM Configuration Management**: Define and manage virtual machine configurations
- **QEMU Wrapper**: Abstraction layer for QEMU/KVM functionality
- **Storage Integration**: Integration with Proxmox storage systems
- **Network Management**: Virtual network configuration and management
- **Snapshot Support**: VM snapshot creation and restoration
- **Migration**: Live VM migration capabilities
- **Resource Management**: CPU, memory, and I/O allocation
- **BIOS/UEFI Support**: SeaBIOS and OVMF firmware selection

## Features

### Core Capabilities

- **Multiple Storage Backends**: Support for local, NFS, iSCSI, Ceph RBD, ZFS, and more
- **Advanced Networking**: Virtual bridges, VLANs, and complex network topologies
- **High Availability**: HA integration for automatic failover
- **Performance Features**: CPU pinning, NUMA support, huge pages, IO threading
- **Security**: AppArmor integration, secure VM isolation
- **Backup Integration**: Integration with Proxmox Backup Server (PBS)

### BIOS and Firmware

#### SeaBIOS (Default)
Traditional BIOS support for maximum compatibility with legacy operating systems.

#### OVMF (UEFI)
Modern Unified Extensible Firmware Interface with support for:
- Secure Boot
- Measured Boot (TPM 2.0)
- Confidential VMs (SEV, SEV-ES, SEV-SNP, TDX)
- EFI variables storage on dedicated disk

### Storage Devices

#### Disk Interfaces
- **IDE**: Legacy parallel ATA (up to 4 devices)
- **SATA**: Serial ATA (up to 6 devices)
- **SCSI**: Small Computer System Interface (up to 31 devices)
- **VirtIO**: Paravirtual block device (up to 16 devices)

#### Advanced Features
- **Caching Modes**: write-back, write-through, none
- **Discard Support**: Thin provisioning support for sparse disks
- **I/O Throttling**: Bandwidth and IOPS limiting
- **Direct Passthrough**: Block device passthrough to guest

### Networking

- **Up to 32 Virtual Network Interfaces**
- **Multiple NIC Models**: virtio (recommended), e1000, rtl8139, vmxnet3, and more
- **VLAN Support**: Tagged and trunk configurations
- **Traffic Shaping**: Rate limiting and QoS
- **Firewall Integration**: Proxmox firewall rules per VM

### CPU Configuration

- **Flexible Topology**: Sockets, cores, and threads configuration
- **CPU Pinning**: Bind vCPUs to host CPUs
- **CPU Flags**: Customize exposed CPU flags
- **NUMA Support**: Virtual NUMA topology for large VMs
- **CPU Types**: Predefined models (kvm64, host) and custom types

### Memory Management

- **Balloon Driver**: Dynamic memory adjustment
- **Huge Pages**: 2MB and 1GB huge page support
- **KSM**: Kernel Samepage Merging for memory deduplication
- **Memory Hotplug**: Add memory without VM restart (requires NUMA)

### Advanced Features

- **Hot Plugging**: Add/remove CPU, memory, network, and disk without restart
- **Snapshots**: Freeze VM state for backup and recovery
- **Templates**: Create VM templates for rapid deployment
- **Cloud-Init**: Automated guest OS configuration
- **Guest Agent**: QEMU Guest Agent for enhanced integration
- **Watchdog Devices**: Hardware watchdog for automatic VM recovery
- **Spice Display**: Remote graphics with USB redirection support
- **TPM**: Virtual Trusted Platform Module support

## Installation

### From .deb Package
```bash
# Download the latest .deb package
apt install /path/to/qemu-server_9.5.0-jaminmc_amd64.deb
```

### From Source (This Repository)
```bash
git clone https://github.com/jaminmc/qemu-server.git
cd qemu-server
make
make install
```

## Configuration

VM configurations are stored as text files in `/etc/pve/qemu-server/` with the format `{vmid}.conf`.

### Basic Configuration Example

```
# VM ID 100 configuration
name: ubuntu-vm
memory: 2048
cores: 2
sockets: 1
ostype: l26

# Network
net0: virtio=12:34:56:78:90:AB,bridge=vmbr0

# Storage
scsi0: local:100/vm-100-disk-0.qcow2,size=20G
ide2: none,media=cdrom

# BIOS (using built-in OVMF)
bios: ovmf
efidisk0: local:100/vm-100-efi.qcow2

# Display
vga: std

# Settings
onboot: 1
autostart: 0
```

### Custom OVMF Configuration Example

```
name: secure-boot-vm
memory: 4096
cores: 4
sockets: 1
ostype: l26

# Network
net0: virtio=12:34:56:78:90:AB,bridge=vmbr0

# Storage
scsi0: local:100/vm-secure-disk-0.qcow2,size=50G

# Custom OVMF firmware with Secure Boot support
bios: ovmf,code=/opt/firmware/OVMF_CODE_secboot.fd,vars=/opt/firmware/OVMF_VARS.fd
efidisk0: local:100/vm-secure-efi.qcow2

# Advanced features
onboot: 1
acpi: 1
kvm: 1
```

### Custom OVMF with Storage Paths Example

```
name: custom-firmware-vm
memory: 8192
cores: 8

# Storage
scsi0: local:100/vm-custom-disk-0.qcow2,size=100G

# Custom OVMF from storage volumes
bios: ovmf,code=local:firmware-store/OVMF_CODE_4M.fd,vars=local:firmware-store/OVMF_VARS.fd
efidisk0: local:100/vm-custom-efi.qcow2
```

### Custom QEMU Binary Example

```
name: custom-qemu-vm
memory: 4096
cores: 4
sockets: 1
ostype: l26
arch: x86_64

# Network
net0: virtio=12:34:56:78:90:AB,bridge=vmbr0

# Storage
scsi0: local:100/vm-custom-qemu-disk.qcow2,size=50G

# Use custom QEMU binary (bypasses OS detection)
qemu_binary_x86_64: /usr/local/qemu-custom/bin/qemu-system-x86_64

# Display
vga: std

# Settings
onboot: 1
```

## Usage

### Command Line Interface (qm)

```bash
# Create a new VM
qm create 100 --name my-vm --memory 2048 --cores 2

# Start a VM
qm start 100

# Stop a VM
qm stop 100

# Edit VM configuration
qm set 100 --memory 4096

# Configure custom QEMU binary for x86_64
qm set 100 --qemu-binary-x86_64 '/usr/local/bin/qemu-custom'

# Configure custom QEMU binary with storage path
qm set 100 --qemu-binary-aarch64 'local:binaries/qemu-custom-aarch64'

# List VMs
qm list

# Show VM status
qm status 100

# Create a snapshot
qm snapshot 100 my-snapshot

# Delete a VM
qm destroy 100
```

### Configuration File Format

The configuration format supports:
- **Simple Values**: `memory: 2048`
- **Structured Values**: `net0: virtio=mac,bridge=vmbr0,rate=100`
- **Multi-line Values**: `args: -arg1 value1 -arg2 value2`

### BIOS Configuration Formats

```
# Simple format (unchanged from previous versions)
bios: seabios
bios: ovmf

# Extended format with custom paths (NEW)
# NOTE: 'code' and 'vars' must BOTH be specified together - they are an all-or-nothing pair
bios: ovmf,code=/path/to/code.fd,vars=/path/to/vars.fd

# Using storage paths
bios: ovmf,code=local:100/code.fd,vars=local:100/vars.fd

# Mixed filesystem and storage paths
bios: ovmf,code=/opt/firmware/CODE.fd,vars=local:200/VARS.fd
```

**Custom OVMF Behavior**:
- If neither `code` nor `vars` are specified: Uses built-in OVMF files
- If both `code` and `vars` are specified: Uses custom firmware files
- If only one is specified: Error - "Custom OVMF requires both 'code' and 'vars' to be specified"

### Custom QEMU Binary Configuration Formats

```
# Simple format - filesystem path (validate executable by default)
qemu_binary_x86_64: /usr/local/qemu/bin/qemu-system-x86_64

# With explicit validation control
qemu_binary_x86_64: /usr/local/qemu/bin/qemu-system-x86_64,validate=1
qemu_binary_x86_64: /usr/local/qemu/bin/qemu-system-x86_64,validate=0

# Using Proxmox storage paths
qemu_binary_x86_64: local:binaries/qemu-custom-x86_64
qemu_binary_aarch64: local:binaries/qemu-custom-aarch64

# Storage path with validation control
qemu_binary_x86_64: local:binaries/qemu-custom,validate=1
```

**Custom QEMU Binary Behavior**:
- Per-architecture configuration: `qemu_binary_x86_64` and `qemu_binary_aarch64` are independent
- When specified, completely overrides default binary selection for that architecture
- Validation enabled by default (checks executable permission)
- Set `validate=0` to skip executable check if needed
- Useful for custom QEMU builds or OS detection bypass

## API Integration

The qemu-server REST API (`PVE::API2::Qemu`) provides endpoints for:
- VM creation, deletion, and modification
- VM lifecycle management (start, stop, reset, etc.)
- Snapshot operations
- Migration and cloning
- Configuration retrieval and updates

### API Examples

```bash
# Get VM configuration
curl https://pve:8006/api2/json/nodes/node/qemu/100/config

# Update VM configuration
curl -X PUT https://pve:8006/api2/json/nodes/node/qemu/100/config \
  -d 'memory=4096'

# Update VM BIOS configuration
curl -X PUT https://pve:8006/api2/json/nodes/node/qemu/100/config \
  -d 'bios=ovmf,code=local:100/code.fd,vars=local:100/vars.fd'

# Update VM QEMU binary configuration
curl -X PUT https://pve:8006/api2/json/nodes/node/qemu/100/config \
  -d 'qemu_binary_x86_64=/usr/local/qemu/bin/qemu-system-x86_64'

# Create VM with custom OVMF
curl -X POST https://pve:8006/api2/json/nodes/node/qemu \
  -d 'vmid=100&name=my-vm&bios=ovmf,code=/path/code.fd,vars=/path/vars.fd'
```

## Supported Guest Operating Systems

- **Linux**: Debian, Ubuntu, Fedora, CentOS, Alpine, Arch, and others
- **Windows**: Windows 10, Windows 11, Windows Server 2016/2019/2022
- **BSD**: FreeBSD, OpenBSD, NetBSD
- **Other**: UNIX, Solaris, OpenIndiana

## Performance Tips

### CPU Performance
- Use host CPU type for best performance: `cpu: host`
- Enable CPU pinning for dedicated workloads
- Use appropriate thread counts (cores × sockets)

### Memory Performance
- Use huge pages for large VMs (1GB pages recommended)
- Enable KSM for memory overcommit scenarios
- Use NUMA for large multi-socket systems

### Storage Performance
- Prefer VirtIO for storage devices
- Use SSD backends for EFI disks
- Enable write-back caching for non-critical data
- Use appropriate disk formats (qcow2 for snapshots, raw for performance)

### Network Performance
- Use VirtIO for network interfaces
- Enable multiple queues for high-throughput scenarios
- Use dedicated network bridges for performance-critical VMs

## Security Considerations

### General
- Keep Proxmox VE and qemu-server updated
- Use AppArmor for additional process isolation
- Limit resource overcommit

### UEFI/Secure Boot
- Use OVMF with Secure Boot for enhanced security
- Store EFI variables on secure storage
- Keep firmware files with proper permissions

### Network Security
- Use firewall rules to restrict VM network access
- Isolate VMs on separate VLANs as needed
- Use encrypted storage for sensitive data

### Custom OVMF Files
- Verify firmware file integrity
- Use trusted firmware sources
- Store firmware files with appropriate permissions (644 or more restrictive)
- Keep firmware files backed up

### Custom QEMU Binaries
- Verify custom binary integrity and authenticity
- Only use binaries from trusted sources
- Store binaries with appropriate permissions (755 or more restrictive)
- Ensure custom binaries are regularly updated and patched
- Document why custom binaries are needed (e.g., OS detection bypass)
- Test custom binaries thoroughly before deploying to production

## Development

### Architecture
- **Backend**: Perl-based server components
- **Storage**: Abstraction layer for multiple storage backends
- **Network**: Linux bridge and VLAN integration
- **QEMU**: Direct wrapper around QEMU/KVM

### Contributing
1. Fork the repository on GitHub
2. Create a feature branch
3. Make your changes with clear commit messages
4. Submit a pull request
5. Ensure tests pass and code follows conventions

### Testing
```bash
# Run tests
make test

# Validate Perl syntax
perl -c src/PVE/QemuServer.pm
```

## Documentation

- **Official Proxmox VE Documentation**: https://pve.proxmox.com/pve-docs/
- **QEMU Documentation**: https://www.qemu.org/documentation/
- **KVM Documentation**: https://www.linux-kvm.org/

## License

Proxmox VE QEMU Server is released under the GNU General Public License (GPL) version 3 or later.

## Support

- **Community Forum**: https://forum.proxmox.com/
- **Issue Tracker**: https://bugzilla.proxmox.com/
- **GitHub Repository**: https://github.com/proxmox/qemu-server

## Custom OVMF CLI Examples

```bash
# Using filesystem paths
qm set 100 --bios 'ovmf,code=/opt/firmware/OVMF_CODE.fd,vars=/opt/firmware/OVMF_VARS.fd'

# Using Proxmox storage paths
qm set 100 --bios 'ovmf,code=local:100/custom-code.fd,vars=local:100/custom-vars.fd'

# Mixed paths
qm set 100 --bios 'ovmf,code=/opt/firmware/CODE.fd,vars=local:200/efi-vars.fd'
```

## Troubleshooting Custom OVMF

**Error: "Custom OVMF requires both 'code' and 'vars' to be specified"**
- This error means you specified only one of the two parameters
- **Solution**: Specify both `code` and `vars` together, or neither (to use defaults)
- Example: `bios: ovmf,code=/path/code.fd,vars=/path/vars.fd` ✅
- Invalid: `bios: ovmf,code=/path/code.fd` ❌
- Valid alternative: `bios: ovmf` (uses built-in firmware) ✅

**Why both are required**:
- `code` is the firmware binary (read-only after boot)
- `vars` is the EFI variables storage (read-write during runtime)
- They must be paired to ensure compatibility and proper UEFI operation
- Mismatched versions could cause boot failures

**Error: "OVMF_code file does not exist at '/path/to/code.fd'"**
- Verify the file path is correct
- Check file permissions (should be readable)
- For storage paths, verify the storage volume exists

**Error: "Invalid OVMF_vars storage path 'invalid:path'"**
- Check storage identifier format (should be `storeid:path/to/file.fd`)
- Verify the storage is configured in Proxmox
- Ensure the file exists in that storage

### General VM Issues

**VM won't start**
- Check VM logs: `journalctl -xe`
- Verify QEMU configuration: `qm config {vmid}`
- Check host resources (CPU, memory, storage)

**Performance issues**
- Monitor resource usage: `qm status {vmid}`
- Check I/O throttling settings
- Verify storage backend performance

**Network connectivity**
- Verify bridge configuration: `ip link show`
- Check VLAN settings if applicable
- Test connectivity from host

## Custom QEMU Binary CLI Examples

```bash
# Using filesystem path for x86_64
qm set 100 --qemu-binary-x86_64 '/usr/local/bin/qemu-custom-x86_64'

# With validation enabled (default)
qm set 100 --qemu-binary-x86_64 '/usr/local/bin/qemu-custom,validate=1'

# Using Proxmox storage path
qm set 100 --qemu-binary-x86_64 'local:binaries/qemu-custom-x86_64'

# aarch64 architecture
qm set 100 --qemu-binary-aarch64 '/usr/local/bin/qemu-custom-aarch64'
```

## Troubleshooting Custom QEMU Binary

**Error: "QEMU binary does not exist at '/path/to/qemu'"**
- Verify the binary path is correct and absolute
- Check that the file exists on the filesystem
- For storage paths, verify the storage volume exists and contains the file

**Error: "QEMU binary is not executable at '/path/to/qemu'"**
- Check file permissions: `ls -la /path/to/qemu`
- Make the binary executable: `chmod +x /path/to/qemu`
- Alternatively, disable validation: `qemu_binary_x86_64: /path/to/qemu,validate=0`

**Error: "don't know how to emulate architecture 'xyz'"**
- The custom binary path wasn't found or couldn't be resolved
- Verify the binary exists for the target architecture (x86_64 or aarch64)
- Check storage path format if using Proxmox storage

**VM fails to start with custom QEMU binary**
- Ensure the custom binary is compatible with the VM's architecture
- Check VM logs: `journalctl -xe`
- Verify the binary works standalone: `/path/to/qemu --version`
- Ensure the binary has all required dependencies installed

## Changelog

### Version 9.5.0-jaminmc
- **NEW**: Custom OVMF firmware configuration support
  - Specify custom OVMF_CODE and OVMF_VARS files via configuration
  - Support for both filesystem and storage paths
  - Automatic path resolution and validation
  - Fully backward compatible with existing configurations
- **NEW**: Custom QEMU binary support per architecture
  - Specify alternative QEMU binaries for x86_64 and aarch64
  - Support for filesystem and storage volume paths
  - Executable validation (configurable)
  - Useful for custom builds and OS detection bypass
  - Per-architecture configuration (independent for each architecture)

### Version 9.1.5
- Fix guest shutdown hookscript
- Query CPU flags for aarch64 host support
- Various bug fixes and improvements

### Previous Versions
See git history for detailed changelog: `git log --oneline`

---

**Maintained by**: Proxmox Community (Modified by JaminMc)  
**Latest Version**: 9.5.0-jaminmc  
**Last Updated**: March 19, 2026
