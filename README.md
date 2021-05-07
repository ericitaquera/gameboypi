

-Baixar a imagem do retropie

https://retropie.org.uk/download/#Pre-made_images_for_the_Raspberry_Pi

-Gravar a imagem no SDcard

-Montar o SDcard 

-Editar o config.txt

Alterar/Descomentar as linhas

framebuffer_width=320
framebuffer_height=240

hdmi_group=1
hdmi_mode=1

hdmi_drive=1

sdtv_mode=2

arm_freq_min=250
core_freq_min=100
sdram_freq_min=150
over_voltage_min=4

arm_freq=1000
gpu_freq=500
core_freq=500
sdram_freq=500
sdram_schmoo=0x02000020
over_voltage=2
sdram_over_voltage=2

initial_turbo=30

dtparam=audio=on
dtparam=spi=on

dtoverlay=pwm-2chan,pin=18,func=2,pin2=13,func2=4

display_rotate=2

audio_pwm_mode=0

disable_audio_dither=1

#Disable the ACT LED on the Pi Zero.
dtparam=act_led_trigger=none
dtparam=act_led_activelow=on

#Disable Bluetooth
dtoverlay=pi3-disable-bt

++++++++++++++++++++++++++++

-criar um arquivo "ssh" no raiz do SDcard (criar um arquivo vazio)

-criar um arquivo wpa_supplicant.conf no raiz do sdcard

country=BR
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
network={
    ssid="<rede>"
    psk="<senha>"
}

-logar 

sudo apt-get update -y

sudo apt-get install bc vim mlocate wiringpi libssl-dev libncurses5-dev -y && sudo updatedb

sudo apt-get upgrade -y

sudo reboot

sudo rpi-update a08ece3d48c3c40bf1b501772af9933249c11c5b
-- https://github.com/Hexxeh/rpi-firmware/commits/master

- install rpi-source

sudo wget https://raw.githubusercontent.com/notro/rpi-source/master/rpi-source -O /usr/bin/rpi-source && sudo chmod +x /usr/bin/rpi-source && sudo /usr/bin/rpi-source -q --tag-update

sudo rpi-source 

-instalar mk_arcade_joystick_rpi

#kernel 4.14.98+

cd /usr/src/

sudo git clone https://github.com/Pinuct/mk_arcade_joystick_rpi.git

sudo mv mk_arcade_joystick_rpi/ mk_arcade_joystick_rpi-0.1.5

sudo dkms build -m mk_arcade_joystick_rpi -v 0.1.5
sudo dkms install -m mk_arcade_joystick_rpi -v 0.1.5

sudo reboot

-editar o /etc/modules

mk_arcade_joystick_rpi

-editar o  /etc/modprobe.d/mk_arcade_joystick_rpi.conf

options mk_arcade_joystick_rpi map=5 gpio=4,17,27,22,20,16,5,6,12,26,19,23

sudo reboot

sudo RetroPie-Setup/retropie_setup.sh

sudo reboot

sudo raspi-config
	
	4 Localisation Options
		I2 Change Timezone
			America
				Sao_Paulo
	

sudo alsamixer

sudo git clone https://github.com/rxbrad/es-theme-gbz35.git /etc/emulationstation/themes/gbz35/

sudo git clone https://github.com/rxbrad/es-theme-gbz35-dark.git /etc/emulationstation/themes/gbz35-dark/


for power savings

Disable HCI Uart service 

sudo systemctl disable hciuart

add to /etc/rc.local

sudo iwconfig wlan0 txpower 0

Disable USB HUB

pi@retropie:~ $ find /sys/devices/ -name `dmesg -t | grep dwc_otg | grep "DWC OTG Controller" | awk '{print $2}' | cut -d":" -f1`

/sys/devices/platform/soc/20980000.usb

Create /usr/bin/pam_session.sh with execute permission

pi@retropie:~ $ sudo cat /usr/bin/pam_session.sh
#!/bin/sh
if [ "$PAM_TYPE" = "close_session" ]; then
        sudo systemctl start cron
        sudo iwconfig wlan0 txpower 0
fi

if [ "$PAM_TYPE" = "open_session" ]; then
        sudo systemctl stop cron
        sudo iwconfig wlan0 txpower 30
fi



Add the following line to /etc/pam.d/sshd

session     optional    pam_exec.so quiet /usr/bin/pam_session.sh

create with execute permission

/usr/bin/stop_wifi.sh

pi@retropie:~ $ cat /usr/bin/stop_wifi.sh

#!/bin/bash

declare -i var=$(cat /var/tmp/counter)

if [ $var -eq 10 ]; then

        echo 0 > /sys/devices/platform/soc/20980000.usb/buspower 2>&1 >> /var/log/teste.log

        sudo iwconfig wlan0 txpower 0 2>&1 >> /var/log/teste.log

        sudo ip link set wlan0 down 2>&1 >> /var/log/teste.log

        var=$var+1

        echo funfou $(date) >> /var/log/teste.log

        exit
fi

var=$var+1

echo $var > /var/tmp/counter



crontab -e

@reboot sudo iwconfig wlan0 txpower 5
@reboot sudo systemctl start systemd-timesyncd.service
@reboot sudo systemctl start ssh.service
@reboot sudo systemctl start smbd.service
@reboot sudo systemctl start nmbd.service
@reboot sudo systemctl start raspi-config.service
@reboot sudo ip link set wlan0 up 2>&1 >> /var/log/teste.log
@reboot sudo echo 0 > /var/tmp/counter
*/1 * * * * sudo /bin/bash /usr/bin/stop_wifi.sh 2>&1 >> /var/log/teste.log


edit vim /etc/network/interfaces

allow-hotplug wlan0
iface wlan0 inet manual
    post-up iw dev $IFACE set power_save off
    wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf


As a workaround, do sudo systemctl edit apt-daily.timer and paste the following text into the editor window:

apt-daily timer configuration override
[Timer]
OnBootSec=15min
OnUnitActiveSec=1d
AccuracySec=1h
RandomizedDelaySec=30min




pi@retropie:~ $ cat performance.sh
echo ""
date
vcgencmd measure_volts
echo "cur_freq=$(cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_cur_freq)"
vcgencmd measure_temp


Performance
https://github.com/RetroPie/RetroPie-Setup/wiki/Speed-Issues



