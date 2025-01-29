# How to OrangePi5+ (OrangePi5plus) pwm fan fix

This manual is for Joshua Riek`s Ubuntu 24.04 (other distro can be adapted).

The device tree with a.o. the info for the PWM fan is in 
Dir: /lib/firmware/6.1.0-1025-rockchip/device-tree/rockchip
File: rk3588-orangepi-5-plus.dtb

I found this in /etc/default/u-boot
variable U_BOOT_FDT

Open terminal in directory /lib/firmware/6.1.0-1025-rockchip/device-tree
Decompile current dtb to text.
dtc -I dtb -O dts -o rk3588-orangepi-5-plus.dts /rockchip/rk3588-orangepi-5-plus.dtb

Open rk3588-orangepi-5-plus.dts and find "pwm-fan" section. Look at "pwms" record.
For example: < A B C D >
"B and D both are zero".
"A" is link to pwm3.
"C" is MOST IMPORTANT - it's PERIOD of ns (frequency).

Here is example (Your pwm-fan section can look slight different) :
    pwm-fan {
        compatible = "pwm-fan";
        #cooling-cells = <0x02>;
        pwms = <0x1f2 0x00 0xc350 0x00>;
        cooling-levels = <0x00 0x32 0x64 0x96 0xc8 0xff>;
        rockchip,temp-trips = <0xc350 0x01 0xd6d8 0x02 0xea60 0x03 0xfde8 0x04 0x11170 0x05>;
        status = "okay";
        phandle = <0x4b5>;
    };

We can see "50000" or "0xc350", at 3rd parameter. It's 50'000 ns period (0xc350) = 20`000Hz = 20kHz.
Pwm-fan schematic has RC-chain (R23+C16): 10K+100nF.
![alt text](https://github.com/metamot/opi5plus_fan_fix/blob/main/Untitled.png?raw=true)
Fc=1/(2pi*R*C)=1/0.00628=160Hz.
So, current DeviceTree settings with 20kHz acts as DC-voltage control (ON/OFF) with max fan speed under any conditions.
You can desolder C16 capacitor, to solve this problem!
(Can you really can locate this cap_0402 on unsilcscreeneed pcb?).

**The other way** is to reduce pwm frequency to 100Hz!
To reduce frequency replace 3rd pwms parameter to "10000000" (one+7zeros) = "10 million" (nanoseconds) = 0.01s = 100Hz.
Here is example:
    pwm-fan {
        compatible = "pwm-fan";
        #cooling-cells = <0x02>;
        pwms = <0x1f2 0x00 10000000 0x00>;
        cooling-levels = <0x00 0x32 0x64 0x96 0xc8 0xff>;
        rockchip,temp-trips = <0xc350 0x01 0xd6d8 0x02 0xea60 0x03 0xfde8 0x04 0x11170 0x05>;
        status = "okay";
        phandle = <0x4b5>;
    };
Take look at pwms record 3rd parameter replaced with 10000000. Your pwm-fan section can look slight different, but only one replacement you need to do, is change 3rd pwms parameter to 10000000.

Compile current dts (source) to dtb (binary)
dtc -I dts -O dtb -o rk3588-orangepi-5-plus.dtb rk3588-orangepi-5-plus.dts

Run the u-boot-update utility to update the device tree to be loaded on the next reboot.
sudo u-boot-update

Reboot
Your OrangePi5+ fan works at different speeds now!

Some of fans are really silent at 100-150Hz of low pwm. Please, try it.
