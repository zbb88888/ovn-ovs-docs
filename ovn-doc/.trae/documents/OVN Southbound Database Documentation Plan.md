I will now proceed with the **Phase 2: OVN Database (Southbound)** verbatim translation and review. I will reorganize the `ovn-manuals-cn/ovn-sb/` directory to ensure all tables from `ovn-sb.5` are covered comprehensively and in a structured manner.

**Plan:**

1.  **`01-System-Physical.md`**: Cover system-wide and physical network tables.
    *   Tables: `SB_Global`, `Chassis`, `Chassis_Private`, `Encap`.
2.  **`02-Logical-Network.md`**: Cover logical network definitions and flows.
    *   Tables: `Address_Set`, `Port_Group`, `Logical_Flow`, `Logical_DP_Group`, `Multicast_Group`, `Mirror`, `Meter`, `Meter_Band`.
3.  **`03-Bindings.md`**: Cover logical-physical bindings.
    *   Tables: `Datapath_Binding`, `Port_Binding`, `MAC_Binding`, `FDB`, `Static_MAC_Binding`.
4.  **`04-Services.md`**: Cover network services.
    *   Tables: `DHCP_Options`, `DHCPv6_Options`, `DNS`, `Service_Monitor`, `IP_Multicast`, `IGMP_Group`.
5.  **`05-Connection-Security.md`**: Cover connectivity and security tables.
    *   Tables: `Connection`, `SSL`, `RBAC_Role`, `RBAC_Permission`.
6.  **`06-HA-Gateway.md`**: Cover high availability and gateway configuration.
    *   Tables: `Gateway_Chassis`, `HA_Chassis`, `HA_Chassis_Group`, `BFD`.
7.  **`07-Load-Balancer.md`**: Cover load balancer configuration.
    *   Tables: `Load_Balancer`.
8.  **`08-Routing-Others.md`**: Cover routing and miscellaneous tables.
    *   Tables: `Advertised_Route`, `Learned_Route`, `ECMP_Nexthop`, `ACL_ID`, `Advertised_MAC_Binding`, `Chassis_Template_Var`, `Controller_Event`.

I will read `source_docs/ovn-sb.5.txt` in chunks and write/rewrite these files strictly following the source text.