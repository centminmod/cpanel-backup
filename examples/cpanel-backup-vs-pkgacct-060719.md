# cpanel-backup.sh vs pkgacct

Below is example run on CentOS 6 cPanel/WHM server testing overall backup speed and compression speed/ratio performance between cPanel native pkgacct versus cpanel-backup.sh backup methods. cPanel pkgacct account method does backup more items than cpanel-backup.sh which can also factor into the size despite zstd compression ratio being more efficient than cPanel's use of gzip via multi-threaded pigz compression algorithm. However, the purpose of cpanel-backup.sh is for speed and compression ratio efficiency for data migration from cPanel/WHM as part of the importer process for importing into [Centmin Mod LEMP stack](https://centminmod.com/) in light of [cPanel announced price increase and licensing changes](https://community.centminmod.com/threads/17849/).

Below example comparison is done on cPanel/WHM VPS server - 4 cpu KVM VPS running CentOS 6 shows:

* cpanel-backup.sh method being `~29.45%` faster + `~6.33%` smaller compressed backup files than cPanel native pkgacct method for a single cPanel user account of ~1.45GB in size for data files + MySQL databases for default set zstd compression levels in cpanel-backup.sh and 
* `~21.26%` faster than cPanel pkgacct when using higher zstd compression levels in cpanel-backup.sh + `12.88%` smaller compressed file backups than pkgacct method.

Where `/home/cpuser1` size wise is ~205 MB for files and ~1.25GB for MySQL databases

```
du -s /home/cpuser1
209436  /home/cpuser1

du -s /var/lib/mysql/cpuser1*
1149376 /var/lib/mysql/cpuser1_db1
183680  /var/lib/mysql/cpuser1_db2
76424   /var/lib/mysql/cpuser1_db3
```

# contents

* [backup times compared](#backup-times-compared)
* [backup sizes compared](#backup-sizes-compared)
* [cpu load averages](#cpu-load-averages)
* [cpu utilisation](#cpu-utilisation)
* [memory usage](#memory-usage)
* [disk I/O usage](#disk-io-usage)
* [cpanel-backup.sh usage options](#cpanel-backupsh-usage-options)
* [pkgacct backup run](#pkgacct-backup-run)
* [cpanel-backup.sh default run](#cpanel-backupsh-backup-run)
* [cpanel-backup.sh higher compression level run](#cpanel-backupsh-higher-compression-level-backup-run)

## backup times compared

* cPanel pkgacct backup time = 110.644 seconds. If you had 100 such cPanel accounts, total backup time would be = 100 x 110.644 seconds = 184.41 minutes
* cpanel-backup.sh backup time = 78.055 seconds `~29.45%` faster. If you had 100 such cPanel accounts, total backup time would be = 100 x 78.055 seconds = 130.09 minutes. Thus saving you ~54.32 minutes compared to pkgacct process.
* cpanel-backup higher zstd level 9 compression backup time = 87.123 seconds `21.26%` faster than pkgacct backup method and `11.62%` slower than default cpanel-backup.sh zstd compression levels set. If you had 100 such cPanel accounts, total backup time would be = 100 x 87.123 seconds = 145.205 minutes. Thus saving you ~39.205 minutes compared to pkgacct process.

## backup sizes compared

* cPanel pkgacct backup size = 489 MB. 100 such cPanel accounts would take = 100 x 489MB = 47.754 GB of disk space.
* cpanel-backup.sh backup size = 947M - 489M = 458 MB `~6.33%` smaller. 100 such cPanel accounts would take = 100x 458MB = 44.727 GB of disk space - ~ 3.027GB less than pkgacct process.
* cpanel-backup higher zstd level 9 compression backup size = 426 MB `12.88%` smaller than pkgacct method and `6.99%` smaller than cpanel-backup.sh default zstd compression level. 100 such cPanel accounts would take = 100x 426MB = 41.6 GB of disk space - ~ 6.154GB less than pkgacct process.

cPanel pkgacct backup (`cpmove-cpuser1.tar.gz`) vs cpanel-backup.sh sizes (060719-040359)

```
ls -lahrt /home/backup-accounts/cpuser1 
total 947M
drwxr-xr-x 5 root root 4.0K Jul  6 03:49 ../
-rw------- 1 root root 489M Jul  6 03:51 cpmove-cpuser1.tar.gz
-rw-r--r-- 1 root root  103 Jul  6 04:03 domain-map-cpuser1-060719-040359.json
-rw-r--r-- 1 root root  338 Jul  6 04:03 domain-map-cpuser1-060719-040359.txt
drwxr-xr-x 2 root root 4.0K Jul  6 04:03 named/
-rw-r--r-- 1 root root   49 Jul  6 04:03 cronjobs-cpuser1-060719-040359.txt
-rw-r--r-- 1 root root 6.5M Jul  6 04:04 public_html-cpuser1-060719-040359.tar.zst
-rw-r--r-- 1 root root  371 Jul  6 04:04 mail-cpuser1-060719-040359.tar.zst
-rw-r--r-- 1 root root 165M Jul  6 04:04 logs-cpuser1-060719-040359.tar.zst
-rw-r--r-- 1 root root 3.5K Jul  6 04:04 ssl-cpuser1-060719-040359.tar.zst
-rw-r--r-- 1 root root 562K Jul  6 04:04 repositories-cpuser1-060719-040359.tar.zst
-rw-r--r-- 1 root root  302 Jul  6 04:04 mysqlgrants-060719-040359.log
-rw-r--r-- 1 root root  682 Jul  6 04:04 mysqlgrants-ssh-060719-040359.log
-rw-r--r-- 1 root root 262M Jul  6 04:05 mysqlbackup-cpuser1_db1-060719-040359.sql.zst
-rw-r--r-- 1 root root  22M Jul  6 04:05 mysqlbackup-cpuser1_db2-060719-040359.sql.zst
drwxr-xr-x 3 root root 4.0K Jul  6 04:05 ./
-rw-r--r-- 1 root root 3.0M Jul  6 04:05 mysqlbackup-cpuser1_db3-060719-040359.sql.zst
```

cPanel pkgacct backup (`cpmove-cpuser1.tar.gz`) vs cpanel-backup.sh sizes (060719-040359) vs cpanel-backup.sh higher zstd lvl 9 compression sizes (060719-044352)

```
ls -lahrt /home/backup-accounts/cpuser1/
total 1.4G
drwxr-xr-x 5 root root 4.0K Jul  6 03:49 ../
-rw------- 1 root root 489M Jul  6 03:51 cpmove-cpuser1.tar.gz
-rw-r--r-- 1 root root  103 Jul  6 04:03 domain-map-cpuser1-060719-040359.json
-rw-r--r-- 1 root root  338 Jul  6 04:03 domain-map-cpuser1-060719-040359.txt
-rw-r--r-- 1 root root   49 Jul  6 04:03 cronjobs-cpuser1-060719-040359.txt
-rw-r--r-- 1 root root 6.5M Jul  6 04:04 public_html-cpuser1-060719-040359.tar.zst
-rw-r--r-- 1 root root  371 Jul  6 04:04 mail-cpuser1-060719-040359.tar.zst
-rw-r--r-- 1 root root 165M Jul  6 04:04 logs-cpuser1-060719-040359.tar.zst
-rw-r--r-- 1 root root 3.5K Jul  6 04:04 ssl-cpuser1-060719-040359.tar.zst
-rw-r--r-- 1 root root 562K Jul  6 04:04 repositories-cpuser1-060719-040359.tar.zst
-rw-r--r-- 1 root root  302 Jul  6 04:04 mysqlgrants-060719-040359.log
-rw-r--r-- 1 root root  682 Jul  6 04:04 mysqlgrants-ssh-060719-040359.log
-rw-r--r-- 1 root root 262M Jul  6 04:05 mysqlbackup-cpuser1_db1-060719-040359.sql.zst
-rw-r--r-- 1 root root  22M Jul  6 04:05 mysqlbackup-cpuser1_db2-060719-040359.sql.zst
-rw-r--r-- 1 root root 3.0M Jul  6 04:05 mysqlbackup-cpuser1_db3-060719-040359.sql.zst
-rw-r--r-- 1 root root  103 Jul  6 04:43 domain-map-cpuser1-060719-044352.json
-rw-r--r-- 1 root root  338 Jul  6 04:43 domain-map-cpuser1-060719-044352.txt
drwxr-xr-x 2 root root 4.0K Jul  6 04:43 named/
-rw-r--r-- 1 root root   49 Jul  6 04:43 cronjobs-cpuser1-060719-044352.txt
-rw-r--r-- 1 root root 6.2M Jul  6 04:43 public_html-cpuser1-060719-044352.tar.zst
-rw-r--r-- 1 root root  364 Jul  6 04:43 mail-cpuser1-060719-044352.tar.zst
-rw-r--r-- 1 root root 165M Jul  6 04:43 logs-cpuser1-060719-044352.tar.zst
-rw-r--r-- 1 root root 3.4K Jul  6 04:44 ssl-cpuser1-060719-044352.tar.zst
-rw-r--r-- 1 root root 562K Jul  6 04:44 repositories-cpuser1-060719-044352.tar.zst
-rw-r--r-- 1 root root  302 Jul  6 04:44 mysqlgrants-060719-044352.log
-rw-r--r-- 1 root root  682 Jul  6 04:44 mysqlgrants-ssh-060719-044352.log
-rw-r--r-- 1 root root 236M Jul  6 04:45 mysqlbackup-cpuser1_db1-060719-044352.sql.zst
-rw-r--r-- 1 root root  17M Jul  6 04:45 mysqlbackup-cpuser1_db2-060719-044352.sql.zst
drwxr-xr-x 3 root root 4.0K Jul  6 04:45 ./
-rw-r--r-- 1 root root 2.6M Jul  6 04:45 mysqlbackup-cpuser1_db3-060719-044352.sql.zst
```

cpanel-backup.sh higher zstd lvl 9 compression sizes = 426 MB

```
ls -lahrt /home/backup-accounts/cpuser1/
total 426M
drwxr-xr-x 5 root root 4.0K Jul  6 04:53 ../
-rw-r--r-- 1 root root  103 Jul  6 04:53 domain-map-cpuser1-060719-045333.json
-rw-r--r-- 1 root root  338 Jul  6 04:53 domain-map-cpuser1-060719-045333.txt
drwxr-xr-x 2 root root 4.0K Jul  6 04:53 named/
-rw-r--r-- 1 root root   49 Jul  6 04:53 cronjobs-cpuser1-060719-045333.txt
-rw-r--r-- 1 root root 6.2M Jul  6 04:53 public_html-cpuser1-060719-045333.tar.zst
-rw-r--r-- 1 root root  364 Jul  6 04:53 mail-cpuser1-060719-045333.tar.zst
-rw-r--r-- 1 root root 165M Jul  6 04:53 logs-cpuser1-060719-045333.tar.zst
-rw-r--r-- 1 root root 3.4K Jul  6 04:53 ssl-cpuser1-060719-045333.tar.zst
-rw-r--r-- 1 root root 562K Jul  6 04:53 repositories-cpuser1-060719-045333.tar.zst
-rw-r--r-- 1 root root  302 Jul  6 04:53 mysqlgrants-060719-045333.log
-rw-r--r-- 1 root root  682 Jul  6 04:53 mysqlgrants-ssh-060719-045333.log
-rw-r--r-- 1 root root 236M Jul  6 04:54 mysqlbackup-cpuser1_db1-060719-045333.sql.zst
-rw-r--r-- 1 root root  17M Jul  6 04:54 mysqlbackup-cpuser1_db2-060719-045333.sql.zst
drwxr-xr-x 3 root root 4.0K Jul  6 04:54 ./
-rw-r--r-- 1 root root 2.6M Jul  6 04:54 mysqlbackup-cpuser1_db3-060719-045333.sql.zst
```

## cpu load averages

* cpanel pkgacct

```
1min 5min 15min min:
0.10 0.11 0.26
1min 5min 15min avg:
2.14 0.71 0.46
1min 5min 15min max:
4.00 1.40 0.70
1min 5min 15min 95%:
4.00 1.36 0.68
```

* cpanel-backup.sh

```
1min 5min 15min min:
0.00 0.18 0.36
1min 5min 15min avg:
1.06 0.43 0.44
1min 5min 15min max:
1.99 0.72 0.53
1min 5min 15min 95%:
1.99 0.72 0.53
```

* cpanel-backup.sh zstd 9 lvl

```
1min 5min 15min min:
0.14 0.61 0.57
1min 5min 15min avg:
1.81 1.01 0.71
1min 5min 15min max:
3.50 1.54 0.90
1min 5min 15min 95%:
3.46 1.49 0.88
```

## cpu utilisation

* cpanel pkgacct

```
%user %nice %system $iowait %steal %idle min:
0.00 0.00 0.75 0.24 0.00 0.25
%user %nice %system $iowait %steal %idle avg:
8.59 29.81 6.12 13.11 0.63 41.74
%user %nice %system $iowait %steal %idle max:
24.37 97.75 14.75 56.96 7.21 97.49
%user %nice %system $iowait %steal %idle 95%:
17.23 93.85 12.88 35.55 2.63 75.78
```

* cpanel-backup.sh

```
%user %nice %system $iowait %steal %idle min:
6.03 0.00 3.55 0.00 0.00 23.68
%user %nice %system $iowait %steal %idle avg:
19.76 7.85 7.07 12.50 0.44 52.38
%user %nice %system $iowait %steal %idle max:
30.65 13.25 13.57 52.64 1.52 82.07
%user %nice %system $iowait %steal %idle 95%:
29.12 12.29 11.50 29.66 1.01 73.50
```

* cpanel-backup.sh zstd 9 lvl

```
%user %nice %system $iowait %steal %idle min:
3.03 0.00 3.30 0.00 0.00 2.26
%user %nice %system $iowait %steal %idle avg:
45.06 6.64 6.72 12.91 0.62 28.06
%user %nice %system $iowait %steal %idle max:
85.71 10.80 14.43 68.61 8.60 85.86
%user %nice %system $iowait %steal %idle 95%:
75.50 9.85 11.65 34.01 3.13 62.05
```

## memory usage

* cpanel pkgacct

```
kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit min:
70480.00 902768.00 88.55 180.00 103624.00 2042472.00 98.76
kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit avg:
94214.09 925321.91 90.76 3190.61 144308.77 2138240.29 103.39
kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit max:
116768.00 949056.00 93.09 15444.00 179012.00 2181324.00 105.47
kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit 95%:
102530.40 943058.40 92.50 10298.40 167748.00 2181168.00 105.47
```

* cpanel-backup.sh

```
kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit min:
70756.00 852616.00 83.63 148.00 96332.00 2006260.00 97.01
kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit avg:
98633.47 920902.53 90.33 4620.11 141466.05 2131570.53 103.07
kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit max:
166920.00 948780.00 93.06 21344.00 208380.00 2201936.00 106.47
kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit 95%:
131658.00 943290.00 92.52 12985.00 180982.00 2200782.00 106.42
```

* cpanel-backup.sh zstd 9 lvl

```
kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit min:
69576.00 816800.00 80.11 124.00 5356.00 2004872.00 96.94
kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit avg:
98038.26 921497.74 90.38 2127.39 71062.87 2171309.46 104.99
kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit max:
202736.00 949960.00 93.18 16552.00 117940.00 2263740.00 109.46
kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit 95%:
129747.20 944803.20 92.67 9864.00 107130.40 2244313.60 108.52
```

## disk I/O usage

* cpanel pkgacct

```
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util min:
25.25 25.25 2320.79 96.97 8.03 0.06 0.19 0.12
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util avg:
1140.88 1140.88 65509.86 32954.28 157.47 2.63 3.99 0.75
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util max:
6264.89 6264.89 386214.14 159385.26 600.58 42.83 84.22 2.69
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util 95%:
2134.15 2134.15 157165.52 137743.14 374.71 7.75 14.07 1.95
```

* cpanel-backup.sh

```
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util min:
203.88 203.88 5252.53 90.72 8.02 0.09 0.25 0.19
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util avg:
1278.34 1278.34 43582.06 12651.34 117.25 1.92 3.66 0.77
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util max:
5014.74 5014.74 147208.00 202576.00 630.88 24.68 92.19 2.79
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util 95%:
3943.38 3943.38 87117.01 97912.04 367.79 5.33 10.51 2.12
```

* cpanel-backup.sh zstd 9 lvl

```
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util min:
174.75 174.75 4040.40 95.05 8.02 0.07 0.18 0.17
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util avg:
1259.52 1259.52 42006.75 13934.29 92.17 1.60 1.83 0.60
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util max:
4963.92 4963.92 133204.04 113446.46 419.44 21.11 21.30 1.93
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util 95%:
2163.72 2163.72 83236.64 78845.28 322.44 5.45 4.77 1.62
```

## cpanel-backup.sh usage options

```
./cpanel-backup.sh 
Usage:

./cpanel-backup.sh {backup}
./cpanel-backup.sh {backup} {cpanel_username}
./cpanel-backup.sh {pkgacct} {cpanel_username}

```

## pkgacct backup run

cPanel pkgacct backup run for cPanel user = cpuser1

```
/root/tools/cpanel-backup.sh pkgacct cpuser1

--------------------------------------------------------
cPanel/WHM backup script 0.9
for data migration to Centmin Mod LEMP stack imports
written by George Liu (centminmod.com)
--------------------------------------------------------

--------------------------------------------------------
list cpanel users
--------------------------------------------------------

cpuser1

--------------------------------------------------------
running pkgacct backup
--------------------------------------------------------

/bin/nice -n 12 /usr/bin/ionice -c2 -n7 pkgacct cpuser1 /home/backup-accounts/cpuser1
[2019-07-06 03:49:44 -0700] pkgacct started.
[2019-07-06 03:49:44 -0700] pkgacct version 10 - user : cpuser1 - tarball: 1 - target mysql : default - split: 0 - incremental: 0 - homedir: 1 - mailman: 1 - backup: 0 - archive version: 4 - running with uid 0
[2019-07-06 03:49:44 -0700] pkgacct using '/usr/local/cpanel/3rdparty/bin/pigz -6 --processes 4 --blocksize 1024 --rsyncable' to compress archives
[2019-07-06 03:49:44 -0700] pkgacct working dir : /home/backup-accounts/cpuser1/cpmove-cpuser1
[2019-07-06 03:49:44 -0700] Copying Reseller Config...[2019-07-06 03:49:44 -0700] Done
[2019-07-06 03:49:44 -0700] Copying Suspension Info (if needed)...[2019-07-06 03:49:44 -0700] Done
[2019-07-06 03:49:44 -0700] Copying installed SSL certificates and keys...[2019-07-06 03:49:44 -0700] Performing “ApacheTLS” component....
[2019-07-06 03:49:44 -0700] Completed “ApacheTLS” component.
[2019-07-06 03:49:44 -0700] Done
[2019-07-06 03:49:44 -0700] Copying Domain Keys....[2019-07-06 03:49:44 -0700] Done
[2019-07-06 03:49:44 -0700] Copying Bandwidth Data....[2019-07-06 03:49:44 -0700] Performing “Bandwidth” component....
Summary databases … done!
[2019-07-06 03:49:45 -0700] Completed “Bandwidth” component.
[2019-07-06 03:49:45 -0700] Done
[2019-07-06 03:49:45 -0700] Copying Dns Zones.......domain1.com...[2019-07-06 03:49:45 -0700] Done
[2019-07-06 03:49:45 -0700] Copying Mail files....[2019-07-06 03:49:45 -0700] Done
[2019-07-06 03:49:45 -0700] Copying proftpd file....[2019-07-06 03:49:45 -0700] Done
[2019-07-06 03:49:45 -0700] Performing “Logs” component....
...log file sizes [0 byte(s)]......domain1.com......domain1.com-ssl_log...[2019-07-06 03:49:45 -0700] Completed “Logs” component.
[2019-07-06 03:49:45 -0700] Copy userdata...[2019-07-06 03:49:45 -0700] Done
[2019-07-06 03:49:45 -0700] Copy custom virtualhost templates...[2019-07-06 03:49:45 -0700] Done
[2019-07-06 03:49:45 -0700] Copying mailman lists and archives....Done copying mailman lists and archives.
[2019-07-06 03:49:45 -0700] Copying homedir.............
[2019-07-06 03:49:45 -0700] Done
[2019-07-06 03:49:45 -0700] Fixing up EA4 .htaccess blocks: Done.
[2019-07-06 03:49:45 -0700] Performing “Postgresql” component....
[2019-07-06 03:49:45 -0700] Completed “Postgresql” component.
[2019-07-06 03:49:45 -0700] Performing “Mysql” component....
[2019-07-06 03:49:45 -0700] Determining mysql dbs...
[2019-07-06 03:49:46 -0700] ...mysqldump version: 5.6.44...[2019-07-06 03:49:46 -0700] ...mysql version: 5.6...[2019-07-06 03:49:46 -0700] Saving mysql privs...[2019-07-06 03:49:46 -0700] Done
[2019-07-06 03:49:46 -0700] ...Done
[2019-07-06 03:49:46 -0700] Storing mysql dbs............
cpuser1_db2.........
[2019-07-06 03:49:55 -0700] (188921305 bytes) 
cpuser1_db1.........
.........
.........
.........
[2019-07-06 03:50:51 -0700] (1070785537 bytes) 
cpuser1_db3[2019-07-06 03:50:53 -0700] (14426511 bytes) 
[2019-07-06 03:50:53 -0700] ...Done
[2019-07-06 03:50:53 -0700] Completed “Mysql” component.
[2019-07-06 03:50:53 -0700] Performing “MysqlRemoteNotes” component....
[2019-07-06 03:50:53 -0700] Completed “MysqlRemoteNotes” component.
[2019-07-06 03:50:53 -0700] Copying cpuser file.......[2019-07-06 03:50:53 -0700] Done
[2019-07-06 03:50:53 -0700] Copying crontab file.......[2019-07-06 03:50:53 -0700] Done
[2019-07-06 03:50:53 -0700] Performing “Quota” component....
[2019-07-06 03:50:54 -0700] Completed “Quota” component.
[2019-07-06 03:50:54 -0700] Performing “Integration” component....
[2019-07-06 03:50:54 -0700] Completed “Integration” component.
[2019-07-06 03:50:54 -0700] Performing “AuthnLinks” component....
[2019-07-06 03:50:54 -0700] Completed “AuthnLinks” component.
[2019-07-06 03:50:54 -0700] Performing “APITokens” component....
[2019-07-06 03:50:54 -0700] Completed “APITokens” component.
[2019-07-06 03:50:54 -0700] Performing “Custom” component....
[2019-07-06 03:50:54 -0700] No custom components to perform.
[2019-07-06 03:50:54 -0700] Completed “Custom” component.
[2019-07-06 03:50:54 -0700] Performing “AutoSSL” component....
[2019-07-06 03:50:54 -0700] Completed “AutoSSL” component.
[2019-07-06 03:50:54 -0700] Storing Subdomains....
[2019-07-06 03:50:54 -0700] Done
[2019-07-06 03:50:54 -0700] Storing Parked Domains....
[2019-07-06 03:50:54 -0700] Done
[2019-07-06 03:50:54 -0700] Storing Addon Domains....
[2019-07-06 03:50:54 -0700] Performing “Password” component....
[2019-07-06 03:50:54 -0700] Completed “Password” component.
[2019-07-06 03:50:54 -0700] Performing “DigestShadow” component....
[2019-07-06 03:50:54 -0700] Completed “DigestShadow” component.
[2019-07-06 03:50:54 -0700] Copying shell.......[2019-07-06 03:50:54 -0700] Done
[2019-07-06 03:50:54 -0700] Performing “PublicContact” component....
[2019-07-06 03:50:54 -0700] Completed “PublicContact” component.
[2019-07-06 03:50:54 -0700] Performing “MailLimits” component....
[2019-07-06 03:50:54 -0700] Completed “MailLimits” component.
[2019-07-06 03:50:54 -0700] Creating Archive ........................
[2019-07-06 03:51:27 -0700] Done
[2019-07-06 03:51:27 -0700] pkgacctfile is: /home/backup-accounts/cpuser1/cpmove-cpuser1.tar.gz
[2019-07-06 03:51:30 -0700] md5sum is: d10fac83b5bef54301b8bc0627f60812
[2019-07-06 03:51:30 -0700] 
[2019-07-06 03:51:30 -0700] size is: 512618589
[2019-07-06 03:51:30 -0700] 
[2019-07-06 03:51:30 -0700] homesize is: 41713664
[2019-07-06 03:51:30 -0700] 
[2019-07-06 03:51:30 -0700] homefiles is: 3827
[2019-07-06 03:51:30 -0700] pkgacct completed

backup start time: Sat Jul  6 03:49:44 PDT 2019
backup end time: Sat Jul  6 03:51:30 PDT 2019
backup time: 106.686
compression processes: 4
compression level: 6
compression ratio: 3.212 (1646594078 / 512618589)
compression ratio: 0.311 (512618589 / 1646594078)
slack notification sending
ok

--------------------------------------------------------
pidstat stats saved at:
/home/backup-accounts/logs/backup_pidstat_stats_060719-034943.log.zst
--------------------------------------------------------
read command:
zstdcat /home/backup-accounts/logs/backup_pidstat_stats_060719-034943.log.zst
--------------------------------------------------------
sar usage stats saved at:
/home/backup-accounts/logs/backup_sar_stats_060719-034943
--------------------------------------------------------
read via sar command:

cpu load avg

sar -q -f /home/backup-accounts/logs/backup_sar_stats_060719-034943 | sed -e "s|$(hostname)|hostname|g"

03:49:44 AM   runq-sz  plist-sz   ldavg-1   ldavg-5  ldavg-15
Average:            2       232      2.14      0.71      0.46

1min 5min 15min min:
0.10 0.11 0.26
1min 5min 15min avg:
2.14 0.71 0.46
1min 5min 15min max:
4.00 1.40 0.70
1min 5min 15min 95%:
4.00 1.36 0.68

cpu utilisation

sar -u -f /home/backup-accounts/logs/backup_sar_stats_060719-034943 | sed -e "s|$(hostname)|hostname|g"

03:49:44 AM     CPU     %user     %nice   %system   %iowait    %steal     %idle
Average:        all      8.54     30.13      6.10     13.05      0.64     41.54

%user %nice %system $iowait %steal %idle min:
0.00 0.00 0.75 0.24 0.00 0.25
%user %nice %system $iowait %steal %idle avg:
8.59 29.81 6.12 13.11 0.63 41.74
%user %nice %system $iowait %steal %idle max:
24.37 97.75 14.75 56.96 7.21 97.49
%user %nice %system $iowait %steal %idle 95%:
17.23 93.85 12.88 35.55 2.63 75.78

memory usage

sar -r -f /home/backup-accounts/logs/backup_sar_stats_060719-034943 | sed -e "s|$(hostname)|hostname|g"

03:49:44 AM kbmemfree kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit
Average:        94214    925322     90.76      3191    144309   2138240    103.39

kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit min:
70480.00 902768.00 88.55 180.00 103624.00 2042472.00 98.76
kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit avg:
94214.09 925321.91 90.76 3190.61 144308.77 2138240.29 103.39
kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit max:
116768.00 949056.00 93.09 15444.00 179012.00 2181324.00 105.47
kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit 95%:
102530.40 943058.40 92.50 10298.40 167748.00 2181168.00 105.47

disk I/O usage

sar -d -f /home/backup-accounts/logs/backup_sar_stats_060719-034943 | sed -e "s|$(hostname)|hostname|g"

03:49:44 AM       DEV       tps  rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz     await     svctm     %util
Average:     dev252-0   1127.56  65748.58  32868.61     87.46      2.62      2.32      0.49     55.77

tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util min:
25.25 25.25 2320.79 96.97 8.03 0.06 0.19 0.12
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util avg:
1140.88 1140.88 65509.86 32954.28 157.47 2.63 3.99 0.75
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util max:
6264.89 6264.89 386214.14 159385.26 600.58 42.83 84.22 2.69
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util 95%:
2134.15 2134.15 157165.52 137743.14 374.71 7.75 14.07 1.95

--------------------------------------------------------
/root/tools/cpanel-backup.sh run log saved at:
/home/backup-accounts/logs/cpanel-backup-060719-034943.log.zst
--------------------------------------------------------
read command:
zstdcat /home/backup-accounts/logs/cpanel-backup-060719-034943.log.zst
--------------------------------------------------------

backup completed
--------------------------------------------------------
total pkgacct cpanel backup time: 110.644 seconds

slack notification sending
ok
--------------------------------------------------------
```

## cpanel-backup.sh backup run

cpanel-backup.sh run for cPanel user = cpuser1

With all backup targets enabled for public_html, logs, mail, mysql, ssl and git repositories

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

And default per backup target zstd compression level tuning as per. Where for logs backup target, cpanel-backup.sh will conditionally reduce zstd compression level to lowest negative 10 (--fast=10) levels to not waste time if the script detects there are already a mix of compressed & uncompressed version of your logs 

```
# regular zstd compression level control for levels 1-19
ZSTD_COMPLEVEL_WEBROOT='6'
ZSTD_COMPLEVEL_MAIL='6'
ZSTD_COMPLEVEL_MYSQL='3'
ZSTD_COMPLEVEL_LOGS='3'
ZSTD_COMPLEVEL_SSL='6'
ZSTD_COMPLEVEL_GITREPOS='6'
```

```
/root/tools/cpanel-backup.sh backup cpuser1              

--------------------------------------------------------
cPanel/WHM backup script 0.9
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
  "parked_domains": [],
  "addon_domains": {},
  "main_domain": "domain1.com",
  "sub_domains": []
}

main_domain=domain1.com

sub_domains:



parked_domains:



cpuser_domainlist=domain1.com

--------------------------------------------------------
domain mapping saved:
/home/backup-accounts/cpuser1/domain-map-cpuser1-060719-040359.txt
--------------------------------------------------------

--------------------------------------------------------
backup cpuser1 domain related DNS zone files
--------------------------------------------------------

backup /var/named/domain1.com.db
cp -af /var/named/domain1.com.db /home/backup-accounts/cpuser1/named/domain1.com-060719-040359.db


--------------------------------------------------------
list cpuser1 cronjobs
--------------------------------------------------------

SHELL="/bin/bash"
#0 0 * * 0 echo "hello world"


--------------------------------------------------------
cpuser1 cronjobs saved:
/home/backup-accounts/cpuser1/cronjobs-cpuser1-060719-040359.txt
--------------------------------------------------------

--------------------------------------------------------
backup cpanel /home/cpuser1/public_html web root
--------------------------------------------------------

/bin/nice -n 12 /usr/bin/ionice -c2 -n7 tar cpf - public_html | zstd -6 -T4 -f --rsyncable > "/home/backup-accounts/cpuser1/public_html-cpuser1-060719-040359.tar.zst"

backup start time: Sat Jul  6 04:03:59 PDT 2019
backup end time: Sat Jul  6 04:04:01 PDT 2019
backup time: 2.456
zstd normal compression mode: enabled
compression level: 6
compression ratio: 4.420 (30092102 / 6807572)
compression ratio: 0.226 (6807572 / 30092102)
slack notification sending
ok

--------------------------------------------------------
backup cpanel /home/cpuser1/mail directory

/bin/nice -n 12 /usr/bin/ionice -c2 -n7 tar cpf - mail | zstd -6 -T4 -f --rsyncable > "/home/backup-accounts/cpuser1/mail-cpuser1-060719-040359.tar.zst"

backup start time: Sat Jul  6 04:04:02 PDT 2019
backup end time: Sat Jul  6 04:04:02 PDT 2019
backup time: 0.066
zstd normal compression mode: enabled
compression level: 6
compression ratio: 231.894 (86033 / 371)
compression ratio: 0.004 (371 / 86033)
slack notification sending
ok

--------------------------------------------------------
backup cpanel /home/cpuser1/logs directory

/bin/nice -n 12 /usr/bin/ionice -c2 -n7 tar cpf - logs | zstd  --fast=10 -T4 -f --rsyncable > "/home/backup-accounts/cpuser1/logs-cpuser1-060719-040359.tar.zst"

backup start time: Sat Jul  6 04:04:02 PDT 2019
backup end time: Sat Jul  6 04:04:05 PDT 2019
backup time: 2.451
zstd normal compression mode: enabled
compression level: fast=10
compression ratio: 1.000 (172750859 / 172749410)
compression ratio: 0.999 (172749410 / 172750859)
slack notification sending
ok

--------------------------------------------------------
backup cpanel /home/cpuser1/ssl directory

/bin/nice -n 12 /usr/bin/ionice -c2 -n7 tar cpf - ssl | zstd -6 -T4 -f --rsyncable > "/home/backup-accounts/cpuser1/ssl-cpuser1-060719-040359.tar.zst"

backup start time: Sat Jul  6 04:04:05 PDT 2019
backup end time: Sat Jul  6 04:04:06 PDT 2019
backup time: 0.099
zstd normal compression mode: enabled
compression level: 6
compression ratio: 7.483 (26116 / 3490)
compression ratio: 0.133 (3490 / 26116)
slack notification sending
ok

--------------------------------------------------------
backup cpanel /home/cpuser1/repositories directory

/bin/nice -n 12 /usr/bin/ionice -c2 -n7 tar cpf - repositories | zstd -9 -T4 -f --rsyncable > "/home/backup-accounts/cpuser1/repositories-cpuser1-060719-040359.tar.zst"

backup start time: Sat Jul  6 04:04:06 PDT 2019
backup end time: Sat Jul  6 04:04:06 PDT 2019
backup time: 0.427
zstd normal compression mode: enabled
compression level: 9
compression ratio: 1.630 (937259 / 574768)
compression ratio: 0.613 (574768 / 937259)
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

mysql username: cpuser1

Grants for cpuser1@localhost
GRANT USAGE ON *.* TO 'cpuser1'@'localhost' IDENTIFIED BY PASSWORD '*9EFEC505F817F592094619681AD824757DA660C6'
GRANT ALL PRIVILEGES ON `cpuser1\\_db1`.* TO 'cpuser1'@'localhost'
GRANT ALL PRIVILEGES ON `cpuser1\\_db2`.* TO 'cpuser1'@'localhost'
GRANT ALL PRIVILEGES ON `cpuser1\\_db3`.* TO 'cpuser1'@'localhost'

replay commands via SSH command line

mysql -e "GRANT USAGE ON *.* TO 'cpuser1'@'localhost' IDENTIFIED BY PASSWORD '*9EFEC505F817F592094619681AD824757DA660C6'" mysql
mysql -e "GRANT ALL PRIVILEGES ON cpuser1\_db1.* TO 'cpuser1'@'localhost'" mysql
mysql -e "GRANT ALL PRIVILEGES ON cpuser1\_db2.* TO 'cpuser1'@'localhost'" mysql
mysql -e "GRANT ALL PRIVILEGES ON cpuser1\_db3.* TO 'cpuser1'@'localhost'" mysql
mysql username: cpuser1_dbuser

Grants for cpuser1_dbuser@localhost
GRANT USAGE ON *.* TO 'cpuser1_dbuser'@'localhost' IDENTIFIED BY PASSWORD '*5DA619872CE4458DC6B0230DC365CD16B4280AC4'
GRANT ALL PRIVILEGES ON `cpuser1\\_db3`.* TO 'cpuser1_dbuser'@'localhost'
GRANT ALL PRIVILEGES ON `cpuser1\\_db2`.* TO 'cpuser1_dbuser'@'localhost'

replay commands via SSH command line

mysql -e "GRANT USAGE ON *.* TO 'cpuser1_dbuser'@'localhost' IDENTIFIED BY PASSWORD '*5DA619872CE4458DC6B0230DC365CD16B4280AC4'" mysql
mysql -e "GRANT ALL PRIVILEGES ON cpuser1\_db3.* TO 'cpuser1_dbuser'@'localhost'" mysql
mysql -e "GRANT ALL PRIVILEGES ON cpuser1\_db2.* TO 'cpuser1_dbuser'@'localhost'" mysql

backup cpanel user's mysql databases

/bin/nice -n 12 /usr/bin/ionice -c2 -n7 mysqldump --default-character-set=utf8 -Q -K --max_allowed_packet=256M --net_buffer_length=65536 --routines --events --triggers --hex-blob "cpuser1_db1" | zstd -3 -T4 -f --rsyncable > "/home/backup-accounts/cpuser1/mysqlbackup-cpuser1_db1-060719-040359.sql.zst"

backup start time: Sat Jul  6 04:04:07 PDT 2019
backup end time: Sat Jul  6 04:05:01 PDT 2019
backup time: 54.666
zstd normal compression mode: enabled
compression level: 3
compression ratio: 4.284 (1175332217 / 274318488)
compression ratio: 0.233 (274318488 / 1175332217)
slack notification sending
ok

/bin/nice -n 12 /usr/bin/ionice -c2 -n7 mysqldump --default-character-set=utf8 -Q -K --max_allowed_packet=256M --net_buffer_length=65536 --routines --events --triggers --hex-blob "cpuser1_db2" | zstd -3 -T4 -f --rsyncable > "/home/backup-accounts/cpuser1/mysqlbackup-cpuser1_db2-060719-040359.sql.zst"

backup start time: Sat Jul  6 04:05:03 PDT 2019
backup end time: Sat Jul  6 04:05:12 PDT 2019
backup time: 9.238
zstd normal compression mode: enabled
compression level: 3
compression ratio: 8.460 (187971591 / 22217671)
compression ratio: 0.118 (22217671 / 187971591)
slack notification sending
ok

/bin/nice -n 12 /usr/bin/ionice -c2 -n7 mysqldump --default-character-set=utf8 -Q -K --max_allowed_packet=256M --net_buffer_length=65536 --routines --events --triggers --hex-blob "cpuser1_db3" | zstd -3 -T4 -f --rsyncable > "/home/backup-accounts/cpuser1/mysqlbackup-cpuser1_db3-060719-040359.sql.zst"

backup start time: Sat Jul  6 04:05:13 PDT 2019
backup end time: Sat Jul  6 04:05:14 PDT 2019
backup time: 1.460
zstd normal compression mode: enabled
compression level: 3
compression ratio: 25.052 (77327310 / 3086646)
compression ratio: 0.039 (3086646 / 77327310)
slack notification sending
ok

--------------------------------------------------------
List cpuser1 backups at /home/backup-accounts/cpuser1
--------------------------------------------------------

+-- [489M]  cpmove-cpuser1.tar.gz
+-- [  49]  cronjobs-cpuser1-060719-040359.txt
+-- [ 103]  domain-map-cpuser1-060719-040359.json
+-- [ 338]  domain-map-cpuser1-060719-040359.txt
+-- [165M]  logs-cpuser1-060719-040359.tar.zst
+-- [ 371]  mail-cpuser1-060719-040359.tar.zst
+-- [262M]  mysqlbackup-cpuser1_db1-060719-040359.sql.zst
+-- [ 21M]  mysqlbackup-cpuser1_db2-060719-040359.sql.zst
+-- [2.9M]  mysqlbackup-cpuser1_db3-060719-040359.sql.zst
+-- [ 302]  mysqlgrants-060719-040359.log
+-- [ 682]  mysqlgrants-ssh-060719-040359.log
+-- [4.0K]  named
|   +-- [1.3K]  domain1.com-060719-040359.db
+-- [6.5M]  public_html-cpuser1-060719-040359.tar.zst
+-- [561K]  repositories-cpuser1-060719-040359.tar.zst
+-- [3.4K]  ssl-cpuser1-060719-040359.tar.zst


--------------------------------------------------------
pidstat stats saved at:
/home/backup-accounts/logs/backup_pidstat_stats_060719-040358.log.zst
--------------------------------------------------------
read command:
zstdcat /home/backup-accounts/logs/backup_pidstat_stats_060719-040358.log.zst
--------------------------------------------------------
sar usage stats saved at:
/home/backup-accounts/logs/backup_sar_stats_060719-040358
--------------------------------------------------------
read via sar command:

cpu load avg

sar -q -f /home/backup-accounts/logs/backup_sar_stats_060719-040358 | sed -e "s|$(hostname)|hostname|g"

04:03:59 AM   runq-sz  plist-sz   ldavg-1   ldavg-5  ldavg-15
Average:            2       230      1.06      0.43      0.44

1min 5min 15min min:
0.00 0.18 0.36
1min 5min 15min avg:
1.06 0.43 0.44
1min 5min 15min max:
1.99 0.72 0.53
1min 5min 15min 95%:
1.99 0.72 0.53

cpu utilisation

sar -u -f /home/backup-accounts/logs/backup_sar_stats_060719-040358 | sed -e "s|$(hostname)|hostname|g"

04:03:59 AM     CPU     %user     %nice   %system   %iowait    %steal     %idle
Average:        all     19.77      7.85      7.08     12.48      0.44     52.38

%user %nice %system $iowait %steal %idle min:
6.03 0.00 3.55 0.00 0.00 23.68
%user %nice %system $iowait %steal %idle avg:
19.76 7.85 7.07 12.50 0.44 52.38
%user %nice %system $iowait %steal %idle max:
30.65 13.25 13.57 52.64 1.52 82.07
%user %nice %system $iowait %steal %idle 95%:
29.12 12.29 11.50 29.66 1.01 73.50

memory usage

sar -r -f /home/backup-accounts/logs/backup_sar_stats_060719-040358 | sed -e "s|$(hostname)|hostname|g"

04:03:59 AM kbmemfree kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit
Average:        98633    920903     90.33      4620    141466   2131571    103.07

kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit min:
70756.00 852616.00 83.63 148.00 96332.00 2006260.00 97.01
kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit avg:
98633.47 920902.53 90.33 4620.11 141466.05 2131570.53 103.07
kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit max:
166920.00 948780.00 93.06 21344.00 208380.00 2201936.00 106.47
kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit 95%:
131658.00 943290.00 92.52 12985.00 180982.00 2200782.00 106.42

disk I/O usage

sar -d -f /home/backup-accounts/logs/backup_sar_stats_060719-040358 | sed -e "s|$(hostname)|hostname|g"

04:03:59 AM       DEV       tps  rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz     await     svctm     %util
Average:     dev252-0   1267.56  43568.59  12597.36     44.31      1.92      1.51      0.43     55.05

tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util min:
203.88 203.88 5252.53 90.72 8.02 0.09 0.25 0.19
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util avg:
1278.34 1278.34 43582.06 12651.34 117.25 1.92 3.66 0.77
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util max:
5014.74 5014.74 147208.00 202576.00 630.88 24.68 92.19 2.79
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util 95%:
3943.38 3943.38 87117.01 97912.04 367.79 5.33 10.51 2.12

--------------------------------------------------------
/root/tools/cpanel-backup.sh run log saved at:
/home/backup-accounts/logs/cpanel-backup-060719-040358.log.zst
--------------------------------------------------------
read command:
zstdcat /home/backup-accounts/logs/cpanel-backup-060719-040358.log.zst
--------------------------------------------------------

backup completed
--------------------------------------------------------
total cpanel backup time: 78.055 seconds

slack notification sending
ok
--------------------------------------------------------
```

## cpanel-backup.sh higher compression level backup run

Test backup cpanel-backup.sh with higher default zstd compression levels for smaller compressed files

cpanel-backup.sh run for cPanel user = cpuser1

With all backup targets enabled for public_html, logs, mail, mysql, ssl and git repositories

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

And default per backup target zstd compression level tuning as per. Where for logs backup target, cpanel-backup.sh will conditionally reduce zstd compression level to lowest negative 10 (--fast=10) levels to not waste time if the script detects there are already a mix of compressed & uncompressed version of your logs 

```
# regular zstd compression level control for levels 1-19
ZSTD_COMPLEVEL_WEBROOT='6'
ZSTD_COMPLEVEL_MAIL='6'
ZSTD_COMPLEVEL_MYSQL='3'
ZSTD_COMPLEVEL_LOGS='3'
ZSTD_COMPLEVEL_SSL='6'
ZSTD_COMPLEVEL_GITREPOS='6'
```

Alter for higher compression levels

```
# regular zstd compression level control for levels 1-19
ZSTD_COMPLEVEL_WEBROOT='9'
ZSTD_COMPLEVEL_MAIL='9'
ZSTD_COMPLEVEL_MYSQL='9'
ZSTD_COMPLEVEL_LOGS='9'
ZSTD_COMPLEVEL_SSL='9'
ZSTD_COMPLEVEL_GITREPOS='9'
```

```
/root/tools/cpanel-backup.sh backup cpuser1                                                                             

--------------------------------------------------------
cPanel/WHM backup script 0.9
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
  "parked_domains": [],
  "addon_domains": {},
  "main_domain": "domain1.com",
  "sub_domains": []
}

main_domain=domain1.com

sub_domains:



parked_domains:



cpuser_domainlist=domain1.com

--------------------------------------------------------
domain mapping saved:
/home/backup-accounts/cpuser1/domain-map-cpuser1-060719-044352.txt
--------------------------------------------------------

--------------------------------------------------------
backup cpuser1 domain related DNS zone files
--------------------------------------------------------

backup /var/named/domain1.com.db
cp -af /var/named/domain1.com.db /home/backup-accounts/cpuser1/named/domain1.com-060719-044352.db


--------------------------------------------------------
list cpuser1 cronjobs
--------------------------------------------------------

SHELL="/bin/bash"
#0 0 * * 0 echo "hello world"


--------------------------------------------------------
cpuser1 cronjobs saved:
/home/backup-accounts/cpuser1/cronjobs-cpuser1-060719-044352.txt
--------------------------------------------------------

--------------------------------------------------------
backup cpanel /home/cpuser1/public_html web root
--------------------------------------------------------

/bin/nice -n 12 /usr/bin/ionice -c2 -n7 tar cpf - public_html | zstd -9 -T4 -f --rsyncable > "/home/backup-accounts/cpuser1/public_html-cpuser1-060719-044352.tar.zst"

backup start time: Sat Jul  6 04:43:52 PDT 2019
backup end time: Sat Jul  6 04:43:55 PDT 2019
backup time: 2.662
zstd normal compression mode: enabled
compression level: 9
compression ratio: 4.634 (30092102 / 6492990)
compression ratio: 0.215 (6492990 / 30092102)
slack notification sending
ok

--------------------------------------------------------
backup cpanel /home/cpuser1/mail directory

/bin/nice -n 12 /usr/bin/ionice -c2 -n7 tar cpf - mail | zstd -9 -T4 -f --rsyncable > "/home/backup-accounts/cpuser1/mail-cpuser1-060719-044352.tar.zst"

backup start time: Sat Jul  6 04:43:56 PDT 2019
backup end time: Sat Jul  6 04:43:56 PDT 2019
backup time: 0.116
zstd normal compression mode: enabled
compression level: 9
compression ratio: 236.354 (86033 / 364)
compression ratio: 0.004 (364 / 86033)
slack notification sending
ok

--------------------------------------------------------
backup cpanel /home/cpuser1/logs directory

/bin/nice -n 12 /usr/bin/ionice -c2 -n7 tar cpf - logs | zstd  --fast=10 -T4 -f --rsyncable > "/home/backup-accounts/cpuser1/logs-cpuser1-060719-044352.tar.zst"

backup start time: Sat Jul  6 04:43:56 PDT 2019
backup end time: Sat Jul  6 04:43:59 PDT 2019
backup time: 3.368
zstd normal compression mode: enabled
compression level: fast=10
compression ratio: 1.000 (172750859 / 172749410)
compression ratio: 0.999 (172749410 / 172750859)
slack notification sending
ok

--------------------------------------------------------
backup cpanel /home/cpuser1/ssl directory

/bin/nice -n 12 /usr/bin/ionice -c2 -n7 tar cpf - ssl | zstd -9 -T4 -f --rsyncable > "/home/backup-accounts/cpuser1/ssl-cpuser1-060719-044352.tar.zst"

backup start time: Sat Jul  6 04:44:00 PDT 2019
backup end time: Sat Jul  6 04:44:00 PDT 2019
backup time: 0.048
zstd normal compression mode: enabled
compression level: 9
compression ratio: 7.537 (26116 / 3465)
compression ratio: 0.132 (3465 / 26116)
slack notification sending
ok

--------------------------------------------------------
backup cpanel /home/cpuser1/repositories directory

/bin/nice -n 12 /usr/bin/ionice -c2 -n7 tar cpf - repositories | zstd -9 -T4 -f --rsyncable > "/home/backup-accounts/cpuser1/repositories-cpuser1-060719-044352.tar.zst"

backup start time: Sat Jul  6 04:44:00 PDT 2019
backup end time: Sat Jul  6 04:44:00 PDT 2019
backup time: 0.440
zstd normal compression mode: enabled
compression level: 9
compression ratio: 1.630 (937259 / 574768)
compression ratio: 0.613 (574768 / 937259)
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

mysql username: cpuser1

Grants for cpuser1@localhost
GRANT USAGE ON *.* TO 'cpuser1'@'localhost' IDENTIFIED BY PASSWORD '*9EFEC505F817F592094619681AD824757DA660C6'
GRANT ALL PRIVILEGES ON `cpuser1\\_db1`.* TO 'cpuser1'@'localhost'
GRANT ALL PRIVILEGES ON `cpuser1\\_db2`.* TO 'cpuser1'@'localhost'
GRANT ALL PRIVILEGES ON `cpuser1\\_db3`.* TO 'cpuser1'@'localhost'

replay commands via SSH command line

mysql -e "GRANT USAGE ON *.* TO 'cpuser1'@'localhost' IDENTIFIED BY PASSWORD '*9EFEC505F817F592094619681AD824757DA660C6'" mysql
mysql -e "GRANT ALL PRIVILEGES ON cpuser1\_db1.* TO 'cpuser1'@'localhost'" mysql
mysql -e "GRANT ALL PRIVILEGES ON cpuser1\_db2.* TO 'cpuser1'@'localhost'" mysql
mysql -e "GRANT ALL PRIVILEGES ON cpuser1\_db3.* TO 'cpuser1'@'localhost'" mysql
mysql username: cpuser1_dbuser

Grants for cpuser1_dbuser@localhost
GRANT USAGE ON *.* TO 'cpuser1_dbuser'@'localhost' IDENTIFIED BY PASSWORD '*5DA619872CE4458DC6B0230DC365CD16B4280AC4'
GRANT ALL PRIVILEGES ON `cpuser1\\_db3`.* TO 'cpuser1_dbuser'@'localhost'
GRANT ALL PRIVILEGES ON `cpuser1\\_db2`.* TO 'cpuser1_dbuser'@'localhost'

replay commands via SSH command line

mysql -e "GRANT USAGE ON *.* TO 'cpuser1_dbuser'@'localhost' IDENTIFIED BY PASSWORD '*5DA619872CE4458DC6B0230DC365CD16B4280AC4'" mysql
mysql -e "GRANT ALL PRIVILEGES ON cpuser1\_db3.* TO 'cpuser1_dbuser'@'localhost'" mysql
mysql -e "GRANT ALL PRIVILEGES ON cpuser1\_db2.* TO 'cpuser1_dbuser'@'localhost'" mysql

backup cpanel user's mysql databases

/bin/nice -n 12 /usr/bin/ionice -c2 -n7 mysqldump --default-character-set=utf8 -Q -K --max_allowed_packet=256M --net_buffer_length=65536 --routines --events --triggers --hex-blob "cpuser1_db1" | zstd -9 -T4 -f --rsyncable > "/home/backup-accounts/cpuser1/mysqlbackup-cpuser1_db1-060719-044352.sql.zst"

backup start time: Sat Jul  6 04:44:01 PDT 2019
backup end time: Sat Jul  6 04:45:01 PDT 2019
backup time: 59.266
zstd normal compression mode: enabled
compression level: 9
compression ratio: 4.766 (1175332217 / 246595276)
compression ratio: 0.209 (246595276 / 1175332217)
slack notification sending
ok

/bin/nice -n 12 /usr/bin/ionice -c2 -n7 mysqldump --default-character-set=utf8 -Q -K --max_allowed_packet=256M --net_buffer_length=65536 --routines --events --triggers --hex-blob "cpuser1_db2" | zstd -9 -T4 -f --rsyncable > "/home/backup-accounts/cpuser1/mysqlbackup-cpuser1_db2-060719-044352.sql.zst"

backup start time: Sat Jul  6 04:45:02 PDT 2019
backup end time: Sat Jul  6 04:45:13 PDT 2019
backup time: 10.664
zstd normal compression mode: enabled
compression level: 9
compression ratio: 11.162 (187971591 / 16839819)
compression ratio: 0.089 (16839819 / 187971591)
slack notification sending
ok

/bin/nice -n 12 /usr/bin/ionice -c2 -n7 mysqldump --default-character-set=utf8 -Q -K --max_allowed_packet=256M --net_buffer_length=65536 --routines --events --triggers --hex-blob "cpuser1_db3" | zstd -9 -T4 -f --rsyncable > "/home/backup-accounts/cpuser1/mysqlbackup-cpuser1_db3-060719-044352.sql.zst"

backup start time: Sat Jul  6 04:45:13 PDT 2019
backup end time: Sat Jul  6 04:45:16 PDT 2019
backup time: 2.434
zstd normal compression mode: enabled
compression level: 9
compression ratio: 28.375 (77327310 / 2725172)
compression ratio: 0.035 (2725172 / 77327310)
slack notification sending
ok

--------------------------------------------------------
List cpuser1 backups at /home/backup-accounts/cpuser1
--------------------------------------------------------

+-- [489M]  cpmove-cpuser1.tar.gz
+-- [  49]  cronjobs-cpuser1-060719-044352.txt
+-- [ 103]  domain-map-cpuser1-060719-044352.json
+-- [ 338]  domain-map-cpuser1-060719-044352.txt
+-- [165M]  logs-cpuser1-060719-044352.tar.zst
+-- [ 364]  mail-cpuser1-060719-044352.tar.zst
+-- [262M]  mysqlbackup-cpuser1_db1-060719-040359.sql.zst
+-- [235M]  mysqlbackup-cpuser1_db1-060719-044352.sql.zst
+-- [ 21M]  mysqlbackup-cpuser1_db2-060719-040359.sql.zst
+-- [ 16M]  mysqlbackup-cpuser1_db2-060719-044352.sql.zst
+-- [2.9M]  mysqlbackup-cpuser1_db3-060719-040359.sql.zst
+-- [2.6M]  mysqlbackup-cpuser1_db3-060719-044352.sql.zst
+-- [ 302]  mysqlgrants-060719-044352.log
+-- [ 682]  mysqlgrants-ssh-060719-044352.log
+-- [4.0K]  named
|   +-- [1.3K]  domain1.com-060719-040359.db
|   +-- [1.3K]  domain1.com-060719-044352.db
+-- [6.2M]  public_html-cpuser1-060719-044352.tar.zst
+-- [561K]  repositories-cpuser1-060719-044352.tar.zst
+-- [3.4K]  ssl-cpuser1-060719-044352.tar.zst


--------------------------------------------------------
pidstat stats saved at:
/home/backup-accounts/logs/backup_pidstat_stats_060719-044351.log.zst
--------------------------------------------------------
read command:
zstdcat /home/backup-accounts/logs/backup_pidstat_stats_060719-044351.log.zst
--------------------------------------------------------
sar usage stats saved at:
/home/backup-accounts/logs/backup_sar_stats_060719-044351
--------------------------------------------------------
read via sar command:

cpu load avg

sar -q -f /home/backup-accounts/logs/backup_sar_stats_060719-044351 | sed -e "s|$(hostname)|hostname|g"

04:43:52 AM   runq-sz  plist-sz   ldavg-1   ldavg-5  ldavg-15
Average:            3       225      1.81      1.01      0.71

1min 5min 15min min:
0.14 0.61 0.57
1min 5min 15min avg:
1.81 1.01 0.71
1min 5min 15min max:
3.50 1.54 0.90
1min 5min 15min 95%:
3.46 1.49 0.88

cpu utilisation

sar -u -f /home/backup-accounts/logs/backup_sar_stats_060719-044351 | sed -e "s|$(hostname)|hostname|g"

04:43:52 AM     CPU     %user     %nice   %system   %iowait    %steal     %idle
Average:        all     45.13      6.64      6.72     12.87      0.64     27.99

%user %nice %system $iowait %steal %idle min:
3.03 0.00 3.30 0.00 0.00 2.26
%user %nice %system $iowait %steal %idle avg:
45.06 6.64 6.72 12.91 0.62 28.06
%user %nice %system $iowait %steal %idle max:
85.71 10.80 14.43 68.61 8.60 85.86
%user %nice %system $iowait %steal %idle 95%:
75.50 9.85 11.65 34.01 3.13 62.05

memory usage

sar -r -f /home/backup-accounts/logs/backup_sar_stats_060719-044351 | sed -e "s|$(hostname)|hostname|g"

04:43:52 AM kbmemfree kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit
Average:        98038    921498     90.38      2127     71063   2171309    104.99

kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit min:
69576.00 816800.00 80.11 124.00 5356.00 2004872.00 96.94
kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit avg:
98038.26 921497.74 90.38 2127.39 71062.87 2171309.46 104.99
kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit max:
202736.00 949960.00 93.18 16552.00 117940.00 2263740.00 109.46
kbmemfree kbmemused %memused kbbuffers kbcached kbcommit %commit 95%:
129747.20 944803.20 92.67 9864.00 107130.40 2244313.60 108.52

disk I/O usage

sar -d -f /home/backup-accounts/logs/backup_sar_stats_060719-044351 | sed -e "s|$(hostname)|hostname|g"

04:43:52 AM       DEV       tps  rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz     await     svctm     %util
Average:     dev252-0   1255.81  42023.22  14029.28     44.63      1.60      1.28      0.42     52.57

tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util min:
174.75 174.75 4040.40 95.05 8.02 0.07 0.18 0.17
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util avg:
1259.52 1259.52 42006.75 13934.29 92.17 1.60 1.83 0.60
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util max:
4963.92 4963.92 133204.04 113446.46 419.44 21.11 21.30 1.93
tps rd_sec/s wr_sec/s avgrq-sz avgqu-sz await svctm %util 95%:
2163.72 2163.72 83236.64 78845.28 322.44 5.45 4.77 1.62

--------------------------------------------------------
/root/tools/cpanel-backup.sh run log saved at:
/home/backup-accounts/logs/cpanel-backup-060719-044351.log.zst
--------------------------------------------------------
read command:
zstdcat /home/backup-accounts/logs/cpanel-backup-060719-044351.log.zst
--------------------------------------------------------

backup completed
--------------------------------------------------------
total cpanel backup time: 87.123 seconds

slack notification sending
ok
--------------------------------------------------------
```