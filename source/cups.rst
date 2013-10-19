Printing
========

The reference is `cups command-line printing and options on the cups.org
site <http://www.cups.org/documentation.php/options.html>`_ or on your
`local server <http://localhost:631/help/options.html>`_.

-  To know the available options:

   ::

        lpoptions -d foo -l

-  get the lists of available sizes and use paper size of A3 and fit to
   page

   ::

       lpoptions -p foo -l| grep 'PageSize'
       lp -d foo -o fitplot:PageSize=A3 /etc/motd

-  get the lists of slots and use a separate Input slot for first page:

   ::

       lpoptions -p foo -l|grep -i slot
       lp -d foo -o 1:InputSlot=UpperCassette -o InputSlot=LowerCassette

-  watermarks ("Draft" ...) on even pages, gray color mode on odd:

   ::

       lp -d foo -o even:Watermark=on -o odd:ColorMode=Gray file

-  Print some pages, and page ranges:

   ::

       lp -d foo1 -o 1,6-10,15,20- file

-  Print multiple copies from a pipe output:

   ::

       program | lp -d foo -n 8

-  idem with collated copies

   ::

       program | lp -d foo -n 8 -o Collate=true

-  Print landscape, duplex with short side tumble:

   ::

       lp -d foo  -o sides=two-sided-short-edge:landscape file

-  Print with a custom media size (example 624pts width, 312pts length)

   ::

       lp -d foo  -o media=Custom.624x312 file

   You can also use a predefined media size: Letter Legal A4 A5 A6 A7 A8
   B5 B6 B7 B8 C5 C6 DL C7 C8 Custom.WIDTHxHEIGHT . WIDTHxHEIGHT is in
   point or is suffixed with in, cm, mm

-  print one side (even with default duplex) and a banner (*standard*:
   without label, *classified,unclassified, secret, topsecret*: with
   corresponding label)

   ::

       lp -o sides=one-sided:job-sheets=standard

-  Print 4-up (or 1,2,4,16) with double border (or single, single-thick,
   double, double-thick), Bottom to top, left to right (btlr or btrl or
   lrbt or rlbt or rltb or tblr or tbrl):

   ::

       lp -o number-up=4:page-border=double:number-up-layout=btlr file

-  **pretty**\ printing 2 columns (or 3, 4, ...) with 12 char/inch
   (default 10) and 8 lines/inch (default 6):

   ::

       lp -o prettyprint:columns=2:cpi=12:lpi=8 file

-  The borders can be forced by ``page-{left,right,bottom,top}``

Managing printers
-----------------

Choosing a printer:

::

    lpstat -p -d

choosing a printer on another server

::

    lpstat -h server -p -d

setting the default printer:

::

    lpoptions -d printer

The user default printer and options are written in
``~/.cups/lpoptions``, which override system wide options in
``/etc/cups/lpoptions``

Printer queue state:

::

    lpstat -p foo

All printer states:

::

    lpstat -p

Physical devices connected to all printers

::

    lpstat -v

All the sattus and defaults of printer *foo*

::

    lpstat -l -p foo


List all jobs on printer *foo*

::

    lpstat -o foo
    lpq -P foo

Canceling job 12345 on printer *foo*:

::

    cancel 12345 foo

Canceling all jobs from printer *foo*:

::

    cancel -a foo

move job 123 to printer *bar*:

::

    lpmove 123 bar

Define the *device-uri* and *ppd file* for the windows printer *foo*
shared thru samba

::

    lpadmin -p foo -v smb://workgroup/user:mypass@windows-server/inkjet -P /root/inkjet.ppd

Enable and accepts jobs on foo:

::

    lpadmin -p foo -E
