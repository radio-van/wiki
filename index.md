# Developing
## general
  * [ai](developing/ai.md)
  * [common](developing/common.md)
  * [compilation](developing/compilation.md)
  * [math](developing/math.md)
  * [network](developing/network.md)
  * [regex](developing/regex.md)
## backend
  * [celery](developing/celery.md)
  * [databases](developing/databases.md)
  * [django](developing/django.md)
  * [asgi/wsgi](developing/asgi_wsgi.md)
  * [python](developing/python.md)
  * [rabbitMQ](developing/rabbitmq.md)
  * [redis](developing/redis.md)
  * [YAML](developing/yaml.md)
## frontend
  * [JS](developing/js.md)
  * [HTML](developing/html.md)
  * [requests](developing/requests.md)
  * [grafana](developing/grafana.md)

# Facts
  * [facts](general/facts.md)

# General
  * [encryption](general/encryption.md)

# Hardware
  * [android](hardware/android.md)
  * [arduino](hardware/arduino.md)
  * [BIOS](hardware/bios.md)
  * [CPU](hardware/cpu.md)
  * [GPU](hardware/gpu.md)
  * [GPD](hardware/gpd.md)
  * [KaiOS](hardware/kaios.md)
  * [mac](hardware/mac.md)
  * [power](hardware/power.md)
  * [radio](hardware/radio.md)
  * [Remarkable](hardware/Remarkable.md)
  * [thinkpad](hardware/thinkpad.md)
  * [video](hardware/video.md)
  * [usb-modems](hardware/modems.md)
  * [xiaomi vacuum](hardware/xiaomi_vacuum.md)

# BSD
  * [pfsense](bsd/pfsense.md)


# Linux
  * [clipboard](linux/clipboard.md)
  * [containers](linux/containers.md)
  * [installation](linux/installation.md)
  * [history](linux/history.md)
  * [filesystem](linux/filesystem.md)
  * [kernel](linux/kernel.md)
  * [keyboard](linux/keyboard.md)
  * [loading](linux/loading.md)
  * [logs](linux/logs.md)
  * [login](linux/login.md)
  * [memory](linux/memory.md)
  * [networking](linux/networking.md)
  * [packages](linux/packages.md)
  * [printing](linux/printing.md)
  * [power](linux/power.md)
  * [sound](linux/sound.md)
  * [terminal](linux/terminal.md)
  * [tips&tricks](linux/tips_tricks.md)
  * [security](linux/security.md)
  * [system](linux/system.md)
  * [usb](linux/usb.md)
  * [video](linux/video.md)
  * [XDG](linux/XDG.md)

## specific distro
  * [android](linux/android.md)
  * [proxmox](linux/proxmox.md)
  * [openwrt](linux/openwrt.md)
  * [truenas](linux/truenas.md)


# Mac OS
  * [apps](macos/apps.md)
  * [installation](macos/installation.md)
  * [login](macos/login.md)
  * [networking](macos/networking.md)
  * [system](macos/system.md)

# Windows
  * [tips&tricks](windows/tips_tricks.md)

# Packages
  * [Home-Assistant](packages/home-assistant.md)
  * [Nextcloud](packages/nextcloud.md)
  * [Retropie](packages/retropie.md)

# Protocols
  * [DNS](protocols/DNS.md)
  * [email](protocols/email.md)

# Services
* audio recognition (aka Shazam) [audd.io](https://audd.io)
    ```bash
    curl https://api.audd.io/ -F url='https://some/mp3/file.mp3' -F return='apple_music,spotify' -F api_token='test'
    ```
* file exchange (aka AirDrop) [sharedrop.io](https://sharedrop.io) and [pairdrop.net](https://pairdrop.net)
* file exchange (aka Dropbox) [temp.sh](https://temp.sh) (file expires after 3 days)
    ```bash
    curl -T <file> temp.sh
    ```
* encoding converter [0xcc.net](http://0xcc.net/jsescape/)
* search engine [yacy](yacy.net)
* location name by coordinates:
    ```bash
    curl 'https://nominatim.openstreetmap.org/reverse?lat=<LAT>&lon=<LON>&format=json'
    ```
    current LON and LAT can be obtained with **geoclue**: `/usr/lib/geoclue-2.0/demos/where-am-i`

# Utilities
  * [adb](utilities/adb.md)
  * [ansible](utilities/ansible.md)
  * [aria](utilities/aria.md)
  * [awk](utilities/awk.md)
  * [bash](utilities/bash.md)
  * [beets](utilities/beets.md)
  * [brew](utilities/brew.md)
  * [clonezilla](utilities/clonezilla.md)
  * [crontab](utilities/crontab.md)
  * [curl](utilities/curl.md)
  * [dd](utilities/dd.md)
  * [discord](utilities/discord.md)
  * [docker](utilities/docker.md)
  * [dunst](utilities/dunst.md)
  * [exiftool](utilities/exiftool.md)
  * [ffmpeg](utilities/ffmpeg.md)
  * [flatpak](utilities/flatpak.md)
  * [find](utilities/find.md)
  * [firefox](utilities/firefox.md)
  * [frida](utilities/frida.md)
  * [fzf](utilities/fzf.md)
  * [git](utilities/git.md) 
  * [gcloud](utilities/gcloud.md) 
  * [gnupg](utilities/gnupg.md) 
  * [imagemagick](utilities/imagemagick.md) 
  * [iptables](utilities/iptables.md) 
  * [ledger](utilities/ledger.md)
  * [luks](utilities/luks.md)
  * [lvm](utilities/lvm.md)
  * [lxc](utilities/lxc.md) 
  * [keynav](utilities/keynav.md) 
  * [kubernetes](utilities/kubernetes.md) 
  * [mitm](utilities/mitm.md) 
  * [mopidy](utilities/mopidy.md) 
  * [mpv](utilities/mpv.md) 
  * [mutt](utilities/mutt.md) 
  * [netcat](utilities/netcat.md) 
  * [newsboat](utilities/newsboat.md) 
  * [netctl](utilities/netctl.md) 
  * [nmap](utilities/nmap.md) 
  * [pass](utilities/pass.md)
  * [qemu](utilities/qemu.md)
  * [rsync](utilities/rsync.md)
  * [sed](utilities/sed.md)
  * [smartctl](utilities/smartctl.md)
  * [ssh](utilities/ssh.md)
  * [sudo](utilities/sudo.md)
  * [syncthing](utilities/syncthing.md)
  * [systemd](utilities/systemd.md)
  * [tailscale](utilities/tailscale.md)
  * [tmux](utilities/tmux.md) 
  * [tar](utilities/tar.md) 
  * [tcpdump](utilities/tcpdump.md) 
  * [top](utilities/top.md) 
  * [tor](utilities/tor.md) 
  * [urxvt](utilities/urxvt.md)
  * [ventoy](utilities/ventoy.md)
  * [vim](utilities/vim.md)
  * [wget](utilities/wget.md)
  * [wireguard](utilities/wireguard.md)
  * [yc](utilities/yc.md)
  * [yt-dlp](utilities/yt-dlp.md)
  * [zip](utilities/zip.md)


# TODO
* nohup
* rename/repren
* shuf
* sort `c-v` tab
* strings
* iconv
* uconv
* split
* datediff
* add
* strptime
* iostat -mxz
* dstat
* mtr
* iftop
* nethogs
* siege
* strace
* ltrace
* ldd
* sar
* expr
* look
* paste
* join
* fmt
* pr
* fold
* column
* seq
* toe
* slurm
* logrotate
* comm
* pv
* apg
* cssh
* dstat
