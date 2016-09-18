
## NOTE

Immediately, I note that I use the development board based stm32f103c8t6, compatible with [ST-NUCLEO-F103RB](https://developer.mbed.org/platforms/ST-Nucleo-F103RB/). I'm using the zsh instead bash shell.


## Install the required packages

```bash
brew install wget autoconf automake pkg-config
```

## ARM Toolchain

Download manually from [Launchpad.net](https://launchpad.net/gcc-arm-embedded/+download), or

```bash
# Make "~/Embed Tools" folder and cd into it
mkdir -p ~/Embed\ Tools && cd "$_"

wget -qO - https://launchpad.net/gcc-arm-embedded/5.0/5-2016-q2-update/+download/gcc-arm-none-eabi-5_4-2016q2-20160622-mac.tar.bz2 | tar xvj
```

Create a symlink to toolchain, if you need to use multiple versions, just unzip them into the directory ~/Embed Tools and, depending on the situation, change the symlink to them.

```bash
sudo ln -s ~/Embed\ Tools/gcc-arm-none-eabi-5_4-2016q2/bin /usr/local/opt/arm-toolchain
```

Find your _rc_ file with:

```bash
$ echo $SHELL
# If /bin/bash in result, then your need ~/.bashrc
/bin/bash
# If /bin/zsh in result, then your need ~/.zshrc
/bin/zsh
```

> __rc__ ([Run commands](https://en.wikipedia.org/wiki/Run_commands)) is a name for any list of commands. .bashrc file, for example, will executed whenever a new terminal session is started in __interactive mode__.


Past next lines at the end of your _rc_ file:

```bash
# ARM Toolchain
export PATH="/usr/local/opt/arm-toolchain:$PATH"
```

Apply changes for this terminal session:

```bash
source ~/.zshrc
# or
source ~/.bashrc
```

Then test ARM Toolchain:

```bash
arm-none-eabi-gcc --version

arm-none-eabi-gcc (GNU Tools for ARM Embedded Processors) 5.4.1 20160609 (release) [ARM/embedded-5-branch revision 237715]
Copyright (C) 2015 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```


## OpenOCD

[OpenOCD](http://openocd.sourceforge.net/) is an open on-chip debugger and progamming tool. It will communicate with gdb to debug and flash the board by using the stlinkv2 debugger.

```bash
brew install openocd --enable_ft2232_libftdi --enable_stlink
```

Once installed, you have to create an empty file called stm32f103c8t6.cfg in your home folder, and paste the following lines:

```bash
# Script for connecting with the STM32F103C8T6 board
source [find interface/stlink-v2.cfg]
source [find target/stm32f1x.cfg]
# reset_config srst_only srst_nogate
```

Now we will test OpenOCD:

```bash
 openocd -f /usr/local/share/openocd/scripts/interface/stlink-v2.cfg -f /usr/local/share/openocd/scripts/target/stm32f1x.cfg
```

Open a Telnet session to another terminal screen:

```bash
telnet localhost 4444
```

Now press and hold the reset button on the devboard, and enter following command in the telnet session screen:

```bash
> reset halt
```

It should get a response:

```bash
timed out while waiting for target halted
TARGET: stm32f1x.cpu - Not halted
in procedure 'reset'
in procedure 'ocd_bouncer'
```

Releasing the reset button and you should see a message about the controller halt:

```bash
target state: halted
target halted due to debug-request, current mode: Thread
xPSR: 0x01000000 pc: 0x08000220 msp: 0x20005000
```

Well, it works!


## Java JDK Installation

[Download](http://www.oracle.com/technetwork/java/javase/downloads/index.html)

## Eclipse Installation

[Download Eclipse Installer or Eclipse for C/C++ Developers](https://www.eclipse.org/downloads/)

### Eclipse GNU ARM plugin

Help->Install New Software
Press Add button and enter:
Name: Gnu Arm Eclipse PlugIn
Location: http://gnuarmeclipse.sourceforge.net/updates

The component «GNU Cross Development Tools” will appear below. Expand, select all the plugins and install it.


## Debugging

### GDB

Run->Debug Configurations

Find the GDB OpenOCD Debugging in left pane.
Right-click on it an select -> New

Name - BlinkTest Debug OpenOCD
Project - enter project name (BlinkTest)
C/C++ Application - enter location and .elf file name (Debug/BlinkTest.elf)
Click on Debugger tab

Tick a «Start OpenOCD locally»
Executable - /usr/local/bin/openocd
GDB port: 3333
Telnet port: 4444
Config Options: -f stm32f103c8t6.cfg

_Note: [stm32f103c8t6.cfg](https://github.com/asakasinsky/STM32/blob/master/STM32F103C8T6_minimal_dev_board/Files/stm32f103c8t6.cfg) must be placed in a «workspace» is running Eclipse, otherwise specify the full path to the file_









-------------------------------

__Проверить по картам памяти и инструкции к OpenOCD__

Programming an STM32F103XXX with a generic "ST Link V2" programmer from Linux · rogerclarkmelbourne/Arduino_STM32 Wiki
https://github.com/rogerclarkmelbourne/Arduino_STM32/wiki/Programming-an-STM32F103XXX-with-a-generic-%22ST-Link-V2%22-programmer-from-Linux


```bash
> init
> reset init
target state: halted
target halted due to debug-request, current mode: Thread
xPSR: 0x01000000 pc: 0xfffffffe msp: 0xfffffffc
> poll off
> adapter_khz 8000
8000 kHz
> dump_image dump.bin 0x08000000 131072
dumped 131072 bytes in 3.291646s (38.886 KiB/s)
> verify_image dump.bin 0x08000000
target state: halted
target halted due to debug-request, current mode: Thread
xPSR: 0x61000000 pc: 0x2000003a msp: 0xfffffffc
verified 131072 bytes in 2.087557s (61.316 KiB/s)
> stm32f1x mass_erase 0
device id = 0x20036410
flash size = 128kbytes
stm32x mass erase complete
> flash write_image dump.bin 0x08000000
target state: halted
target halted due to debug-request, current mode: Thread
xPSR: 0x61000000 pc: 0x2000003e msp: 0xfffffffc
wrote 131072 bytes from file dump.bin in 4.109462s (31.148 KiB/s)
> verify_image dump.bin 0x08000000
target state: halted
target halted due to debug-request, current mode: Thread
xPSR: 0x61000000 pc: 0x2000003a msp: 0xfffffffc
verified 131072 bytes in 2.094413s (61.115 KiB/s)
> load_image dump.bin 0x20000000 bin 0x20000000 20480
20480 bytes written at address 0x20000000
downloaded 20480 bytes in 0.494348s (40.457 KiB/s)
> shutdown
shutdown command invoked
> Connection closed by foreign host.
```

-------------------------------



## Семихостинг

__Не проверял__

Выводит printf() в консоль OpenOCD с помощью JTAG. Для этого в опции линкера в Make-файле надо добавить:

-specs=nosys.specs -specs=nano.specs -specs=rdimon.specs -lc -lrdimon
Для корректного вывода float добавить ключ

LDFLAGS += -u _printf_float
В Initialization Commands на вкладке Startup в параметрах конфигурации отладки OpenOCD в Eclipse добавить:

monitor arm semihosting enable



 -------------------

 -----


Отладка с помощью EmbSysRegView

Можно смотреть все регистры на паузе, заливка на плату автоматически, вывод в консоль жклипса




 -------------------



Выбираем HAL

## STM32CubeMX


Не зависимо от библиотеки в результате компиляции вы должны получить бинарные файлы .hex или .bin для прошивки контроллера и .elf файл для отладки.


Генератор проектов STM32CubeMX работает как отдельное приложение под Java. Чтобы его установить под Linux или MacOS надо переименовать установочный exe-файл в jar и выполнить:

    java -jar SetupSTM32CubeMX-4.11.0.jar

Также есть версия STM32CubeMX в виде плагина к Eclipse (http://www.st.com/web/en/catalog/tools/PF257931). Для ее установки нужно в Eclipse выбрать:

    Help > Install New Software...

Нажать кнопки Add..., затем Archive... и выбрать zip-архив с плагином.

STM32CubeMX умеет генерировать проекты для нескольких IDE: EWARM, MDK-ARM 4 и 5, TrueStudio и SW4STM32. Eclipse пока напрямую не поддерживается.

Чтобы скомпилировать проект и работать с ним в Eclipse через обычные Make-файлы необходимо сгенерировать проект для IDE SW4STM32 с помощью STM32CubeMX, а затем сгенерировать make-файл с помощью скрипта https://github.com/baoshi/CubeMX2Makefile. Для этого понадобится Python 2.7 и команда:

    CubeMX2Makefile.py путь_к_проекту_SW4STM32
    
    usage: cubemximporter.py [-h] [-v VERBOSE] [--dryrun]
                         eclipse_project_folder cubemx_project_folder

Потом открыть сгенерированный файл и удалить кавычки в строчке с CDEFS. Например,

    C_DEFS = -D__weak="__attribute__\(\(weak\)\)" -D__packed="__attribute__\(\(__packed__\)\)" -DUSE_HAL_DRIVER -DSTM32F303xC

заменить на

     C_DEFS = -D__weak=__attribute__\(\(weak\)\) -D__packed=__attribute__\(\(__packed__\)\) -DUSE_HAL_DRIVER -DSTM32F303xC

После этого скомпилировать проект можно командой make в каталоге проекта. Для Windows предварительно нужно будет установить GNU Make (http://gnuwin32.sourceforge.net/packages/make.htm) и GNU Core Utilites (http://gnuwin32.sourceforge.net/packages/coreutils.htm), под Ubuntu/Debian Linux достаточно установить пакет make.

Импортировать проект в Eclipse так:

    File > New > Makefile Project With Existing Code
