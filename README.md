
# Software and scripts for stratum-1 NTP server project

This repository includes all scripts for this project. Make sure the repository is located in /opt/ntp <br>
Therefore subfolder "bin" should have the absolut path "/opt/ntp/bin"

## Prerequsits

* Hardware and OS
* Self compiled NTP in /usr/local <br>
* Self compiled PTPDv2

### OS

For my environment I used a BananaPi M1 with GNU/Linux.

### ntp

I always take the latest source from http://www.ntp.org/downloads.html. NTPsec is currently not supported and it will not work. To compile the source do the following:

<pre>
export CC=gcc
export LDFLAGS=-L/usr/lib/arm-linux-gnueabihf

./configure  '--with-sntp' '--enable-RAWDCF' --enable-SHM --enable-ATOM -enable-NMEA '--enable-autokey' '--enable-simulator'  \
--with-crypto --with-openssl-libdir=/usr/lib/arm-linux-gnueabihf

make
sudo make install
</pre>

This will install all necessary file below /usr/local.

Make sure that "ntpq" doesn't ask for "key" and "password" if trying to modify the configuration. See "ntpq.c.patch" below.


### ptpd2

"ptp" protocol is used for the initial synchronization. It synchronize the system much faster than "ntpd". The reference server must be the master. The stratum-1 NTP server will be the client for short time. "ptpd2" is available from a git repository:

    https://github.com/ptpd/ptpd.git



## Subfolder "bin"

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

usage: ppsavgoffset_ksh [-d] pps-device

This script is used by "timesync_ksh". It read timestamps of a pps-device sources and calculates the average offset and deviation. Using option "-d" shows 18 meassuring points. The last line is average and deviation. Both must be interger for further calculation in "timesync_ksh".


### ppsverify_ksh

usage: ppsverify_ksh [-d] pps-device

This script is executed at the beginning of "timesync_ksh" to check if the given pps-device is available or not. It will terminate with exit status 0 if this device is available otherwise with status 1.

## subfolder "etc"

### ntp.conf.example

This is an example of ntp.conf where you have adopt a lot of things. The final location is typically /etc/ntp.conf <br>
The most important changes are: <br>
* defining the pps-device
* defining the preferred server
* defining the key for authorization

Make sure that "ntpd" is NOT started at the boot process. It will be started by "timesync_ksh". A second important step is that "ntpq" can configure the deamon.

## subfolder "src"

### ntpq.c.patch

This is a patch for "ntpq.c". It prevents that you will be asked for a password configuring "ntpd". Of course you have to configure properly /etc/ntp.conf and /root/.ntprc
