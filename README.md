# linux_kernel_recover
Recover operation system after upgrade to new kernel 


После неудачной попытки обновить ядро операционная система перестала грузиться.
Иходные данные:
Linux Mint 19.2, kernel = 5.4.0-70-generic, boot device nvme0n1p3, root device nvme0n1p4
Если подробнее, то получилось это сделать следующим образом:
1. Для каталога /boot оставил слишком мала места (235Mbyte)
2. Накопилось несколько ядер 5.4.0-69-generic 5.4.0-70-generic 5.4.0-71-generic
3. При обновлении с 70 до 71 не заметил как update-initramfs вывалилось с ошибкой и версия ядра осталась на 70
4. Обновил до 72 версии ядра и проигнорировал сообщение об ошибке update-initramfs которая говорила о нехватке свободного места в каталоке /boot
5. Поспешил, не проверил не какой версии ядра загружен удалил вручную версии 69, 70
6. После удаления снова не запустил команду update-initramfs

В результате получил систему которая не могла загрузиться. 
Чтобы вернуть в рабочее состоянии сделал следующее:

1. Из другой операционной системы стоявшей на компьютере создал загрузочную флешку со своим дистрибутивом Mint 20
2. Загрузился с загрузочной флешки
3. Осуществил настройку для chroot

`sudo mount /dev/nvme0n1p4 /mnt`<br>
`sudo mount /dev/nvme0n1p3 /mnt/boot`<br>

`sudo mount -t proc /proc /mnt/proc/`<br>
`sudo mount -t sysfs /sys /mnt/sys/`<br>
`sudo mount -o bind /tmp /mnt/tmp/`<br>
`sudo mount -o bind /dev /mnt/dev/`<br>

4. Осуществляем chroot в /mnt <br>

`sudo chroot /mnt /bin/bash`<br>

И вот мы в можем выполнять действия от основной операционной системы

5. Удаляем все старые и текущие ядра

`apt remove linux-image-5.4.0-71-generic`<br>
`apt remove linux-image-5.4.0-72-generic`<br>
`apt autoremove`<br>

6. Очищаем каталог /boot от файлов суффиксами в названиях 5.4.0-71 и 5.4.0-72 <br>

7. Устанавливаем по новой ядро 5.4.0-72 <br>

`apt install linux-image-5.4.0-72-generic`<br>

`update-initramfs -c -k 5.4.0-72-generic`<br>
`update-grub`

8. Проверяем содержимое файла modules.symbols на наличие драйверов сети и wifi
`cat /lib/modules/5.4.0-72-generic/modules.symbols | grep rtl`
- если содержимое не отображается, то при перезагрузке не будут работать wifi и еще другие устройства (например звук)
- если содержимое отобразилось, то можно пропустить следующий пункт и перейти на пункт 10

9. В моем случае помогла установка версии ядра lowlatency
`apt install linux-image-5.4.0-72-lowlatency`<br>
`update-initramfs`<br>
`update-grub`<br>

10. Выходим из chroot
`exit`<br>

11. Перезагружаемся в свою родную операционную систему.

В большинстве случаев данных действий достаточно. 

