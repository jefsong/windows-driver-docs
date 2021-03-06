---
title: Provisioning for hotspot authentication
description: Provisioning for hotspot authentication
ms.assetid: bfb4e1ec-9887-4b25-bfcc-be642b1a0101
ms.author: windowsdriverdev
ms.date: 04/20/2017
ms.topic: article
ms.prod: windows-hardware
ms.technology: windows-devices
---

# Provisioning for hotspot authentication


For an app to participate in the hotspot authentication process, it must first create one or more profiles for Wi-Fi hotspots. This is done by using the Provisioning Agent interface that is discussed in [Using metadata to configure mobile broadband experiences](using-metadata-to-configure-mobile-broadband-experiences.md). The hotspot must use open authentication and must include the **HotspotProfile** element. The following provisioning file sample shows how to associate an SSID with your app:

``` syntax
<WLANProfile xmlns="http://www.microsoft.com/networking/CarrierControl/WLAN/v1">
  <name>Contoso Wi-Fi</name>
  <SSIDConfig>
    <SSID>
      <name>Contoso Wi-Fi___33</name>
    </SSID>
  </SSIDConfig>
  <MSM>
    <security>
      <authEncryption>
        <authentication>open</authentication>
        <encryption>none</encryption>
        <useOneX>false</useOneX>
      </authEncryption>
      <HotspotProfile xmlns="http://www.microsoft.com/networking/WLAN/HotspotProfile/v1">
        <ExtAuth>
          <ExtensionId>YourAppIdGoesHere</ExtensionId>
        </ExtAuth>
        <TrustedDomains>
          <TrustedDomain>www.mycaptiveportal.com</TrustedDomain>
        </TrustedDomains>
        <UserAgent>contoso</UserAgent>
      </HotspotProfile>
    </security>
  </MSM>
</WLANProfile>
```

The ExtensionId field contains the package family name of the app that generates hotspot credentials. The package family name is automatically generated by Visual Studio. To find the package family name for your application, open the **package.appxmanifest** file in your Visual Studio solution and go to the Packaging window.

After the provisioning file is processed, the app that has the package family name “YourAppIdGoesHere” must register for the Hotspot Authentication event. It is required that the provisioning file is processed first to grant the specified app access to this event. An app can register a single handler for this event. The event registration remains valid as long as there is at least one profile that refers to the corresponding app.

## <span id="sign"></span><span id="SIGN"></span>Sign the provisioning file


Because provisioning modifies system settings that persist after the user has exited or even uninstalled the app, a stricter measure of verification is required than for most APIs. This verification is provided by a combination of operator-specific hardware (the SIM), cryptographic signatures, and user confirmation. The following table lists the verification requirements:

<table>
<colgroup>
<col width="25%" />
<col width="25%" />
<col width="25%" />
<col width="25%" />
</colgroup>
<thead>
<tr class="header">
<th>SIM present</th>
<th>Provisioning source</th>
<th>Signature requirement</th>
<th>User confirmation requirement</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p>Yes, MB provider</p></td>
<td><p>Mobile broadband app</p></td>
<td><p>None</p></td>
<td><p>None</p></td>
</tr>
<tr class="even">
<td><p>Yes, MB provider</p></td>
<td><p>Operator web site</p></td>
<td><p>Certificate must:</p>
<ul>
<li><p>Chain back to trusted root CA</p></li>
<li><p>Be associated with mobile broadband hardware in APN database or experience metadata</p></li>
</ul></td>
<td><p>None</p></td>
</tr>
<tr class="odd">
<td><p>No, Wi-Fi provider</p></td>
<td><p>Mobile broadband app or web site</p></td>
<td><p>Certificate must:</p>
<ul>
<li><p>Chain back to trusted root CA</p></li>
<li><p>Be marked for Extended Validation</p></li>
</ul></td>
<td><p>User is prompted to confirm the first time the certificate is used; none thereafter.</p></td>
</tr>
</tbody>
</table>

 

## <span id="related_topics"></span>Related topics


[WISPr authentication](wispr-authentication.md)

 

 






