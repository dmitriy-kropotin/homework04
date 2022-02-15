#  Домашняя работа по ZFS

1. Создаем виртуальную машину `vagrant up`. Все необходимые компоненты, в том числе модуль ядра zfs, автоматически устанавливаются скриптом zfs-mod.sh

Результат:
```
........
zfs04: Installed:
zfs04:   zfs.x86_64 0:0.8.5-1.el7
........

```
2. Захожу на виртуальную машину `vagrant ssh`
3. Проверяю zfs `zfs version`
```
lzfs-0.8.5-1
zfs-kmod-0.8.5-1
```
4. Проверяю, все диски на месте `lsblk`
```
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   40G  0 disk
`-sda1   8:1    0   40G  0 part /
sdb      8:16   0  512M  0 disk
sdc      8:32   0  512M  0 disk
sdd      8:48   0  512M  0 disk
sde      8:64   0  512M  0 disk
sdf      8:80   0  512M  0 disk
sdg      8:96   0  512M  0 disk
sdh      8:112  0  512M  0 disk
sdi      8:128  0  512M  0 disk
```
5. Теперь можно заходить под su`sudo -i` , и выполнять домашнее задание 
6. Создаю pool,
```
[root@zfs04 ~]# zpool create otus1 mirror /dev/sd{b,c}
[root@zfs04 ~]# zpool create otus2 mirror /dev/sd{d,e}
[root@zfs04 ~]# zpool create otus3 mirror /dev/sd{f,g}
[root@zfs04 ~]# zpool create otus3 mirror /dev/sd{h,i}
cannot create 'otus3': pool already exists
[root@zfs04 ~]# zpool create otus4 mirror /dev/sd{h,i}
```
7. Проверяю пулы командой `zpool list`
```
[root@zfs04 ~]# zpool list
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
otus1   480M  91.5K   480M        -         -     0%     0%  1.00x    ONLINE  -
otus2   480M  91.5K   480M        -         -     0%     0%  1.00x    ONLINE  -
otus3   480M   100K   480M        -         -     0%     0%  1.00x    ONLINE  -
otus4   480M  91.5K   480M        -         -     0%     0%  1.00x    ONLINE  -
```
8. Командой `zpool status`
```
[root@zfs04 ~]# zpool status
  pool: otus1
 state: ONLINE
  scan: none requested
config:

	NAME        STATE     READ WRITE CKSUM
	otus1       ONLINE       0     0     0
	  mirror-0  ONLINE       0     0     0
	    sdb     ONLINE       0     0     0
	    sdc     ONLINE       0     0     0

errors: No known data errors

  pool: otus2
 state: ONLINE
  scan: none requested
config:

	NAME        STATE     READ WRITE CKSUM
	otus2       ONLINE       0     0     0
	  mirror-0  ONLINE       0     0     0
	    sdd     ONLINE       0     0     0
	    sde     ONLINE       0     0     0

errors: No known data errors

  pool: otus3
 state: ONLINE
  scan: none requested
config:

	NAME        STATE     READ WRITE CKSUM
	otus3       ONLINE       0     0     0
	  mirror-0  ONLINE       0     0     0
	    sdf     ONLINE       0     0     0
	    sdg     ONLINE       0     0     0

errors: No known data errors

  pool: otus4
 state: ONLINE
  scan: none requested
config:

	NAME        STATE     READ WRITE CKSUM
	otus4       ONLINE       0     0     0
	  mirror-0  ONLINE       0     0     0
	    sdh     ONLINE       0     0     0
	    sdi     ONLINE       0     0     0

errors: No known data errors
```

9. Так же интересен вывод команды `df -h`. Пулы уже примонтированы, и готовы к использованию
```
[root@zfs04 ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        489M     0  489M   0% /dev
tmpfs           496M     0  496M   0% /dev/shm
tmpfs           496M  6.8M  489M   2% /run
tmpfs           496M     0  496M   0% /sys/fs/cgroup
/dev/sda1        40G  3.2G   37G   8% /
tmpfs           100M     0  100M   0% /run/user/1000
otus1           352M  128K  352M   1% /otus1
otus2           352M  128K  352M   1% /otus2
otus3           352M  128K  352M   1% /otus3
otus4           352M  128K  352M   1% /otus4
```
10. Включаем сжатие
```
[root@zfs04 ~]# zfs set compression=lzjb otus1
[root@zfs04 ~]# zfs set compression=lz4 otus2
[root@zfs04 ~]# zfs set compression=gzip-9 otus3
[root@zfs04 ~]# zfs set compression=zle otus4
```
11. Проверяю
```
[root@zfs04 ~]# zfs get all | grep comp
otus1  compressratio         1.00x                  -
otus1  compression           lzjb                   local
otus1  refcompressratio      1.00x                  -
otus2  compressratio         1.00x                  -
otus2  compression           lz4                    local
otus2  refcompressratio      1.00x                  -
otus3  compressratio         1.00x                  -
otus3  compression           gzip-9                 local
otus3  refcompressratio      1.00x                  -
otus4  compressratio         1.00x                  -
otus4  compression           zle                    local
otus4  refcompressratio      1.00x                  -
```
12. Скачиваю на каждый пул один и тот же файл, для проверки сжатия `for i in {1..4}; do wget -P /otus$i https://gutenberg.org/cache/epub/2600/pg2600.converter.log; done`
```
[root@zfs04 ~]# for i in {1..4}; do wget -P /otus$i https://gutenberg.org/cache/epub/2600/pg2600.converter.log; done
--2022-02-13 18:41:15--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log
Resolving gutenberg.org (gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47
Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 40784369 (39M) [text/plain]
Saving to: '/otus1/pg2600.converter.log'

100%[================================================================================================================================================================>] 40,784,369  3.25MB/s   in 13s

2022-02-13 18:41:30 (2.98 MB/s) - '/otus1/pg2600.converter.log' saved [40784369/40784369]

--2022-02-13 18:41:30--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log
Resolving gutenberg.org (gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47
Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 40784369 (39M) [text/plain]
Saving to: '/otus2/pg2600.converter.log'

100%[================================================================================================================================================================>] 40,784,369  2.93MB/s   in 14s

2022-02-13 18:41:45 (2.73 MB/s) - '/otus2/pg2600.converter.log' saved [40784369/40784369]

--2022-02-13 18:41:45--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log
Resolving gutenberg.org (gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47
Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 40784369 (39M) [text/plain]
Saving to: '/otus3/pg2600.converter.log'

100%[================================================================================================================================================================>] 40,784,369  2.14MB/s   in 16s

2022-02-13 18:42:02 (2.36 MB/s) - '/otus3/pg2600.converter.log' saved [40784369/40784369]

--2022-02-13 18:42:02--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log
Resolving gutenberg.org (gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47
Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 40784369 (39M) [text/plain]
Saving to: '/otus4/pg2600.converter.log'

100%[================================================================================================================================================================>] 40,784,369  3.82MB/s   in 11s

2022-02-13 18:42:14 (3.49 MB/s) - '/otus4/pg2600.converter.log' saved [40784369/40784369]

```
13. Проверяю `ls -l /otus*`
```
root@zfs04 ~]# ls -l /otus*
/otus1:
total 22015
-rw-r--r--. 1 root root 40784369 Feb  2 09:01 pg2600.converter.log

/otus2:
total 17970
-rw-r--r--. 1 root root 40784369 Feb  2 09:01 pg2600.converter.log

/otus3:
total 10948
-rw-r--r--. 1 root root 40784369 Feb  2 09:01 pg2600.converter.log

/otus4:
total 39856
-rw-r--r--. 1 root root 40784369 Feb  2 09:01 pg2600.converter.log
```
14. Видно, что максимальное сжатие у gzip-9, но на него файл записывался дольше всех (16 секунд)
15. Так же посмотрю место и сжатие командами `zfs list` и `zfs get all | grep comp | grep -v ref`
```
[root@zfs04 ~]# zfs list
NAME    USED  AVAIL     REFER  MOUNTPOINT
otus1  21.6M   330M     21.5M  /otus1
otus2  17.6M   334M     17.6M  /otus2
otus3  10.8M   341M     10.7M  /otus3
otus4  39.0M   313M     38.9M  /otus4
[root@zfs04 ~]# zfs get all | grep comp | grep -v ref
otus1  compressratio         1.81x                  -
otus1  compression           lzjb                   local
otus2  compressratio         2.22x                  -
otus2  compression           lz4                    local
otus3  compressratio         3.64x                  -
otus3  compression           gzip-9                 local
otus4  compressratio         1.00x                  -
otus4  compression           zle                    local
```
16. Еще я хочу проверить как работает dedup. Для этого очищаю otus4 и отключаю на нем сжатие
```
[root@zfs04 ~]# rm -rf /otus4/*
[root@zfs04 ~]# ls -la /otus4/
total 5
drwxr-xr-x.  2 root root    2 Feb 13 19:10 .
dr-xr-xr-x. 21 root root 4096 Feb 13 18:23 ..
[root@zfs04 ~]# zfs set compression=off otus4
```
17. Включаю на otus4 dedup `zfs set dedup=on otus4` и проверяю
```
[root@zfs04 ~]# zfs get all | grep dedup
otus1  dedup                 off                    default
otus2  dedup                 off                    default
otus3  dedup                 off                    default
otus4  dedup                 on                     local
```
18. Копирую один и тот же файл дважды, под разными именами 
```
[root@zfs04 ~]# cp /otus3/pg2600.converter.log /otus4/pg2600-log.log
[root@zfs04 ~]# cp /otus3/pg2600.converter.log /otus4/pg2600-log2.log
```

20. Проверяю `zpool list`
```
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
otus1   480M  21.6M   458M        -         -     0%     4%  1.00x    ONLINE  -
otus2   480M  17.7M   462M        -         -     0%     3%  1.00x    ONLINE  -
otus3   480M  10.9M   469M        -         -     0%     2%  1.00x    ONLINE  -
otus4   480M  39.8M   440M        -         -     1%     8%  1.97x    ONLINE  -

[root@zfs04 ~]# ll /otus4/
total 79926
-rw-r--r--. 1 root root 40784369 Feb 13 19:11 pg2600-log.log
-rw-r--r--. 1 root root 40784369 Feb 13 19:12 pg2600-log2.log
```

21. Видно, файлы размером 79 мегабайт, занимают 40 мегабайт файловой системы!
22. Для следующего задания необходимо скачать файл `wget -O archive.tar.gz --no-check-certificate 'https://drive.google.com/u/0/uc?id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg&export=download'`
```
...
Saving to: 'archive.tar.gz'

100%[======================================>] 7,275,140   10.2MB/s   in 0.7s

2022-02-14 17:07:01 (10.2 MB/s) - 'archive.tar.gz' saved [7275140/7275140]

[root@zfs04 ~]# ll
total 7124
-rw-------. 1 root root    5570 Apr 30  2020 anaconda-ks.cfg
-rw-r--r--. 1 root root 7275140 Feb 14 17:07 archive.tar.gz
-rw-------. 1 root root    5300 Apr 30  2020 original-ks.cfg
```
23. Распаковка архива `tar -xzvf archive.tar.gz`
```
[root@zfs04 ~]# tar -xzvf archive.tar.gz
zpoolexport/
zpoolexport/filea
zpoolexport/fileb
[root@zfs04 ~]# ll
total 7124
-rw-------. 1 root root    5570 Apr 30  2020 anaconda-ks.cfg
-rw-r--r--. 1 root root 7275140 Feb 14 17:07 archive.tar.gz
-rw-------. 1 root root    5300 Apr 30  2020 original-ks.cfg
drwxr-xr-x. 2 root root      32 May 15  2020 zpoolexport
```
24. Проверяю снап `zpool import -d zpoolexport/`
```
[root@zfs04 ~]# zpool import -d zpoolexport/
   pool: otus
     id: 6554193320433390805
  state: ONLINE
 action: The pool can be imported using its name or numeric identifier.
 config:

	otus                         ONLINE
	  mirror-0                   ONLINE
	    /root/zpoolexport/filea  ONLINE
	    /root/zpoolexport/fileb  ONLINE
```
25. Делаю импорт пула из снапа. `zpool import -d zpoolexport/ otus` , и проверяю `zpool status`	
```
[root@zfs04 ~]# zpool status otus
  pool: otus
 state: ONLINE
  scan: none requested
config:

        NAME                         STATE     READ WRITE CKSUM
        otus                         ONLINE       0     0     0
          mirror-0                   ONLINE       0     0     0
            /root/zpoolexport/filea  ONLINE       0     0     0
            /root/zpoolexport/fileb  ONLINE       0     0     0

errors: No known data errors
```
26. Просмотр параметров пула `zpool get all otus` и файловой системы  `zfs get all otus`
```
[root@zfs04 ~]# zpool get all otus
NAME  PROPERTY                       VALUE                          SOURCE
otus  size                           480M                           -
otus  capacity                       0%                             -
otus  altroot                        -                              default
otus  health                         ONLINE                         -
otus  guid                           6554193320433390805            -
otus  version                        -                              default
otus  bootfs                         -                              default
otus  delegation                     on                             default
otus  autoreplace                    off                            default
otus  cachefile                      -                              default
otus  failmode                       wait                           default
otus  listsnapshots                  off                            default
otus  autoexpand                     off                            default
otus  dedupditto                     0                              default
otus  dedupratio                     1.00x                          -
otus  free                           478M                           -
otus  allocated                      2.09M                          -
otus  readonly                       off                            -
otus  ashift                         0                              default
otus  comment                        -                              default
otus  expandsize                     -                              -
otus  freeing                        0                              -
otus  fragmentation                  0%                             -
otus  leaked                         0                              -
otus  multihost                      off                            default
otus  checkpoint                     -                              -
otus  load_guid                      16766692604483253338           -
otus  autotrim                       off                            default
otus  feature@async_destroy          enabled                        local
otus  feature@empty_bpobj            active                         local
otus  feature@lz4_compress           active                         local
otus  feature@multi_vdev_crash_dump  enabled                        local
otus  feature@spacemap_histogram     active                         local
otus  feature@enabled_txg            active                         local
otus  feature@hole_birth             active                         local
otus  feature@extensible_dataset     active                         local
otus  feature@embedded_data          active                         local
otus  feature@bookmarks              enabled                        local
otus  feature@filesystem_limits      enabled                        local
otus  feature@large_blocks           enabled                        local
otus  feature@large_dnode            enabled                        local
otus  feature@sha512                 enabled                        local
otus  feature@skein                  enabled                        local
otus  feature@edonr                  enabled                        local
otus  feature@userobj_accounting     active                         local
otus  feature@encryption             enabled                        local
otus  feature@project_quota          active                         local
otus  feature@device_removal         enabled                        local
otus  feature@obsolete_counts        enabled                        local
otus  feature@zpool_checkpoint       enabled                        local
otus  feature@spacemap_v2            active                         local
otus  feature@allocation_classes     enabled                        local
otus  feature@resilver_defer         enabled                        local
otus  feature@bookmark_v2            enabled                        local
[root@zfs04 ~]# zfs get all otus
NAME  PROPERTY              VALUE                  SOURCE
otus  type                  filesystem             -
otus  creation              Fri May 15  4:00 2020  -
otus  used                  2.04M                  -
otus  available             350M                   -
otus  referenced            24K                    -
otus  compressratio         1.00x                  -
otus  mounted               yes                    -
otus  quota                 none                   default
otus  reservation           none                   default
otus  recordsize            128K                   local
otus  mountpoint            /otus                  default
otus  sharenfs              off                    default
otus  checksum              sha256                 local
otus  compression           zle                    local
otus  atime                 on                     default
otus  devices               on                     default
otus  exec                  on                     default
otus  setuid                on                     default
otus  readonly              off                    default
otus  zoned                 off                    default
otus  snapdir               hidden                 default
otus  aclinherit            restricted             default
otus  createtxg             1                      -
otus  canmount              on                     default
otus  xattr                 on                     default
otus  copies                1                      default
otus  version               5                      -
otus  utf8only              off                    -
otus  normalization         none                   -
otus  casesensitivity       sensitive              -
otus  vscan                 off                    default
otus  nbmand                off                    default
otus  sharesmb              off                    default
otus  refquota              none                   default
otus  refreservation        none                   default
otus  guid                  14592242904030363272   -
otus  primarycache          all                    default
otus  secondarycache        all                    default
otus  usedbysnapshots       0B                     -
otus  usedbydataset         24K                    -
otus  usedbychildren        2.01M                  -
otus  usedbyrefreservation  0B                     -
otus  logbias               latency                default
otus  objsetid              54                     -
otus  dedup                 off                    default
otus  mlslabel              none                   default
otus  sync                  standard               default
otus  dnodesize             legacy                 default
otus  refcompressratio      1.00x                  -
otus  written               24K                    -
otus  logicalused           1020K                  -
otus  logicalreferenced     12K                    -
otus  volmode               default                default
otus  filesystem_limit      none                   default
otus  snapshot_limit        none                   default
otus  filesystem_count      none                   default
otus  snapshot_count        none                   default
otus  snapdev               hidden                 default
otus  acltype               off                    default
otus  context               none                   default
otus  fscontext             none                   default
otus  defcontext            none                   default
otus  rootcontext           none                   default
otus  relatime              off                    default
otus  redundant_metadata    all                    default
otus  overlay               off                    default
otus  encryption            off                    default
otus  keylocation           none                   default
otus  keyformat             none                   default
otus  pbkdf2iters           0                      default
otus  special_small_blocks  0                      default
```
27. С помощью `grep` посмотрю конкретные параметры, например дедупликацию
```
[root@zfs04 ~]# zpool get all otus | grep dedup
otus  dedupditto                     0                              default
otus  dedupratio                     1.00x                          -
[root@zfs04 ~]# zfs get all otus | grep dedup
otus  dedup                 off                    default
```
28. ... и сжатие 
```
[root@zfs04 ~]# zpool get all otus | grep compr
otus  feature@lz4_compress           active                         local
[root@zfs04 ~]# zfs get all otus | grep compr
otus  compressratio         1.00x                  -
otus  compression           zle                    local
otus  refcompressratio      1.00x                  -
```
29. Для следующего задания качаю файл со снапшотом `wget -O otus_task2.file --no-check-certificate 'https://drive.google.com/u/0/uc?id=1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG&export=download'`
```
...
Length: 5432736 (5.2M) [application/octet-stream]
Saving to: 'otus_task2.file'

100%[===================================================================================================================================================================================================>] 5,432,736   10.0MB/s   in 0.5s

2022-02-15 12:28:54 (10.0 MB/s) - 'otus_task2.file' saved [5432736/5432736]
```
30. Восстановление файловой системы из снапшота `zfs receive otus/test@today < otus_task2.file`
31. Ищу файл с именем "secret_messag" `find /otus/test -name "secret_message"`
```
[root@zfs04 ~]# find /otus/test -name "secret_message"
/otus/test/task1/file_mess/secret_message
```
32. Просмотр содержимого через cat
```
[root@zfs04 ~]# cat /otus/test/task1/file_mess/secret_message
https://github.com/sindresorhus/awesome
```
33. Репозиторий `https://github.com/sindresorhus/awesome`

