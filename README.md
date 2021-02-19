# raspi-webcam

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


corrction of sound delay...

```
/usr/bin/raspivid -t 0 -n -a 12 -b 2500000 -pf high --mode 5 -fps 25 -g 50 --flush -ih -o - | ffmpeg \
-f alsa -thread_queue_size 1024 -ar 44100 -channel_layout mono -ac 1 -i hw:1,0 -itsoffset 15 \
-f h264 -thread_queue_size 1024 -i - \
-map 1:0 -c:v copy -map 0:0 -c:a aac -b:a 64k \
-f rtsp rtsp://localhost:8554/streaming
```
