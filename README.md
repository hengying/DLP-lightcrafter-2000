# DLP-lightcrafter-2000
I bought a DLP lightcrafter 2000, but I can't make it display the Raspberry Pi 4b's GUI. After many reading and tests, finally I successfully make the DPI projected on the DLP. Here's the key points.

# Devices
1. DLP lightcrafter 2000 (DLP2000DMD)
2. Raspberry Pi 4b

# Basic Pin connections
Same as: [Build a Pi Zero W pocket projector! ](https://www.mickmake.com/post/build-a-pi-zero-w-pocket-projector-project/) on [mickmake.com](https://www.mickmake.com/)

# Additional Pin connections
* DLP's *HOST_PRESENTZ* should be connected to *GND*
* DLP's *VINTF* should be connected to *3V3*
* DLP's *PROJ_ON_EXT* connected to RP4's *GPIO 22*

# Raspberry Pi 4b's configuration
According to [Raspberry Pi Hat for TI LightCrafter Display DLP2000](https://www.intellar.ca/blog/raspberry-pi-evm2000) on [intellar.ca](https://www.intellar.ca/)

## /boot/config.txt 
```
dtoverlay=dpi18
overscan_left=0
overscan_right=0
overscan_top=0
overscan_bottom=0
framebuffer_width=640
framebuffer_height=360
enable_dpi_lcd=1
display_default_lcd=0
dpi_group=2
dpi_mode=87
dpi_output_format=458773
hdmi_timings=640 0 14 4 12 360 0 2 3 9 0 0 0 60 0 13824000 3  
#config i2c to use pins 23,24 for communication with dlp hat
dtoverlay=i2c-gpio,i2c_gpio_sda=23,i2c_gpio_scl=24,i2c_gpio_delay_us=2,bus=22
```

## Do **NOT** add these lines into /etc/rc.local
```
# Do NOT add these lines into /etc/rc.local
i2cset -y 22 0x1b 0x0c 0x00 0x00 0x00 0x1B i
i2cset -y 22 0x1b 0x0b 0x00 0x00 0x00 0x00 i
```

# I2C address
My DLP's I2C address is 22. It could checked it by "i2cdetect -l".
On my RP4, I got this:
```
i2c-20  i2c             fef04500.i2c                            I2C adapter
i2c-1   i2c             bcm2835 (i2c@7e804000)                  I2C adapter
i2c-21  i2c             fef09500.i2c                            I2C adapter
i2c-22  i2c             ffffffff00000002.i2c                    I2C adapter
```

# Turn on DLP script
## set the RP4's GPIO's drive strength
```
raspi-gpio set 1-27 ip pn
sleep 1
sudo gpio drive 0 5
raspi-gpio set 0-21 a2 pn
sleep 1
```
[WiringPi](https://projects.drogon.net/raspberry-pi/wiringpi/download-and-install/) or [pigpio](https://abyz.me.uk/rpi/pigpio/download.html) should be installed to set the drive strength of RP4's GPIO.

## turn on DLP
```
gpio -g mode 22 out
gpio -g write 22 1
```

## set DLP's source to 'Parallel I/F' (RP4's DPI)
```
sleep 0.5
echo pixel format
i2cset -y 22 0x1b 0x0d 0x00 0x00 0x00 0x02 i
echo resolution
i2cset -y 22 0x1b 0x0c 0x00 0x00 0x00 0x1B i
echo parallel input
i2cset -y 22 0x1b 0x0b 0x00 0x00 0x00 0x00 i
```

# Turn off DLP script
```
gpio -g write 22 0
```

# More details
1. I made several PCBs from [JLCPCB](https://jlcpcb.com/)
2. I made a simple 3D printed shell

![PCB](/pics/PCB.png)

![PCB](/pics/3d_printed_shell.jpg)


