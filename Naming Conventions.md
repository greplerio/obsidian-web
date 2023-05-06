---
sticker: 1f4d5
public: true
tags:
  - Public
  - site
  - zone
  - it_system
  - production
  - ZFS
---

#Public 

### Sites

Sites are not organized using city or other location identifiers (since we assume that the locations may change over time). We use a unique *index* for each site.

Thus, we refer to them as:

`Site 1 = S1
`Site 2 = S2`
`Site 3 = S3`
etc.`

For fixed locations we would use an acronym for the city, and the site index i.e. the fourth site, located in Montreal would be `MO4`. 

### Host Naming

Physical hosts (i.e., not LXC containers or VMs, but the actual brick and mortal computer systems that support the infrastructure) will have the following nomenclature:

`<SITE ID>-<CHASSIS ID>-<SYSTEM ID>`

The components are:
- `Site ID`: see above
- `Chassis ID`: a numerical identifier for the physical case/server frame the system resides within. Some servers have multiple blades/nodes, so we could have multiple hosts within a single 2U rack unit.
- `System ID`: An identifier for what the system is, comprising a function prefix and three-digit zero-padded index. See the list below.

Example: `S1-C001-PM001` = Site 1, Chassis 1, Proxmox system

| File                                                                                                | Functional Prefix |
| --------------------------------------------------------------------------------------------------- | ----------------- |
| [AlmaLinux](AlmaLinux.md#)                                     | ALN               |
| [MergerFS](MergerFS.md#)                                        | MFS               |
| [Nexus Respository OSS](Nexus%20Respository%20OSS.md#) | NREP              |
| [Proxmox](Proxmox.md#)                                           | PM                |
| [OpenMediaVault](OpenMediaVault.md#)                      | OMV               |



### VM Identifiers

The VMID should be a combination of:
- #site (as #)
- #zone  (as ##) 
	- For devices with *multiple* interfaces (like routers or gateways), the zone used as their **primary internet gateway** shall be selected for this code. This should be the first zone in the `zones` metadata attribute.
- a two-digit integer
	- generally different #it_system groups will get their own range of ints, so that they are grouped together, but this is just a guideline, not a hard-and-fast rule

For example, the primary firewall for [Site 1](Site%201.md#) is:
 - Site Octet: `= [Site 1](Site%201.md#).site_id` 
 - Primary Zone: `= [Site 1](Site%201.md#).zones[0].ip_octet`
 - VM Index: 01
Results in: `VMID = 10001



### Network Interfaces

While we have previously used Tailscale to mesh together our physical hosts and configured Proxmox SDN to facilitate communication between the systems, this introduced a SPoF ([Tailscale](./Tailscale.md#)) and introduced unncessary dependencies and complexity.

Our objectives (namely, [Like a Pheonix](./Design%20Document.md#like-a-pheonix), and [Self-contained](./Design%20Document.md#self-contained)) mean that we do not want our hypervisors to require any additional internet-connected software installs (like `openvswitch` of `ifupdown2`) after initial .ISO installation. 

The #production networking configurations must be such that they can be copied onto the system via ssh without any dependencies.


### Storage Pools and Datasets

Each host's storage pools (typically #ZFS) follow this naming pattern:

Fixed Operational pools:
 - `local-nvme`
 - `local-ssd`
 - `local-hdd`

Dedicated Backup disks:
 - `backup-hdd`
	 - this convention is still in development, since we want to be able to rotate this disks and store them offline for safe-keeping.



### Container Folder Structures

Each application stack gets it's own configuration folder.
Within, there will be the following:

 - `docker-compose.yml`
 - `*-variables.env` - container specific environment variable files, excluded from git.
 - `*-variables.env-example` - copy of the above env with sensitive information removed. saved to git.
 - `./container-data/` - where container bind mounts reside. Excluded from git.

