
# Software and scripts for stratum-1 NTP server project

This repository includes all scripts for this project. Make sure the repository is located in /opt/ntp
Therefore subfolder "bin" is absolut "/opt/ntp/bin"

# Prerequsits

* Self compiled NTP in /usr/local <br>
** Source from http://www.ntp.org/downloads.html
* Self compiled PTPDv2

### timesync_ksh

This is the shell script to start the NTP process. It is started by the cron process "vfy_ntp_proc_ksh" and does the following:

* sets the system time from the reference server
* correct the 7-staging counter so that the 1PPS starts at beginning of a second
* starts "ntpd"

It can take much more than one hour to synchronize the system.

### vfy_ntp_proc_ksh

This is a sript which will be started from "cron" in an intervall of 30 minutes. If "ntpd" is not running it will start "timesync_ksh"

/etc/cron.d/root should look like

    1,31 * * * * root /opt/ntp/bin/vfy_ntp_proc_ksh


### ppsavgoffset_ksh

### ppscalcdiff_ksh

### ppsverify_ksh

### mydrift
