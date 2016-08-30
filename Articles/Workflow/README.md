
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
