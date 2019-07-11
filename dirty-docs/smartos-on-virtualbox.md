+--------------------------------------------------------------------------+
<div class="pageheader">

<span class="pagetitle"> SmartOS Documentation : SmartOS on VirtualBox
</span>

</div>

<div class="pagesubheading">

This page last changed on Oct 14, 2014 by
<font color="#0050B2">nahamu</font>.

</div>

<div class="panelMacro">

  ---------------------------------------------------------------------
------------------------------------------------------------------------
-----------
  ![](images/icons/emoticons/information.gif){width="16" height="16"}
**Just a stub for now.**\

Need to double check OSX 10.8 and 10.7 - networking seems to just not wo
rk there.
  ---------------------------------------------------------------------
------------------------------------------------------------------------
-----------

</div>

Some people seem to have networking issues while testing SmartOS in
VirtualBox.

- Networking seems to be fine in the global zone
- the GZ can ping a provisioned zone
- the zone can ping the GZ
- the zone itself has no connection to the outer world

Networking seems to be fine after reconfiguring the NIC settings in
VirtualBox. The NIC should be set to bridged mode and to allow promisc
traffic for the Host and all VMs.\
Here is an importable appliance to get that done:
<https://dl.dropbox.com/u/2265989/SmartOS/SmartOS.ova>\
You just need to add the SmartOS iso to be able to boot it.

This seems to work on Windows and Linux (Ubuntu) - on OSX 10.8 this
seems to not work.

<div class="panelMacro">

  ----------------------------------------------------------------- ----
------------------------------------------------------------------------
------------------------------------------------------------------------
------------------------------------------------------------------------
-----------------------------------
  ![](images/icons/emoticons/warning.gif){width="16" height="16"}   **De
pending on how much you value your time, if you are running Mavericks on
 a Mac, you may want to just pony up for VMWare...**\
                                                                    On O
ctober 14, 2014, jritorto commented on IRC that "TIL: fought with virtua
lbox for the whole day trying to get network in zones to work. Finally g
ave up and tried VMWare like I said I was going to at 10:00 this morning
. It worked perfectly first try."
  ----------------------------------------------------------------- ----
------------------------------------------------------------------------
------------------------------------------------------------------------
------------------------------------------------------------------------
-----------------------------------

</div>
+--------------------------------------------------------------------------+

  ----------------------------------------------------------------------------------
  ![](images/border/spacer.gif){width="1" height="1"}
  <font color="grey">Document generated by Confluence on Jul 07, 2019 00:15</font>
  ----------------------------------------------------------------------------------

