# Example configuration files
This example assumes the following:
- You bond all 4 interface and running VM manager;
- You have a data VLAN 2 with MTU 9000;
- You have a management VLAN 10 with MTU 1500;
- You may (later) want to used large MTU for your VMs.

## Explanations
1. The `ovs_bond99` interface is for VM Manager to attach tap interfaces;
	- For some reason my VM Manager automatically selected this interface, so I didn't have to do any special configuration, YMMV.
2. In `ifcfg-ovs_bond0` the most important line related to this mod is the `OVS_PATCH`;
	- It establishes a patch link to `ovs_bond99` so VMs can get their network packets.
3. In `ifcfg-ovs_bond1` the most important line related to this mod is the `SLAVE_LIST`;
	- It enables DSM to see this interface as "connected".
4. In `ifcfg-ovs_bond99`:
	- The `OVS_PATCH` line describes the other half of the patch link to/from `ovs_bond0`;
	- The `SLAVE_LIST` line enables DSM to see this interface as "connected";
	- The `OVS_VIRTUAL` line prevents bringing down the `eth` interfaces when this interface is removed (from DSM GUI);
		- Although I wouldn't recommend touching the network config part of the GUI after this modification.

## How about MTU?
That needs an additional step of hack:
- Edit your `/etc/synoinfo.conf`, add/modify the following keys:
	```
	eth0_mtu="9000"
	eth1_mtu="9000"
	eth2_mtu="9000"
	eth3_mtu="9000"
	bond0_mtu="9000"
	bond1_mtu="1500"
	bond99_mtu="9000"
	```
