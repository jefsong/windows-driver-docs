---
title: KSPROPERTY\_DVDCOPY\_TITLE\_KEY
description: The KSPROPERTY\_DVDCOPY\_TITLE\_KEY property retrieves the title key information from the current content for the DVD copyright protection authentication process.
ms.assetid: 7c07bf75-cbc4-4319-a1a6-4f05d228d91a
keywords: ["KSPROPERTY_DVDCOPY_TITLE_KEY Streaming Media Devices"]
topic_type:
- apiref
api_name:
- KSPROPERTY_DVDCOPY_TITLE_KEY
api_location:
- ksmedia.h
api_type:
- HeaderDef
ms.author: windowsdriverdev
ms.date: 11/28/2017
ms.topic: article
ms.prod: windows-hardware
ms.technology: windows-devices
---

# KSPROPERTY\_DVDCOPY\_TITLE\_KEY


The KSPROPERTY\_DVDCOPY\_TITLE\_KEY property retrieves the title key information from the current content for the DVD copyright protection authentication process.

## <span id="ddk_ksproperty_dvdcopy_title_key_ks"></span><span id="DDK_KSPROPERTY_DVDCOPY_TITLE_KEY_KS"></span>


### <span id="Usage_Summary_Table"></span><span id="usage_summary_table"></span><span id="USAGE_SUMMARY_TABLE"></span>Usage Summary Table

<table>
<colgroup>
<col width="20%" />
<col width="20%" />
<col width="20%" />
<col width="20%" />
<col width="20%" />
</colgroup>
<thead>
<tr class="header">
<th>Get</th>
<th>Set</th>
<th>Target</th>
<th>Property descriptor type</th>
<th>Property value type</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p>No</p></td>
<td><p>Yes</p></td>
<td><p>Pin</p></td>
<td><p>[<strong>KSPROPERTY</strong>](https://msdn.microsoft.com/library/windows/hardware/ff564262)</p></td>
<td><p>[<strong>KS_DVDCOPY_TITLEKEY</strong>](https://msdn.microsoft.com/library/windows/hardware/ff567640)</p></td>
</tr>
</tbody>
</table>

 

The property value (operation data) is a KS\_DVDCOPY\_TITLEKEY structure that describes the current title key.

Remarks
-------

For more information about the title key, see [DVD Copyright Protection](https://msdn.microsoft.com/library/windows/hardware/ff558736).

Requirements
------------

<table>
<colgroup>
<col width="50%" />
<col width="50%" />
</colgroup>
<tbody>
<tr class="odd">
<td><p>Header</p></td>
<td>Ksmedia.h (include Ksmedia.h)</td>
</tr>
</tbody>
</table>

## <span id="see_also"></span>See also


[**KS\_DVDCOPY\_TITLEKEY**](https://msdn.microsoft.com/library/windows/hardware/ff567640)

 

 






