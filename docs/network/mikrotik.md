---
authors:
  - harry
---
# Mikrotik

## Firewall

### Chain

* forward - **Used to process packets passing through the router**
* input - **Used to process packets entering the router through one of the interfaces with the destination IP address which is one of the router's addresses. Packets passing through the router are not processed against the rules of the input chain**
* output - **Used to process packets originated from the router and leaving it through on of the interfaces. Packets passing through the router are not process against the rules of the output chain**
* It is also possible to create own chains

## DNS

### DOH setup with Quad9

[Quad9](https://docs.quad9.net/Setup_Guides/Open-Source_Routers/MikroTik_RouterOS_%28Encrypted%29/)

### Update DNS Records from DHCP Leases

see [MikroTik Script for Automatic DNS Records from DHCP Leases](https://blog.pessoft.com/2019/09/06/mikrotik-script-automatic-dns-records-from-dhcp-leases/)

Add the script to IP -> DHCP Server -> DHCP (tab) -> DHCP server instance -> Script (tab) -> Lease Script and change the `:local dnsDomain "dynamic.example.local"`

```sh
# When "1" all DNS entries with IP address of DHCP lease are removed
:local dnsRemoveAllByIp "1"
# When "1" all DNS entries with hostname of DHCP lease are removed
:local dnsRemoveAllByName "1"
# When "1" addition and removal of DNS entries is always done also for non-FQDN hostname
:local dnsAlwaysNonfqdn "1"
# DNS domain to add after DHCP client hostname
:local dnsDomain "dynamic.example.local"
# DNS TTL to set for DNS entries
:local dnsTtl "00:15:00"
# Source of DHCP client hostname, can be "lease-hostname" or any other lease attribute, like "host-name" or "comment"
:local leaseClientHostnameSource "lease-hostname"

:local leaseComment "dhcp-lease-script_$leaseServerName_$leaseClientHostnameSource"
:local leaseClientHostname
:if ($leaseClientHostnameSource = "lease-hostname") do={
  :set leaseClientHostname $"lease-hostname"
} else={
  :set leaseClientHostname ([:pick \
    [/ip dhcp-server lease print as-value where server="$leaseServerName" address="$leaseActIP" mac-address="$leaseActMAC"] \
    0]->"$leaseClientHostnameSource")
}
:local leaseClientHostnames "$leaseClientHostname"
:if ([:len [$dnsDomain]] > 0) do={
  :if ($dnsAlwaysNonfqdn = "1") do={
    :set leaseClientHostnames "$leaseClientHostname.$dnsDomain,$leaseClientHostname"
  } else={
    :set leaseClientHostnames "$leaseClientHostname.$dnsDomain"
  }
}
:if ($dnsRemoveAllByIp = "1") do={
  /ip dns static remove [/ip dns static find comment="$leaseComment" and address="$leaseActIP"]
}
:foreach h in=[:toarray value="$leaseClientHostnames"] do={
  :if ($dnsRemoveAllByName = "1") do={
    /ip dns static remove [/ip dns static find comment="$leaseComment" and name="$h"]
  }
  /ip dns static remove [/ip dns static find comment="$leaseComment" and address="$leaseActIP" and name="$h"]
  :if ($leaseBound = "1") do={
    :delay 1
    /ip dns static add comment="$leaseComment" address="$leaseActIP" name="$h" ttl="$dnsTtl"
  }
}
```

## IPv6 Networking

see https://administrator.de/tutorial/ipv6-mittels-prefix-delegation-bei-pppoe-mikrotik-632633.html

### DHCPv6 Client Settings - Telekom

* Interface: pppoe-out1
* Request: prefix
* Pool Name: pool-ipv6
* Pool Prefix Length: 64
* Check: Use Peer DNS, Rapid Commit, Add Default Route

!!! warning
    * Pool Prefix Length must be set to 64!!!
    * From https://forum.mikrotik.com/viewtopic.php?t=153437#p757747
    * *In other words, if the ISP gives you a /56 and you set "Pool Prefix Length" to 56, you are telling the MikroTik to create a pool of /56 subnets that just has a single /56 in it (since the ISP only gives you one). -> I think what you actually want is a "Pool Prefix Length" of 64 which means that when the ISP gives you a /56 you slice that up into 256 individual /64's which go into the pool.*

-> Check Status

### Set IP Address

* `/ipv6 address add address=::1/64 from-pool=pool-ipv6 interface=bridge`
* `/ipv6 address add address=::1/64 from-pool=pool-ipv6 interface=<VLAN>`

## Wireguard VPN

### EXTFW - OPNSense

* Public IP Address: vpn.dmdy.de

#### WireGuard Instance

* Name: wgEXTFW
* Public key: tcJ....
* Private key: *****
* Listen Port: 51280
* Tunnel address: 172.16.0.1/24

### router - MikroTik

### WireGuard - Interface

* Name: wgEXT
* Type: WireGuard
* MTU: 1420
* Public Key: LOx..............
