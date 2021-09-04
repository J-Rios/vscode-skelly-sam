# vscode-skelly-sam

VSCode SAM skelleton base example project to work (develop and debug) with Microchip-Atmel SAM microcontrollers on a Linux system without the need of a full IDE.

This project example target a SAMD20-Xplained-Pro board, but the repository can be used as a reference to create new projects for other targets.

## Requirements

- Install required libraries:

```bash
sudo apt-get update
sudo apt-get -y install libncurses5 libncurses5-dev libncursesw5-dev
sudo apt-get -y install libc6 libc6-dev
sudo apt-get -y install libudev libudev-dev
```

- Install the [ARM Toolchain](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads):

```bash
cd /opt
sudo wget https://developer.arm.com/-/media/Files/downloads/gnu-rm/10.3-2021.07/gcc-arm-none-eabi-10.3-2021.07-x86_64-linux.tar.bz2
sudo tar -xf gcc-arm-none-eabi-10.3-2021.07-x86_64-linux.tar.bz2
sudo rm -f gcc-arm-none-eabi-10.3-2021.07-x86_64-linux.tar.bz2
sudo ln -sf /opt/gcc-arm-none-eabi-10.3-2021.07/bin/* /usr/bin/
```

- Install Debugging and Flashing Tools:

```bash
sudo apt-get -y install openocd

cd ~
git clone https://github.com/ataradov/edbg
cd edbg
make all
sudo cp -a ./edbg /usr/local/bin/
sudo chmod +x /usr/local/bin/edbg
cd ~
rm -rf edbg
```

- In vscode, install C/C++ and Cortex-Debug extensions.

## How to Create a SAM project supported by vscode

Here is the steps that has been followed to create current project:

1. Create SAM projects directories and setup current project:

```bash
mkdir ~/sam
mkdir ~/sam/vscode-skelly-sam
cd ~/sam/vscode-skelly-sam
mkdir build doc inc lib src
mkdir -p asf_hal/samd20
```

2. Get [Microchip-Atmel ASF (Advanced Software Framework)](https://www.microchip.com/mplab/avr-support/advanced-software-framework) and setup the specific version that you want to use:

```bash
cd ~/sam/
git clone https://github.com/J-Rios/microchip-asf
cd microchip-asf
git log
git checkout <COMMIT_OF_ASF_VERSION>
```

3. Use an application example from ASF as startup point and setup current project with them:

```bash
cd ~/sam/
cp -a microchip-asf/sam0/applications/led_toggle/led_toggle.c vscode-skelly-sam/src/main.c
cp -a microchip-asf/sam0/applications/led_toggle/samd20_xplained_pro/gcc/asf.h vscode-skelly-sam/asf_hal/samd20/
cp -a microchip-asf/sam0/applications/led_toggle/samd20_xplained_pro/gcc/Makefile vscode-skelly-sam/build/
cp -a microchip-asf/sam0/applications/led_toggle/samd20_xplained_pro/gcc/config.mk vscode-skelly-sam/build/config.mk.orig
cp -a microchip-asf/sam0/applications/led_toggle/samd20_xplained_pro/conf_board.h vscode-skelly-sam/asf_hal/samd20/
cp -a microchip-asf/sam0/applications/led_toggle/samd20_xplained_pro/conf_clocks.h vscode-skelly-sam/asf_hal/samd20/
```

4. Open the created project directory inside vscode.

5. Create **makebuild** bash script (and all needed scripts) to ease build, modify **Makefile** to point to ASF internal Makefile path, and configure **config.mk.orig** file to point to ASF path directory and all project specific files and directory paths.

6. Create a **./vscode/settings.json** file and setup specific Toolchain and tools path in there.

7. Create a **./vscode/c_cpp_properties.json** file and setup C & C++ standard versions, the **arm-none-eabi-gcc compiler path**, **Defines** and the **compiler arguments** that are used in project config.mk to setup correctly intellisense functionality.

8. Create a **./vscode/tasks.json** file and setup *makebuild* script as command to execute, and tasks availables to be used from vscode.

9. Create a **./vscode/launch.json** file and setup each available debuggers to be used and their specific configurations for target device.

## Usage

Build and Debug:

1. Physically connect the hardware programmer to target device and the PC.

2. Open main.c file.

3. Go to **Run and Debug** left panel menu and select some launch configuration according to programmer and debugger that will be used (*openocd*).

4. Press Run button.

Release Flash:

1. Press Ctrl+Shift+B to show the configured Tasks of the project.

2. Execute one of the Task to flash the firmware into the device through any of the supported and configured interfaces (*edbg*).

## Notes

- Remember that compiler optimizations will translate the C/C++ code to an efficient memory/speed ASM, causing that lines execution behaviour when debugging will confuse a human. To keep a sequentially line-by-line execution a -Og flag/parameter needs to be passed to the compiler (change that in Makefile). Of course, you won't want this debug oriented optimization for Release, so you should use -Os for firmware Release.
