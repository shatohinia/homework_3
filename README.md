1. Добавил на VM 2 HDD  2G и 5 Gb
2. На каждом диске сделал два раздела с помощью fdisk (`sudo fdisk /dev/sdb` `sudo fdisk /dev/sdc`)
3. Cобрать VG с именем tms-vg из конкретных разделов: `/dev/sdb1` и `/dev/sdc2`
 - `sudo pvcreate /dev/sdb1 /dev/sdc2`
 - `sudo vgcreate tms-vg /dev/sdb1 /dev/sdc2`
- `sudo vgs`
4. Создал LV для data и bkp по 1G на каждый
 - `sudo lvcreate -n data -L 1G tms-vg`
 - `sudo lvcreate -n bkp -L 1G tms-vg`
 - `sudo lvs`
 4.1 Отформатировал тома `data и bkp` в файловые системы `xfs и ext4`
 - `sudo mkfs.xfs /dev/tms-vg/data`
 - `sudo mkfs.ext4 /dev/tms-vg/bkp`
 4.2 Создал точки монтирования (`sudo mkdir -p /opt/data /opt/backups`)
 - `sudo mount /dev/tms-vg/data /opt/data`
 - `sudo mount /dev/tms-vg/bkp /opt/backups`
5. Добавил в volume group `/dev/sdb2`  
 - `sudo pvcreate /dev/sdb2`
 - `sudo vgextend tms-vg /dev/sdb2`
6. Расширил LV data и увеличить размер ФС
 - `sudo lvextend -l +100%FREE /dev/tms-vg/data`
 - `sudo xfs_growfs /opt/data`
 - `df -h /opt/data`
7. Из оставшегося `/dev/sdc2`  собрал VG с названием `reserved`
 - `sudo pvcreate /dev/sdc1`
 - `sudo vgcreate reserved /dev/sdc1`
8. На полученной `VG reserved` создал `lV` с названием `mylv` и размером на всю VG
 - `sudo lvcreate -n mylv -l 100%FREE reserved`
9. Смонтировал `lV reserved` в `/opt/tests`, тип ФС `ext4`
 - `sudo mkfs.ext4 /dev/reserved/mylv`
 - `sudo mkdir -p /opt/tests`
 - `sudo mount /dev/reserved/mylv /opt/tests` 
10. Создать два файла в `/opt/tests`
 - `sudo bash -c 'echo "This is file one" > /opt/tests/file1.txt`
 - `sudo bash -c 'echo "This is file two" > /opt/tests/file2.txt`
11. Отмонтировал LV и проверил файлы
 - `sudo umount /opt/tests`
 - `ls -l /opt/tests`
 11.1 Создал файл `bigdata.txt` и примонтировал обратно
 - `sudo bash -c 'echo "Some big data" > /opt/tests/bigdata.txt`
 - `sudo mount /dev/reserved/mylv /opt/tests`
(`Когда диск не примонтирован файл 'bigdata.txt' виден на корневом диске. Когда мы монтируем '/opt/tests',
 мы видим содержимое диска 'mylv' с файлами 'file1' и 'file2', а файл 'bigdata.txt' оказался скрыт под точкой монтирования
 Он не удалился, он просто недоступен, пока мы снова не выполним umount`)
