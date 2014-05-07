---
title: Seeing a High Number of Trespasses from a CLARiiON Array with ESX Hosts
author: Karim Elatov
layout: post
permalink: /2012/04/seeing-a-high-number-of-trespasses-from-a-clariion-array-with-esx-hosts/
dsq_thread_id:
  - 1404673236
categories:
  - Storage
  - VMware
tags:
  - Active/Non-Optimal Path
  - Active/Optimal Path
  - Active/Passive
  - ALUA
  - Assymetric Logical Unit Access
  - CLARiiON
  - Explicit/Implicit Failover
  - failover mode 4
  - NDU
  - Preferred Path
  - Storage Processor
  - TGP
  - TGPS
  - Trespass
  - useANO
---
I was recently working with a customer who was experiencing a high number of Trespasses on his CLARiiON Array with 3 ESX 4.0 hosts. While this was happening I saw the following in the logs across all the hosts:

[code]  
Apr 19 09:25:54 ESX\_Host vmkernel: 0:18:55:02.430 cpu2:4262)VMW\_SATP\_ALUA: satp\_alua_activatePaths: Activation disallowed due to follow-over.  
Apr 19 09:36:36 ESX\_Host vmkernel: 0:19:05:45.146 cpu0:4478)NMP: nmp\_CompleteCommandForPath: Command 0x2a (0x410001032580) to NMP device &quot;naa.60060160467029002acca5f7836fe111&quot; failed on physical path &quot;vmhba2:C0:T1:L3&quot; H:0x0 D:0x2 P:0x0 Valid sense data: 0x6 0x2a 0x6.  
Apr 19 09:36:36 ESX\_Host vmkernel: 0:19:05:45.146 cpu0:4478)WARNING: NMP: nmp\_DeviceRetryCommand: Device &quot;naa.60060160467029002acca5f7836fe111&quot;: awaiting fast path state update for failover with I/O blocked. No prior reservation exists on the device.  
Apr 19 09:36:37 ESX\_Host vmkernel: 0:19:05:46.147 cpu2:4222)WARNING: NMP: nmp\_DeviceAttemptFailover: Retry world failover device &quot;naa.60060160467029002acca5f7836fe111&quot; - issuing command 0x410001032580  
Apr 19 09:36:37 ESX\_Host vmkernel: 0:19:05:46.147 cpu0:4096)NMP: nmp\_CompleteRetryForPath: Retry world recovered device &quot;naa.60060160467029002acca5f7836fe111&quot;  
[/code]

The customer had setup the CLARiiON to use ALUA (Assymetric Logical Unit Access) mode (fail over mode 4). From <a href="http://www.emc.com/collateral/hardware/white-papers/h1416-emc-clariion-intgtn-vmware-wp.pdf" onclick="javascript:_gaq.push(['_trackEvent','download','http://www.emc.com/collateral/hardware/white-papers/h1416-emc-clariion-intgtn-vmware-wp.pdf']);">EMC CLARiiON Integration with VMware ESX</a>:

> On VMware 4.x servers with CX4 arrays, the FIXED or Round Robin policy is supported. The FIXED policy on the CX4 provides failback capability. To use the FIXED policy, you must be running FLARE release 28 version 04.28.000.5.704 or later. Also, the failovermode mode must be set to 4 (ALUA mode or Asymmetric Active/Active mode). The default failovermode for ESX 4.x is 1. Use the Failover Setup Wizard within Navisphere to change the failovermode from 1 to 4.

Since all CLARiiON arrays are active/passive, from <a href="http://virtualgeek.typepad.com/virtual_geek/2009/09/a-couple-important-alua-and-srm-notes.html" onclick="javascript:_gaq.push(['_trackEvent','outbound-article','http://virtualgeek.typepad.com/virtual_geek/2009/09/a-couple-important-alua-and-srm-notes.html']);">A couple important (ALUA and SRM) notes</a>:

> CLARiiON arrays are in VMware’s nomenclature an “active/passive” array – they have a LUN ownership  
> model where internally a LUN is “owned” by one storage processor or the other. This is common in  
> mid-range (“two brain” aka “storage processors”) arrays who’s core architecture was born in  
> the 1990s (and is similar to NetApp’s Fibre Channel – but not iSCSI &#8211; implementation, HP EVA and others)

Setting it for ALUA mode is actually a good idea. Before we get too far ahead of our selves, first let&#8217;s define what active/passive means and then we can figure out what ALUA is. From this great post, <a href="http://gestaltit.com/all/tech/storage/bas/assymetric-logical-unit-access-alua-mode-emc-clariion" onclick="javascript:_gaq.push(['_trackEvent','outbound-article','http://gestaltit.com/all/tech/storage/bas/assymetric-logical-unit-access-alua-mode-emc-clariion']);">Assymetric Logical Unit Access (ALUA) mode on CLARiiON</a>:

> Normally a path to a CLARiiON is considered active when we are connected to the service processor that is currently serving you the LUN. CLARiiON arrays are so called “active/passive” arrays, meaning that only one service processor is in charge of a LUN, and the secondary service processor is just waiting for a signal to take over the ownership in case of a failure. The array will normally receive a signal that tells it to switch from one service processor to the other one. This routine is called a “trespass” and happens so fast that you usually don’t really notice such a failover.

Now what does ALUA allow us to do, from the same post:

> To put it more simply, ALUA allows me to reach my LUN via the active and the inactive service  
> processor. Oversimplified this just means that all traffic that is directed to the non-active service  
> processor will be routed internally to the active service processor.
> 
> On a CLARiiON that is using ALUA mode this will result in the host seeing paths that are in an optimal  
> state, and paths that are in an non-optimal state. The optimal path is the path to the active storage  
> processor and is ready to perform I/O and will give you the best performance, and the non-optimal path  
> is also ready to perform I/O but won’t give you the best performance since you are taking a detour.

Now how does the array actually do this, from the article &#8220;<a href="http://www.emc.com/collateral/hardware/white-papers/h2890-emc-clariion-asymm-active-wp.pdf" onclick="javascript:_gaq.push(['_trackEvent','download','http://www.emc.com/collateral/hardware/white-papers/h2890-emc-clariion-asymm-active-wp.pdf']);">EMC CLARiiON Asymmetric Active/Active Feature</a>:

> In dual-SP systems, such as a CLARiiON, I/O can be routed through either SP. For example, if I/O for a LUN is sent to an SP that does not own the LUN, that SP redirects the I/O to the SP that does own the LUN. This redirection is done through internal communication within the storage system. It is transparent to the host, and the host is not aware that the other SP processed the I/O. Hence, a trespass is not required when I/O is sent to the non-owning (or non- optimal) SP.
> 
> Dual-SP storage systems that support ALUA define a set of target port groups for each LUN. One target port group is defined for the SP that currently owns the LUN, and the other target port group is defined for the SP that does not own the LUN. Standard ALUA commands enable the host failover software to determine the state of a LUN’s path.

Inside that same document there is an excellent picture that depicts how ALUA works. Check it out:

<a href="http://virtuallyhyper.com/wp-content/uploads/2012/04/ALUA.png" onclick="javascript:_gaq.push(['_trackEvent','outbound-article','http://virtuallyhyper.com/wp-content/uploads/2012/04/ALUA.png']);"><img class="alignnone size-full wp-image-1255" title="ALUA" src="http://virtuallyhyper.com/wp-content/uploads/2012/04/ALUA.png" alt="ALUA Seeing a High Number of Trespasses from a CLARiiON Array with ESX Hosts" width="452" height="480" /></a>

ALUA uses Target Port Groups (TGP) for multiple aspects, so it&#8217;s very important to understand  
what those are. There is a great picture of TGP in this article, <a href="http://deinoscloud.wordpress.com/2011/07/04/it-all-started-with-this-question/" onclick="javascript:_gaq.push(['_trackEvent','outbound-article','http://deinoscloud.wordpress.com/2011/07/04/it-all-started-with-this-question/']);">It All Started With This Question…</a>:

<a href="http://virtuallyhyper.com/wp-content/uploads/2012/04/asymmetricaccessstate.png" onclick="javascript:_gaq.push(['_trackEvent','outbound-article','http://virtuallyhyper.com/wp-content/uploads/2012/04/asymmetricaccessstate.png']);"><img class="alignnone size-full wp-image-1256" title="asymmetricaccessstate" src="http://virtuallyhyper.com/wp-content/uploads/2012/04/asymmetricaccessstate.png" alt="asymmetricaccessstate Seeing a High Number of Trespasses from a CLARiiON Array with ESX Hosts" width="600" height="264" /></a>

Also from <a href="http://www.emc.com/collateral/hardware/white-papers/h2890-emc-clariion-asymm-active-wp.pdf" onclick="javascript:_gaq.push(['_trackEvent','download','http://www.emc.com/collateral/hardware/white-papers/h2890-emc-clariion-asymm-active-wp.pdf']);">EMC CLARiiON Asymmetric Active/Active Feature</a>:

> Target port group — A set of target ports that are in either primary SP ports or secondary SP ports.

Now once you understand what those are, you can actually query TPGs and get information back regarding  
the state of the Group, and the result is called Target Port Group Support (TGPS). From the same document:

> The Report Target Port Group command provides a report about the following three attributes:
> 
> Preferred — Indicates whether this port group is the default (preferred) port group.  
> Asymmetric access state — Indicates the state of the port group. *Port group states include Active/Optimal,Active/Non-optimal, Standby, and Unavailable.*  
> Attribute — Indicates whether the current asymmetric access state was explicitly set by a command or  
> was implicitly set or changed by the storage system.
> 
> The SET TARGET PORT GROUP command allows the access state (*Active/Optimal, Active/Non-optimal, Standby,and Unavailable*) of each port group to be set or changed.
> 
> Access states are:
> 
> *Active/Optimal* — Best performing path, does not require upper level redirection to complete I/O  
> *Active/Non-optimal* — Requires upper level redirection to complete I/O  
> *Standby* — This state is not supported by CLARiiON  
> *Unavailable* — Returned for port groups on a SP that is down. The user cannot set it by using set  
> target port groups.
> 
> Target port group commands are implemented in the ALUA layer of the storage system. However, host-  
> based path management software executes the commands and manages the paths. The explicit ALUA approach is similar to the traditional path management mechanisms except that ALUA has standardized the previously vendor-specific mechanisms

In short, you define a port group that SPs are part of and these port groups can return status of LUNs  
depending the owner and preference.

Now that we got that out of the way let&#8217;s back to logs, so we saw  
the following sense code returned: *H:0&#215;0 D:0&#215;2 0&#215;6 0x2a 0&#215;6*. Translating that we get the following:

[code]  
ASC/ASCQ: 2Ah / 06h  
ASC Details: ASYMMETRIC ACCESS STATE CHANGED  
Status: 02h - CHECK CONDITION  
Hostebyte: 0x00 - DID_OK - No error  
Sensekey: 6h  
Sensekey MSG: UNIT ATTENTION  
[/code]

We are getting an *Access State Changed*, and that is expected since this was a trespass. Now looking at  
the output of the &#8216;nmp device&#8217; we saw the following

[code]  
\# esxcli nmp device list -d naa.600601604670290026cca5f7836fe111  
naa.600601604670290026cca5f7836fe111  
Device Display Name: DGC Fibre Channel Disk (naa.600601604670290026cca5f7836fe111)  
Storage Array Type: VMW\_SATP\_ALUA_CX  
Storage Array Type Device Config: {navireg=on, ipfilter=on}{implicit\_support=on;explicit\_support=on;explicit\_allow=on;alua\_followover=on;{TPG\_id=1,TPG\_state=ANO}{TPG\_id=2,TPG\_state=AO}}  
Path Selection Policy: VMW\_PSP\_FIXED  
Path Selection Policy Device Config: {preferred=vmhba2:C0:T1:L1;current=vmhba2:C0:T1:L1}  
Working Paths: vmhba2:C0:T1:L1  
[/code]

The most important line is the following:

[code]  
Storage Array Type Device Config: {navireg=on, ipfilter=on}{implicit\_support=on;explicit\_support=on;explicit\_allow=on;alua\_followover=on;{TPG\_id=1,TPG\_state=ANO}{TPG\_id=2,TPG\_state=AO}}  
[/code]

Definitions of all the settings can be found at VMware KB <a href="http://kb.vmware.com/kb/1022030" onclick="javascript:_gaq.push(['_trackEvent','outbound-article','http://kb.vmware.com/kb/1022030']);">1022030</a>:

> implicit_support=on
> 
> This parameter shows whether or not the device supports implicit ALUA. You cannot set this option as it is a property of the LUN.
> 
> explicit_support
> 
> This parameter shows whether or not the device supports explicit ALUA. You cannot set this option as it is a property of the LUN.
> 
> explicit_allow
> 
> This parameter shows whether or not the user allows the SATP to exercise its explicit ALUA capability  
> if the need arises during path failure. This only matters if the device actually supports explicit ALUA  
> (that is, explicit\_support is on). This option is turned on using the esxcli command enable\_explicit_alua  
> and turned off using the esxcli command disable\_explicit\_alua.
> 
> alua_followover
> 
> This parameter shows whether or not the user allows the SATP to exercise the follow-over policy,  
> which prevents path thrashing in multi-host setups. This option is turned on using the esxcli command  
> enable\_alua\_followover and turned off using the esxcli command disable\_alua\_followover.

Just to elaborate, the article &#8216;<a href="http://www.emc.com/collateral/hardware/white-papers/h2890-emc-clariion-asymm-active-wp.pdf" onclick="javascript:_gaq.push(['_trackEvent','download','http://www.emc.com/collateral/hardware/white-papers/h2890-emc-clariion-asymm-active-wp.pdf']);">EMC CLARiiON Asymmetric Active/Active Feature</a>&#8216;, has a good paragraph  
about implicit and explicit failover:

> Explicit and implicit failover software  
> Explicit and implicit failover software should not be confused with the explicit and implicit trespass  
> commands mentioned earlier.
> 
> Failover software that supports *explicit* ALUA commands monitors and adjusts the active paths between optimal and non-optimal using the REPORT TARGET PORT GROUPS and SET TARGET PORT GROUPS commands, respectively. The SET TARGET PORT GROUPS command, when issued on one controller, trespasses the LUN to the other controller. PowerPath has the same net effect as an explicit trespass, although it uses traditional CLARiiON proprietary commands to initiate a trespass. Windows 2008 MPIO supports explicit ALUA commands.  
> It is possible for host multipathing software to support *implicit* ALUA functionality. The host could redirect traffic to the non-optimal path for some reason that is not apparent to the storage system. In this case, the storage system must implicitly trespass once it detects enough I/O on the non-optimal paths. To get the optimal path information, the failover software uses the REPORT TARGET PORT GROUPS command. AIX MPIO only supports implicit ALUA commands.  
> Both *explicit* and *implicit* failover software may or may not provide autorestore capability. Without the  
> autorestore capability, after an NDU, all LUNs could end up on the same SP. Cluster software could also make use of this functionality.

Also, if you want to know what follow-over is, I would suggest reading this &#8216;[Configuration Settings for ALUA  
Devices][1]&#8216; .

Lastly we also see the following from the *nmp device* command that we haven&#8217;t defined yet:

[code]  
{TPG\_id=1,TPG\_state=AO}{TPG\_id=2,TPG\_state=ANO}  
[/code]

and from the above blog:

> There are two Target Port Groups. TPG\_id=1 has all the Active Optomized paths (AO). TPG\_id=2 has all the Active Non-Optomized paths (ANO)

Now running the *nmp path* commands, we saw the following:

[code]  
\# esxcli nmp path list -d naa.600601604670290026cca5f7836fe111  
fc.20000000c96eb647:10000000c96eb647-fc.50060160bea038b6:5006016d3ea038b6-naa.600601604670290026cca5f7836fe111  
Runtime Name: vmhba2:C0:T1:L1  
Device: naa.600601604670290026cca5f7836fe111  
Device Display Name: DGC Fibre Channel Disk (naa.600601604670290026cca5f7836fe111)  
Group State: active  
Storage Array Type Path Config: {TPG\_id=2,TPG\_state=AO,RTP\_id=14,RTP\_health=UP}  
Path Selection Policy Path Config: {current: yes; preferred: yes}

fc.20000000c96eb647:10000000c96eb647-fc.50060160bea038b6:500601643ea038b6-naa.600601604670290026cca5f7836fe111  
Runtime Name: vmhba2:C0:T0:L1  
Device: naa.600601604670290026cca5f7836fe111  
Device Display Name: DGC Fibre Channel Disk (naa.600601604670290026cca5f7836fe111)  
Group State: active unoptimized  
Storage Array Type Path Config: {TPG\_id=1,TPG\_state=ANO,RTP\_id=5,RTP\_health=UP}  
Path Selection Policy Path Config: {current: no; preferred: no}  
[/code]

So the top path is going through TGP2 and the state of that TPG is AO (active/optimal) and that is the  
preferred path. The bottom path is going through TPG1 and the state is ANO (Active/non-optimal) and  
that is not the preffered path. I checked the other hosts and they looked the same, so no user preffered  
paths have been set and all the host are going through the AO path.  
I also found VMware KB <a href="http://kb.vmware.com/kb/2005369" onclick="javascript:_gaq.push(['_trackEvent','outbound-article','http://kb.vmware.com/kb/2005369']);">2005369</a>  and it talks about how with Round Robin you can allow the ESX host to use Active Non-Optimized paths:

[code]  
#esxcli nmp psp setconfig --config useANO=1 --device <device uid>  
[/code]

But we weren&#8217;t using Round Robin and all the hosts were consistent in their preffered path to be the Active  
Optimized Path. The article <a href="http://www.emc.com/collateral/hardware/white-papers/h1416-emc-clariion-intgtn-vmware-wp.pdf" onclick="javascript:_gaq.push(['_trackEvent','download','http://www.emc.com/collateral/hardware/white-papers/h1416-emc-clariion-intgtn-vmware-wp.pdf']);">EMC CLARiiON Integration with VMware ESX</a>, talks about some pros and cons of Round Robin Vs. FIXED:

> When using the FIXED policy, the auto-restore or failback capability distributes the LUNs to their  
> respective default storage processors (SPs) after an NDU operation. This prevents the LUNs from all being on a single storage processor after an NDU (Non-Disruptive Upgrade). When using the FIXED policy, ensure the preferred path setting is configured to be on the same storage processor for all ESX hosts accessing a given LUN.
> 
> With the FIXED policy, there is some initial setup that is required to select the preferred path; the preferred path should also be the optimal path when using the ALUA mode. If set up properly, there should not be any performance impact when using failovermode 4 (ALUA). Note that FIXED sends I/O down only a single path. However, if you have multiple LUNs in your environment, you could very well choose a preferred path for a given LUN that is different for other LUNs and achieve static I/O load balancing FIXED performs an automatic restore, hence LUNs won&#8217;t end up on a single SP after an NDU.
> 
> When using Round Robin there is no auto-restore functionality, hence after an NDU all LUNs will end up  
> on a single SP. A user would need to manually trespass some LUNs to the other SP in order to balance the load. The benefit of Round Robin is that not too many manual setups are necessary during initial setup; by default it uses the optimal path and does primitive load balancing (however, it still sends I/O down only a single path at a time). If multiple LUNs are used in the environment, you might see some performance boost. If you had a script that takes care of the manual trespass issue, then Round Robin would be the way to avoid manual configuration.

It turned out that the SAN Admin was doing some maintenance on the array and was manually trespassing the LUN to perform updates on the SPs. So the above logs were expected and there was nothing to worry about <img src="http://virtuallyhyper.com/wp-includes/images/smilies/icon_smile.gif" alt="icon smile Seeing a High Number of Trespasses from a CLARiiON Array with ESX Hosts" class="wp-smiley" title="Seeing a High Number of Trespasses from a CLARiiON Array with ESX Hosts" /> 

<div class="SPOSTARBUST-Related-Posts">
  <H3>
    Related Posts
  </H3>
  
  <ul class="entry-meta">
    <li class="SPOSTARBUST-Related-Post">
      <a title="ESX(i) Host Experiencing a lot of Active Path Changes and Disconnects to/from VNX 5300 over iSCSI" href="http://virtuallyhyper.com/2012/08/esxi-host-experiencing-a-lot-of-active-path-changes-and-disconnects-to-vnx-5300-over-iscsi/" onclick="javascript:_gaq.push(['_trackEvent','outbound-article','http://virtuallyhyper.com/2012/08/esxi-host-experiencing-a-lot-of-active-path-changes-and-disconnects-to-vnx-5300-over-iscsi/']);" rel="bookmark">ESX(i) Host Experiencing a lot of Active Path Changes and Disconnects to/from VNX 5300 over iSCSI</a>
    </li>
  </ul>
</div>

<p class="wp-flattr-button">
  <a class="FlattrButton" style="display:none;" href="http://virtuallyhyper.com/2012/04/seeing-a-high-number-of-trespasses-from-a-clariion-array-with-esx-hosts/" title=" Seeing a High Number of Trespasses from a CLARiiON Array with ESX Hosts" rev="flattr;uid:virtuallyhyper;language:en_GB;category:text;tags:Active/Non-Optimal Path,Active/Optimal Path,Active/Passive,ALUA,Assymetric Logical Unit Access,CLARiiON,Explicit/Implicit Failover,failover mode 4,NDU,Preferred Path,Storage Processor,TGP,TGPS,Trespass,useANO,blog;button:compact;">I was recently working on case which had the following setup: Over night the even hosts would start having a lot of disconnects and path changes/thrashing. On the array end...</a>
</p>

 [1]: http://blogs.vmware.com/vsphere/2012/02/configuration-settings-for-alua-devices.html