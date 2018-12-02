Wpa_supplicant
==============

*See also* `MzLinux Wifi Page <http://www.mzlinux.org/node/253>`_.

-  `Linux WPA/WPA2/IEEE 802.1X
   Supplicant <http://w1.fi/wpa_supplicant/>`_, has support for WPA and
   WPA2 Supported wireless cards/drivers it includes Host AP driver for
   Prism2/2.5/3, Linuxant DriverLoader, Agere Systems Inc. Linux Driver
   (Hermes-I/Hermes-II chipset), madwifi (Atheros ar521x) ATMEL
   AT76C5XXx, Linux ndiswrapper, Broadcom wl.o driver, Intel ipw2100.
-  `wpa supplicant
   README <http://w1.fi/gitweb/gitweb.cgi?p=hostap.git;a=blob_plain;f=wpa_supplicant/README>`_.
-  `Example wpa\_supplicant configuration
   file <http://w1.fi/gitweb/gitweb.cgi?p=hostap.git;a=blob_plain;f=wpa_supplicant/wpa_supplicant.conf>`_
-  `Debian: WiFi How To Use <http://wiki.debian.org/WiFi/HowToUse>`_
-  `WPA support in Debian <http://wiki.debian.org/WPA>`_
-  `Quick Start Guide to Debian and WPA Wireless
   Security <http://www.fmepnet.org/debian_wpa.html>`_ by Mike Shuey -
   2006.
-  ArchWiki: :archwiki:`WPA supplicant`.
-  Ubuntu `WiFi Doc <https://help.ubuntu.com/community/WifiDocs>`_: `WPA
   HowTo <https://help.ubuntu.com/community/WifiDocs/WPAHowTo>`_, `WiFi
   HowTo <https://help.ubuntu.com/community/WifiDocs/WiFiHowTo>`_

Using wpa\_supplicant
---------------------

If you want dispense from using an heavy network manager; or if you
don't have the graphic desktop; or the resources to do it you can
connect to a roaming wifi ap directly with wpa\_supplicant and wpa\_cli.
If you have a graphic interface wpa\_gui will make the work a lot
simpler.

You have first to set in your ``/etc/network/interface`` a configuration
entry like::

  iface roam inet manual
  wpa-driver wext wpa-roam

In ``/etc/wpa_supplicant_roam.conf``::

  ctrl_interface=DIR=/var/run/wpa_supplicant
  ctrl_interface_group=netdev
  update_config=1

  network={
      key_mgmt=NONE
      disabled=1
  }

It must be writable by the
group from which you will control the interface::

  $ sudo chown root:netdev /etc/wpa_supplicant/roam.conf
  $ sudo chmod 660 /etc/wpa_supplicant/roam.conf


To test your configuration you can test with debug as::

   $ wpa_supplicant  -Dwext -iwlan0 -d -C/var/run/wpa_sup

Then to background the daemon::

   $ wpa_supplicant  -Dwext -iwlan0 -B -C/var/run/wpa_sup

In production you usually launch wpa\_supplicant with::

  $ sudo ifup -v wlan0=roam

You can inspect the scanned networks and change the configuration
either with :man:`wpa\_gui` or with the command line :man:`wpa\_cli`.

:man:`wpa\_gui` is quite simple just do scan, then edit the network following
what you found, and connect.

:man:`wpa\_cli` is less easy, but you can do the following:

::

    > scan
    SCANNING
    > scan_results
    bssid / frequency / signal level / flags / ssid
    00:0f:66:56:f1:b3       2432    196     [WPA-PSK-TKIP][ESS]     myap
    > set_network 1 key_mgmt WPA-PSK
    > set_network 1 psk "mysecretpassword"
    > set_network 1  pairwise TKIP

You may prefer to use ``iwlist`` to scan your network as it gives more
information.

::

    $ iwlist wlan0 scanning
    wlan0     Scan completed :
              Cell 01 - Address: 00:0F:66:56:F1:B3
                        Channel:5
                        Frequency:2.432 GHz (Channel 5)
                        Quality=54/70  Signal level=-56 dBm
                        Encryption key:on
                        ESSID:"myap"
                        Bit Rates:1 Mb/s; 2 Mb/s; 5.5 Mb/s; 11 Mb/s; 18 Mb/s
                                  24 Mb/s; 36 Mb/s; 54 Mb/s
                        Bit Rates:6 Mb/s; 9 Mb/s; 12 Mb/s; 48 Mb/s
                        ....
                        IE: WPA Version 1
                            Group Cipher : TKIP
                            Pairwise Ciphers (1) : TKIP
                            Authentication Suites (1) : PSK



An example of full session, adding a new AP is

::


   # wpa_cli -g/var/run/wpa_sup
   > status
   wpa_state=INACTIVE
   address=00:c9:33:34:4f:ed
   > scan
   OK
   > scan_result
   bssid / frequency / signal level / flags / ssid
   de:ad:be:ef:70:e2	2422	-48	[WPA2-EAP-TKIP+CCMP][ESS]	FreeAP_secure
   de:ad:be:ef:70:e0	2422	-48	[WPA-PSK-CCMP][ESS]	AP-123456
   de:ad:be:ef:70:e1	2422	-60	[ESS]	FreeAP
   > add_network
   0
   > set_network 0 ssid "AP-123456"
   OK
   > set_network 0 psk "mysecretpassword"
   OK
   > enable_network 0
   OK
   > status
   bssid=de:ad:be:ef:70:e0
   ssid=AP-123456
   id=0
   mode=station
   pairwise_cipher=CCMP
   group_cipher=CCMP
   key_mgmt=WPA-PSK
   wpa_state=COMPLETED
   address=00:c9:33:34:4f:ed
   > set update_config 1
   OK
   > set_network 0 proto WPA
   OK
   > set_network 0 key_mgmt WPA-PSK
   OK
   > set_network 0 pairwise CCMP
   OK
   > set_network 0 group CCMP
   OK
   > save_config
   OK
