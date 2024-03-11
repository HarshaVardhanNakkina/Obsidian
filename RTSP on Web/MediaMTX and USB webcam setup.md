- get the list of devices like this
```bash
ffmpeg -list_devices true -f dshow -i dummy
```

- run the MediaMTX server like this
```bash
docker run --rm -it -e MTX_PROTOCOLS=tcp -p 8554:8554 -p 1936:1935 -p 8888:8888 -p 8889:8889 aler9/rtsp-simple-server
```

- publish live webcam stream using ffmpeg like this:
```bash
ffmpeg -f dshow -i video="Logi C310 HD WebCam" -rtsp_transport tcp -c:v libx264 -preset ultrafast -tune zerolatency -an -f rtsp rtsp://localhost:8554/ship-stream
```

- run the SRS server like this
```bash
docker run --rm -it -p 1937:1935 -p 1986:1985 -p 8090:8080 ossrs/srs:v6 ./objs/srs -c conf/docker.conf
```

- publish to SRS server like this
```bash
ffmpeg -rtsp_transport tcp -i rtsp://localhost:8554/ship-stream -c:v copy -c:a aac -f flv rtmp://localhost:1937/live/ship-stream
```
