# Загрузка Linux
## Попасть в систему без пароля.
### Первый способ `init=/bin/sh`

Во время загрузки GRUB меню жамем `e` и переходим к редактированию параметров загрузки.

В конце строки, начинающуюся с `linux16` дописываем `init=/bin/sh` и жмем `сtrl-x`.

![#1-1-1](https://user-images.githubusercontent.com/114483769/216231455-bafb8dd8-8709-40f1-8584-08547bfd6486.jpeg)

Перемонтируем файловую систему с правами записи: 
```
mount -rw -o remount /
```

После чего проверяем выполнив команду 

```
mount | grep root
``` 

или создав файл.

![#1-1-2](https://user-images.githubusercontent.com/114483769/216231498-2252f912-a47c-4ef9-8f32-69b732a91e5d.jpeg)

### Второй способ `rd.break`

Во время загрузки GRUB меню жамем `e` и переходим к редактированию параметров загрузки.

В конце строки, начинающуюся с `linux16` дописываем `rd.break` и жмем `сtrl-x` (проваливаемся в emergency mode).

Перемонтируем корневую файловую систему с правами записи
```
mount -rw -o remount /sysroot
```

Выполняем операцию изменения корневого каталога 
```
chroot /sysroot
```

При помощи команды `passwd root` пароль для `root`.

Обновляем контекст SELinux 
```
touch /.autorelabel
```
![#1-1](https://user-images.githubusercontent.com/114483769/216231575-6e5a0f46-486a-455e-84ff-a0ff6fdd8f70.jpeg)

Перегружаем машину и входим с новым паролем.

### Третий способ `rw init=/sysroot/bin/sh`

Во время загрузки GRUB меню жамем `e` и переходим к редактированию параметров загрузки.

В конце строки, начинающуюся с `linux16` заменяем `ro` на `rw init=/sysroot/bin/sh` и жмем `сtrl-x` .

Дальше как во втором примере с той разницей, что файловая система сразу монтируется в режиме записи. 

## Установить систему с LVM, после чего переименовать VG

Проверяем исходное слостояние командой `vgs` и меняем имя VG командой 
```
vgrename centos OtusHW7
```
![#1-2](https://user-images.githubusercontent.com/114483769/216231613-46ae9e65-26a7-45e5-bd62-f345ad4f61fc.jpeg)

Редактируем файлы: `/etc/fstab, /etc/default/grub, /boot/grub2/grub.cfg.` Меняе везде название старой VG `centos` на новое `OtusHW7`.

Пересобираем `initrd image` с новыми параметрами командой 
```
mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
```

Перезагружаемся и проверяем результат.

![#1-3](https://user-images.githubusercontent.com/114483769/216231642-a00bd968-9bb8-4de0-ad57-cf65246f71c5.jpeg)

## Добавить модуль в initrd

В каталог, где хранятся скрипты модулей `/usr/lib/dracut/modules.d/` добавляем тестовый каталог `01test` и в него помещаем два срипта:

`module-setup.sh` - устанавливает модуль и вызывает скрипт `test.sh`

```
#!/bin/bash

check() {
    return 0
}

depends() {
    return 0
}

install() {
    inst_hook cleanup 00 "${moddir}/test.sh"
}
```

`test.sh` - вызываемый скрипт, который выводит сообщение и делает паузу вывода на 10 секунд

```
#!/bin/bash

exec 0<>/dev/console 1<>/dev/console 2<>/dev/console
cat <<'msgend'
Hello! You are in dracut module!
 ___________________
< Hello OTUS 2023!!! >
 -------------------
   \
    \
        .--.
       |o_o |
       |:_/ |
      //   \ \
     (|     | )
    /'\_   _/`\
    \___)=(___/
msgend
sleep 10
echo " continuing...."
```

Пересобираем `initrd image` с новыми параметрами командой 
```
mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
```
Перегружаемся.

Во время загрузки GRUB меню жамем `e` и переходим к редактированию параметров загрузки.

В строке, начинающуюся с `linux16` убираем параметры `rghb` и `quiet` и жмем `сtrl-x` .

Смотрим вывод

![#2-1](https://user-images.githubusercontent.com/114483769/216231726-2a8a4811-1a37-4061-87a9-e087bc6e3530.jpeg)

