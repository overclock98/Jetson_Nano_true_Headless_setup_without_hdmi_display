# Jetson Nano headless fresh setup (WiFi, SSH and VNC) even without a HDMI Display


  1- Download image from [Jetson Download Center]
  
  [Jetson Download Center]: <https://developer.nvidia.com/embedded/downloads>
  
  - Write Image to the microSD Card: [steps][df1]
  
  [df1]: <https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-devkit#write>
  
  - Initial Setup Headless Mode: [steps][df2]
  
  [df2]: <https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-devkit#setup>
  
  - Insert the microSD card (with system image already written to it) into the slot on the underside of the Jetson Nano module.
  
  - Jumper the J48 Power Select Header pins.
  
  - Connect your other computer to the Jetson Nano’s Micro-USB port.
  
  - Connect a DC power supply to the J25 Power Jack. The Jetson Nano will power on automatically.
  
  - Allow 1 minute for the Jetson Nano to boot.
  
  2- On your other computer, use the serial terminal application to connect via host serial port to the Jetson Nano. I use putty (board rate: 115200)
  
  - Review and accept NVIDIA Jetson software EULA
  
  - Select system language, keyboard layout, and time zone
  
  - Create username, password, and computer name
  
  - Select APP partition size—it is recommended to use the max size suggested
  
  - Select primary interface as dummy0 (Wi-Fi will configure later)
  
  - Choose “do not configure network at this time” and proceed
  
  - Login with username and password
  
  3- Insert wifi dongle in to usb port
  
  - Setup WiFi: [steps][df3]
  
  [df3]: <https://core.docs.ubuntu.com/en/stacks/network/network-manager/docs/configure-wifi-connections>
  
  - Check dongle with
  ```sh
  $ nmcli d
  ```
  - Turn on the wifi
  ```sh
  $ nmcli r wifi on
  ```
  - List out wifi “nmcli d wifi list”
  
  -	Enter ssid and password
  ```sh
  $ sudo nmcli d wifi connect <ssid> password <password>
  ```
  - Check ip with
  ```sh
  $ ifconfig wlan0
  ```
  4- Reboot and login via ssh on putty with above ip
  
  In my experience NOMACHINE is better than VNC. If you wand NOMACHINE checkout the [link][df11]
  
  [df11]: <https://www.nomachine.com/AR02R01074>
  
  - Update with
  ```sh
  $ sudo apt update
  ```
  - Install nano text editor
  ```sh
  $ sudo apt-get install nano
  ```
  5- Enabling Desktop Sharing: [steps][df4]
  
  [df4]: <https://www.hackster.io/news/getting-started-with-the-nvidia-jetson-nano-developer-kit-43aa7c298797>
  
  - Install vino
  ```sh
  $ sudo apt install vino
  ```  
  - Open the schema in nano editor
  ```sh
  $ sudo nano /usr/share/glib-2.0/schemas/org.gnome.Vino.gschema.xml
  ```
  and go ahead and add the following key into the XML file
  ```sh
   <key name='enabled' type='b'>
      <summary>Enable remote access to the desktop</summary>
      <description>
        If true, allows remote access to the desktop via the RFB
        protocol. Users on remote machines may then connect to the
        desktop using a VNC viewer.
      </description>
      <default>false</default>
    </key>
  ```
  - Then compile the Gnome schemas
  ```sh
  $ sudo glib-compile-schemas /usr/share/glib-2.0/schemas
  ```
  - Disable encryption for the VNC server
  ```sh
  $ gsettings set org.gnome.Vino require-encryption false
  $ gsettings set org.gnome.Vino prompt-enabled false
  ```
  - Find the UUIDs of your connections
  ```sh
  $ nmcli connection show
  ```
  -  Replace UUID with yours as a comma separated list in the square brackets of the configuration line
  ```sh
  $ dconf write /org/gnome/settings-daemon/plugins/sharing/vino-server/enabled-connections "['UUID']"
  ```
  6-Enabling Automatic Login (otherwise cant start from ssh): [steps][df5]
  
  [df5]: <https://vitux.com/how-to-enable-disable-automatic-login-in-ubuntu-18-04-lts/>
  
  - Open the custom.conf file in the Nano editor
  ```sh
  $ sudo nano /etc/gdm3/custom.conf
  ```
  - commented-out these lines and replace user1 with your username (mine is nano)
  ```sh
  AutomaticLoginEnable = true
  AutomaticLogin = nano
  ```
  7- Reboot and start Vino
  ```sh
  $ sudo reboot
  $ export DISPLAY=:0 && /usr/lib/vino/vino-server
  ```
  - Open VNC viewer with ip of nano and port number 5900 (mine is 192.168.43.166:5900 from  Real VNC)
  
  - From GUI open startup application and add (othervise we have to manualy start vnc everytime)
  ```sh
  /usr/lib/vino/vino-server
  ```
  8- To fix low resolution issue modify xorg.conf file
  ```sh
  $ sudo nano /etc/X11/xorg.conf
  ```
  and add these lines to end of the file (you can also choose resolution by changing 1280 800)
  ```sh
  Section "Monitor"
     Identifier "DSI-0"
     Option    "Ignore"
  EndSection

  Section "Screen"
     Identifier    "Default Screen"
     Monitor        "Configured Monitor"
     Device        "Default Device"
     SubSection "Display"
         Depth    24
         Virtual 1280 800
     EndSubSection
  EndSection
  ```
  9- Reboot Jetson Nano and login via VNC
  ```sh
  $ sudo reboot
  ```
  - ¯\\\_(ツ)\_/¯
