
# Netgear WNDR4500 Recovery

I flashed dd-wrt on my Netgear WNDR4500 router a couple years ago, and it
ran fine until some months ago when apparently a power outage corrupted the
firmware, and the router stopped working as intended. It became essentialy a
network switch, and the LED kept blinking a green light. Other that this, only
the lan port LEDs were lit when a cable was connected, and wifi was completely
off. Also, there was no GUI accessible, no DHCP, and the router reverted to its
default IP (192.168.1.1).

Searching around with Google, I found a handful of useful links explaining what
I could do to recover my router's firmware, and they gave me a little understanding
about what state my router was on.

https://wiki.dd-wrt.com/wiki/index.php/Netgear_WNDR4500
https://wiki.dd-wrt.com/wiki/index.php/TFTP_flash
https://wiki.dd-wrt.com/wiki/index.php/Recover_from_a_Bad_Flash
https://kb.netgear.com/000059633/How-to-upload-firmware-to-a-NETGEAR-router-using-TFTP-client
https://www.kunxi.org/2015/12/reinstall-netgear-n900-firmware-with-a-mac/
https://medium.com/@mr_joshkendrick/fixing-netgear-wndr4500-blinking-green-power-light-corrupted-firmware-a907e31bbeee

Basically, what I could summarize is that the firmware was corrupted, but the
bootloader CFE was running, and that it allowed flashing the firmware using an
utility called TFTP (which is actually a file transfer protocol based on UDP
and uses port 69 by default) to transfer the firmware during a narrow window
of some seconds during the bootloader initialization.

The first time my firmware was corrupted, I flashed back the dd-wrt firmware
I had originally flashed my router with, and it worked without much hassle.

I couple days ago the problem ocurred again, but I couldn't reflash the original
dd-wrt firmware back. I tried every combination of hard-resets, tftp options and
timing of the flash commands, but nothing worked. When I finally decided to try
the stock firmware (downloaded from the netgear website), it worked after a couple
tries (it usually require some tries because the timing is sensitive).

So this is what I tried and what worked:

- I'm using macOS 10.14.6, and it comes with the tftp command by default
- My Netgear WNDR4500 is a version 1 router
- The ping command (also default) can be used to indicate what state the route
is in. The CFE bootloader has TTL=100, and the loaded application firmware
has TTL=64. In my case, the following response was printed:
```64 bytes from 192.168.1.1: icmp_seq=502 ttl=100 time=0.243 ms```
- The full collection of TFTP command options that I've recommended were:
```$ tftp
connect 192.168.1.1
binary
rexmt 1
timeout 60
put dd-wrt.XXXXXXXX.chk```
- I managed to make it worked after all (with the stock firmware) using the
following commands:
```$ tftp -e 192.168.1.1
timeout 60
put dd-wrt.XXXXXXXX.chk```
- I did a hard-reset just before trying to flash just to be sure, but it
doesn't seem so be always required.
- After doing the reset, and turning off and on the router one last time,
I could only flash after the LED light was blinking green, and a couple seconds
later I started to receive ping responses (with TTL=100). As soon as I had
a ping response, I ran the tftp commands. It sent the data in about 10 seconds,
and soon after the router LED lights started flashing in various patterns all
around. When the flash didn't work (when I tried the dd-wrt firmware), the main
LED light just kept blinking green forever.
- The dd-wrt firmware I tried to flash was `dd-wrt.v24-26138_NEWD-2_K3.x_mega-WNDR4500.chk`,
and it successfully flashed from the stock firmware GUI, just didn't worked
when sending via TFTP (file is in the repo)
- The stock firmware I used was `WNDR4500-V1.0.1.46_1.0.76.chk`, downloaded
from the netgear website (file is in the repo also).

So this was my experience recovering this router, any questions, suggestions
or issues with my explanation, please report by opening an issue in this repository.

