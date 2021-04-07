# raspi-webcam

## disable swap

`$ sudo dphys-swapfile swapoff && sudo dphys-swapfile uninstall && update-rc.d dphys-swapfile remove && systemctl disable dphys-swapfile`

## mount NFS folder under Mac
`$ sudo mount -t nfs -o nolocks,resvport,locallocks 192.168.0.5:/nfs/dev ./CodeRemote`

## v4l2rtspserver

> [https://github.com/mpromonet/v4l2rtspserver](v4l2rtspserver)
> $ sudo apt install snapd
> $ sudo reboot
> $ sudo snap install v4l2rtspserver --edge



## rtsp-simple-server & ffmpeg streaming with audio

https://github.com/cdgriffith/pi_streaming_setup/tree/master

### rtsp

`sudo python3 streaming_setup.py --rtsp`

edit `/etc/systemd/system/stream_camera.service`

```
ffmpeg -nostdin -hide_banner -loglevel error \
-f v4l2 -input_format h264 -s 1280x720 -i /dev/video0 \
-f alsa -ar 44100 -channel_layout mono -ac 1 -i hw:1,0 \
-map 0:0 -c:v copy -map 1:0 -c:a aac -b:a 64k \
-f rtsp rtsp://localhost:8554/streaming
```
### Mpeg-dash

`sudo python3 streaming_setup.py`

edit `/etc/systemd/system/stream_camera.service`

```
ffmpeg -nostdin -hide_banner -loglevel error \
-f v4l2 -input_format h264 -s 1280x720 -i /dev/video0 \
-f alsa -ar 44100 -channel_layout mono -ac 1 -i hw:1,0 \
-map 0:0 -c:v copy -map 1:0 -c:a aac -b:a 64k \
-f dash -remove_at_exit 1 -window_size 5 -use_timeline 1 -use_template 1 -hls_playlist 1 /dev/shm/streaming/manifest.mpd
```

### RTSP

simple h264 stream from raspicam

```
/usr/bin/raspivid -t 0 -n -a 12 -b 1000000 -pf high --mode 5 -fps 15 -g 30 -ih -o - | \
ffmpeg -use_wallclock_as_timestamps 1 -fflags nobuffer -hide_banner -loglevel error \
-f h264 -framerate 15 -thread_queue_size 1024 -i - \
-c:v copy \
-f rtsp rtsp://localhost:8554/streaming
```

h264 raspicam correction of sound delay...

```
/usr/bin/raspivid -t 0 -n -a 12 -b 1000000 -pf high --mode 5 -fps 15 -g 30 -ih -rot 180 -o - | \
ffmpeg -use_wallclock_as_timestamps 1 -fflags nobuffer -nostdin -hide_banner -loglevel error \
-f alsa -thread_queue_size 1024 -ar 44100 -channel_layout mono -ac 1 -i hw:1,0 -itsoffset 15 \
-f h264 -thread_queue_size 1024 -i - \
-map 1:0 -c:v copy -map 0:0 -c:a aac -b:a 64k \
-f rtsp rtsp://localhost:8554/streaming1
```

h264 stream from v4l2 device

```
v4l2-ctl --set-fmt-video=width=1920,height=1080,pixelformat="H264" -d /dev/video1

ffmpeg -use_wallclock_as_timestamps 1 -fflags nobuffer -fflags +igndts -nostdin -hide_banner -loglevel error \
-f v4l2 -thread_queue_size 1024 -input_format h264 -s 1920x1080 -r 15 -i /dev/video1 \
-c:v copy \
-f rtsp rtsp://localhost:8554/streaming2
```

