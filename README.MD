# Домашнее задание: работа с загрузчиком

## Попадаем в систему без пароля несколькими способами

Будем загружать ОС АЛЬТ.

![Alt text](<Screenshots/1. Loading.png>)
При загрузке нажимаем `e` 
![Alt text](<Screenshots/2. Loading -e.png>)

### Способ 1. init=/bin/sh:

В конце строки, начинающейся с `linux` добавляем init=/bin/bash 
![Alt text](<Screenshots/3. Loading init_bin_bash.png>)
и нажимаем `F10` для загрузки в систему.
![Alt text](<Снимок экрана от 2024-01-21 21-59-31.png>)

Проверяем, что создать файл невозможно, т.к. файловая система примонтирована в режиме только для чтения.
Вводим команду `mount / rw,remount` для перемонтирования системного диска в режиме записи.
Проверяем, что создать файл возможно.
![Alt text](<Screenshots/4. Remount fs rw.png>)

### Способ 2. rd.break:

Теперь в конце строки, которая начинается с `linux`, дописываем `rd.break`.
![Alt text](<Screenshots/6. Loading ALT rd.break.png>)
и нажимаем `F10` для загрузки в систему.
Альт Линукс загрузился в обычном режиме. Пока не понятно, почему.
Попробуем с CentOS:
![Alt text](<Screenshots/7. Loading CentOS rd.break.png>)
Получилось:
![Alt text](<Screenshots/8. Root_pass_changed.png>)
`touch /.autorelabel ` было нужно для из-за того, что в Centos SElinux включен и надо, чтобы при перезагрузке этот самый relable выполнился.
Загружаемся с новым паролем и пробуем стать рутом с новым паролем. Работает.
![Alt text](<Screenshots/9. Test_new_rootpass.png>)

### Способ 3. rw init=/sysroot/bin/sh

В строке, начинающейся с `linux` заменяем `ro` на `rw` `init=/sysroot/bin/sh` и нажимаем сtrl-x для загрузки в систему.
![Alt text](<Screenshots/10. init possible 3.png>)

Загрузились:

![Alt text](Screenshots/11.init_possile3_complete.png)

Поменяем название `VolumeGroup` LVM.
Для этого:
1. Создадим каталог /tmp/test
2. Примонтируем туда /dev/mapper/centos/root
3. Примонтируем mount --bind /dev в /tmp/test/dev, аналогично сделаем то же для /sys и /proc
4. Сделаем chroot в /tmp/test
5. Посмотрим vgs
![Alt text](Screenshots/12.Show_vgs.png)

Меняем название с помощью vgrename centos otus, исправляем /etc/fstab так:

![Alt text](Screenshots/13.fstab.png)

Далее вносим исправления в /etc/default/grub, заменяя `centos`на `otus`.

А вот для того, чтобы исправить /boot/grub2/grub.cfg пришлось примонтироваро /dev/vda1 в /boot, поскольку /boot в моём случае находился на другом разделе.

Пересоздаём initrd image и перезагружаемся. Получилось:

![Alt text](Screenshots/14.rebooted.png)


### Добавляем модуль с помощью dracut

Создаём папку `01test` в `/usr/lib/dracut/modules.d`, а в ней скрипты.

Запускаем `dracut -f -v` и проверяем результат:
![alt text](Screenshots/15.Including_module.png)

Перезагружаемся и видим:

![alt text](Screenshots/16.Tux.png)