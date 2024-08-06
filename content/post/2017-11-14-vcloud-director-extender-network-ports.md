---
title: vCloud Director Extender – Network Ports
author: Jon Waite
type: post
date: 2017-11-14T01:08:14+00:00
url: /2017/11/vcloud-director-extender-network-ports/
categories:
  - Firewall
  - Networking
  - vCloud Director
  - vCloud Director Extender
  - VMware
tags:
  - Firewall
  - Installation
  - Networking
  - NSX
  - Ports
  - vCloud Director Extender

---
One of the things which appears to be missing from the [published documentation][1] on vCloud Director Extender (CX) is any mention of the communications internally between the deployed appliances and other VMware infrastructure components (vCenter, vCloud Director etc.) In a service provider context it is unlikely that the appliances will be deployed into the same network/security zone as these components so it is important to know what these communication requirements are.

Using the Flow Monitoring functionality in VMware NSX I was able to capture all traffic flows during vCloud Extender migrations and produce the drawing below detailing these traffic flows.

<h3 style="text-align: center;">
  <strong>Network Traffic Flows for vCloud Extender (Provider Side)</strong>
</h3>

[<img loading="lazy" decoding="async" class="aligncenter size-full wp-image-395" src="https://kiwicloud.ninja/wp-content/uploads/2017/11/CX-Network-Flows-during-migration-2.png" alt="" width="638" height="619" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/11/CX-Network-Flows-during-migration-2.png 638w, https://kiwicloud.ninja/wp-content/uploads/2017/11/CX-Network-Flows-during-migration-2-300x291.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/11/CX-Network-Flows-during-migration-2-155x150.png 155w, https://kiwicloud.ninja/wp-content/uploads/2017/11/CX-Network-Flows-during-migration-2-150x146.png 150w" sizes="(max-width: 638px) 100vw, 638px" />][2]

&nbsp;

Note that the http (tcp/80) access from the replicator appliance to the ESXi hosts appears anomolous &#8211; I would have expected this to be on https (tcp/443) at the very least and this probably needs further investigation.

The 8044/tcp port to the replication manager can be NAT&#8217;d from a different external (public) port if necessary &#8211; this can be configured using the &#8221;Public Endpoint URL&#8221; field when activating the replication manager appliance during vCloud Extender deployment (see my post: <http://152.67.105.113/2017/10/vcloud-director-extender-part-2-cloud-provider-setup/>).

The 44045/tcp port to the replicator appliance can also be NAT&#8217;d from a different external (public) port if necessary &#8211; this can be configured using the &#8220;Public Endpoint URL&#8221; field when activating the replicator appliance during vCloud Extender deployment  (see my post: <http://152.67.105.113/2017/10/vcloud-director-extender-part-2-cloud-provider-setup/>).

Be careful when activating the &#8220;Replication Manager&#8221; and &#8220;Replicator&#8221; appliances &#8211; the configuration screens look very similar and it is reasonably easy to get them mixed up and enter incorrect parameters.

Also note that this diagram only depicts traffic flows for migration activity and doesn&#8217;t capture additional flows involved in L2 network extensions (which typically will be from a hosted NSX edge to either the tenant NSX edge or standalone NSX appliance in the tenant site).

At least the information presented should allow other service providers to configure appropriate network security to protect their internal vCloud and vSphere environments when deploying vCloud Extender components into a DMZ network (for example).

As always, comments and feedback appreciated.

Jon

 [1]: https://docs.vmware.com/en/vCloud-Director-Extender/index.html
 [2]: https://kiwicloud.ninja/wp-content/uploads/2017/11/CX-Network-Flows-during-migration-2.png