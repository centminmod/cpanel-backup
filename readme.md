# cpanel-backup.sh

`cpanel-backup.sh` is a cPanel/WHM backup script I was hired to write for a client to allow data migration from cPanel/WHM to [Centmin Mod LEMP stack](https://centminmod.com/) as a result of the [cPanel announced price increase and licensing changes](https://community.centminmod.com/threads/17849/). This is the backup part with an accompanying import part to come and backup data transfer part as well - when combined will make up the data migrator for importing into Centmin Mod LEMP stack based servers. Centmin Mod LEMP stack isn't for shared hosting, so isn't always an alternative to cPanel. However, Centmin Mod LEMP stack is best suited for single site owner managed own sites where there's trust. Note: there is already a manual guide for [cPanel/WHM to Centmin Mod data migration](https://community.centminmod.com/threads/how-to-transfer-cpanel-whm-sites-to-centmin-mod-lemp-servers.11610/).

# features

* `cpanel-backup.sh` will have features from [dbbackup.sh](https://community.centminmod.com/threads/dbbackup-sh-quick-mysql-database-backups-for-centmin-mod-stack.4573/) including conditional db character set detection & 
  pure innodb database detection for conditional single-transaction options and using nice/ionice for tar backups. 
* `cpanel-backup.sh` will also have sar & pidstat cpu, memory and disk usage stats recorded allowing you to fine tune your backup and compression parameters for your specific server and backup requirements i.e. lower memory & cpu usage.
* pidstat logs and backup logs also also zstd level 1 compressed to save space.
* `cpanel-backup.sh` will support the following compression algorithms, gzip via multi-threaded pigz, xz via multi-threaded pxz and zstd ([zstd is the default due to speed and compression ration performance](https://community.centminmod.com/threads/round-3-compression-comparison-benchmarks-zstd-vs-brotli-vs-pigz-vs-bzip2-vs-xz-etc.17259/))
* `cpanel-backup.sh` compression algorithm level settings have finer granular control on a per backup target basis, so public_html, mail, logs, ssl or database backup targets each have their own compression level control so you can optimise compression speed and compressed file size as well as control the amount of server resources used (cpu, memory etc)
* zstd compression is the default compression used with tar backups and has both normal and a low memory mode to reduce memory used for compression. 
* For cPanel log files if you opt to back them up, `cpanel-backup.sh` will conditionally reduce zstd compression level to lowest negative 10 (--fast=10) levels to not waste time if the script detects there are already a mix of compressed & uncompressed version of your logs (due to logrotate). FYI, zstd has compression levels from fastest to slowest (smallest compressed file size) from -10 to 19 and then 3 ultra levels 20-22.
* `cpanel-backup.sh` will support backing up all cpanel user accounts in same session as well as per cpanel user account backups on command line.
* `cpanel-backup.sh` will support alias name masking for domain name, username and database name to allow you to publicly demo Slack notifications on live cPanel user data but still keep actual domain name, username, and database names private.
* `cpanel-backup.sh` will backup cPanel user's domain mapping for main domain, subdomains, parked domains, git repositories, cronjobs and DNS zone files if they exist.
* `cpanel-backup.sh` will also optionally sending slack channel notifications on successful or failed backup targets i.e. public_html, mail, logs, ssl or database backups. All notifications are colour coded - green = successful or red = failed for backup status for each backup target. This allows quick visual inspection of which backup targets failed their backup runs.

# slack channel notifications

cpanel user = cpuser1 public_html backup

![](/screenshots/cpanel-backup.sh-slack-cpuser1-publichtml-01.png)

mysql database backup using zstd level 3 compression

![](/screenshots/cpanel-backup.sh-slack-cpuser1-mysql-01.png)

slack channel searching backup notification logs

![](/screenshots/cpanel-backup.sh-slack-cpuser1-search-01.png)

## alias masked slack channel notifications

For privacy reasons, alias masked notifictions for public demos of Slack channel notifications.

cpanel user = cpuser1 public_html backup

![](/screenshots/cpanel-backup.sh-slack-cpuser1-publichtml-aliasmasked-01.png)

mysql database backup using zstd level 3 compression

![](/screenshots/cpanel-backup.sh-slack-cpuser1-mysql-aliasmasked-01.png)

## failed backup notifications

Example of failed mysql backup with alias db name masking enabled. Colour coded Slack notification, red = failed backup

![](/screenshots/cpanel-backup.sh-slack-cpuser1-mysql-aliasmasked-failed-01.png)

# raw output example

`cpanel-backup.sh` preview for single cpanel user backup mode to back cpanel user = `cpuser1` with alias domain name, cPanel username and database name masking enabled only for Slack channel notifications (script outputs the real names) and default zstd compression algorithm used and zstd compressed logs.

with all backup targets enabled for public_html, logs, mail, mysql, ssl and git repositories

```
# backup /home/username/public_html
BACKUP_WEBROOT='y'
# backup /home/username/mail
BACKUP_MAIL='y'
# backup MySQL databases prefixed with cpanel username_*
BACKUP_MYSQL='y'
# backup /home/username/logs
BACKUP_LOGS='y'
# backup /home/username/ssl
BACKUP_SSL='y'
# backup /home/username/repositories
BACKUP_GITREPOS='y'
```

where `/home/cpuser1` size wise is ~1.1GB for files and ~1.2GB for MySQL databases

```
du -s /home/cpuser1
1058348 /home/cpuser1

du -s /var/lib/mysql/cpuser1*
1204552 /var/lib/mysql/cpuser1_db1
8       /var/lib/mysql/cpuser1_db2
8       /var/lib/mysql/cpuser1_db3
```

```
/root/tools/cpanel-backup.sh cpuser1

-------------------------------
cPanel/WHM backup script 0.8
for data migration to Centmin Mod LEMP stack imports
written by George Liu (centminmod.com)
--------------------------------------------------------

--------------------------------------------------------
list cpanel users
--------------------------------------------------------

cpuser1

--------------------------------------------------------
list cpuser1 domain mapping
--------------------------------------------------------

{
  "parked_domains": [
    "domain1.biz",
    "domain1.info"
  ],
  "addon_domains": {},
  "main_domain": "domain1.com",
  "sub_domains": [
    "ads.domain1.com",
    "m.domain1.com"
  ]
}

main_domain=domain1.com

sub_domains:

ads.domain1.com
m.domain1.com

parked_domains:

domain1.biz
domain1.info

cpuser_domainlist=domain1.com ads.domain1.com m.domain1.com domain1.biz domain1.info

--------------------------------------------------------
domain mapping saved:
/home/backup-accounts/cpuser1/domain-map-cpuser1-040719-094323.txt
--------------------------------------------------------

--------------------------------------------------------
backup cpuser1 domain related DNS zone files
--------------------------------------------------------

backup /var/named/domain1.com.db
cp -af /var/named/domain1.com.db /home/backup-accounts/cpuser1/named/domain1.com.db

backup /var/named/ads.domain1.com.db
cp -af /var/named/ads.domain1.com.db /home/backup-accounts/cpuser1/named/ads.domain1.com.db

backup /var/named/m.domain1.com.db
cp -af /var/named/m.domain1.com.db /home/backup-accounts/cpuser1/named/m.domain1.com.db

backup /var/named/domain1.biz.db
cp -af /var/named/domain1.biz.db /home/backup-accounts/cpuser1/named/domain1.biz.db

backup /var/named/domain1.info.db
cp -af /var/named/domain1.info.db /home/backup-accounts/cpuser1/named/domain1.info.db


--------------------------------------------------------
list cpuser1 cronjobs
--------------------------------------------------------

#SHELL="/usr/local/cpanel/bin/jailshell"
#*/15 * * * * echo "dummy cronjob"

--------------------------------------------------------
cpuser1 cronjobs saved:
/home/backup-accounts/cpuser1/cronjobs-cpuser1-040719-094323.txt
--------------------------------------------------------

--------------------------------------------------------
backup cpanel /home/cpuser1/public_html web root
--------------------------------------------------------

/bin/nice -n 12 /bin/ionice -c2 -n7 tar cpf - public_html | zstd -6 -T2 -f --rsyncable > "/home/backup-accounts/cpuser1/public_html-cpuser1-040719-094323.tar.zst"

backup start time: Thu Jul  4 09:43:23 UTC 2019
backup end time: Thu Jul  4 09:43:25 UTC 2019
backup time: 1.745
zstd normal compression mode: enabled
compression level: 6
compression ratio: 2.193 (22470527 / 10246463)
compression ratio: 0.455 (10246463 / 22470527)
slack notification sending
ok

--------------------------------------------------------
backup cpanel /home/cpuser1/mail directory

/bin/nice -n 12 /bin/ionice -c2 -n7 tar cpf - mail | zstd -6 -T2 -f --rsyncable > "/home/backup-accounts/cpuser1/mail-cpuser1-040719-094323.tar.zst"

backup start time: Thu Jul  4 09:43:25 UTC 2019
backup end time: Thu Jul  4 09:43:25 UTC 2019
backup time: 0.008
zstd normal compression mode: enabled
compression level: 6
compression ratio: 81.681 (24586 / 301)
compression ratio: 0.012 (301 / 24586)
slack notification sending
ok

--------------------------------------------------------
backup cpanel /home/cpuser1/logs directory

/bin/nice -n 12 /bin/ionice -c2 -n7 tar cpf - logs | zstd -1 -T2 -f --rsyncable > "/home/backup-accounts/cpuser1/logs-cpuser1-040719-094323.tar.zst"

backup start time: Thu Jul  4 09:43:26 UTC 2019
backup end time: Thu Jul  4 09:43:40 UTC 2019
backup time: 13.998
zstd normal compression mode: enabled
compression level: 1
compression ratio: 12.733 (1049492951 / 82422177)
compression ratio: 0.078 (82422177 / 1049492951)
slack notification sending
ok

--------------------------------------------------------
backup cpanel /home/cpuser1/ssl directory

/bin/nice -n 12 /bin/ionice -c2 -n7 tar cpf - ssl | zstd -6 -T2 -f --rsyncable > "/home/backup-accounts/cpuser1/ssl-cpuser1-040719-094323.tar.zst"

backup start time: Thu Jul  4 09:43:40 UTC 2019
backup end time: Thu Jul  4 09:43:40 UTC 2019
backup time: 0.007
zstd normal compression mode: enabled
compression level: 6
compression ratio: 46.545 (4096 / 88)
compression ratio: 0.021 (88 / 4096)
slack notification sending
ok

--------------------------------------------------------
backup cpanel /home/cpuser1/repositories directory

/bin/nice -n 12 /bin/ionice -c2 -n7 tar cpf - repositories | zstd -6 -T2 -f --rsyncable > "/home/backup-accounts/cpuser1/repositories-cpuser1-040719-094323.tar.zst"

backup start time: Thu Jul  4 09:43:41 UTC 2019
backup end time: Thu Jul  4 09:43:41 UTC 2019
backup time: 0.066
zstd normal compression mode: enabled
compression level: 6
compression ratio: 1.623 (932311 / 574381)
compression ratio: 0.616 (574381 / 932311)
slack notification sending
ok

--------------------------------------------------------
list cpanel user: cpuser1 mysql databases
--------------------------------------------------------

cpuser1_db1
cpuser1_db2
cpuser1_db3

--------------------------------------------------------
list cpanel user: cpuser1 mysql usernames & grants
--------------------------------------------------------

mysql username: cpuser1_dbuser

Grants for cpuser1_dbuser@localhost
GRANT USAGE ON *.* TO 'cpuser1_dbuser'@'localhost' IDENTIFIED BY PASSWORD '*1F315FCA7B4226EFD886026773262F1429508DF3'
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, TRIGGER ON `cpuser1_db1`.* TO 'cpuser1_dbuser'@'localhost'
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, TRIGGER ON `cpuser1_db2`.* TO 'cpuser1_dbuser'@'localhost'
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, TRIGGER ON `cpuser1_db3`.* TO 'cpuser1_dbuser'@'localhost'

replay commands via SSH command line

mysql -e "GRANT USAGE ON *.* TO 'cpuser1_dbuser'@'localhost' IDENTIFIED BY PASSWORD '*1F315FCA7B4226EFD886026773262F1429508DF3'" mysql
mysql -e "GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, TRIGGER ON cpuser1_db1.* TO 'cpuser1_dbuser'@'localhost'" mysql
mysql -e "GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, TRIGGER ON cpuser1_db2.* TO 'cpuser1_dbuser'@'localhost'" mysql
mysql -e "GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, TRIGGER ON cpuser1_db3.* TO 'cpuser1_dbuser'@'localhost'" mysql

backup cpanel user's mysql databases

/bin/nice -n 12 /bin/ionice -c2 -n7 mysqldump --default-character-set=utf8 -Q -K --max_allowed_packet=256M --net_buffer_length=65536 --routines --events --triggers --hex-blob "cpuser1_db1" | zstd -3 -T2 -f --rsyncable > "/home/backup-accounts/cpuser1/mysqlbackup-cpuser1_db1-040719-094323.sql.zst"

backup start time: Thu Jul  4 09:43:41 UTC 2019
backup end time: Thu Jul  4 09:44:23 UTC 2019
backup time: 42.023
zstd normal compression mode: enabled
compression level: 3
compression ratio: 4.309 (1232429907 / 285958928)
compression ratio: 0.232 (285958928 / 1232429907)
slack notification sending
ok

/bin/nice -n 12 /bin/ionice -c2 -n7 mysqldump --default-character-set=utf8 --single-transaction -Q -K --max_allowed_packet=256M --net_buffer_length=65536 --routines --events --triggers --hex-blob "cpuser1_db2" | zstd -3 -T2 -f --rsyncable > "/home/backup-accounts/cpuser1/mysqlbackup-cpuser1_db2-040719-094323.sql.zst"

backup start time: Thu Jul  4 09:44:24 UTC 2019
backup end time: Thu Jul  4 09:44:24 UTC 2019
backup time: 0.014
zstd normal compression mode: enabled
compression level: 3
compression ratio: 7.873 (4157 / 528)
compression ratio: 0.127 (528 / 4157)
slack notification sending
ok

/bin/nice -n 12 /bin/ionice -c2 -n7 mysqldump --default-character-set=utf8 --single-transaction -Q -K --max_allowed_packet=256M --net_buffer_length=65536 --routines --events --triggers --hex-blob "cpuser1_db3" | zstd -3 -T2 -f --rsyncable > "/home/backup-accounts/cpuser1/mysqlbackup-cpuser1_db3-040719-094323.sql.zst"

backup start time: Thu Jul  4 09:44:25 UTC 2019
backup end time: Thu Jul  4 09:44:25 UTC 2019
backup time: 0.013
zstd normal compression mode: enabled
compression level: 3
compression ratio: 7.888 (4157 / 527)
compression ratio: 0.126 (527 / 4157)
slack notification sending
ok

--------------------------------------------------------
List cpuser1 backups at /home/backup-accounts/cpuser1
--------------------------------------------------------

+-- [  76]  cronjobs-cpuser1-040719-094323.txt
+-- [ 190]  domain-map-cpuser1-040719-094323.json
+-- [ 533]  domain-map-cpuser1-040719-094323.txt
+-- [ 79M]  logs-cpuser1-040719-094323.tar.zst
+-- [ 301]  mail-cpuser1-040719-094323.tar.zst
+-- [273M]  mysqlbackup-cpuser1_db1-040719-094323.sql.zst
+-- [ 528]  mysqlbackup-cpuser1_db2-040719-094323.sql.zst
+-- [ 527]  mysqlbackup-cpuser1_db3-040719-094323.sql.zst
+-- [ 835]  mysql-grants-040719-094323.log
+-- [ 861]  mysql-grants-ssh-040719-094323.log
+-- [4.0K]  named
|   +-- [   0]  ads.domain1.com.db
|   +-- [   0]  domain1.biz.db
|   +-- [   0]  domain1.com.db
|   +-- [   0]  domain1.info.db
|   +-- [   0]  m.domain1.com.db
+-- [9.8M]  public_html-cpuser1-040719-094323.tar.zst
+-- [561K]  repositories-cpuser1-040719-094323.tar.zst
+-- [  88]  ssl-cpuser1-040719-094323.tar.zst


--------------------------------------------------------
pidstat stats saved at:
/home/backup-accounts/logs/backup_pidstat_stats_040719-094323.log.zst
--------------------------------------------------------
read command:
zstdcat /home/backup-accounts/logs/backup_pidstat_stats_040719-094323.log.zst
--------------------------------------------------------
sar usage stats saved at:
/home/backup-accounts/logs/backup_sar_stats_040719-094323
--------------------------------------------------------
read via sar command:

cpu load avg

sar -q -f /home/backup-accounts/logs/backup_sar_stats_040719-094323 | sed -e "s|$(hostname)|hostname|g"

09:43:23 AM   runq-sz  plist-sz   ldavg-1   ldavg-5  ldavg-15   blocked
Average:            3       147      1.04      0.28      0.16         0

1min 5min 15min min:
0.00 0.01 0.07
1min 5min 15min avg:
1.04 0.28 0.16
1min 5min 15min max:
1.96 0.56 0.25
1min 5min 15min 95%:
1.94 0.55 0.25

cpu utilisation

sar -u -f /home/backup-accounts/logs/backup_sar_stats_040719-094323 | sed -e "s|$(hostname)|hostname|g"

09:43:23 AM     CPU     %user     %nice   %system   %iowait    %steal     %idle
Average:        all     72.65      7.34      3.78      3.81      0.24     12.18

%user %nice %system $iowait %steal %idle min:
0.50 0.00 1.01 0.00 0.00 0.00
%user %nice %system $iowait %steal %idle avg:
72.66 7.33 3.79 3.83 0.24 12.15
%user %nice %system $iowait %steal %idle max:
90.36 16.92 8.00 51.79 1.55 98.49
%user %nice %system $iowait %steal %idle 95%:
89.85 15.66 6.31 29.02 0.52 64.84

memory usage

sar -r -f /home/backup-accounts/logs/backup_sar_stats_040719-094323 | sed -e "s|$(hostname)|hostname|g"

09:43:23 AM kbmemfree kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit  kbactive   kbinact   kbdirty
Average:       103236    911500     89.83     64251    441303   1372402     66.51    246046    527449     21496

kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit kbactive kbinact kbdirty min:
70180.00 855908.00 84.35 64168.00 376364.00 1279200.00 62.00 180744.00 465176.00 760.00
kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit kbactive kbinact kbdirty avg:
103236.44 911499.56 89.83 64251.05 441303.30 1372402.35 66.51 246046.29 527449.33 21496.44
kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit kbactive kbinact kbdirty max:
158828.00 944556.00 93.08 64776.00 518116.00 1435092.00 69.55 302512.00 612688.00 37280.00
kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit kbactive kbinact kbdirty 95%:
142049.60 937571.20 92.39 64768.00 511324.80 1435092.00 69.55 302402.80 602478.40 35894.00

disk I/O usage

sar -d -f /home/backup-accounts/logs/backup_sar_stats_040719-094323 | sed -e "s|$(hostname)|hostname|g"

09:43:23 AM       DEV       tps  rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz     await     svctm     %util
Average:     dev253-0    176.77  61579.22  10937.98    410.24      0.30      1.74      0.28      5.00

tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util min:
7.00 7.00 520.00 0.00 11.81 0.00 0.00 0.00
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util avg:
176.85 176.85 61571.99 10930.42 725.01 0.30 3.06 0.45
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util max:
2624.00 2624.00 180435.64 76546.53 945.40 1.16 7.69 1.20
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util 95%:
351.60 351.60 163840.00 67071.20 914.42 0.84 5.57 0.74

--------------------------------------------------------
/root/tools/cpanel-backup.sh run log saved at:
/home/backup-accounts/logs/cpanel-backup-040719-094323.log.zst
--------------------------------------------------------
read command:
zstdcat /home/backup-accounts/logs/cpanel-backup-040719-094323.log.zst
--------------------------------------------------------
backup completed
--------------------------------------------------------
total cpanel backup time: 64.361 seconds

slack notification sending
ok
--------------------------------------------------------
```

# sar & pidstat logging

Inspecting pidstat zstd compressed logs

At public_html backup stage

```
zstdcat /home/backup-accounts/logs/backup_pidstat_stats_020719-233152.log.zst
Linux 3.10.0-957.21.3.el7.x86_64 (cpanelhost.domain.com)      07/02/2019      _x86_64_        (2 CPU)

#      Time   UID       PID    %usr %system  %guest    %CPU   CPU  minflt/s  majflt/s     VSZ    RSS   %MEM   kB_rd/s   kB_wr/s kB_ccwr/s  Command
 1562110313     0         9    0.00    0.97    0.00    0.97     0      0.00      0.00       0      0   0.00      0.00      0.00      0.00  rcu_sched
 1562110313     0     21125    0.00    0.00    0.00    0.00     1     42.72      0.00  113704   1540   0.15      0.00      0.00      0.00  /bin/bash /root/tools/cpanel-backup.sh cpuser1 
 1562110313     0     21127    0.00    0.00    0.00    0.00     0     10.68      0.00  108036    784   0.08      0.00      0.00      0.00  sar -o /home/backup-accounts/logs/backup_sar_stats_020719-233152 1 
 1562110313     0     21128    0.00    0.97    0.00    0.97     0    318.45      0.00  108184   1052   0.10      0.00      0.00      0.00  pidstat -durhl 1 
 1562110313     0     21132    0.00    0.00    0.00    0.00     0     36.89      0.00  113244    996   0.10      0.00     11.65      0.00  sadc 1 -z -S ALL /home/backup-accounts/logs/backup_sar_stats_020719-233152 
 1562110313     0     21143    0.97   13.59    0.00   14.56     0    830.10      0.00  123532   1312   0.13  23840.78      0.00      0.00  tar cpf - public_html 
 1562110313     0     21144  100.00    0.97    0.00  100.00     0   8762.14      0.00  234432  35032   3.45      0.00   5231.07      0.00  zstd -6 -T2 -f --rsyncable 

```

At start of mysql database backup stage via mysqldump

```
#      Time   UID       PID    %usr %system  %guest    %CPU   CPU  minflt/s  majflt/s     VSZ    RSS   %MEM   kB_rd/s   kB_wr/s kB_ccwr/s  Command
 1562110331     0      1264    0.00    1.00    0.00    1.00     0      0.00      0.00       0      0   0.00      0.00  19368.00      0.00  jbd2/vda1-8
 1562110331     0      1707    0.00    0.00    0.00    0.00     0      0.00      0.00   55572    180   0.02      0.00      4.00      0.00  /sbin/auditd 
 1562110331     0      2201    0.00    0.00    0.00    0.00     1      2.00      0.00   21584    408   0.04      0.00      0.00      0.00  /usr/sbin/irqbalance --foreground 
 1562110331   997      4592   25.00    4.00    0.00   29.00     0   1019.00      7.00  517624  60604   5.97  37008.00   1784.00   1784.00  /usr/sbin/mysqld 
 1562110331     0     21125    0.00    0.00    0.00    0.00     0    393.00      0.00  113704   1600   0.16   4996.00      8.00      0.00  /bin/bash /root/tools/cpanel-backup.sh cpuser1 
 1562110331     0     21126    0.00    0.00    0.00    0.00     0      0.00      0.00  107952    672   0.07      0.00      4.00      0.00  tee /home/backup-accounts/logs/cpanel-backup-020719-233152.log 
 1562110331     0     21128    0.00    1.00    0.00    1.00     1    330.00      0.00  108184   1128   0.11      0.00      0.00      0.00  pidstat -durhl 1 
 1562110331     0     21132    0.00    0.00    0.00    0.00     0     38.00      0.00  113244    996   0.10      0.00      8.00      0.00  sadc 1 -z -S ALL /home/backup-accounts/logs/backup_sar_stats_020719-233152 
 1562110331     0     21396   24.00    2.00    0.00   26.00     0   1824.00      1.00   54284   4924   0.49   3980.00      0.00      0.00  mysqldump --default-character-set=utf8 -Q -K --max_allowed_packet=256M --net_buffer_length=65536 --routines --events --triggers
 1562110331     0     21397   90.00    3.00    0.00   93.00     0   9848.00      0.00  211320  38196   3.76      0.00   4320.00      0.00  zstd -3 -T2 -f --rsyncable 
```

Inspecting sar recorded resource usage

cpu load avearages

```
sar -q -f /home/backup-accounts/logs/backup_sar_stats_020719-233152 | sed -e "s|$(hostname)|hostname|g"
Linux 3.10.0-957.21.3.el7.x86_64 (hostname)     07/02/2019      _x86_64_        (2 CPU)

11:31:52 PM   runq-sz  plist-sz   ldavg-1   ldavg-5  ldavg-15   blocked
11:31:53 PM         3       147      0.00      0.06      0.16         1
11:31:54 PM         1       144      0.00      0.06      0.16         0
11:31:55 PM         1       144      0.00      0.06      0.16         0
11:31:56 PM         3       147      0.00      0.06      0.16         0
11:31:57 PM         3       147      0.00      0.06      0.16         0
11:31:58 PM         3       147      0.24      0.11      0.17         0
11:31:59 PM         0       147      0.24      0.11      0.17         2
11:32:00 PM         3       147      0.24      0.11      0.17         0
11:32:01 PM         3       147      0.24      0.11      0.17         0
11:32:02 PM         3       147      0.24      0.11      0.17         0
11:32:03 PM         0       147      0.38      0.14      0.18         2
11:32:04 PM         1       147      0.38      0.14      0.18         2
11:32:05 PM         3       148      0.38      0.14      0.18         1
11:32:06 PM         3       147      0.38      0.14      0.18         0
11:32:07 PM         3       147      0.38      0.14      0.18         0
11:32:08 PM         3       147      0.51      0.17      0.19         0
11:32:09 PM         1       144      0.51      0.17      0.19         0
11:32:10 PM         2       146      0.51      0.17      0.19         0
11:32:11 PM         2       147      0.51      0.17      0.19         1
11:32:12 PM         3       147      0.51      0.17      0.19         0
11:32:13 PM         3       147      0.87      0.25      0.22         1
11:32:14 PM         6       149      0.87      0.25      0.22         1
11:32:15 PM         4       148      0.87      0.25      0.22         2
11:32:16 PM         1       150      0.87      0.25      0.22         3
11:32:17 PM         3       149      0.87      0.25      0.22         0
11:32:18 PM         5       149      1.12      0.31      0.24         0
11:32:19 PM         3       149      1.12      0.31      0.24         0
11:32:20 PM         3       149      1.12      0.31      0.24         0
11:32:21 PM         3       149      1.12      0.31      0.24         0
11:32:22 PM         3       149      1.12      0.31      0.24         0
11:32:23 PM         3       149      1.35      0.38      0.26         0
11:32:24 PM         2       149      1.35      0.38      0.26         0
11:32:25 PM         4       149      1.35      0.38      0.26         0
11:32:26 PM         4       147      1.35      0.38      0.26         0
11:32:27 PM         2       147      1.35      0.38      0.26         0
11:32:28 PM         3       147      1.49      0.42      0.27         0
11:32:29 PM         3       147      1.49      0.42      0.27         0
11:32:30 PM         3       147      1.49      0.42      0.27         0
11:32:31 PM         3       147      1.49      0.42      0.27         0
11:32:32 PM         3       147      1.49      0.42      0.27         0
11:32:33 PM         2       147      1.61      0.46      0.29         0
11:32:34 PM         3       149      1.61      0.46      0.29         0
11:32:35 PM         4       149      1.61      0.46      0.29         0
11:32:36 PM         1       149      1.61      0.46      0.29         0
11:32:37 PM         4       149      1.61      0.46      0.29         0
11:32:38 PM         4       149      1.64      0.49      0.30         0
11:32:39 PM         3       149      1.64      0.49      0.30         0
11:32:40 PM         2       149      1.64      0.49      0.30         0
11:32:41 PM         2       149      1.64      0.49      0.30         0
11:32:42 PM         3       149      1.64      0.49      0.30         0
11:32:43 PM         3       149      1.75      0.53      0.31         0
11:32:44 PM         3       149      1.75      0.53      0.31         0
11:32:45 PM         2       147      1.75      0.53      0.31         0
11:32:46 PM         4       147      1.75      0.53      0.31         0
11:32:47 PM         1       146      1.75      0.53      0.31         0
11:32:48 PM         0       144      1.61      0.52      0.31         0
11:32:49 PM         0       144      1.61      0.52      0.31         0
11:32:50 PM         0       144      1.61      0.52      0.31         0
Average:            3       147      1.03      0.31      0.24         0
```

sar cpu load average extended stats for min, avg, max and 95th percentile values

```
cat /home/backup-accounts/logs/sar-cpuload-data-formatted-020719-233152.log                            
1min 5min 15min min:
0.00 0.06 0.16
1min 5min 15min avg:
1.03 0.31 0.24
1min 5min 15min max:
1.75 0.53 0.31
1min 5min 15min 95%:
1.75 0.53 0.31
```

sar disk I/O usage extended stats for min, avg, max and 95% percentile values

```
cat /home/backup-accounts/logs/sar-disk-data-formatted-030719-002643.log
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util min:
0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util avg:
166.84 166.84 132239.08 9618.89 790.93 0.66 3.63 0.56
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util max:
279.00 279.00 194312.87 57968.00 914.07 2.34 11.32 1.06
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util 95%:
231.06 231.06 181390.35 55487.20 910.22 1.20 5.81 0.88
```