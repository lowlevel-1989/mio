### documentación oficial

- https://ffmpeg.org/ffmpeg-all.html
- https://ffmpeg.org/ffmpeg-devices.html#lavfi
- https://trac.ffmpeg.org/wiki/HWAccelIntro
- https://trac.ffmpeg.org/wiki/Hardware/VAAPI
- https://trac.ffmpeg.org/wiki/Hardware/QuickSync

~~~
$ ffmpeg [global_options] {[input_file_options] -i input_url} ... \
	{[output_file_options] output_url} ...
~~~


~~~
 _______              ______________

|       |            |              |
| input |  demuxer   | encoded data |   decoder
| file  | ---------> | packets      | -----+
|_______|            |______________|      |
                                           v
                                       _________
                                      |         |
                                      | decoded |
                                      | frames  |
                                      |_________|
 ________             ______________       |
|        |           |              |      |
| output | <-------- | encoded data | <----+
| file   |   muxer   | packets      |   encoder
|________|           |______________|

~~~

### Complex filtergraphs

~~~
 _________
|         |
| input 0 |\                    __________
|_________| \                  |          |
             \   _________    /| output 0 |
              \ |         |  / |__________|
 _________     \| complex | /
|         |     |         |/
| input 1 |---->| filter  |\
|_________|     |         | \   __________
               /| graph   |  \ |          |
              / |         |   \| output 1 |
 _________   /  |_________|    |__________|
|         | /
| input 2 |/
|_________|

~~~

### overlay

~~~
El overlay filtro requiere exactamente dos entradas de    video,
pero ninguna está especificada, por lo que se utilizan las dos
primeras secuencias de video disponibles
~~~

###          https://ffmpeg.org/ffmpeg-all.html
### REF: 5.1 Especificadores de flujo <- documentacion importante


### listar dispositivos

~~~
$ v4l2-ctl --list-devices
$ v4l2-ctl -d /dev/video3 --list-formats-ext
~~~

### ver decodes habilitados

~~~
$ ffmpeg -decoders
~~~

### ver decodes habilitados

~~~
$ ffmpeg -encoders
$ ffmpeg -h encoder=h264_qsv
~~~

### capturar con el cpu

~~~
$ ffmpeg -f alsa  -i hw:1    -f v4l2 -framerate 60 -video_size 1920x1080 -input_format mjpeg -i /dev/video5 -vcodec copy output.mkv
$ ffmpeg -f pulse -i default -f v4l2 -framerate 60 -video_size 1920x1080 -input_format mjpeg -i /dev/video5 -vcodec copy output.mkv
~~~


### intel Quick Sync va-api

### capturar con tarjeta de video
### intel /dev/dri/renderD128
### amd   /dev/dri/renderD129

~~~
$ ffmpeg -vaapi_device /dev/dri/renderD128 -video_size 1280x720 -i /dev/video5 -filter_complex format=nv12,hwupload -vcodec h264_vaapi output.mkv
~~~

### intel Quick Sync qsv

~~~
$ ffmpeg -y -f alsa -i hw:1 -init_hw_device qsv=hw -filter_hw_device hw     \
	-f v4l2 -i /dev/video5 -filter_complex hwupload=extra_hw_frames=64  \
	-vcodec h264_qsv -global_quality 15 -preset 7 -profile high output.mkv
~~~

### Configuración utilizada

~~~
$ ffmpeg -y -f alsa -i hw:1 -init_hw_device qsv=hw -filter_hw_device hw \
	-f v4l2 -i /dev/video5 -f v4l2 -i /dev/video3                   \
	-filter_complex overlay=1590:10,hwupload=extra_hw_frames=64     \
	-vcodec h264_qsv -global_quality 15 -preset 7 -profile high output.mp4
~~~

### NOTA: extra_hw_frames=64 <- el valor no me queda claro su función

### alsa mixer

~~~
$ alsamixer
~~~

### jack para control de audio

~~~
$ pactl list
$ pactl list sink
$ pactl list source

$ pactl load-module   module-jack-sink
$ pactl unload-module module-jack-sink

$ pactl load-module   module-jack-source
$ pactl unload-module module-jack-source

$ cat /proc/asound/cards
$ alsa_in -j 'Console Source' -d hw:1
~~~

### loopback audio

~~~
$ pactl load-module   module-loopback
$ pactl unload-module module-loopback
~~~

### loopback video
~~~
$ sudo modprobe v4l2loopback
~~~

### ip webcam
~~~
$ gst-launch-1.0 souphttpsrc location="http://192.168.0.11:8080/video" is_live=true ! jpegdec ! autovideosink
$ gst-launch-1.0 souphttpsrc location="http://192.168.0.11:8080/video" is_live=true ! jpegdec ! videoconvert ! v4l2sink device=/dev/video1
~~~
