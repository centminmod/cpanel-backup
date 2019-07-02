# cpanel-backup.sh

`cpanel-backup.sh` is a cPanel/WHM backup script I was hired to write for a client to allow data migration from cPanel/WHM to [Centmin Mod LEMP stack](https://centminmod.com/) as a result of the [cPanel announced price increase and licensing changes](https://community.centminmod.com/threads/17849/). This is the backup part with an accompanying import part to come and backup data transfer part as well - when combined will make up the data migrator for importing into Centmin Mod LEMP stack based servers. Centmin Mod LEMP stack isn't for shared hosting, so isn't always an alternative to cPanel. However, Centmin Mod LEMP stack is best suited for single site owner managed own sites where there's trust.

# features

* `cpanel-backup.sh` will have features from [dbbackup.sh](https://community.centminmod.com/threads/dbbackup-sh-quick-mysql-database-backups-for-centmin-mod-stack.4573/) including conditional db character set detection & 
  pure innodb database detection for conditional single-transaction options and using nice/ionice for tar backups. 
* `cpanel-backup.sh` will also have sar & pidstat cpu, memory and disk usage stats recorded allowing you to fine tune your backup and compression parameters for your specific server and backup requirements.
* `cpanel-backup.sh` will support the following compression algorithms, gzip via multi-threaded pigz, xz via multi-threaded pxz and zstd ([zstd is the default due to speed and compression ration performance](https://community.centminmod.com/threads/round-3-compression-comparison-benchmarks-zstd-vs-brotli-vs-pigz-vs-bzip2-vs-xz-etc.17259/))
* `cpanel-backup.sh` compression algorithm level settings have finer granular control on a per backup target basis, so public_html, mail, logs, ssl or database backup targets each have their own compression level control so you can optimise compression speed and compressed file size as well as control the amount of server resources used (cpu, memory etc)
* zstd compression is the default compression used with tar backups and has both normal and a low memory mode to reduce memory used for compression. 
* For cPanel log files if you opt to back them up, `cpanel-backup.sh` will conditionally reduce zstd compression level to lowest negative 10 (--fast=10) levels to not waste time if the script detects there are already a mix of compressed & uncompressed version of your logs (due to logrotate). FYI, zstd has compression levels from fastest to slowest (smallest compressed file size) from -10 to 19 and then 3 ultra levels 20-22.
* `cpanel-backup.sh` will support backing up all cpanel user accounts in same session as well as per cpanel user account backups on command line.
* `cpanel-backup.sh` will also optionally sending slack channel notifications on successful or failed backup targets i.e. public_html, mail, logs, ssl or database backups. All notifications are colour coded - green = successful or red = failed for backup status for each backup target

# slack channel notifictaions

cpanel user = cpuser1 public_html backup

![](/screenshots/cpanel-backup.sh-slack-cpuser1-publichtml-01.png)

mysql database backup using zstd level 3 compression

![](/screenshots/cpanel-backup.sh-slack-cpuser1-mysql-01.png)

slack channel searching backup notification logs

![](/screenshots/cpanel-backup.sh-slack-cpuser1-search-01.png)

# raw output example

`cpanel-backup.sh` preview for single cpanel user backup mode to back cpanel user = `cpuser1`

```
/root/tools/cpanel-backup.sh cpuser1                                      

--------------------------------------------------------
cPanel/WHM backup script 0.2
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

/bin/nice -n 12 /bin/ionice -c2 -n7 tar cpf - public_html | zstd -6 -T2 -f --rsyncable > "/home/backup-accounts/cpuser1/public_html-cpuser1-010719-184816.tar.zst"

backup start time: Mon Jul  1 18:48:16 UTC 2019
backup end time: Mon Jul  1 18:48:17 UTC 2019
backup time: 1.052
compression level: -6
compression ratio: 2.071 (22470527 / 10849889)
compression ratio: 0.482 (10849889 / 22470527)
slack notification sending
ok

--------------------------------------------------------
backup cpanel /home/cpuser1/mail directory

/bin/nice -n 12 /bin/ionice -c2 -n7 tar cpf - mail | zstd -6 -T2 -f --rsyncable > "/home/backup-accounts/cpuser1/mail-cpuser1-010719-184816.tar.zst"

backup start time: Mon Jul  1 18:48:18 UTC 2019
backup end time: Mon Jul  1 18:48:18 UTC 2019
backup time: 0.006
compression level: -6
compression ratio: 81.681 (24586 / 301)
compression ratio: 0.012 (301 / 24586)
slack notification sending
ok

--------------------------------------------------------
backup cpanel /home/cpuser1/logs directory

/bin/nice -n 12 /bin/ionice -c2 -n7 tar cpf - logs | zstd -3 -T2 -f --rsyncable > "/home/backup-accounts/cpuser1/logs-cpuser1-010719-184816.tar.zst"

backup start time: Mon Jul  1 18:48:18 UTC 2019
backup end time: Mon Jul  1 18:48:33 UTC 2019
backup time: 14.885
compression level: -3
compression ratio: 12.336 (1049492951 / 85069054)
compression ratio: 0.081 (85069054 / 1049492951)
slack notification sending
ok

--------------------------------------------------------
backup cpanel /home/cpuser1/ssl directory

/bin/nice -n 12 /bin/ionice -c2 -n7 tar cpf - ssl | zstd -6 -T2 -f --rsyncable > "/home/backup-accounts/cpuser1/ssl-cpuser1-010719-184816.tar.zst"

backup start time: Mon Jul  1 18:48:34 UTC 2019
backup end time: Mon Jul  1 18:48:34 UTC 2019
backup time: 0.011
compression level: -6
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

/bin/nice -n 12 /bin/ionice -c2 -n7 mysqldump --default-character-set=utf8 -Q -K --max_allowed_packet=256M --net_buffer_length=65536 --routines --events --triggers --hex-blob "cpuser1_db1" | zstd -3 -T2 -f --rsyncable > "/home/backup-accounts/cpuser1/mysqlbackup-cpuser1_db1-010719-184816.sql.zst"

backup start time: Mon Jul  1 18:48:34 UTC 2019
backup end time: Mon Jul  1 18:49:11 UTC 2019
backup time: 36.716
compression level: -3
compression ratio: 4.239 (1232429907 / 290671453)
compression ratio: 0.235 (290671453 / 1232429907)
slack notification sending
ok

/bin/nice -n 12 /bin/ionice -c2 -n7 mysqldump --default-character-set=utf8 --single-transaction -Q -K --max_allowed_packet=256M --net_buffer_length=65536 --routines --events --triggers --hex-blob "cpuser1_db2" | zstd -3 -T2 -f --rsyncable > "/home/backup-accounts/cpuser1/mysqlbackup-cpuser1_db2-010719-184816.sql.zst"

backup start time: Mon Jul  1 18:49:11 UTC 2019
backup end time: Mon Jul  1 18:49:11 UTC 2019
backup time: 0.024
compression level: -3
compression ratio: 7.888 (4157 / 527)
compression ratio: 0.126 (527 / 4157)
slack notification sending
ok

/bin/nice -n 12 /bin/ionice -c2 -n7 mysqldump --default-character-set=utf8 --single-transaction -Q -K --max_allowed_packet=256M --net_buffer_length=65536 --routines --events --triggers --hex-blob "cpuser1_db3" | zstd -3 -T2 -f --rsyncable > "/home/backup-accounts/cpuser1/mysqlbackup-cpuser1_db3-010719-184816.sql.zst"

backup start time: Mon Jul  1 18:49:12 UTC 2019
backup end time: Mon Jul  1 18:49:12 UTC 2019
backup time: 0.013
compression level: -3
compression ratio: 7.888 (4157 / 527)
compression ratio: 0.126 (527 / 4157)
slack notification sending
ok

--------------------------------------------------------
List cpuser1 backups at /home/backup-accounts/cpuser1
--------------------------------------------------------

-rw-r--r--. 1 root root  11M Jul  1 18:48 public_html-cpuser1-010719-184816.tar.zst
-rw-r--r--. 1 root root  301 Jul  1 18:48 mail-cpuser1-010719-184816.tar.zst
-rw-r--r--. 1 root root  82M Jul  1 18:48 logs-cpuser1-010719-184816.tar.zst
-rw-r--r--. 1 root root   88 Jul  1 18:48 ssl-cpuser1-010719-184816.tar.zst
-rw-r--r--. 1 root root  835 Jul  1 18:48 mysql-grants-010719-184816.log
-rw-r--r--. 1 root root  861 Jul  1 18:48 mysql-grants-ssh-010719-184816.log
-rw-r--r--. 1 root root 278M Jul  1 18:49 mysqlbackup-cpuser1_db1-010719-184816.sql.zst
-rw-r--r--. 1 root root  527 Jul  1 18:49 mysqlbackup-cpuser1_db2-010719-184816.sql.zst
-rw-r--r--. 1 root root  527 Jul  1 18:49 mysqlbackup-cpuser1_db3-010719-184816.sql.zst


--------------------------------------------------------
sar usage stats saved at
/home/backup-accounts/logs/backup_sar_stats_010719-184816
--------------------------------------------------------
read via sar command:

cpu load avg

sar -q -f /home/backup-accounts/logs/backup_sar_stats_010719-184816 | sed -e "s|$(hostname)|hostname|g"

06:48:16 PM   runq-sz  plist-sz   ldavg-1   ldavg-5  ldavg-15   blocked
Average:            3       146      1.03      0.28      0.13         0

1min 5min 15min min:
0.16 0.05 0.06
1min 5min 15min avg:
1.03 0.28 0.13
1min 5min 15min max:
1.55 0.45 0.19
1min 5min 15min 95%:
1.55 0.45 0.19

cpu utilisation

sar -u -f /home/backup-accounts/logs/backup_sar_stats_010719-184816 | sed -e "s|$(hostname)|hostname|g"

06:48:16 PM     CPU     %user     %nice   %system   %iowait    %steal     %idle
Average:        all     72.43      7.01      3.08      3.24      0.05     14.20

%user %nice %system $iowait %steal %idle min:
0.50 0.00 0.00 0.00 0.00 1.01
%user %nice %system $iowait %steal %idle avg:
72.44 7.01 3.08 3.24 0.05 14.18
%user %nice %system $iowait %steal %idle max:
93.94 15.66 8.08 68.53 0.51 99.00
%user %nice %system $iowait %steal %idle 95%:
87.94 14.21 6.64 18.83 0.50 85.07

memory usage

sar -r -f /home/backup-accounts/logs/backup_sar_stats_010719-184816 | sed -e "s|$(hostname)|hostname|g"

06:48:16 PM kbmemfree kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit  kbactive   kbinact   kbdirty
Average:       103532    911204     89.80     26091    511394   1327279     64.33    184846    588241     22697

kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit kbactive kbinact kbdirty min:
69412.00 841544.00 82.93 24820.00 431512.00 1278260.00 61.95 156940.00 513432.00 0.00
kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit kbactive kbinact kbdirty avg:
103532.35 911203.65 89.80 26090.53 511394.04 1327278.67 64.33 184846.04 588241.33 22696.70
kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit kbactive kbinact kbdirty max:
173192.00 945324.00 93.16 38180.00 552112.00 1380248.00 66.89 228360.00 620260.00 42984.00
kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit kbactive kbinact kbdirty 95%:
135009.60 928285.60 91.48 38173.60 538869.60 1380248.00 66.89 227361.60 616761.60 38702.40

disk I/O usage

sar -d -f /home/backup-accounts/logs/backup_sar_stats_010719-184816 | sed -e "s|$(hostname)|hostname|g"

06:48:16 PM       DEV       tps  rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz     await     svctm     %util
Average:     dev253-0    181.67  66801.54  12273.95    435.27      0.34      1.91      0.33      5.92

tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util min:
5.00 5.00 608.00 0.00 14.22 0.00 0.05 0.05
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util avg:
181.81 181.81 66834.62 12258.28 758.92 0.34 4.21 0.68
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util max:
4396.00 4396.00 180224.00 83392.00 942.70 1.14 37.90 5.37
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util 95%:
215.40 215.40 164856.00 73265.60 911.79 0.86 12.77 2.55

--------------------------------------------------------
/root/tools/cpanel-backup.sh run log saved at
/home/backup-accounts/logs/cpanel-backup-010719-184816.log
--------------------------------------------------------
backup completed
--------------------------------------------------------
total cpanel backup time: 57.342 seconds

slack notification sending
ok
--------------------------------------------------------
```