Memory and swap management
==========================

.. highlight:: bash


Inspecting and tuning ram
-------------------------

.. _free_mem:

Free memory
~~~~~~~~~~~

.. code-block:: bash

   $ free -m

output::

                total    used     free   shared  buffers cached
   Mem:          2003    1618      385        0      136    592
   -/+ buffers/cache:     889     1113
   Swap:         2047       0     2047

It means you have 2G ram 1.6G are used 889M actively by applications and
728M can be reclaimed 136M from buffers and 592M from cached data; the
total amount of space that could be used is 1.1G

2G of swap are unused.

You can find more explanations in `Linux ate my ram
<http://www.linuxatemyram.com/play.html>`_
and in `Tips for Optimizing Linux Memory Usage
<https://www.linuxjournal.com/article/2770>`_.

A very good ref is also
`Tuning the Memory Management Subsystem
<http://doc.opensuse.org/documentation/html/openSUSE/opensuse-tuning/cha.tuning.memory.html#cha.tuning.memory.usage>`_.
chapter 15 of `openSUSE System Analysis and Tuning Guide
<http://doc.opensuse.org/documentation/html/openSUSE/opensuse-tuning/>`_.

Managing swap space
-------------------


Swap Info
~~~~~~~~~

::

   $ swapon -s


The output is::

  Filename            Type		Size	Used	Priority
  /dev/mapper/vg-swap   partition               2097148 0       -1

or :ref:`free -m <free_mem>`


Swap partition
~~~~~~~~~~~~~~

::

   $ mkswap /dev/mapper/vg-swap
   $ swapon /dev/mapper/vg-swap

In `fstab`::

   /dev/mapper/vg-swap none  swap  sw 0 0

Swap file
~~~~~~~~~

::

   $ dd if=/dev/zero of=/swapfile bs=1M count=512

if the file system is ext4 or brtfs *but not ext2/3* you can also use
the quicker::

   $ fallocate -l 512M /swapfile

Then::

  $ chmod 600 /swapfile
  $ mkswap /swapfile
  $ swapon /swapfile

  /swapfile none swap defaults 0 0

Swappiness
~~~~~~~~~~

A low value of kernel swappiness parameter will reduce swapping from RAM,
default is 60 and current value is::

  $ cat /proc/sys/vm/swappiness

To test swapiness::

  $ echo 5 > /proc/sys/vm/swappiness

To set it at boot put in `/etc/sysctl.conf`::

  vm.swappiness = 5
