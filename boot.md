
# Table of Contents

1.  [Загрузка через GRUB](#org957321a)
    1.  [Подгружаемые GRUB-модули](#org5c44532)
    2.  [Настройка терминала](#orge2d2f13)
    3.  [Тема оформления GRUB](#org2773ce5)
    4.  [Пункты меню](#org1bb7556)
        1.  [Обычная загрузка](#org9b7e657)
        2.  [Загрузка с отключенным remote desktop](#org025522c)
        3.  [Загрузка с отключенным floppy](#orgd503882)
        4.  [Загрузка без сети](#org2338537)
        5.  [Быстрая загрузка](#org5d99181)
        6.  [NatSuki](#org81dbd81)
        7.  [Reboot](#orgd603f3f)
        8.  [Power off](#orgc677ee5)
2.  [Сборка загрузчика](#orgc205b97)
    1.  [Сборка ассемблерной части](#org6393e02)
    2.  [Сборка сишной части](#org1dea19d)
4.  [Граф зависимостей в Makefile](#org87234c7)



<a id="org957321a"></a>

# Загрузка через GRUB

SayoriOS использует GRUB2 в качестве загрузчика, его конфиг лежит в iso/boot/grub/grub.cfg


<a id="org5c44532"></a>

## Подгружаемые GRUB-модули

Для загрузки нам понадобятся необходимые модули:

    # Импорт модулей
    insmod vbe
    insmod vga
    insmod video_bochs
    insmod video_cirrus
    insmod gfxterm
    insmod gfxmenu
    insmod tga


<a id="orge2d2f13"></a>

## Настройка терминала

Теперь нам нужно настроить терминал, его input и output.

Допустимые настройки зависят от платформы, и могут включать:

-   console - родная консоль платформы
-   serial - последовательный терминал
-   serial_{port} - последовательный терминал с явным выбором порта
-   at_keyboard - клавиатура ПК AT
-   usb_keyboard - USB-клавиатура использовующая HID Boot Protocol

По умолчанию используется "console", но тут мы указываем ее явно.

Допустимые output терминалов также зависят от платформы, и могут включать:

-   console - родная консоль платформы
-   serial - последовательный терминал
-   serial\_<port> - последовательный терминал с явным выбором порта
-   gfxterm - вывод в графическом режиме
-   vga_text - вывод текста VGA
-   mda_text - (текстовый вывод MDA),
-   morse - вывод через Beeper используя код Морзе
-   spkmodem -простой протокол передачи данных с использованием системного динамика. Он полезен, когда последовательный порт недоступен. Подключите выход от отправляющей системы (где работает GRUB) к линейному входу принимающей системы (обычно машина для проявки). На принимающей машине скопмплируйте "spkmodem-recv" из запустите:

    parecord --channels=1 --rate=48000 --format=s16le | ./spkmodem-recv

Мы используем gfxterm - вывод в графическом режиме. Это приводит к тому, что GRUB загружает все доступные видео GRUB драйверы и использует тот, который наиболее подходит для вашего оборудования. Поэтому для минимизации размера эту опцию можно изменить.

    # Настройка терминала
    terminal_input console
    terminal_output gfxterm


<a id="org2773ce5"></a>

## Тема оформления GRUB

Мы используем тему оформления, которая лежит в iso/boot/grub/theme.txt

    # Тема оформления
    set theme=/boot/grub/theme.txt
    export theme

Эта тема добавляет фоновую картинку и описывает внешний вид меню

    desktop-color: "#24282c"
    desktop-image: "background.tga"

    + boot_menu {
        left = 15%
        width = 70%
        top = 30%
        height = 45%
        item_height = 30
        item_padding = 5
        icon_width = 32
        icon_height = 32
        item_icon_space = 20
        item_spacing = 5
        item_color = "#cccccc"
        selected_item_color = "#00ff00"
    }

    + hbox {
        left = 50%-250
        top = 80%
        width = 500
        + label { width = 250 height = 20 align = "center" color = "#ffffff" text = "E = Edit Boot Options" }
        + label { width = 250 height = 20 align = "center" color = "#ffffff" text = "C = GRUB Commandline" }
    }


<a id="org1bb7556"></a>

## Пункты меню

Несколько пунктов меню описывают различные варианты загрузки SayoryOS. Все они пока используют boot/kernel.elf и bot/ramdisk


<a id="org9b7e657"></a>

### Обычная загрузка

    # Обычная загрузка
    menuentry "SayoriOS Soul v0.3.4 Alpha" {
        multiboot /boot/kernel.elf
        module /boot/ramdisk initrd_tarfs;
        boot
    }


<a id="org025522c"></a>

### Загрузка с отключенным ACPI

    # Загрузка с отключенным remote desktop (это пимник написал)
    menuentry --hotkey=w "SayoriOS Soul v0.3.4 Alpha [VMWare]" {
        multiboot /boot/kernel.elf disable=rdsp
        module /boot/ramdisk initrd_tarfs;
        boot
    }


<a id="orgd503882"></a>

### Загрузка с отключенным floppy

    # Загрузка с отключенным floppy
    menuentry --hotkey=f "SayoriOS Soul v0.3.4 Alpha [No Floppy Support]" {
        multiboot /boot/kernel.elf disable=floppy
        module /boot/ramdisk initrd_tarfs;
        boot
    }


<a id="org2338537"></a>

### Загрузка без сети

    # Загрузка без сети
    menuentry "SayoriOS Soul v0.3.x Alpha [Minimal Bootscreen]" {
        multiboot /boot/kernel.elf bootscreen=minimal disable=network
        module /boot/ramdisk initrd_tarfs;
        boot
    }


<a id="org5d99181"></a>

### Быстрая загрузка

    # Быстрая загрузка
    menuentry --hotkey=f "SayoriOS Soul v0.3.4 Alpha [FAST+]" {
        multiboot /boot/kernel.elf disable=coms disable=ac97 disable=pc-speaker disable=floppy disable=network bootscreen=minimal bootscreen=no-logs
        module /boot/ramdisk initrd_tarfs;
        boot
    }


<a id="org81dbd81"></a>

### NatSuki

    # NatSuki
    menuentry --hotkey=n "SayoriOS Soul v0.3.4 Alpha [NatSuki]" {
        multiboot /boot/kernel.elf
        module /boot/ramdisk initrd_tarfs;
        boot
    }


<a id="orgd603f3f"></a>

### Reboot

    # Reboot
    menuentry --hotkey=r 'Reboot' {
       reboot
    }


<a id="orgc677ee5"></a>

### Power off

    # Power off
    menuentry --hotkey=h 'Power off' {
       halt
    }


<a id="orgc205b97"></a>

# Сборка загрузчика

Мы можем собирать загрузчик с использованием GRUB или GRUB2:

    GRUB=$(ls $PREFIX/bin/grub-mkrescue 2> /dev/null)
    GRUB2=$(ls $PREFIX/bin/grub2-mkrescue 2> /dev/null)

    if [ -n "$GRUB2" ]; then
        echo $GRUB2
    elif [ -n "$GRUB" ]; then
        echo $GRUB
    fi

И это используется в таргете Makefile, который называется "geniso":

    geniso: $(KERNEL)
        $(shell bash tools/grub.sh) -o "kernel.iso" iso/ -V kernel

"geniso"-цель зависит от цели "KERNEL", через которую определен файл с ядром (определяется в config.mk):

    KERNEL = iso/boot/kernel.elf

А в основном Makefile эта цель описана так (я временно удалил все связанное с Rust для упрощения, а также stripping):

    $(KERNEL): $(KERNEL_NEED)
        @echo -e '\x1b[32mLINK \x1b[0m' $(KERNEL)
        @rm -f $(KERNEL)
        @$(LD) $(LDFLAGS) -o $(KERNEL) $(KERNEL_NEED)
        @bash tools/genmap.sh
        @bash tools/insertmap.sh
        @ls -lh $(KERNEL)
        @-rm kernel.map

KERNEL-цель зависит от:

-   "KERNEL_NEED"
-   "LD"
-   "LDFLAGS"

а также от файлов:

-   tools/genmap.sh (todo)
-   tools/insertmap.sh (todo)

Таргет "KERNEL_NEED" определяется в config.mk:

    KERNEL_NEED = $(ASM) $(OBJS) $(CPP_CODE)

    LD ?= ld.lld

    LDFLAGS=-T kernel/asm/link.ld

Теперь у нас накопились цели:

-   "ASM" (сборка ассемблерной части)
-   "OBJS" (сборка сишной части)
-   "CPP_CODE" (полностью закомментированные сорцы)

а также файл:

-   kernel/asm/link.ld (todo)


<a id="org6393e02"></a>

## Сборка ассемблерной части

Тут мы достигаем ассеблерного исходного кода:

    ASM=$(ASM_SRC:%.s=$(OBJ_DIRECTORY)/%.o)

что добавляет к списку целей:

-   "ASM_SRC"
-   "OBJ_DIRECTORY"


"OBJ_DIRECTORY":

    OBJ_DIRECTORY = objects

    $(OBJ_DIRECTORY)/%.o : %.s | $(OBJ_DIRECTORY)
        @echo -e '\x1b[32mASM  \x1b[0m' $@
        @$(AS) $< $(ASFLAGS) -o $@

    $(OBJ_DIRECTORY)/%.o : %.c | $(OBJ_DIRECTORY)
        @echo -e '\x1b[32mC    \x1b[0m' $@
        @$(CC) $(CFLAGS) -c -o $@ $<

    $(OBJ_DIRECTORY)/%.o : %.cpp | $(OBJ_DIRECTORY)
        @echo -e '\x1b[32mCPP  \x1b[0m' $@
        @$(CXX) $(CPP_FLAGS) -c -o $@ $<


<a id="org1dea19d"></a>

## Сборка сишной части

Тут мы достигаем сишного кода:

    OBJS = $(SOURCES:%.c=$(OBJ_DIRECTORY)/%.o)

Так как "OBJ_DIRECTORY" уже определена, остается разобраться с "SOURCES"

Теперь нам нужен еще "GAMEBOY" (я не знаю зачем нужна его поддежка в ядре):

    GAMEBOY = $(wildcard kernel/src/ports/gameboy/*.c)

В файле config.mk определена еще переменная "GAMEBOY_OBJS" которая нигде не используется

    GAMEBOY_OBJS = $(GAMEBOY:.c=.o)


<a id="org87234c7"></a>

# Граф зависимостей в Makefile

Есть способ получить его в виде svg-файла с помощью ключей самого make и graphviz:

    make -Bnd | make2graph > out.dot && dot -Tsvg  out.dot > out.svg && firefox --new-window out.svg