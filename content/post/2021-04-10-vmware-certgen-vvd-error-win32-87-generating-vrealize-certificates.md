---
title: 'VMware Certgen VVD Error Win32: 87 Generating vRealize Certificates'
author: Jon Waite
type: post
date: 2021-04-10T06:04:47+00:00
url: /2021/04/vmware-certgen-vvd-error-win32-87-generating-vrealize-certificates/
categories:
  - Uncategorized

---
Whenever I hit an obscure issue using someone else&#8217;s code, I try and write up a quick post if I manage to find the cause (and fix it). Hopefully in this case you&#8217;ve found this article after encountering the same issue and this will provide an easy way to resolve it.

In this case, I was working on deploying a lab environment using VMware Cloud Foundation 4.2 and using vRealize Suite Lifecycle Manager (vRSLCM) to deploy the vRealize Suite components into the VCF environment once deployed. VMware have published a utility (available from the KB page [here][1]{.broken_link}) called &#8216;Certificate Generation Utility for VMware Validated Design for Software-Defined Data Center 6.x&#8217; (yes, bit of a mouthful). This extremely useful article has a script attached which will take a comma-separated csv file generated using the VMware Validated Design (VVD) spreadsheet and automatically request all of the required PKI certificates from a Microsoft Certificate Authority and then bundle up all of the generated certificate files ready for use.

The issue I hit when I first ran this utility was that no mention is made in the pre-requisites section that the Java keytool utility is required to generate some of the certificate bundle formats. Bit of an odd omission, but easy enough to install a current JDK distribution and ensure that the keytool.exe command is included in the PATH.

The next issue I hit was a bit harder to find though, starting the utility from a PowerShell prompt and validating the configuration appeared to show everything was ok:

<img loading="lazy" decoding="async" width="832" height="642" class="wp-image-68901 size-full aligncenter" src="https://kiwicloud.ninja/wp-content/uploads/2021/04/Pasted-3.png" srcset="https://kiwicloud.ninja/wp-content/uploads/2021/04/Pasted-3.png 832w, https://kiwicloud.ninja/wp-content/uploads/2021/04/Pasted-3-300x231.png 300w, https://kiwicloud.ninja/wp-content/uploads/2021/04/Pasted-3-800x617.png 800w, https://kiwicloud.ninja/wp-content/uploads/2021/04/Pasted-3-768x593.png 768w, https://kiwicloud.ninja/wp-content/uploads/2021/04/Pasted-3-150x116.png 150w, https://kiwicloud.ninja/wp-content/uploads/2021/04/Pasted-3-194x150.png 194w" sizes="(max-width: 832px) 100vw, 832px" /> 

When I attempted to generate the CSRs and certificates though I got errors:

<img loading="lazy" decoding="async" class="aligncenter wp-image-68888 size-full" src="https://kiwicloud.ninja/wp-content/uploads/2021/04/Pasted-1.png" alt="Script error" width="840" height="183" srcset="https://kiwicloud.ninja/wp-content/uploads/2021/04/Pasted-1.png 840w, https://kiwicloud.ninja/wp-content/uploads/2021/04/Pasted-1-300x65.png 300w, https://kiwicloud.ninja/wp-content/uploads/2021/04/Pasted-1-800x174.png 800w, https://kiwicloud.ninja/wp-content/uploads/2021/04/Pasted-1-768x167.png 768w, https://kiwicloud.ninja/wp-content/uploads/2021/04/Pasted-1-150x33.png 150w, https://kiwicloud.ninja/wp-content/uploads/2021/04/Pasted-1-250x54.png 250w" sizes="(max-width: 840px) 100vw, 840px" /> 

If I continued with the process by supplying keystore passwords, I then got error pop-ups for each key to be created:

<img loading="lazy" decoding="async" class="aligncenter wp-image-68890 size-full" src="https://kiwicloud.ninja/wp-content/uploads/2021/04/Pasted-2.png" alt="certutil error dialog" width="352" height="152" srcset="https://kiwicloud.ninja/wp-content/uploads/2021/04/Pasted-2.png 352w, https://kiwicloud.ninja/wp-content/uploads/2021/04/Pasted-2-300x130.png 300w, https://kiwicloud.ninja/wp-content/uploads/2021/04/Pasted-2-150x65.png 150w, https://kiwicloud.ninja/wp-content/uploads/2021/04/Pasted-2-250x108.png 250w" sizes="(max-width: 352px) 100vw, 352px" /> 

Very odd, doing a heap of troubleshooting and PowerShell analysis, I finally found the problem &#8211; my Windows Root CA server has spaces in it&#8217;s name (since I set it to be a more &#8216;friendly&#8217; name than the default CN of the server).

The actual name of the CA is &#8216;Auckland Lab DPC Root CA&#8217;, but as you can see in the first screenshot above this has been truncated to &#8216;AucklandLabDPCRootCA&#8217;. Once I&#8217;d worked out what the issue was, finding and fixing the code was reasonably straightforward, in the function &#8216;get-ca-name&#8217; (around line 145 in the CertGen-6.2.002.ps1 script which was the latest available at the time of writing):

<code class="EnlighterJSRAW" data-enlighter-language="powershell">Function get-ca-name {
$val = certutil -adca
$f = ($val | Select-String -pattern 'cn = (.*)').matches[0]
$f = $f -replace " ",""
$CAName = ($f -split "=")[1]
Write-Host "*****[INFO] CA Name = $CAName"
return $CAName
}</code>

Obviously the global replace in $f of all space characters will cause this issue.

Changing the function as follows allows the original CA name (including space characters) to be retained. Note the extra space character after the &#8216;=&#8217; sign in the line $CAName = &#8230;

<code class="EnlighterJSRAW" data-enlighter-language="powershell">Function get-ca-name {
$val = certutil -adca
$f = ($val | Select-String -pattern 'cn = (.*)').matches[0]
$CAName = ($f -split "= ")[1]
Write-Host "*****[INFO] CA Name = $CAName"
return $CAName
}</code>

After making this change to the script I was able to use CertGen to generate all of the vRealize Suite component certificates successfully.

Hopefully this helps out others who hit the same issue.

Jon.

 [1]: https://kb.vmware.com/s/article/78246