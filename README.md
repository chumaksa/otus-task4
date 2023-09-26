# otus-task4

# Определить алгоритм с наилучшим сжатием.

## Определить какие алгоритмы сжатия поддерживает zfs (gzip gzip-N, zle lzjb, lz4);

* создать 4 файловых системы на каждой применить свой алгоритм сжатия;
* Для сжатия использовать либо текстовый файл либо группу файлов:
* Cкачать файл “Война и мир” и расположить на файловой системе wget -O War_and_Peace.txt http://www.gutenberg.org/ebooks/2600.txt.utf-8, либо скачать файл ядра распаковать и расположить на файловой системе.

### Результат:
* список команд которыми получен результат с их выводами;
* вывод команды из которой видно какой из алгоритмов лучше.

### Решение

Для начала посмотрим какие диски у нас есть.
[root@zfs ~]# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   40G  0 disk
└─sda1   8:1    0   40G  0 part /
sdb      8:16   0  512M  0 disk
sdc      8:32   0  512M  0 disk
sdd      8:48   0  512M  0 disk
sde      8:64   0  512M  0 disk
sdf      8:80   0  512M  0 disk
sdg      8:96   0  512M  0 disk
sdh      8:112  0  512M  0 disk
sdi      8:128  0  512M  0 disk

Всего доступно 8 свободных дисков. Разделим диски по парам, из пар соберём raid1 и сделаем 4 файловые системы zfs.
[root@zfs ~]# zpool create zfsdisk1 mirror /dev/sdb /dev/sdc
[root@zfs ~]# zpool create zfsdisk2 mirror /dev/sdd /dev/sde
[root@zfs ~]# zpool create zfsdisk3 mirror /dev/sdf /dev/sdg
[root@zfs ~]# zpool create zfsdisk4 mirror /dev/sdh /dev/sdi

Для проверки посмотрим что содержит наш zpool.
[root@zfs ~]# zpool list
NAME       SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
zfsdisk1   480M  91.5K   480M        -         -     0%     0%  1.00x    ONLINE  -
zfsdisk2   480M  91.5K   480M        -         -     0%     0%  1.00x    ONLINE  -
zfsdisk3   480M  91.5K   480M        -         -     0%     0%  1.00x    ONLINE  -
zfsdisk4   480M  91.5K   480M        -         -     0%     0%  1.00x    ONLINE  -

Для выяснения лучшего алгоритма сжатия для каждой файловой системы zfs добавим свой алгоритм.
[root@zfs ~]# zfs set compression=lzjb zfsdisk1
[root@zfs ~]# zfs set compression=lz4 zfsdisk2
[root@zfs ~]# zfs set compression=gzip-9 zfsdisk3
[root@zfs ~]# zfs set compression=zle zfsdisk4

Проверим факт применения настроек компрессии для каждой файловой системы zfs.
[root@zfs ~]# zfs get compression zfsdisk1
NAME      PROPERTY     VALUE     SOURCE
zfsdisk1  compression  lzjb      local
[root@zfs ~]# zfs get compression zfsdisk2
NAME      PROPERTY     VALUE     SOURCE
zfsdisk2  compression  lz4       local
[root@zfs ~]# zfs get compression zfsdisk3
NAME      PROPERTY     VALUE     SOURCE
zfsdisk3  compression  gzip-9    local
[root@zfs ~]# zfs get compression zfsdisk4
NAME      PROPERTY     VALUE     SOURCE
zfsdisk4  compression  zle       local

Запишем на каждую файловую систему zfs одинаковый размер данных.
[root@zfs ~]# for i in {1..4}; do wget wget -O /zfsdisk$i/War_and_Peace.txt  http://www.gutenberg.org/ebooks/2600.txt.utf-8; done
--2023-09-24 17:29:24--  http://wget/
Resolving wget (wget)... failed: Name or service not known.
wget: unable to resolve host address ‘wget’
--2023-09-24 17:29:24--  http://www.gutenberg.org/ebooks/2600.txt.utf-8
Resolving www.gutenberg.org (www.gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47
Connecting to www.gutenberg.org (www.gutenberg.org)|152.19.134.47|:80... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://www.gutenberg.org/ebooks/2600.txt.utf-8 [following]
--2023-09-24 17:29:24--  https://www.gutenberg.org/ebooks/2600.txt.utf-8
Connecting to www.gutenberg.org (www.gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: http://www.gutenberg.org/cache/epub/2600/pg2600.txt [following]
--2023-09-24 17:29:25--  http://www.gutenberg.org/cache/epub/2600/pg2600.txt
Connecting to www.gutenberg.org (www.gutenberg.org)|152.19.134.47|:80... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://www.gutenberg.org/cache/epub/2600/pg2600.txt [following]
--2023-09-24 17:29:26--  https://www.gutenberg.org/cache/epub/2600/pg2600.txt
Connecting to www.gutenberg.org (www.gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3359630 (3.2M) [text/plain]
Saving to: ‘/zfsdisk1/War_and_Peace.txt’

100%[=====================================================================================================================================================================================================================================>] 3,359,630    833KB/s   in 4.3s

2023-09-24 17:29:31 (764 KB/s) - ‘/zfsdisk1/War_and_Peace.txt’ saved [3359630/3359630]

FINISHED --2023-09-24 17:29:31--
Total wall clock time: 6.9s
Downloaded: 1 files, 3.2M in 4.3s (764 KB/s)
--2023-09-24 17:29:31--  http://wget/
Resolving wget (wget)... failed: Name or service not known.
wget: unable to resolve host address ‘wget’
--2023-09-24 17:29:31--  http://www.gutenberg.org/ebooks/2600.txt.utf-8
Resolving www.gutenberg.org (www.gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47
Connecting to www.gutenberg.org (www.gutenberg.org)|152.19.134.47|:80... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://www.gutenberg.org/ebooks/2600.txt.utf-8 [following]
--2023-09-24 17:29:31--  https://www.gutenberg.org/ebooks/2600.txt.utf-8
Connecting to www.gutenberg.org (www.gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: http://www.gutenberg.org/cache/epub/2600/pg2600.txt [following]
--2023-09-24 17:29:32--  http://www.gutenberg.org/cache/epub/2600/pg2600.txt
Connecting to www.gutenberg.org (www.gutenberg.org)|152.19.134.47|:80... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://www.gutenberg.org/cache/epub/2600/pg2600.txt [following]
--2023-09-24 17:29:33--  https://www.gutenberg.org/cache/epub/2600/pg2600.txt
Connecting to www.gutenberg.org (www.gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3359630 (3.2M) [text/plain]
Saving to: ‘/zfsdisk2/War_and_Peace.txt’

100%[=====================================================================================================================================================================================================================================>] 3,359,630   1.08MB/s   in 3.0s

2023-09-24 17:29:37 (1.08 MB/s) - ‘/zfsdisk2/War_and_Peace.txt’ saved [3359630/3359630]

FINISHED --2023-09-24 17:29:37--
Total wall clock time: 5.8s
Downloaded: 1 files, 3.2M in 3.0s (1.08 MB/s)
--2023-09-24 17:29:37--  http://wget/
Resolving wget (wget)... failed: Name or service not known.
wget: unable to resolve host address ‘wget’
--2023-09-24 17:29:37--  http://www.gutenberg.org/ebooks/2600.txt.utf-8
Resolving www.gutenberg.org (www.gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47
Connecting to www.gutenberg.org (www.gutenberg.org)|152.19.134.47|:80... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://www.gutenberg.org/ebooks/2600.txt.utf-8 [following]
--2023-09-24 17:29:37--  https://www.gutenberg.org/ebooks/2600.txt.utf-8
Connecting to www.gutenberg.org (www.gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: http://www.gutenberg.org/cache/epub/2600/pg2600.txt [following]
--2023-09-24 17:29:38--  http://www.gutenberg.org/cache/epub/2600/pg2600.txt
Connecting to www.gutenberg.org (www.gutenberg.org)|152.19.134.47|:80... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://www.gutenberg.org/cache/epub/2600/pg2600.txt [following]
--2023-09-24 17:29:38--  https://www.gutenberg.org/cache/epub/2600/pg2600.txt
Connecting to www.gutenberg.org (www.gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3359630 (3.2M) [text/plain]
Saving to: ‘/zfsdisk3/War_and_Peace.txt’

100%[=====================================================================================================================================================================================================================================>] 3,359,630   1.05MB/s   in 3.0s

2023-09-24 17:29:43 (1.05 MB/s) - ‘/zfsdisk3/War_and_Peace.txt’ saved [3359630/3359630]

FINISHED --2023-09-24 17:29:43--
Total wall clock time: 5.9s
Downloaded: 1 files, 3.2M in 3.0s (1.05 MB/s)
--2023-09-24 17:29:43--  http://wget/
Resolving wget (wget)... failed: Name or service not known.
wget: unable to resolve host address ‘wget’
--2023-09-24 17:29:43--  http://www.gutenberg.org/ebooks/2600.txt.utf-8
Resolving www.gutenberg.org (www.gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47
Connecting to www.gutenberg.org (www.gutenberg.org)|152.19.134.47|:80... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://www.gutenberg.org/ebooks/2600.txt.utf-8 [following]
--2023-09-24 17:29:43--  https://www.gutenberg.org/ebooks/2600.txt.utf-8
Connecting to www.gutenberg.org (www.gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: http://www.gutenberg.org/cache/epub/2600/pg2600.txt [following]
--2023-09-24 17:29:44--  http://www.gutenberg.org/cache/epub/2600/pg2600.txt
Connecting to www.gutenberg.org (www.gutenberg.org)|152.19.134.47|:80... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://www.gutenberg.org/cache/epub/2600/pg2600.txt [following]
--2023-09-24 17:29:44--  https://www.gutenberg.org/cache/epub/2600/pg2600.txt
Connecting to www.gutenberg.org (www.gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3359630 (3.2M) [text/plain]
Saving to: ‘/zfsdisk4/War_and_Peace.txt’

100%[=====================================================================================================================================================================================================================================>] 3,359,630    981KB/s   in 3.3s

2023-09-24 17:29:49 (981 KB/s) - ‘/zfsdisk4/War_and_Peace.txt’ saved [3359630/3359630]

FINISHED --2023-09-24 17:29:49--
Total wall clock time: 6.2s
Downloaded: 1 files, 3.2M in 3.3s (981 KB/s)

Проверим, сколько места занимает один и тот же файл в разных файловых системах:
[root@zfs ~]# zfs list
NAME       USED  AVAIL     REFER  MOUNTPOINT
zfsdisk1  2.48M   350M     2.41M  /zfsdisk1
zfsdisk2  2.09M   350M     2.02M  /zfsdisk2
zfsdisk3  1.30M   351M     1.23M  /zfsdisk3
zfsdisk4  3.30M   349M     3.23M  /zfsdisk4

Теперь посмотрим степень сжатия:
[root@zfs ~]# zfs get all | grep compressratio | grep -v ref
zfsdisk1  compressratio         1.35x                  -
zfsdisk2  compressratio         1.62x                  -
zfsdisk3  compressratio         2.64x                  -
zfsdisk4  compressratio         1.01x                  -

Видно, что самая высокая степень сжатия имеет файловая система zfsdisk3. Проверим какой алгоритм сжатия применяется на этой файловой системе:
[root@zfs ~]# zfs get compression zfsdisk1
NAME      PROPERTY     VALUE     SOURCE
zfsdisk1  compression  lzjb      local

Из этого можно сдеалть вывод, что для текстовых файлов максимальную степень сжатия можно получить используя алгоритм lzjb.

## Определение настроек пула

* загрузить архив с файлами локально. https://drive.google.com/open?id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg Распаковать.
* с помощью команды zfs import собрать pool ZFS;
* командами zfs определить настройки:
* размер хранилища;
* тип pool;
* значение recordsize;
* какое сжатие используется;
* какая контрольная сумма используется.

### Результат:
* список команд которыми восстановили pool . Желательно с Output команд;
* файл с описанием настроек settings.

### Решение

Скачиваем файл - https://drive.google.com/open?id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg
[root@zfs ~]# wget -O zfs_task1.tar.gz https://drive.google.com/uc?id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg&export=download
[1] 32035
[root@zfs ~]# --2023-09-24 18:10:31--  https://drive.google.com/uc?id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg
Resolving drive.google.com (drive.google.com)... 64.233.163.194, 2a00:1450:4010:c07::c2
Connecting to drive.google.com (drive.google.com)|64.233.163.194|:443... connected.
HTTP request sent, awaiting response... 303 See Other
Location: https://doc-0c-bo-docs.googleusercontent.com/docs/securesc/ha0ro937gcuc7l7deffksulhg5h7mbp1/ni4nq61gklgabsfk54pfjrhrbhndu8im/1695742200000/16189157874053420687/*/1KRBNW33QWqbvbVHa3hLJivOAt60yukkg?uuid=5e3a5ead-b0b0-4999-9e9f-da0066822e4a [following]
Warning: wildcards not supported in HTTP.
--2023-09-24 18:10:35--  https://doc-0c-bo-docs.googleusercontent.com/docs/securesc/ha0ro937gcuc7l7deffksulhg5h7mbp1/ni4nq61gklgabsfk54pfjrhrbhndu8im/1695742200000/16189157874053420687/*/1KRBNW33QWqbvbVHa3hLJivOAt60yukkg?uuid=5e3a5ead-b0b0-4999-9e9f-da0066822e4a
Resolving doc-0c-bo-docs.googleusercontent.com (doc-0c-bo-docs.googleusercontent.com)... 216.58.210.129, 2a00:1450:4026:808::2001
Connecting to doc-0c-bo-docs.googleusercontent.com (doc-0c-bo-docs.googleusercontent.com)|216.58.210.129|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 7275140 (6.9M) [application/x-gzip]
Saving to: ‘zfs_task1.tar.gz’

100%[=====================================================================================================================================================================================================================================>] 7,275,140   2.28MB/s   in 3.0s

2023-09-24 18:10:38 (2.28 MB/s) - ‘zfs_task1.tar.gz’ saved [7275140/7275140]


[1]+  Done                    wget -O zfs_task1.tar.gz https://drive.google.com/uc?id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg

Далее разархивируем его
[root@zfs ~]# tar -xzvf zfs_task1.tar.gz
zpoolexport/
zpoolexport/filea
zpoolexport/fileb

Теперь посмотрим подробнее на получившийся каталог и проверим возможность импортирования файловой системы
 [root@zfs ~]# zpool import -d zpoolexport/
   pool: otus
     id: 6554193320433390805
  state: ONLINE
 action: The pool can be imported using its name or numeric identifier.
 config:

        otus                         ONLINE
          mirror-0                   ONLINE
            /root/zpoolexport/filea  ONLINE
            /root/zpoolexport/fileb  ONLINE
			
Видим, что перед нами pool с именем otus, который представляет из себя зеркало и может быть импортирован в нашу систему.

Сделаем импорт
[root@zfs ~]# zpool import -d zpoolexport/ otus

Проверим успешность импорта
[root@zfs ~]# zpool status otus
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

Определяем объём (размер хранилища)
[root@zfs ~]# zfs get available otus
NAME  PROPERTY   VALUE  SOURCE
otus  available  350M   -

Определяем тип
[root@zfs ~]# zfs get type otus
NAME  PROPERTY  VALUE       SOURCE
otus  type      filesystem  -

Определяем значение recordsize
[root@zfs ~]# zfs get recordsize otus
NAME  PROPERTY    VALUE    SOURCE
otus  recordsize  128K     local

Определяем какое сжатие используется
[root@zfs ~]# zfs get compression otus
NAME  PROPERTY     VALUE     SOURCE
otus  compression  zle       local

Определяем какая контрольная сумма используется
[root@zfs ~]# zfs get checksum otus
NAME  PROPERTY  VALUE      SOURCE
otus  checksum  sha256     local
