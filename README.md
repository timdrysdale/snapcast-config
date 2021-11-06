# snapcast-config
Notes for self on my setup of librespot and snapcast on ubuntu

## Background

Snapcast stopped working a while ago and I thought I'd make some notes about theconfig as I try to set it up again, to speed up this process if it breaks again in future.

Symptoms: connecting to spotify premium as 'librespot on ubuntu', but playing tracks does not appear to work

```
Nov 06 11:41:57 ubuntu systemd[1]: Started librespot.
Nov 06 11:41:57 ubuntu librespot[1407]: [2021-11-06T11:41:57Z INFO  librespot] librespot UNKNOWN (UNKNOWN). Built on 2020-10-10. Build ID: LI1nt55P
Nov 06 12:25:40 ubuntu librespot[1407]: [2021-11-06T12:25:40Z INFO  librespot_core::session] Connecting to AP "gew1-accesspoint-a-dhkn.ap.spotify.com:4070"
Nov 06 12:25:40 ubuntu librespot[1407]: [2021-11-06T12:25:40Z INFO  librespot_core::session] Authenticated as "treddlytim" !
Nov 06 12:25:40 ubuntu librespot[1407]: [2021-11-06T12:25:40Z INFO  librespot_core::session] Country: "GB"
Nov 06 12:25:41 ubuntu librespot[1407]: [2021-11-06T12:25:41Z INFO  librespot_connect::spirc] No more tracks left in queue
Nov 06 13:00:10 ubuntu librespot[1407]: [2021-11-06T13:00:10Z INFO  librespot_connect::spirc] No more tracks left in queue
```
The track does not play in the spotify app on Ubuntu desktop, and the systemctl status on the raspberry pi says there are no more tracks.


## Guide

I probably originally followed this guide here: https://oyvn.no/multi-room-audio-with-snapcast which is why I didn't write this up the first time around.

## The fix....

It looks like librespot is outdated, so let's start with updating it. The guide talks about compiling slowly on the rpi - but I figure, I can make a cup of tea, so on the rpi server:
```
cargo install librespot
```
Ok - make that a long cup of tea.

```
$ which librespot
/home/ubuntu/.cargo/bin/librespot
```

So it seems like no need to move it as already on path - but the service file specifically refers to `/usr/local/bin/librespot'

```
sudo mv /home/ubuntu/.cargo/bin/librespot /usr/local/bin/librespot
```


```
$ sudo systemctl restart librespot
$ sudo systemctl status librespot
<snip>
Nov 06 13:24:11 ubuntu systemd[1]: Started librespot.
Nov 06 13:24:11 ubuntu librespot[6636]: [2021-11-06T13:24:11Z INFO  librespot] librespot 0.3.1 UNKNOWN (Built on 2021-11-06, Build ID: 82oxdXBd, Profile: release)
```

Librespot on ubuntu is not showing in the webclient list of players, but is showing in the native app. Selecting a new track and switching to librespot on uubutu, then checking status shows it looks like the track is getting to the librespot service now

```
Nov 06 13:25:56 ubuntu librespot[6660]: [2021-11-06T13:25:56Z INFO  librespot_playback::player] Loading <Roxanne> with Spotify URI <spotify:track:7avW5jZNXipvuB23HuwQ>
Nov 06 13:25:56 ubuntu librespot[6660]: [2021-11-06T13:25:56Z INFO  librespot_playback::player] <Roxanne> (192666 ms) loaded
```

Snapcast still looks idle (http://192.168.0.62:1780/), but the track is now playing directly out of the raspberry pi.

restart the snapserver just in case it needs it...

```
sudo systemctl restart snapserver
```

check hardware devices:

```
**** List of PLAYBACK Hardware Devices ****
card 0: Headphones [bcm2835 Headphones], device 0: bcm2835 Headphones [bcm2835 Headphones]
  Subdevices: 7/8
  Subdevice #0: subdevice #0
  Subdevice #1: subdevice #1
  Subdevice #2: subdevice #2
  Subdevice #3: subdevice #3
  Subdevice #4: subdevice #4
  Subdevice #5: subdevice #5
  Subdevice #6: subdevice #6
  Subdevice #7: subdevice #7
```

Now connect instead to the snapcast stream in the desktop spotify client .... and the sound is coming out of all the connected clients.

Adjusting the volume from the mobile app works fine too ... so just seems like librespot needed updating and the snapserver restarting....


## Deprecated stuff

### librespot

I updated librespot on the desktop - although this is now disabled in favour of running it on a raspberry pi. 

0. Obtain latest version
```
cargo install librespot
sudo mv /home/tim/.cargo/bin/librespot /usr/local/bin/librespot
```
0. Restart librespot service
```
sudo systemctl restart librespot
```
0. Check latest version is being used
```
sudo systemctl status librespot
```
with the result:
```
Nov 06 12:38:01 w6625 systemd[1]: Started Librespot connection.
Nov 06 12:38:01 w6625 librespot[2524644]: [2021-11-06T12:38:01Z INFO  librespot] librespot 0.3.1 UNKNOWN (Built on 2021-11-06, Build ID: iBsKY6Iy, Profile: release)
```
 
The service file `/lib/systemd/system/librespot.service`: 

```
[Unit]
Description=Librespot connection
After=network.target 

[Service]
ExecStart=librespot -n "Librespot" -b 320 -c ./cache --enable-volume-normalisation --initial-volume 75 --backend pipe
Restart=on-failure

[Install]
WantedBy=snapserver.service
```

