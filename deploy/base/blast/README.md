# Blast core configuration

The "core" of blast, an system to auto-provision hardware.  A few notes:

- Assumes you wish to run a proxy DHCP, with another primary DHCP server
  used for most activity
- The hostname for "matchbox.<domain>.<tld>" must be resovlable by the DNS
  provided by your primary DHCP server.  You could leverage DNS by your primary
  cluster, depending on your home equipment.  For a unifi system the following command
  can be used to forward requests:

```sh
configure
set service dns forwarding options server=/r15cookie.lan/172.18.110.81
commit
save
```

## Templating System

With no adjustments, a default blast cluster will provide a system that will boot live k3os 
PXE systems.  While this can be used for manually installation, it's more efficient to allow
automatic installation.  The basic system.

- Matchbox has a series of configmaps that contain the default files, as well as templates for use.
Those configmaps are
  - matchbox-groups-base: Definitions to match MAC addresses to profiles, as well as metadata that can be used in templates.
  - matchbox-profiles-base:  Profiles that specify boot parameters
  - matchbox-ignition-base:  Profile configuration files for the k3os installer (or any other installer).   
- Customization can be made by provided a few customized configmaps/secrets in your own kustomization
  - PVC can be overridden to use longhorn if pernament storage of the "assets" volumn is required
  - Env:
    redownload:  If set to true, then will redownload assets, even if they exist.  Otherwise will skip if already downloaded
  - Configmap: 
    - matchbox-assets-url:  A name/value pair of name:URL that will be auto-downloaded to prepopulate the assets local store 
  - Secrets:
    - ipparams: name:value paris.  Names below
      - ipcidr:  Network Address and cidr.  For instance, 192.168.1.0/24
      - startaddr:  Starting network address for DHCP, if offered.  i.e: 192.168.1.100
      - endaddr:  Ending address for DHCP, if offered.  i.e.: 192.168.1.200
      - dns:  DNS server(s), if offered, comma seperated.  i.e.: 8.8.8.8, 8.8.4.4
      - bootstrap-host:  hostname of bootstrap host (this HAS to exist in host table)
    - hosts: name:value pair of host:ip  Note that bootstrap host HAS to be defined.
    - mac: name:value pair matching mac:host
    - profile: name:value pair mapping host:profile.  Profile can be either controller or worker.
  - An initContainer that does the following:
    - Checks matchbox-assets-url, and downloads any if necessary
    - Populates the real configures.
      - Copies base configs
      - Populate real DNS configurated based on hosts table (used for dns-masq DNS)
      - Populate matchbox-groups based on profile pair

## Virtualbox testing

The build in iPXE system didn't appear to work with Matchbox's changed iPXE.  To get around

1. Download [iPXE ISO](http://boot.ipxe.org/ipxe.iso)
2. Boot from ISO
3. Break into iPXE command line, and enter the following commands:

```sh
dhcp
chain http://matchbox.r15cookie.lan/ipxe
```

## Notes

- [Matchbox Concepts](https://matchbox.psdn.io/matchbox/)
