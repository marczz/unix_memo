=======
Systemd
=======

systemctl
=========
Reference:
   :man:`systemctl`

.. csv-table::
   :delim: %
   :widths: 50, 60

   $ systemctl status%Show system status
   $ systemctl list-units%List running units
   $ systemctl%List running units
   $ systemctl --failed%List failed units
   $ systemctl --all%List running and inactive units
   $ systemctl list-unit-files%State of all installed units
   $ systemctl status unit%unit status
   # systemctl start unit%start
   # systemctl stop unit%stop
   # systemctl restart unit%restart the unit
   # systemctl reload unit%reload configuration
   $ systemctl is-enabled unit%enabled or disabled?
   # systemctl enable unit%enable to be started on boot
   # systemctl disable unit%disable not to be started on boot
   # systemctl mask unit%forbid starting the unit
   # systemctl unmask unit%unmask
   # systemctl help unit%help for the unit
   # systemctl daemon-reload%reload systemd
   $ systemctl reboot%reboot system
   $ systemctl poweroff%power-off
   $ systemctl suspend%suspend to memory
   $ systemctl hibernate% :wikipedia:`hibernate on disk<Hibernation_(computing)>`
   $ systemctl hybrid-sleep% :wikipedia:`hibernate then sleep<Hibernation_(computing)>`

journalctl
==========
Reference:
   :man:`journalctl`


.. csv-table::
   :delim: %
   :widths: 50, 60

   $ journalctl%system log
   $ journalctl -b%journal since boot
   $ journalctl -b 1%journal of previous boot
   $ journalctl --since "2016-04-07 12:00:00"%journal since some date
   $ journalctl --since "2h ago"%journal since some time
   $ journalctl -u httpd --since=00:00 --until=9:30
   $ journalctl -u ntp%journal for unit ntp
   $ journalctl -f%follow new messages
   $ journalctl -e%jump at end of journal
   $ journalctl -n 1000%show at most 1000 entries
   # usermod -a -G adm lennart%allow user lennart to see logs
