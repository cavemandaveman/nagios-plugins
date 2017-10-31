# check_lvm_thinpools
A plugin for checking the size of LVM thinpools through SNMPv3

## Usage
```
This script will check LVM thinpool disk utilization of a remote machine via SNMPv3.
It requires all of the following options to be defined.

       -u  SNMPv3 read-only user
       -a  Auth protocol (sha or md5)
       -A  Auth password
       -P  Priv password
       -H  Hostname or IP to check against
       -w  Warning threshold
       -c  Critical threshold
```

`-w` and `-c` should be any number 0.00 - 100.00 and will reflect the percentage of the thinpool used.

## Notes
Checking thinpool usage is something that needs to be done with root privileges, which can be achieved many ways. You could edit your sudoers file to whitelist the `lvs` command or whitelist a script that utilizes `lvs`. You could have snmpd or Nagios be run as the root user on your hosts. While one of these might be an easier approach, they are less secure.

This script has a lot of moving parts to accommodate the fact that remote snmp commands are not being run as the root user, but it is a more secure approach. The requirements for this script to work are:

*   A cron job on the remote host you want to check, which writes the output we want to file.
*   A net-snmp extension on the remote host that reads from said file.

The following process has only been tested on a CentOS 7 remote host, but should work for Debian/Ubuntu as well.

#### Cron Job
1.  Create a new directory for monitoring:
```
mkdir /opt/monitoring/lvm/
```
2.  Create a new file `/etc/cron.d/lvs_thinpool_output` with the following contents:
```
*/3 * * * * root /usr/sbin/lvs --noheadings --separator ':' -o lv_name,data_percent -S 'data_percent >= 0' | /usr/bin/sed 's/ //g' > /opt/monitoring/lvm/lvs_thinpool_output.txt
```
This will have the root user write the output we want from `lvs` every 3 minutes to a file in the directory we just created. Depending on how often you have Nagios poll, you can change it to run at whatever interval suits you.

#### Extend Net-SNMP
What we want to do is create an extension to net-snmp that we can use in our Nagios script. If you want to read more in depth about extending net-snmp, the [RHEL docs](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/sect-system_monitoring_tools-net-snmp#sect-System_Monitoring_Tools-Net-SNMP-Extending) are a great resource.
1.  Add the following to the end of `/etc/snmp/snmpd.conf`:
```
# Extending Net-SNMP
extend check_lvm_thinpools /bin/cat /opt/monitoring/lvm/lvs_thinpool_output.txt
```
2.  Restart snmp:
```
sudo systemctl restart snmpd
```

That's it! Now this script will invoke our custom snmp extension and return the results we want.
