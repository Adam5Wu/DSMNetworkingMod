# DSM Networking Mod
Getting Synology DSM to work with OpenVSwitch + Bond + Multi-VLANs + Jumbo Packet with support for Virtualization

## Why you need this?
If you are here, you probably share a similar adventure with me:
1. Sinked a fortune for the all mighty Synology NAS;
2. Let's LACP bond this baby, make the maximum use of the powerfulness;
3. Ooops, I have a dedicated management VLAN, what do I do?
    1. Googling around a bit, found [some instructions](http://www.mybenke.org/?p=2373), created additional `ifcfg-bond0.(tag)` files;
    2. It works! Although a bit of hacking was needed, not bad.
4. Hmm, there is some VM Manager / Docker add-on, seems quite interesting;
    1. Heck why not, this thing got an Xeon inside, better make full use of the power I paid for;
    2. Plan and architect the VMs / containers, the whole new world became quite attractive!
5. Install VM Manager / Docker, which turned on OpenVSwitch, and the world instantly darkens...
    1. Your existing bonded multiple VLAN setup is blown into oblivion;
    2. And follow the old method of re-creating ifcfg file couldn't bring anything back.
6. Double, triple Googling around, finally found [another guide](https://community.synology.com/enu/forum/12/post/123052);
    1. Apparently Synology has never considered the scenario of OpenVSwitch + Bond + Multi-VLANs;
    2. As a result, the `/etc/rc.network` has to be modified, adding additional logic;
    3. Following the new guide, you are able to establish multiple bonded VLAN interfaces, success!
7. But the hack seems incomplete, and you are still bothered by the following problems:
    1. Except the first bond interface, all other interfaces are shown as "disconnected";
        - As a result, you are not able to use other interfaces from GUI, such as when you install MailPlus server;
    2. Unable to use jumbo packet on any of the bond interfaces when you start a VM;
        - As soon as you start any VM, all bond interfaces' MTU reset to 1500.

This modification extends existing guide on OpenVSwitch + Bond + Multi-VLANs, and tries to cover all corner aspects, enabling a complete solution.

## How did I solve the problem?
1. Disconnected interface problem:
    - Through trial-and-error, I found having a non-empty "SLAVE_LIST" with interfaces that is in up state makes DSM think the bond interface is connected;
    - But of course I noticed some logic in `/etc/rc.network` that acks upon values of "SLAVE_LIST";
    - So more logic is injected into `/etc/rc.network` to avoid trigger operations on the wrong scenario.
2. Jumbo Packet problem:
    - OpenVSwitch automatically reduce MTU on all interfaces on a bridge to the minimum value allowed across all attached interfaces;
    - When you start a VM, a tap interface is attached to the bridge which all your other network interfaces attach to, and the VM Manager seems only want to create ones with MTU of 1500. As a result all interfaces are forced downgrade to non-jumbo mode;
        - Of course you could manually upgrade each tap interface's MTU, and get back the larger MTU after upgrading all taps;
        - But as soon as you create / start another VM, the problem is back again.
    - In order to permenantly solve the problem, you cannot have the VM manager attach taps to the same bridge.
        - Luckily, OpenVSwitch already have a solution for scenario;
            1. Create another bridge, which is not backed by any physical interface, dedicated for VMs;
            2. Create a pair of peering "patch" interfaces, with each end on your existing bridge (with physical bond interface) and VM bridge;
            3. Let the VM manager automatically attach taps to the VM bridge instead;
        - Note that, if you want you VMs to use jumbo packets, you still need to manually adjust the MTU of the tap interfaces. But at least your physical NAS will always respect your MTU configurations now.
    - Again, more logics are injected into `/etc/rc.network` for ability to create patch interfaces.

## Other changes I made (to `/etc/rc.network`)
1. Refactored the code related to MTU and dhcp client;
2. Cleaned up some mixed use of local and non-local variables;
3. Fixed a typo.

## How do I apply this solution?
1. You should be using `DSM 6.2.2-24922`;
2. Apply the `/etc/rc.network` patch file;
    ```bash
    cp /etc/rc.network /volume1/backup/
    patch /etc/rc.network rc.network.patch
    ```
    - If you have already modified yours, don't worry, you can get back the original copy from `/etc.defaults/rc.network`;
3. Refer to sample configurations in the [examples directory](https://github.com/Adam5Wu/DSMNetworkingMod/blob/master/examples), adjust for your needs.
