# TA-meraki
TA-meraki 1.1.3
Cisco Meraki Technology Adapter
Supports all tested versions of Splunk (tested up to 7.x)

This application is primarily supported via the following splunk answers forum:
https://github.com/AlaskaSSO/TA-meraki
https://answers.splunk.com/app/questions/3018.html

This is a Splunk Technology Adapter for Cisco Meraki;  the nomenclature has changed in recent releases to be called a "splunk add-on".  This app uses the classic naming.

All you need to do ahead of time is make sure Meraki logs are tagged as sourcetype=meraki AND in index meraki and you should be good to go.  If this is not the case, then I recommend creating a duplicate in locals which overrides the hard coded index searches.

Request:  Currently most of all of the log lines are being correctly identified.  The ones that are not are likely found with the following search:
"sourcetype=meraki app="meraki-events"".

If you have log lines which match that search I would appreciate anyone sending them in so I add additional extractions/identification.


CHANGELOG: 
Version 1.1.3
Thank you Peter McCarthy from SageNet for finding at dvc_ip regex problem

Version 1.1.2
Thank you Jason Mantor for finding an WARN in the lease_scope.

Version 1.1.1
Handle both strip and non-strip of \d{4,}\.\d{4,}

Version 1.1.0
Started to fill out CIM for wireless ids and network session start/stop.  I don't have a definition for wireless ips, so
I put everything as informational and blocked for now

Version 1.0.9
minor fixes

Version 1.0.8
Preliminary support for WAP Meraki devices thank you John Ward from spicosolutions

Version 1.0.7
Released under (CC BY-SA 3.0) license

Version 1.0.6
changed category field to a multi-field in order to pick up multiple category websites

Version 1.0.5
bugfix for AP, flows were reported at the end of the log line unlike the other devices.  Removed to blank space check at the end of [meraki_dest_port2]

Version 1.0.4
cleanup on DHCP portion to make it easier to read
cleanup on signature_id to become more useful (applied signature_id to dhcp entries based on Microsoft DHCP error ids)
applied coalesce to a few different variables that were being reported by different regex's (signature_id,meraki_action,meraki_priority)

Version 1.0.3
bugfix report of different logformat for flows on MX access point.  Added new extractions for flows on AP.
bugfix regarding if you search for a signature created in the dhcp portion it was being overwritten by #FIELDALIAS-signature = category AS signature from the web portion; changed to a coalesce so now you can do a regular search on both without searching the model

Version 1.0.2
jgrayccm - bugfix typo in dhcp events for eventtype=meraki-dhcp

Version 1.0.1
added meraki_dhcp_lease_release extraction
added meraki_events_ad extraction, basic identification for Active Directory activity
added meraki_date_clipper, if added to indexer or heavy forwarder and feed comes through syslog this removes unix timestamp date and saves about 19 bytes of data per log file (normal syslog timestamp still exists)

Version 1.0.0
Minor update
updated version for Splunk Certification

Version 0.0.9
Minor update
cleanup for app certification status

Version 0.0.8
Minor update
cleanup for app certification

Version 0.0.7
Minor update
changed default app status to disabled for app certification status
modified documentation/icons and contact information for app certification process
This version will be submitted to application certification, app has been around for a while and I believe all/most major/minor bugs have been squashed

Version 0.0.6
Minor update
added extraction for port status change (will eventually be added to CIM change analysis)
added extraction for authentication log under events-authentication
TODO: deal with more unparsed event types (i.e. vrrp); feel free to submit log samples
TODO: still deal with block messages regarding CIM compliance

Version 0.0.5
Added DHCP CIM compliance (via meraki_app="events-dhcp")
moved status codes/rule messages to a lookup app so it will be more forward compatible in the future
moved web blocks to a new meraki_app called events-content_filtering_block, unfortunately with no source IP address I can't add it into the CIM.  Maybe put the device as the src IP?
TODO: deal with more unparsed event types (i.e. vrrp); feel free to submit log samples

Version 0.0.4
Added Web CIM compliance
todo: still deal with block messages regarding CIM compliance

Version 0.0.3
Added ICMP Code Type resolution
Split Meraki into 2 sub event types instead of having them all as one
(IDS, and Network Traffic) for CIM compliance
Fixed a bug with splunk 6.3.2 regarding concat of fields; moved to running 2 extractions instead of one.  Previously fieldname:$1$2 which no longer works
Fixed a typo in signature and signature-id
added a couple of fields to start on the progress of adding web CIM support to Meraki

TODO:  Web CIM compliance

0.0.2

These are a set of Meraki extractions that are partially CIM compliant.  These were developed from me seeing the events on my own system.

TA-meraki should be installed index side and search side.

These are likely NOT complete; but currently everything on my system is detected.  Please feel free to submit logs or things that don't work
and I'll fix them.

---
File: README-inputs.txt
 
This TA-app assumes the following:

Cisco Meraki logs will be deposited into index meraki
Cisco Meraki logs will all have sourcetype meraki

This is a technology adapter that enables front end applications to view meraki data via the common information model. If the front end is written to CIM standards your meraki data will automatically appear in that app. Examples include Splunk Enterprise Security (and likely others).

This app provides the following common information models:
[eventtype=meraki-ids-alerts]
ids = enabled
attack = enabled
[eventtype=meraki-flows]
network = enabled
communicate = enabled
[eventtype=meraki-urls]
web = enabled
proxy = enabled
[eventtype=meraki-dhcp]
network = enabled
session = enabled
dhcp = enable

Due to difficulty in sometimes identifying the various services meraki provides; I recommend opening up a separate port on your syslog server with a filter as listed below; or adding a new UDP high address port on a heavy forwarder, or if you only had one indexer that box and then configuring that box as a syslog server with the UDP high address port chosen.

inputs.conf

[default]
host_segment = 4
[monitor:///logpartition/logs/meraki/*/2016/...]
SHOULD_LINEMERGE=false
sourcetype = meraki
index=meraki

[monitor:///logpartition/logs/meraki/*/2017/...]
SHOULD_LINEMERGE=false
sourcetype = meraki
index=meraki

Sample config for syslog-ng

port to process meraki

source s_ext_udp_15146 {
udp(so_rcvbuf(1073741823) log_fetch_limit(10000) port(15146));
};
filter f_meraki { facility(local0) };
log {
source(s_ext_udp_15146);
filter(f_meraki);
destination(d_meraki);
};
destination d_meraki {
file("/logpartition/logs/meraki/$HOST/$YEAR/$MONTH/$DAY/meraki-$YEAR-$MONTH-$DAY"
owner(root) group(adm) perm(0640) dir_perm(0751) dir_group(adm) create_dirs(yes) template("$ISODATE $HOST $MSGHDR$MSGONLY\n"));
};
