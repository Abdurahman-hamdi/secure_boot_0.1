# secure_boot_0.1
This a secure boot bootloader runs on stm32f4 target, the project integerates Mbedtls library to enable aes-cbc to establish secure communication, and SHA25 to verfiy the App binary every reset.

## Introduction to How the project works
```
Secure boot makes a secure commuincation between a flasher TEST tool(I will build it in the feature) that could be remote tool or a local tool communicating via UART siral port.
This verion enables AES_CBC decption to decrypt the Application HEX file received from the TEST flasher tool, and SHA256 to verfiy the Applicaion Consistency each time the system reset, Also a CRC_32 is enabled to build reliable communication between TEST and our bootloader. 
CRC is a good choice for small data chunks to be signed, Meanwhile SHA256 is suitable for large files.
```
flasher TEST tool works on sending an AES_CBC encrypted string cointains:
```
1-for writting to the flash: write_cmd+(flash memory address+its CRC word)+(data to be writen(16bytes aligned) +its CRC_32).
2-for erasing the application memory sector : erase_cmd+(flash memory address and its CRC).
3-for jump and that is after writing the new SW : jump_cmb+(flash memory address/start addres +itc crc)
```
```
the flasher tool shall also compute the SHA256 for the application and send the hash word to our bootloader, after reciving this word, its written to flash memory address =(uint32_t)0x0800C000(shall be known be both sides the flasher and our bootloader), and after reset the system calculate the SHA256 for the app and compare it with the hash word received from the tool, if they are the same then run the application.
```
for further information of how the apllication should be, plz check : my rep https://github.com/Abdurahman-hamdi/stm32f407_bootloader.

##Project structure
```
├── bootloader.elf.launch
├── bootloader.hex
├── build
├── CMakeFiles
├── CMakeLists.txt
├── gcc-arm-none-eabi.cmake
├── gtest
├── Libraries
├── Middlewares //holds Mbedtls library
├── PROJECT
├── README.md
├── src
├── stm32f4_flash.ld
├── user_include
└── user_port_manage
```
Most important directories:  
Under src direcory, bootloader.c utilses AES_CBC_decrypt function to decrypt the recieved string from the tool, it also implements check_imageconsistency function which reads the whole application from the flash memory and performs SHA256 and generate the Hash word to be compared with the reserved one.  
The folder user_port_manage contains configuration for the project( Flash start adress, SHA256 hash word..etc ) open it!  
The folder Middlewares contains Mbedtls library you can clone it from https://github.com/STMicroelectronics/stm32-mw-mbedtls.git, and follow intructions in its readme to build it for arm-none-eabi-gcc compiler.  
The folder Library contains STD drivers for stm32f4.  
the file gcc-arm-none-eabi.cmake contains compiler and linker setting.  
the file CMakeLists.txt is to build the project and create its make file, and also to integerate Mbedtls library.  
Ignore gtest as it will be integerated and used in the near feature.  


## How to setup and build the project:  
This project is build on ubuntu 20.8.  
install the compiler for ARM and place its binaries in /opt/arm-gcc/.  
Unzip the project.  
After building Mbedtls copy them under Midleware dirctory.  
Ensure that your cmake version >3.12 to  be able to use it.  
At project directory run the cmake command "cmake -Bbuild -DCMAKE_TOOLCHAIN_FILE=gcc-arm-none-eabi.cmake -DCMAKE_BUILD_TYPE=Debug -DCMAKE_EXPORT_COMPILE_COMMANDS=true"  
this command will generate build directory and development commands like make flash, enter this directory and Run "make -j8" to compile, link, and generate .elf,.hex,.bin project files.  
Now you can burn the stm32f4 by run " make flash"



