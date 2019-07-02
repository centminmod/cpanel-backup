# cpanel-backup.sh

`cpanel-backup.sh` is a cPanel/WHM backup script I was hired to write for a client to allow data migration from cPanel/WHM to [Centmin Mod LEMP stack](https://centminmod.com/) as a result of the [cPanel announced price increase and licensing changes](https://community.centminmod.com/threads/17849/). This is the backup part with an accompanying import part to come and backup data transfer part as well - when combined will make up the data migrator for importing into Centmin Mod LEMP stack based servers. Centmin Mod LEMP stack isn't for shared hosting, so isn't always an alternative to cPanel. However, Centmin Mod LEMP stack is best suited for single site owner managed own sites where there's trust.

# features

* `cpanel-backup.sh` will have features from [dbbackup.sh](https://community.centminmod.com/threads/dbbackup-sh-quick-mysql-database-backups-for-centmin-mod-stack.4573/) including conditional db character set detection & 
  pure innodb database detection for conditional single-transaction options and using nice/ionice for tar backups. 
* `cpanel-backup.sh` will also have sar & pidstat cpu, memory and disk usage stats recorded allowing you to fine tune your backup and compression parameters for your specific server and backup requirements.
* pidstat logs and backup logs also also zstd level 1 compressed to save space.
* `cpanel-backup.sh` will support the following compression algorithms, gzip via multi-threaded pigz, xz via multi-threaded pxz and zstd ([zstd is the default due to speed and compression ration performance](https://community.centminmod.com/threads/round-3-compression-comparison-benchmarks-zstd-vs-brotli-vs-pigz-vs-bzip2-vs-xz-etc.17259/))
* `cpanel-backup.sh` compression algorithm level settings have finer granular control on a per backup target basis, so public_html, mail, logs, ssl or database backup targets each have their own compression level control so you can optimise compression speed and compressed file size as well as control the amount of server resources used (cpu, memory etc)
* zstd compression is the default compression used with tar backups and has both normal and a low memory mode to reduce memory used for compression. 
* For cPanel log files if you opt to back them up, `cpanel-backup.sh` will conditionally reduce zstd compression level to lowest negative 10 (--fast=10) levels to not waste time if the script detects there are already a mix of compressed & uncompressed version of your logs (due to logrotate). FYI, zstd has compression levels from fastest to slowest (smallest compressed file size) from -10 to 19 and then 3 ultra levels 20-22.
* `cpanel-backup.sh` will support backing up all cpanel user accounts in same session as well as per cpanel user account backups on command line.
* `cpanel-backup.sh` will support alias name masking for domain name, username and database name to allow you to publicly demo Slack notifications on live cPanel user data but still keep actual domain name, username, and database names private.
* `cpanel-backup.sh` will also optionally sending slack channel notifications on successful or failed backup targets i.e. public_html, mail, logs, ssl or database backups. All notifications are colour coded - green = successful or red = failed for backup status for each backup target. This allows quick visual inspection of which backup targets failed their backup runs.

# slack channel notifictaions

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

# raw output example

`cpanel-backup.sh` preview for single cpanel user backup mode to back cpanel user = `cpuser1` with alias domain name, cPanel username and database name masking enabled only for Slack channel notifictaions (script outputs the real names) and default zstd compression algorithm used and zstd compressed logs.

```
/root/tools/cpanel-backup.sh cpuser1

--------------------------------------------------------
cPanel/WHM backup script 0.5
for data migration to Centmin Mod LEMP stack imports
written by George Liu (centminmod.com)
--------------------------------------------------------

--------------------------------------------------------
list cpanel users
--------------------------------------------------------

cpuser1

--------------------------------------------------------
backup cpanel /home/cpuser1/public_html web root
--------------------------------------------------------

/bin/nice -n 12 /bin/ionice -c2 -n7 tar cpf - public_html | zstd -6 -T2 -f --rsyncable > "/home/backup-accounts/cpuser1/public_html-cpuser1-020719-233152.tar.zst"

backup start time: Tue Jul  2 23:31:52 UTC 2019
backup end time: Tue Jul  2 23:31:54 UTC 2019
backup time: 1.611
zstd normal compression mode: enabled
compression level: 6
compression ratio: 2.193 (22470527 / 10246463)
compression ratio: 0.455 (10246463 / 22470527)
slack notification sending
ok

--------------------------------------------------------
backup cpanel /home/cpuser1/mail directory

/bin/nice -n 12 /bin/ionice -c2 -n7 tar cpf - mail | zstd -6 -T2 -f --rsyncable > "/home/backup-accounts/cpuser1/mail-cpuser1-020719-233152.tar.zst"

backup start time: Tue Jul  2 23:31:55 UTC 2019
backup end time: Tue Jul  2 23:31:55 UTC 2019
backup time: 0.011
zstd normal compression mode: enabled
compression level: 6
compression ratio: 81.681 (24586 / 301)
compression ratio: 0.012 (301 / 24586)
slack notification sending
ok

--------------------------------------------------------
backup cpanel /home/cpuser1/logs directory

/bin/nice -n 12 /bin/ionice -c2 -n7 tar cpf - logs | zstd -1 -T2 -f --rsyncable > "/home/backup-accounts/cpuser1/logs-cpuser1-020719-233152.tar.zst"

backup start time: Tue Jul  2 23:31:55 UTC 2019
backup end time: Tue Jul  2 23:32:09 UTC 2019
backup time: 13.127
zstd normal compression mode: enabled
compression level: 1
compression ratio: 12.733 (1049492951 / 82422177)
compression ratio: 0.078 (82422177 / 1049492951)
slack notification sending
ok

--------------------------------------------------------
backup cpanel /home/cpuser1/ssl directory

/bin/nice -n 12 /bin/ionice -c2 -n7 tar cpf - ssl | zstd -6 -T2 -f --rsyncable > "/home/backup-accounts/cpuser1/ssl-cpuser1-020719-233152.tar.zst"

backup start time: Tue Jul  2 23:32:09 UTC 2019
backup end time: Tue Jul  2 23:32:09 UTC 2019
backup time: 0.010
zstd normal compression mode: enabled
compression level: 6
compression ratio: 46.545 (4096 / 88)
compression ratio: 0.021 (88 / 4096)
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

/bin/nice -n 12 /bin/ionice -c2 -n7 mysqldump --default-character-set=utf8 -Q -K --max_allowed_packet=256M --net_buffer_length=65536 --routines --events --triggers --hex-blob "cpuser1_db1" | zstd -3 -T2 -f --rsyncable > "/home/backup-accounts/cpuser1/mysqlbackup-cpuser1_db1-020719-233152.sql.zst"

backup start time: Tue Jul  2 23:32:10 UTC 2019
backup end time: Tue Jul  2 23:32:47 UTC 2019
backup time: 37.158
zstd normal compression mode: enabled
compression level: -3
compression ratio: 4.309 (1232429907 / 285958928)
compression ratio: 0.232 (285958928 / 1232429907)
slack notification sending
ok

/bin/nice -n 12 /bin/ionice -c2 -n7 mysqldump --default-character-set=utf8 --single-transaction -Q -K --max_allowed_packet=256M --net_buffer_length=65536 --routines --events --triggers --hex-blob "cpuser1_db2" | zstd -3 -T2 -f --rsyncable > "/home/backup-accounts/cpuser1/mysqlbackup-cpuser1_db2-020719-233152.sql.zst"

backup start time: Tue Jul  2 23:32:48 UTC 2019
backup end time: Tue Jul  2 23:32:48 UTC 2019
backup time: 0.014
zstd normal compression mode: enabled
compression level: -3
compression ratio: 7.873 (4157 / 528)
compression ratio: 0.127 (528 / 4157)
slack notification sending
ok

/bin/nice -n 12 /bin/ionice -c2 -n7 mysqldump --default-character-set=utf8 --single-transaction -Q -K --max_allowed_packet=256M --net_buffer_length=65536 --routines --events --triggers --hex-blob "cpuser1_db3" | zstd -3 -T2 -f --rsyncable > "/home/backup-accounts/cpuser1/mysqlbackup-cpuser1_db3-020719-233152.sql.zst"

backup start time: Tue Jul  2 23:32:49 UTC 2019
backup end time: Tue Jul  2 23:32:49 UTC 2019
backup time: 0.012
zstd normal compression mode: enabled
compression level: -3
compression ratio: 7.858 (4157 / 529)
compression ratio: 0.127 (529 / 4157)
slack notification sending
ok

--------------------------------------------------------
List cpuser1 backups at /home/backup-accounts/cpuser1
--------------------------------------------------------

-rw-r--r--. 1 root root 9.8M Jul  2 23:31 public_html-cpuser1-020719-233152.tar.zst
-rw-r--r--. 1 root root  301 Jul  2 23:31 mail-cpuser1-020719-233152.tar.zst
-rw-r--r--. 1 root root  79M Jul  2 23:32 logs-cpuser1-020719-233152.tar.zst
-rw-r--r--. 1 root root   88 Jul  2 23:32 ssl-cpuser1-020719-233152.tar.zst
-rw-r--r--. 1 root root  835 Jul  2 23:32 mysql-grants-020719-233152.log
-rw-r--r--. 1 root root  861 Jul  2 23:32 mysql-grants-ssh-020719-233152.log
-rw-r--r--. 1 root root 273M Jul  2 23:32 mysqlbackup-cpuser1_db1-020719-233152.sql.zst
-rw-r--r--. 1 root root  528 Jul  2 23:32 mysqlbackup-cpuser1_db2-020719-233152.sql.zst
-rw-r--r--. 1 root root  529 Jul  2 23:32 mysqlbackup-cpuser1_db3-020719-233152.sql.zst


--------------------------------------------------------
pidstat stats saved at:
/home/backup-accounts/logs/backup_pidstat_stats_020719-233152.log.zst
--------------------------------------------------------
read command:
zstdcat /home/backup-accounts/logs/backup_pidstat_stats_020719-233152.log.zst
--------------------------------------------------------
sar usage stats saved at:
/home/backup-accounts/logs/backup_sar_stats_020719-233152
--------------------------------------------------------
read via sar command:

cpu load avg

sar -q -f /home/backup-accounts/logs/backup_sar_stats_020719-233152 | sed -e "s|$(hostname)|hostname|g"

11:31:52 PM   runq-sz  plist-sz   ldavg-1   ldavg-5  ldavg-15   blocked
Average:            3       147      1.03      0.31      0.24         0

1min 5min 15min min:
0.00 0.06 0.16
1min 5min 15min avg:
1.03 0.31 0.24
1min 5min 15min max:
1.75 0.53 0.31
1min 5min 15min 95%:
1.75 0.53 0.31

cpu utilisation

sar -u -f /home/backup-accounts/logs/backup_sar_stats_020719-233152 | sed -e "s|$(hostname)|hostname|g"

11:31:52 PM     CPU     %user     %nice   %system   %iowait    %steal     %idle
Average:        all     68.77      7.07      3.61      4.29      0.07     16.19

%user %nice %system $iowait %steal %idle min:
0.50 0.00 0.50 0.00 0.00 0.00
%user %nice %system $iowait %steal %idle avg:
68.80 7.07 3.62 4.26 0.07 16.18
%user %nice %system $iowait %steal %idle max:
91.50 16.58 8.33 40.40 0.53 99.00
%user %nice %system $iowait %steal %idle 95%:
88.51 15.81 6.61 28.31 0.51 92.13

memory usage

sar -r -f /home/backup-accounts/logs/backup_sar_stats_020719-233152 | sed -e "s|$(hostname)|hostname|g"

11:31:52 PM kbmemfree kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit  kbactive   kbinact   kbdirty
Average:       101801    912935     89.97    130308    377143   1367578     66.28    302071    472112     20589

kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit kbactive kbinact kbdirty min:
69592.00 874056.00 86.14 130232.00 314356.00 1278800.00 61.98 235400.00 411972.00 592.00
kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit kbactive kbinact kbdirty avg:
101800.83 912935.17 89.97 130307.66 377143.10 1367578.00 66.28 302071.10 472112.14 20589.17
kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit kbactive kbinact kbdirty max:
140680.00 945144.00 93.14 130720.00 452252.00 1428484.00 69.23 358248.00 553476.00 36432.00
kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit kbactive kbinact kbdirty 95%:
131916.00 936719.00 92.31 130708.00 437348.20 1426468.00 69.14 358094.40 537221.40 33403.60

disk I/O usage

sar -d -f /home/backup-accounts/logs/backup_sar_stats_020719-233152 | sed -e "s|$(hostname)|hostname|g"

11:31:52 PM       DEV       tps  rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz     await     svctm     %util
Average:     dev253-0    202.15  67822.85  12029.73    395.02      0.34      1.72      0.29      5.93

tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util min:
1.00 1.00 0.00 0.00 8.00 0.00 0.00 0.00
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util avg:
202.55 202.55 67877.05 12043.87 668.44 0.34 3.24 0.54
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util max:
2844.00 2844.00 194669.31 81768.00 940.42 1.43 32.53 4.56
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util 95%:
566.65 566.65 180224.00 69632.00 910.22 0.86 4.94 1.16

--------------------------------------------------------
/root/tools/cpanel-backup.sh run log saved at:
/home/backup-accounts/logs/cpanel-backup-020719-233152.log.zst
--------------------------------------------------------
read command:
zstdcat /home/backup-accounts/logs/cpanel-backup-020719-233152.log.zst
--------------------------------------------------------
backup completed
--------------------------------------------------------
total cpanel backup time: 59.582 seconds

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