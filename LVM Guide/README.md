# Оглавление

[Что такое LVM?](#что-такое-lvm)

[Установка](#установка)

[Начало работы](#начало-работы)

# Объясняю работу с LVM на пальцах, как новенький(правки рекомендуются)
## Что такое LVM?
- *__Logical Volume Manager (LVM) - это очень мощная система управления томами с данными для Linux. Она позволяет создавать поверх физических разделов (или даже не разбитых винчестеров) логические тома, которые в самой системе будут видны как обычные блочные устройства с данными (т.е. как обычные разделы). Основные преимущества LVM в том, что во-первых одну группу логических томов можно создавать поверх любого количества физических разделов, а во-вторых размер логических томов можно легко менять прямо во время работы. Кроме того, LVM поддерживает механизм снапшотов, копирование разделов «на лету» и зеркалирование, подобное RAID-1.(https://help.ubuntu.ru/wiki/lvm)__*
- Проще говоря, *__LVM__* - это технология, которая позволяет нам работать с дисками(физическими томами) устройства и их логическими томами. Чаще всего рядовые пользователи сталкиваются с этим при работе на *__Windows__* в *__diskmgmt.msc__* в случаях, когда у пользователя имеется лишь один диск, и ему необходимо отдельное место, например, для установки дополнительной ОС. Процесс разделения одного физического тома, на несколько логических.
- Есть 3 уровня абстракции:
  - *__PV (Physical Volume)__* — физические тома (это могут быть разделы или целые «не разбитые» диски)
  - *__VG (Volume Group)__* — группа томов (объединяем физические тома *__(PV)__* в группу, создаём единый диск, который будем дальше разбивать так, как нам хочется)
  - *__LV (Logical Volume)__* — логические разделы, собственно раздел нашего нового «единого диска» ака Группы Томов, который мы потом форматируем и используем как обычный раздел, обычного жёсткого диска.
## Установка
- Проверим наличие установленных пакетов LVM командой: `apt list lvm2`. Если все окей, то мы увидим примерно следующее:
```bash
:~$ apt list lvm2
Вывод списка… Готово
lvm2/jammy,now 2.03.11-2.1ubuntu4 amd64 [установлен, автоматически]
```
- Так же можно воспользоваться командой `apt-cache list policy lvm2` или `dpkg -l | grep lvm2`
```bash
:~$ apt-cache policy lvm2
lvm2:
  Установлен: 2.03.11-2.1ubuntu4
  Кандидат:   2.03.11-2.1ubuntu4
  Таблица версий:
 *** 2.03.11-2.1ubuntu4 500
        500 http://ru.archive.ubuntu.com/ubuntu jammy/main amd64 Packages
        100 /var/lib/dpkg/status
```
```bash
:~$ dpkg -l | grep -i lvm2
ii  liblvm2cmd2.03:amd64                  2.03.11-2.1ubuntu4                      amd64        LVM2 command library
ii  lvm2                                  2.03.11-2.1ubuntu4                      amd64        Linux Logical Volume Manager
```
- Если LVM у вас не установлен, то используйте команду `apt-get install lvm2`
## Начало работы
Для начала проверим список уже имеющихся физических и логических томов командой `lsblk`:
```bash
:~$ lsblk
NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0                       7:0    0    62M  1 loop /snap/core20/1587
loop1                       7:1    0  79,9M  1 loop /snap/lxd/22923
loop3                       7:3    0  40,8M  1 loop /snap/snapd/20092
loop4                       7:4    0  63,5M  1 loop /snap/core20/2015
loop5                       7:5    0 111,9M  1 loop /snap/lxd/24322
sda                         8:0    0    50G  0 disk
├─sda1                      8:1    0     1M  0 part
├─sda2                      8:2    0     2G  0 part /boot
└─sda3                      8:3    0    48G  0 part
  └─ubuntu--vg-ubuntu--lv 253:1    0    48G  0 lvm  /
sdb                         8:16   0    50G  0 disk
sr0                        11:0    1   1,4G  0 rom
```
Видим два подключенных физических тома: *__sda__* и *__sdb__*. Заметим, что на ФТ *_sda_* уже имеются разделы, в том числе и логический том, выделенный под операционную систему. Важно заметить, что сама операционная система не занимает отмеченные 48GB, просто раздел, в котором она содержится был расширен до максимально возможного размера. Темы расширения разделов мы коснемся позже.

- Будем работать с *__sdb__*
  - Во-первых, мы можем вообще не делать на диске никаких разделов, а просто инициализировать наше устройство, как физический том.
```bash
:~$ pvcreate /dev/sdb
Physical volume "/dev/sdb" successfully created
```