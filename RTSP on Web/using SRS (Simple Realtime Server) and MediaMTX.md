[SRS](https://ossrs.io/lts/en-us/docs/v4/doc/getting-started) is a simple, high-efficiency, real-time video server supporting RTMP, WebRTC, HLS, HTTP-FLV, SRT, MPEG-DASH, and GB28181.
SRS can be run as Docker container, like this:

```bash
docker run --rm -it -p 1935:1935 -p 1985:1985 -p 8080:8080 \
    ossrs/srs:v6 ./objs/srs -c conf/docker.conf
```

A live stream can be published to this container, like this:
```bash
ffmpeg -re -i <input file/stream> -c copy -f flv rtmp://localhost/live/<stream_key>
```

The stream can be played:
- RTMP (with VLC): rtmp://localhost/live/<stream_key>
- HTML5 (HTTP-FLV): http://localhost:8080/live/<stream_key>.flv
	- for this you need flash video player JS library like [flv-h265](https://www.npmjs.com/package/flv-h265)
- HTML5 (HLS): http://localhost:8080/live/<stream_key>.m3u8

SRS also allows to stream in WebRTC, should be explored.

---

[MediaMTX](https://github.com/bluenviron/mediamtx) is a Ready-to-use RTSP / RTMP / LL-HLS / WebRTC server and proxy that allows to read, publish and proxy video and audio streams. Formerly known as rtsp-simple-server.

MediaMTX can be run as Docker container, like this:
```bash
docker run --rm -it -e MTX_PROTOCOLS=tcp -p 8554:8554 -p 1935:1935 -p 8888:8888 -p 8889:8889 aler9/rtsp-simple-server
```

> The `--network=host` flag is mandatory since Docker can change the source port of UDP packets for routing reasons, and this doesn't allow the server to find out the author of the packets. This issue can be avoided by disabling the UDP transport protocol

A live stream can be published to this container, like this:
```bash
ffmpeg -re -stream_loop -1 -i file.ts -c copy -f rtsp rtsp://localhost:8554/mystream
```