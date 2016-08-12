#snapshooter

##1 Introduction


snapshooter is a bash script which allows you to create btrfs snapshots and send them incrementally to local or remote locations through ssh. Remote servers can be woken up using Wake-on-LAN. It automatically keeps a configurable number of snapshots.


##2 Configuration


All configuration files are by default located in /usr/local/etc/snapshooter/ 
If you want to change this, edit the configdir variable in the actual script.

###2.1 Main configuration file

/usr/local/etc/snapshooter/snapshooter.conf:

	# the periods of your snapshots must be set here. These are common examples.
	periods=( weekly daily hourly )
	
	# Wake-on-LAN binary which should be used
	# This is the default one for arch linux
	wol='/bin/wol'

	# The default time in seconds which the script should wait
	# for the server to be booted. Can be overridden by an individual config.
	defaultboottime=120
	
	# if you want to automatically poweroff the remote server, set this to yes
	# or override it by an individual config.
	poweroffserver='no'
	
	
###2.2 Configs for creation of snapshots:

/usr/local/etc/snapshooter/create/[configname].conf

The individual config files, for subvolumes, which should be snapshoted follow this syntax:


	# name of the subvolume: 'subvol0'
	# or with ',' separated names: 'subvol0,subvol1'
	# recursive snapshooting is supported, just set name to '*'
	name='subvol0'
	
	# if set to yes, the script will search for a subvolume named $name
	# in the $snapdir
	# if you want to snapshot the snapdir itself, for example /,
	# set this to no
	recursive='yes'

	# directory where the subvolume is located
	snapdir='/pool'
	
	# destination of the snapshot, must be on the same physical disk
	destination='/pool/snapshots'
	
	# optional prefix and postfix for the snapshot can be set here. 
	# for example the hostname of the machine, to avoid confusion, 
	# when snapshots of the root filesystem of multiple machines
	# are be backed up on one server
	prefix='laptop'
	postfix='read-only'
	
	# numbers of snapshots, that should be kept.
	# Each period must be specified in snapshooter.conf.
	weekly=5
	daily=6
	hourly=3

A snapshot will be named:
[prefix]\_[name]\_[date]\_[period]\_[postfix]:

laptop_subvol0_2016-07-24_06:00:00_hourly_read-only


###2.3 Configs for sending of snapshots:

/usr/local/etc/snapshooter/send/[configname].conf

config file syntax for snapshots which should be sent:

	# name of the subvolume: 'subvol0'
	# or with ',' separated names: 'subvol0,subvol1'
	# recursive snapshooting is supported, just set name to '*'
	name='subvol0'

	# directory where the snapshots are located.
	# this must be the same directory as the destination
	# of the corresponding "create config"
	snapdir='/pool/snapshots'

	# destination, where snapshots should be sent to
	destination='/mnt/usb-hdd'

	# optional prefix and postfix for the snapshot can be set here. 
	# for example the hostname of the machine, to avoid confusion, 
	# when snapshots of the root filesystem of multiple machines
	# are be backed up on one server
	prefix='laptop'
	postfix=''

	# numbers of snapshots, that should be kept.
	# Each period must be specified in snapshooter.conf.
	weekly=5
	daily=6
	hourly=3

	# if snapshots should be sent to a remote location
	# public key authentification must be set up.
	sshlogin='root@destination-server'
	# if you use a custom ssh port:
	port='xxxxx'

	# if the server should be woken up, set the mac address here
	mac='XX:XX:XX:XX:XX:XX'
	# optional ip address of server
	ip='192.168.1.45'

	# optionally override the default boot time ( in seconds )
	boottime=180

	# it the server shoud be shut down, after the sending of snapshots,
	# set to yes
	poweroffserver='yes'


##3 Using the script:


###3.1 creating snapshots:

snapshooter create [period]

[period] is one of the periods specified in the main config file, for example daily.

The script will then create daily snapshots for every config in /usr/local/etc/snapshooter/create/ 
and get rid of old snapshots automatically.



###3.2 sending snapshots:

snapshooter send [configname]

[configname] is the name of the config, which will be used.
specifying multiple with ',' separated confignames is supported.
if you omit the [configname] option, then all confis in /usr/local/etc/snapshooter/send/ will be used.

The script will now determine whether a parent snapshot is present and if not it will bootstrap it.
Once it exists it will send the following snapshots incrementally.
Old snapshots will be deleted automatically.



##4 Automation

The script is supposed be automaticly run using systemd or cron.

https://wiki.archlinux.org/index.php/Systemd/Timers  
This article will help you create these units if you do not already know how to write them.

NOTE: The script must be run as root, since the btrfs commands require it.

##5 Miscellaneous

Disclaimer:
I'm in no way responsible for any damage or dataloss the script may cause, through bugs or incorrect configuration.
Use at your own risk.

The script is Provided under a GNU GPLv3 License
