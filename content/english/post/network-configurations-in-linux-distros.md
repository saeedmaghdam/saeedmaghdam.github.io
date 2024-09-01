---
title: "Network Configurations in Linux Distros"
date: 2024-09-01T11:46:47+02:00
Description: ""
Tags: []
Categories: []
DisableComments: false
---

## Changing IP Address

Changing the IP address on a Linux system can be done in several ways, depending on whether you want to change it temporarily (for the current session) or permanently (so it persists across reboots). The method you choose will depend on your specific needs and the environment you're working in.

### Methods to Change the IP Address:

1. **Using the `ip` Command**:
   - **Temporary Change**: The `ip` command can change the IP address for the current session.
   - **Command**:
     ```bash
     sudo ip addr add <new-ip-address>/<subnet-mask> dev <interface>
     sudo ip addr del <old-ip-address>/<subnet-mask> dev <interface>
     ```
   - **Example**:
     ```bash
     sudo ip addr add 192.168.1.100/24 dev eth0
     sudo ip addr del 192.168.1.50/24 dev eth0
     ```
   - **Persistent?**: No, this change will be lost after a reboot.

2. **Using the `ifconfig` Command**:
   - **Temporary Change**: Similar to the `ip` command but older and less powerful.
   - **Command**:
     ```bash
     sudo ifconfig <interface> <new-ip-address> netmask <subnet-mask>
     ```
   - **Example**:
     ```bash
     sudo ifconfig eth0 192.168.1.100 netmask 255.255.255.0
     ```
   - **Persistent?**: No, this change will also be lost after a reboot.

3. **Editing Network Configuration Files**:
   - **Permanent Change**: This method ensures the IP address change persists across reboots.
   - **For Ubuntu/Debian**:
     - Edit the `/etc/network/interfaces` file:
       ```bash
       sudo nano /etc/network/interfaces
       ```
     - Example configuration:
       ```bash
       auto eth0
       iface eth0 inet static
           address 192.168.1.100
           netmask 255.255.255.0
           gateway 192.168.1.1
       ```
     - Apply changes:
       ```bash
       sudo systemctl restart networking
       ```
   - **For Red Hat/CentOS/Fedora**:
     - Edit the configuration file for your interface in `/etc/sysconfig/network-scripts/` (e.g., `ifcfg-eth0`):
       ```bash
       sudo nano /etc/sysconfig/network-scripts/ifcfg-eth0
       ```
     - Example configuration:
       ```bash
       BOOTPROTO=static
       IPADDR=192.168.1.100
       NETMASK=255.255.255.0
       GATEWAY=192.168.1.1
       ```
     - Apply changes:
       ```bash
       sudo systemctl restart network
       ```

4. **Using Network Manager (for GUI-based environments)**:
   - **Temporary or Permanent Change**: This method is more common in desktop environments or systems using `NetworkManager`.
   - **Command Line with `nmcli`**:
     ```bash
     sudo nmcli con mod <connection-name> ipv4.addresses <new-ip-address>/<subnet-mask>
     sudo nmcli con up <connection-name>
     ```
   - **GUI Method**:
     - Go to Network settings.
     - Select the network interface.
     - Modify the IPv4 settings to static, then enter the new IP address, subnet mask, and gateway.
   - **Persistent?**: Yes, changes made through `nmcli` or the GUI will persist after reboots.

### Choosing the Right Method:

- **Temporary vs. Permanent**:
  - If you need a **temporary** change (e.g., for testing or troubleshooting), use the `ip` or `ifconfig` command.
  - For a **permanent** change (to ensure the IP address stays the same after reboot), edit the appropriate network configuration files or use Network Manager.

- **Distribution**:
  - On **Ubuntu/Debian** systems, the `/etc/network/interfaces` method is commonly used, although newer systems may rely on `netplan` (which uses YAML configuration files).
  - On **Red Hat/CentOS/Fedora** systems, you typically edit `/etc/sysconfig/network-scripts/`.

- **GUI vs. Command Line**:
  - If you prefer a **GUI** and are using a desktop environment, the Network Manager tool (accessible through system settings) might be the easiest option.
  - For **servers or command-line only** environments, editing the network configuration files or using `nmcli` is recommended.

### Additional Considerations:
- **DHCP vs. Static**: Ensure you’re aware of whether your system should be using a DHCP or a static IP. Switching from DHCP to static may require additional configuration, such as setting up DNS servers.
- **Network Services**: After changing the IP address, you may need to restart network services or the system to apply changes fully.
- **Compatibility**: Ensure that the method you choose is compatible with your Linux distribution and version, as there might be slight variations in the file paths or commands.

By choosing the right method based on these factors, you can effectively change the IP address on your Linux system to suit your needs.

## Configure DNS Servers

Setting DNS servers in Linux can be done in several ways, depending on your distribution, system configuration, and whether the change should be temporary or permanent. Here's a breakdown of the different methods and how to choose the best one for your needs.

### Methods to Set DNS Servers in Linux

1. **Modifying `/etc/resolv.conf` Directly**:
   - **Description**: The `/etc/resolv.conf` file contains the DNS server addresses that the system will use for resolving domain names.
   - **Command**:
     ```bash
     sudo nano /etc/resolv.conf
     ```
   - **Example**:
     ```bash
     nameserver 8.8.8.8
     nameserver 8.8.4.4
     ```
   - **Persistent?**: No, this method is not persistent across reboots if your system uses a service like `NetworkManager`, `systemd-resolved`, or DHCP, which may overwrite this file.

2. **Using Network Manager**:
   - **GUI Method**:
     - Go to Network settings in your desktop environment.
     - Select your connection, then go to the "IPv4" or "IPv6" settings.
     - Set the DNS servers under the DNS field.
   - **Command Line with `nmcli`**:
     ```bash
     sudo nmcli con mod <connection-name> ipv4.dns "8.8.8.8 8.8.4.4"
     sudo nmcli con up <connection-name>
     ```
   - **Persistent?**: Yes, changes made through `NetworkManager` will persist across reboots.
   - **Note**: This is common in desktop environments or systems where NetworkManager manages the network settings.

3. **Using `systemd-resolved`**:
   - **Description**: `systemd-resolved` is a service that provides network name resolution to local applications.
   - **Edit Configuration**:
     ```bash
     sudo nano /etc/systemd/resolved.conf
     ```
   - **Example**:
     ```bash
     [Resolve]
     DNS=8.8.8.8 8.8.4.4
     FallbackDNS=1.1.1.1 1.0.0.1
     ```
   - **Apply Changes**:
     ```bash
     sudo systemctl restart systemd-resolved
     ```
   - **Link to `/etc/resolv.conf`**:
     ```bash
     sudo ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf
     ```
   - **Persistent?**: Yes, changes are persistent and managed by `systemd-resolved`.

4. **Editing Interface-Specific Configuration Files**:
   - **For Ubuntu/Debian** (with `netplan`):
     - **Edit Netplan Configuration**:
       ```bash
       sudo nano /etc/netplan/01-netcfg.yaml
       ```
     - **Example**:
       ```yaml
       network:
         version: 2
         ethernets:
           eth0:
             dhcp4: yes
             nameservers:
               addresses: [8.8.8.8, 8.8.4.4]
       ```
     - **Apply Changes**:
       ```bash
       sudo netplan apply
       ```
   - **For Red Hat/CentOS/Fedora**:
     - **Edit Interface Configuration**:
       ```bash
       sudo nano /etc/sysconfig/network-scripts/ifcfg-eth0
       ```
     - **Example**:
       ```bash
       DNS1=8.8.8.8
       DNS2=8.8.4.4
       ```
     - **Restart Network Service**:
       ```bash
       sudo systemctl restart network
       ```
   - **Persistent?**: Yes, these changes are persistent across reboots.

5. **Configuring DHCP Client**:
   - **Description**: If you're using DHCP and want to configure specific DNS servers, you can set this in the DHCP client configuration.
   - **For `dhclient`**:
     - **Edit the configuration**:
       ```bash
       sudo nano /etc/dhcp/dhclient.conf
       ```
     - **Example**:
       ```bash
       supersede domain-name-servers 8.8.8.8, 8.8.4.4;
       ```
   - **Restart Network Interface**:
     ```bash
     sudo dhclient -r && sudo dhclient
     ```
   - **Persistent?**: Yes, this method is persistent and instructs the DHCP client to use specified DNS servers instead of those provided by the DHCP server.

### Choosing the Right Method:

1. **Temporary vs. Permanent**:
   - If you need a **temporary change** for testing or troubleshooting, directly editing `/etc/resolv.conf` might be sufficient, but it won't persist across reboots.
   - For a **permanent solution**, use Network Manager, `systemd-resolved`, or modify the interface-specific configuration files.

2. **Distribution and Configuration Management**:
   - **Ubuntu/Debian**: If you’re using Netplan or `systemd-resolved`, follow those specific methods. Older systems or minimal installations may still use the traditional `/etc/network/interfaces` method.
   - **Red Hat/CentOS/Fedora**: Configuration typically happens within `/etc/sysconfig/network-scripts/`. If you're using `NetworkManager`, that should be your go-to method.

3. **Network Manager vs. Manual Configuration**:
   - If **NetworkManager** is managing your network, use `nmcli` or the GUI. It’s the preferred tool as it manages all aspects of networking on systems where it's enabled.
   - For **servers or non-GUI environments**, where NetworkManager may not be installed, manual configuration of the network scripts or using `systemd-resolved` is more appropriate.

4. **Consistency and Control**:
   - If you want full control over DNS settings and consistency across different networking tools and environments, consider using `systemd-resolved`. It centralizes DNS resolution and can be more robust in complex setups.

5. **DHCP Considerations**:
   - If your system receives IP configuration via **DHCP** but you need to override the DNS settings, modify the DHCP client configuration (`dhclient.conf`) to enforce the use of specific DNS servers.

By selecting the method that best matches your system setup, administrative practices, and persistence needs, you can effectively manage DNS server settings on your Linux system.