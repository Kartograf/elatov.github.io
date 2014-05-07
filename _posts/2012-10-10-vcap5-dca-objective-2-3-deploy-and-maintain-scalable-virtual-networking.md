---
title: VCAP5-DCA Objective 2.3 – Deploy and Maintain Scalable Virtual Networking
author: Karim Elatov
layout: post
permalink: /2012/10/vcap5-dca-objective-2-3-deploy-and-maintain-scalable-virtual-networking/
dsq_thread_id:
  - 1404673378
categories:
  - Certifications
  - VCAP5-DCA
  - VMware
tags:
  - VCAP5-DCA
---
### Identify VMware NIC Teaming policies

From &#8220;<a href="http://pubs.vmware.com/vsphere-50/topic/com.vmware.ICbase/PDF/vsphere-esxi-vcenter-server-50-networking-guide.pdf" onclick="javascript:_gaq.push(['_trackEvent','download','http://pubs.vmware.com/vsphere-50/topic/com.vmware.ICbase/PDF/vsphere-esxi-vcenter-server-50-networking-guide.pdf']);">vSphere Networking ESXi 5.0</a>&#8221;

> **Edit Failover and Load Balancing Policy for a vSphere Standard Switch**  
> Use Load Balancing and Failover policies to determine how network traffic is distributed between adapters and how to reroute traffic in the event of an adapter failure.
> 
> The Failover and Load Balancing policies include the following parameters:
> 
> *   Load Balancing policy: The Load Balancing policy determines how outgoing traffic is distributed among the network adapters assigned to a standard switch. Incoming traffic is controlled by the Load Balancing policy on the physical switch.
> *   Failover Detection: Link Status/Beacon Probing
> *   Network Adapter Order (Active/Standby)
> 
> In some cases, you might lose standard switch connectivity when a failover or failback event occurs. This causes the MAC addresses used by virtual machines associated with that standard switch to appear on a different switch port than they previously did. To avoid this problem, put your physical switch in portfast or portfast trunk mode.

From the same document:

> In the Load Balancing list, select an option for how to select an uplink.
> 
> <a href="http://virtuallyhyper.com/wp-content/uploads/2012/09/load_balancing_algorithms.png" onclick="javascript:_gaq.push(['_trackEvent','outbound-article','http://virtuallyhyper.com/wp-content/uploads/2012/09/load_balancing_algorithms.png']);"><img src="http://virtuallyhyper.com/wp-content/uploads/2012/09/load_balancing_algorithms.png" alt="load balancing algorithms VCAP5 DCA Objective 2.3 – Deploy and Maintain Scalable Virtual Networking " title="load_balancing_algorithms" width="572" height="167" class="alignnone size-full wp-image-4017" /></a>
> 
> In the Network failover detection list, select the option to use for failover detection.
> 
> <a href="http://virtuallyhyper.com/wp-content/uploads/2012/09/network_failover_detection.png" onclick="javascript:_gaq.push(['_trackEvent','outbound-article','http://virtuallyhyper.com/wp-content/uploads/2012/09/network_failover_detection.png']);"><img src="http://virtuallyhyper.com/wp-content/uploads/2012/09/network_failover_detection.png" alt="network failover detection VCAP5 DCA Objective 2.3 – Deploy and Maintain Scalable Virtual Networking " title="network_failover_detection" width="562" height="179" class="alignnone size-full wp-image-4018" /></a> 

Also from the same document, here is a summary of all the policies:

> Specify the settings in the Policy Exceptions group
> 
> <a href="http://virtuallyhyper.com/wp-content/uploads/2012/09/portgroup_network_policies.png" onclick="javascript:_gaq.push(['_trackEvent','outbound-article','http://virtuallyhyper.com/wp-content/uploads/2012/09/portgroup_network_policies.png']);"><img src="http://virtuallyhyper.com/wp-content/uploads/2012/09/portgroup_network_policies.png" alt="portgroup network policies VCAP5 DCA Objective 2.3 – Deploy and Maintain Scalable Virtual Networking " title="portgroup_network_policies" width="558" height="724" class="alignnone size-full wp-image-4019" /></a> 

### Identify common network protocols

When it comes to virtual switch talking to the physical network these are involved:

STP &#8211; Spanning Tree Issue (it&#8217;s recommended to be disable or at least set portfast, more information in VMware KB <a href="http://kb.vmware.com/kb/1003804" onclick="javascript:_gaq.push(['_trackEvent','outbound-article','http://kb.vmware.com/kb/1003804']);">1003804</a> )  
LACP &#8211; Link Aggregation Control Protocol only supported with 5.1  
Etherchannel &#8211; Supported only with the IP_hash Algorithtm (for sample config check out VMware KB <a href="http://kb.vmware.com/kb/1004048" onclick="javascript:_gaq.push(['_trackEvent','outbound-article','http://kb.vmware.com/kb/1004048']);">1004048</a>  
QOS &#8211; Quality of Service, with VMware only 802.1p tagging is supported (for a good example check out <a href="http://www.vmware.com/files/pdf/techpaper/Whats-New-VMware-vSphere-50-Networking-Technical-Whitepaper.pdf" onclick="javascript:_gaq.push(['_trackEvent','download','http://www.vmware.com/files/pdf/techpaper/Whats-New-VMware-vSphere-50-Networking-Technical-Whitepaper.pdf']);">What’s New in VMware vSphere 5.0 Networking</a>)

### Understand the NIC Teaming failover types and related physical network settings

There are two options as described above, &#8220;Link Status Only&#8221; and Beacon Probing. For &#8220;Link Status Only&#8221; make sure you have a consistent negotiation policy set. For 1GB it&#8217;s recommended to set auto/auto negotiation from the physical switch and from the physical NIC. If that is set properly then failover should be pretty smooth.

### Determine and apply Failover settings

This was discussed in the above objectives.

### Configure explicit failover to conform with VMware best practices

If you are using Software iSCSI and you want to utilized MPIO (MultiPath IO) then you will need to bind each vmk to a vmnic.

From &#8220;<a href="http://pubs.vmware.com/vsphere-50/topic/com.vmware.ICbase/PDF/vsphere-esxi-vcenter-server-50-storage-guide.pdf" onclick="javascript:_gaq.push(['_trackEvent','download','http://pubs.vmware.com/vsphere-50/topic/com.vmware.ICbase/PDF/vsphere-esxi-vcenter-server-50-storage-guide.pdf']);">vSphere Storage ESXi 5.0</a>&#8220;:

> **Change Port Group Policy for iSCSI VMkernel Adapters**  
> If you use a single vSphere standard switch to connect VMkernel to multiple network adapters, change the port group policy, so that it is compatible with the iSCSI network requirements.
> 
> By default, for each virtual adapter on the vSphere standard switch, all network adapters appear as active. You must override this setup, so that each VMkernel interface maps to only one corresponding active NIC. For example, vmk1 maps to vmnic1, vmk2 maps to vmnic2, and so on.
> 
> **Prerequisites**  
> Create a vSphere standard switch that connects VMkernel with physical network adapters designated for iSCSI traffic. The number of VMkernel adapters must correspond to the number of physical adapters on the vSphere standard switch.
> 
> **Procedure**  
> 1 Log in to the vSphere Client and select the host from the inventory panel.  
> 2 Click the Configuration tab and click Networking.  
> 3 Select the vSphere standard switch that you use for iSCSI and click Properties.  
> 4 On the Ports tab, select an iSCSI VMkernel adapter and click Edit.  
> 5 Click the NIC Teaming tab and select Override switch failover order.  
> 6 Designate only one physical adapter as active and move all remaining adapters to the Unused Adapters category.  
> 7 Repeat Step 4 through Step 6 for each iSCSI VMkernel interface on the vSphere standard switch. 

If you are limited on NICs you can have one vSwitch with two uplinks. Inside the vswitch you can have two portgroups, 1 for Management and 1 for vMotion (each should be on a separate VLAN). You can set vmnic0 to be active for Management and set vmnic1 as standby. Set the failover policy the opposite for vMotion, vmnic1 active and vmnic0 as standvy. With this setup you can have redundancy and no two types of traffic sharing the same NIC (isolation). 

### Configure port groups to properly isolate network traffic

Use VST and trunk ports on the physical switch and try to isolate your traffic by physical nics as well. Here is a pretty good example:

<a href="http://virtuallyhyper.com/wp-content/uploads/2012/10/vswitch_example.png" onclick="javascript:_gaq.push(['_trackEvent','outbound-article','http://virtuallyhyper.com/wp-content/uploads/2012/10/vswitch_example.png']);"><img src="http://virtuallyhyper.com/wp-content/uploads/2012/10/vswitch_example.png" alt="vswitch example VCAP5 DCA Objective 2.3 – Deploy and Maintain Scalable Virtual Networking " title="vswitch_example" width="477" height="660" class="alignnone size-full wp-image-4271" /></a>

<p class="wp-flattr-button">
  <a class="FlattrButton" style="display:none;" href="http://virtuallyhyper.com/2012/10/vcap5-dca-objective-2-3-deploy-and-maintain-scalable-virtual-networking/" title=" VCAP5-DCA Objective 2.3 – Deploy and Maintain Scalable Virtual Networking" rev="flattr;uid:virtuallyhyper;language:en_GB;category:text;tags:VCAP5-DCA,blog;button:compact;">Identify VMware NIC Teaming policies From &#8220;vSphere Networking ESXi 5.0&#8221; Edit Failover and Load Balancing Policy for a vSphere Standard Switch Use Load Balancing and Failover policies to determine how...</a>
</p>