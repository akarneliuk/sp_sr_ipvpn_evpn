# Automated Service Provider Fabric

This part is dedicated for automated deployment of:
1) Underlaying MPLS infrastructure (ISIS, Segment Routing, TI-LFA)
2) Underlaying BGP signaling (VPN-IPV4/VPN-IPV6 UNICAST, EVPN address-families)
3) Overlay services (IP VPN (IPv4/IPv6), EVPN (L2/L3))

# Powered by OpenConfig

Wherever possible, OpenConfig YANG models are used. More preciesly OpenConfig is used:
1) In Arista EOS and Cisco IOS XR for configuration of underlay MPLS fabric running ISIS as IGP with Segment Routing extensions
2) In Cisco IOS XR for configuration of underlay BGP sginaling for EVPN, VPNV4/VPNV6 unicast

Important notes:
1) As of today the most current Nokia SR OS release 16.0.R5 doesn't support Segment Routing in OpenConfig YANG models. Nevertheless the common templates `openconfig-interfaces.j2` and `openconfig-network-instance.j2` are prepared and can properly configure ISIS on all NOSes including Nokia SR OS. As soon as Nokia's OpenConfig YANG models will support Segment Routing, the Ansible playbooks will be changed so that Nokia start use OpenConfig as well. Temporary Nokia is using CLI-based configuration using `sros_config` Ansible module.
2) Arista EOS in version 4.21.1.1F supports OpenConfig YANG modules only for IPV4/IPV6 UNICAST AFI/SAFI, hence Ansible module `eos_config` is used to configure underlay BGP. For Nokia SR OS also Ansible `sros_config` is used.
3) During creation of automation using OpenConfig it turned out that there is no node for `update-source` wihtin BGP module, that's why native Cisco IOS XR YANG module is used to fix that. There is no JTOX used for that, but rather XML for NETCONF is constructed directly.

# How to use Service Provider Fabric automation

There are several possible options how you can automatically deploy Service Provider Fabric. The prerequisite is copy the initial configuration files called `sp_{{ hostname }}_initial.conf` from `files` folder to the running config of your network functions. Then use the following tags:
1) To deploy undelay routing (ISIS) and MPLS (Segment Routing) use `ansible-playbook --inventory=hosts --tags=underlay_mpls`.
2) To deploy undelay BGP signaling use `ansible-playbook --inventory=hosts --tags=underlay_bgp`.
3) To deploy the whole underlay infrastructure (ISIS + Segment Routing + BGP) use `ansible-playbook --inventory=hosts --tags=underlay`.

# Temporary limitations

Currently there is no tags, which allow you to create any overlay service.

# How to use YANG tools

There are some Ansible playbooks, which help a lot for development of automation based on YANG (NETCONF/YANG, RESTCONF/YANG, GNOI/YANG):
1) Tool `yang_extractor_config.yml` is used to extract from the network function actual configuration and associated stated in particular YANG module. You can use calling `ansible-playbook oc_extractor.yml --inventory=hosts --limit=X1 --tags=X2`, where `X1` is hostname of the network function (i.e. EOS1, SR1, XR1, etc) and `X2` defines the intersted YANG module. Thr following modules are currently supported:
- Tag `oc-if` stands for OpenConfig YANG module `openconfig-interfaces.yang` including all `openconfig-if-*.yang` extensions.
- Tag `oc-netinst` stands for OpenConfig YANG module `openconfig-network-instance.yang` including all its imports.
- Tag `oc-lldp` stands for OpenConfig YANG module `openconfig-lldp.yang`.
- Tag `ipv4-bgp-cfg` stands for Cisco native YANG module `Cisco-IOS-XR-ipv4-bgp-cfg.yang`.