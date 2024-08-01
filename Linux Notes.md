## Hosts File Format [#](https://linuxize.com/post/how-to-edit-your-hosts-file/#hosts-file-format)

Entries in the hosts file have the following format:

```txt
IPAddress DomainName [DomainAliases]
```

The IP address and the domain names should be separated by at least one space or tab. The lines starting with `#` are comments and are ignored.

To add an entry to the hosts file, simply open the file in your text editor. Below is a sample hosts file:

```txt
# Static table lookup for hostnames.
# See hosts(5) for details.

127.0.1.1 linuxize.desktop linuxize
127.0.0.1 localhost
```

The hosts file changes take effect immediately except in cases where the DNS entries are cached by applications.

To undo the changes, simply open the file and remove the lines you added.

---

## Netplan static ip step by step instructions

To configure a static IP address on your Ubuntu server you need to find and modify a relevant netplan network configuration file. See the above section for all possible Netplan configuration file locations and forms. For example you might find there a default netplan configuration file called `50-cloud-init.yaml` with a following content instructing the `networkd` deamon to configure your network interface via DHCP:

```yaml
# This file describes the network interfaces available on your system
# For more information, see netplan(5).
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: yes
```

To set your network interface `enp0s3` to static IP address `192.168.1.222` with gateway `192.168.1.1` and DNS server as `8.8.8.8` and `8.8.4.4` replace the above configuration with the one below.

> [!warning] Warning:
> You must adhere to a correct code indent for each line of the block. In other words, the number of spaces before each configuration stanza matters. Otherwise you may end up with an error message similar to:
> 
> **Invalid YAML at //etc/netplan/01-netcfg.yaml line 7 column 6: did not find expected key**

```yaml
#network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: false
      dhcp6: false
      addresses:
      - 192.168.1.122/24
      routes:
      - to: default
        via: 192.168.1.1
      nameservers:
       addresses: [8.8.8.8,8.8.4.4]
```

> [!note] Note:
> Due to recent updates in network configuration methods, the error message "gateway4 has been deprecated, use default routes instead" signifies the need to substitute the outdated `gateway4` option with the `routes` configuration. Users should define default and specific routes within the network interface settings, as illustrated in the provided example, to ensure more flexible and straightforward network routing. This adjustment is integral to the continuous efforts to standardize and streamline network configurations.

To permanently disable cloud-init's network configuration capabilities on systems where configurations are typically overridden upon reboot, you can create a configuration file in the /etc/cloud/cloud.cfg.d/ directory. Use the command:

```bash
$ sudo bash -c 'echo "network: {config: disabled}" > /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg'
```

This will write the necessary settings directly. This action will ensure that changes to network configurations do not persist across reboots, effectively disabling cloud-init’s control over network settings. Creating this file informs cloud-init to leave network configurations as manually set, rather than resetting them based on the instance’s data source. It is crucial to implement this in environments where consistent, manual network configuration is necessary.

Once ready apply the new Netplan configuration changes with the following commands:
```bash
sudo netplan apply
```

In case you run into some issues execute:
```bash
sudo netplan --debug apply
```

