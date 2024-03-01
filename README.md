# Using SONiC on AIR 
General notes on using the automation in this repository to deploy SONiC L3 routed BGP configs.  

Sections broken into the following categories:
1. General Notes
2. Scope of Automation Code
3. Configuration Backup
4. Configuration Restore
5. Verification of Demo

## General Notes

The SONiC image on AIR currently uses statically defined interface naming conventions.  
In the future, the hard coded requirements will be changed to be more flexible with custom topologies.  
Today, use the below interface naming/mapping as a reference for which `physical` interface from the Linux OS maps to the SONiC network interface.

**Static Interface Naming:**  
- Ethernet0
- Ethernet4
- Ethernet8
- Ethernet(N+4) ...

**Physical Interface to SONiC Interface Mapping**
- eth1 <---> Ethernet0
- eth2 <---> Ethernet4
- eth3 <---> Ethernet8
- eth(N+1) <---> Ethernet(N+4)

**SONiC Default Login:**  

SSH Keys are automatically deployed in this demo so that SSH from the `oob-mgmt-server` is streamlined to each of the SONiC nodes. For completions sake, below is the default username and password to access the SONiC nodes:

| Credential | Value |
|----------|:--------|
| Username | admin |
| Password | YourPaSsWoRd |


## Scope of this Repository
The goal of this repository is to supply a quick demo environment based on the [cldemo2](https://gitlab.com/cumulus-consulting/goldenturtle/cldemo2-air-builder/) architecture with the alteration of replacing all network nodes as SONiC nodes.

Configurations have been created and backed up within this branch to be easily deployed.  
To pair with that, simple automation has been created for the (re)deployment aspect.

Automation that renders configuration via templates has not been developed yet, that is to come later.  

## Configuration Backup Info
Configs have been backed up in the `automation/playbooks/backup_files` directory for the following devices.  

* Border
* Spine
* Leaf
* Server

Other devices are currently not within scope of this exercise.  
Please review the directories for their file contents and to view the configurations for reference.  

If desired, you can leverage the `backup` playbooks in the `automation/playbooks/` directory to update configurations in the `backup_files` directory.  
This allows a user to redeploy the configurations saved/changed while using the demo infrastructure.  

_**Note:** non-NVIDIA users will not be able to save changes the repository._  
_You may ZIP or FORK the repo when making personal changes you wish to reuse in the future._


## Configuration Restore Info
How to perform a configuration deployment of L3 Routed BGP for the topology containing 100% SONiC ...  

First, note the various playbooks `./automation/playbooks/` directory.  
The `restore_files.yml` playbook has been updated and the `restore_sonic.yml` playbook is new.  
In order to completely deploy/restore the backed up configurations in this repository both of those playbooks should be run.  

**Execute the following ansible commands to initiate the configuration restore process:**

1. Clone this repository to the `oob-mgmt-server`
```
$ git clone https://github.com/mhenimerz/sonic_demo_network.git
```
2. Move into the newly created project directory
```
$ cd sonic_demo_network
```
3. Move into the `automation` directory
```
$ cd automation
```
4. Run the Ansible restore automation for the `SONiC` nodes
```
$ ansible-playbook -i inventories/pod1/ playbooks/restore_sonic.yml
```
5. Run the Ansible restore automation for the `non-SONiC` nodes
```
$ ansible-playbook -i inventories/pod1/ playbooks/restore_files.yml
```
7. Log into various nodes and confirm BGP sessions are up for IPv4 AF and EVPN

### Playbook Structure

The playbooks have the following important structure:
* Inventory is stored in the following file `automation/inventories/pod1/hosts`
* Variables do not exist for this automation as the configurations are backups (vs rendered from templates)
* Ansible playbooks are stores in `automation/playbooks/`
* Backup configurations are stored in `automation/playbooks/backup_files/`

## Verification of Demo

To verify the demo, follow along with the following steps.

1. Show BGP summary on `leaf01` to verify neighbors:

```
admin@leaf01:~$ show ip bgp summary

IPv4 Unicast Summary:
BGP router identifier 10.255.255.1, local AS number 65101 vrf-id 0
BGP table version 23
RIB entries 29, using 5568 bytes of memory
Peers 7, using 152712 KiB of memory
Peer groups 3, using 192 bytes of memory


Neighbhor       V     AS    MsgRcvd    MsgSent    TblVer    InQ    OutQ  Up/Down      State/PfxRcd  NeighborName
------------  ---  -----  ---------  ---------  --------  -----  ------  ---------  --------------  --------------
10.0.101.101    4  65201          9          8         0      0       0  00:05:24                1  SERVER
10.0.101.102    4  65201          9          8         0      0       0  00:05:24                1  SERVER
10.0.101.103    4  65201          9          8         0      0       0  00:05:24                1  SERVER
172.0.1.0       4  65199        137        143         0      0       0  00:06:12                5  spine01
172.0.2.0       4  65199        132        134         0      0       0  00:06:04                5  spine02
172.0.3.0       4  65199        133        136         0      0       0  00:06:05                5  spine03
172.0.4.0       4  65199        138        143         0      0       0  00:06:12                5  spine04

Total number of neighbors 7
```

2. Show BGP learned routes on `leaf01`

```
admin@leaf01:~$ show ip bgp network
BGP table version is 23, local router ID is 10.255.255.1, vrf id 0
Default local pref 100, local AS 65101
Status codes:  s suppressed, d damped, h history, * valid, > best, = multipath,
               i internal, r RIB-failure, S Stale, R Removed
Nexthop codes: @NNN nexthop's vrf id, < announce-nh-self
Origin codes:  i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 10.0.101.0/24    0.0.0.0                  0         32768 i
*= 10.0.103.0/24    172.0.2.0                              0 65199 65102 i
*=                  172.0.3.0                              0 65199 65102 i
*=                  172.0.1.0                              0 65199 65102 i
*>                  172.0.4.0                              0 65199 65102 i
*= 10.0.104.0/24    172.0.2.0                              0 65199 65102 i
*=                  172.0.3.0                              0 65199 65102 i
*=                  172.0.1.0                              0 65199 65102 i
*>                  172.0.4.0                              0 65199 65102 i
*> 10.255.255.1/32  0.0.0.0                  0         32768 i
*= 10.255.255.3/32  172.0.2.0                              0 65199 65102 i
*=                  172.0.3.0                              0 65199 65102 i
*=                  172.0.1.0                              0 65199 65102 i
*>                  172.0.4.0                              0 65199 65102 i
*= 10.255.255.4/32  172.0.2.0                              0 65199 65102 i
*=                  172.0.3.0                              0 65199 65102 i
*=                  172.0.1.0                              0 65199 65102 i
*>                  172.0.4.0                              0 65199 65102 i
*> 10.255.255.101/32
                    172.0.1.0                0             0 65199 i
*> 10.255.255.102/32
                    172.0.2.0                0             0 65199 i
*> 10.255.255.103/32
                    172.0.3.0                0             0 65199 i
*> 10.255.255.104/32
                    172.0.4.0                0             0 65199 i
*> 192.168.255.1/32 10.0.101.101             0             0 65201 ?
*> 192.168.255.2/32 10.0.101.102             0             0 65201 ?
*> 192.168.255.3/32 10.0.101.103             0             0 65201 ?

Displayed  13 routes and 25 total paths

```

3. View the BGP summary on the server:

```
cumulus@server01:~$ sudo vtysh

Hello, this is FRRouting (version 7.5.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

server01# show ip bgp summary

IPv4 Unicast Summary:
BGP router identifier 192.168.255.1, local AS number 65201 vrf-id 0
BGP table version 1
RIB entries 1, using 192 bytes of memory
Peers 2, using 43 KiB of memory

Neighbor        V         AS   MsgRcvd   MsgSent   TblVer  InQ OutQ  Up/Down State/PfxRcd   PfxSnt
10.0.101.1      4      65101        11        12        0    0    0 00:08:39            0        1
10.0.102.1      4      65101        11        12        0    0    0 00:08:39            0        1

Total number of neighbors 2
server01#
```


