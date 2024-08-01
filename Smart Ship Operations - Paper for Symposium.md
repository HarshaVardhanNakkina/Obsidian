# RTSP stream to browser

## Using HLS (HTTP Live Streaming)

`HTTP Live Streaming` protocol AKA `HLS` is a streaming protocol developed by Apple. HTTP Live Streaming (HLS) is a highly popular video streaming protocol. Despite its name, it is utilized for both on-demand and live streaming. HLS divides video files into smaller, downloadable HTTP files and delivers them via the HTTP protocol. A list of available streams, encoded at various bit rates, is provided to the client using an extended M3U playlist. Client devices then load these files and play them back as video.

One advantage of HLS is its compatibility with all Internet-connected devices, as they all support HTTP. This makes HLS easier to implement compared to streaming protocols that need specialized servers. Another benefit is that HLS can adjust video quality based on network conditions without interrupting playback. This is why video quality can fluctuate while a user is watching. This capability, known as "adaptive bitrate video delivery" or "adaptive bitrate streaming," prevents slow network conditions from completely halting video playback.

>[!note] Note:
>One of the best things about HLS is that it supports live, on-demand, and event- playlists. This means that you can use the same protocol to live stream an event as well as an existing video file stored on your server. However, since HLS prioritizes video quality over latency, the end-to-end usage of the streaming protocol can lead to a delay of up to 45 seconds in streams. This challenge is typically resolved by using a different protocol (such as RTMP) for ingestion. It’s a part of the LL-HLS extension and brings the latency down to approx. 2 seconds.

### converting RTSP stream to HLS

There are multiple streaming servers that does this like Wowza, Red5. A simple and quick solution is to use `Nginx` with `nginx-rtmp-module`. First you need to create a RTMP server like this:
```nginx
rtmp {
	server {
		listen 1935;

		application live {
			live on;

			hls on;
			hls_path /tmp/hls;
			hls_fragment 15s;
		}
	}
}
```

this will tell Nginx to start listening on the port `1935` (standard RTMP publishing port). When an RTSP stream is published on this port, it gets broken down into fragments of defined size and these fragments are stored in the specified. A playlist file `m3u8` is created with the list of all these fragments which client read and download to play the video as mentioned earlier

Create another server that will serve this HLS stream:
```nginx
	server {
		listen 80;

		location / {
			root /tmp/hls;
			expires $expires;
		}
	}
```

publish a live stream to Nginx using `ffmpeg`
```bash
ffmpeg -re -i <input> -c:v copy -c:a aac -ar 44100 -ac 1 -f flv rtmp://localhost/hls/<stream_name>
```

One can play this HLS stream in the browser natively using Media Source Extensions (MSE), but there are libraries that do that for you

access the published stream as HLS using `video.js` library:
```html
  <video id="my-video" class="video-js" controls preload="auto" width="1280" height="720" data-setup="{}">
    <source src="http://localhost/hls/<stram_name>.m3u8" type="application/vnd.apple.mpegurl" />
    <p class="vjs-no-js">
      To view this video please enable JavaScript, and consider upgrading to a
      web browser that
      <a href="https://videojs.com/html5-video-support/" target="_blank">supports HTML5 video</a>
    </p>
  </video>
```

However, since HLS prioritizes video quality over latency, the end-to-end usage of the streaming protocol can lead to a delay of up to 45 seconds in streams. One of the biggest challenge of using HLS is **latency**.

>[!danger] Challenge:
>Since HLS breaks down the stream and stores them as chunks of specified duration, the client has to wait for that specified duration before even starting the playback. Along with that, HLS keeps a buffer of few segments for seamless delivery and experience

There are several ways to solve this problem:
- reducing the fragment size can help reduce the latency
- pre-loading the segments beforehand when you know the location of the next stream
- replacing the encoder for a faster one, which is what `Low Latency HLS (LLHLS)` protocol does

However, we didn't go down this road further.

## Encapsulating RTSP into FLV (Adobe Flash Video)

`http-flv (HTTP FLV live stream)` is a mechanism to deliver a live stream in `flv (flash video)` format by HTTP.

>[!note] Note:
>Unlike file downloads, live streams have an indefinite or uncertain length, so they are usually implemented using the HTTP Chunked protocol.

There are several media servers that support this method, such as [SRS](https://ossrs.io/lts/en-us/docs/v4/doc/getting-started), [MediaMTX](https://github.com/bluenviron/mediamtx) . Both can run as docker containers like this:
```bash
docker run --rm -it -p 1935:1935 -p 1985:1985 -p 8080:8080 ossrs/srs:v6 ./objs/srs -c conf/docker.conf
```

```bash
docker run --rm -it -e MTX_PROTOCOLS=tcp -p 8554:8554 -p 1935:1935 -p 8888:8888 -p 8889:8889 aler9/rtsp-simple-server
```

A live stream can be published to the container, like this:
```bash
ffmpeg -re -i <input file/stream> -c copy -f rtsp rtmp://localhost/live/<stream_key>
```

The HTTP-FLV stream can be accessed by using http://localhost:8080/live/<stream_key>.flv

There are several challenges with this approach too:
- HTTP-FLV streaming has the same delay as RTSP (~ 3-5 seconds)
- Adobe Flash Video format has come to end of life in 2021, so using adobe flash player extension to play the flv video poses a security risk.
- for this you need flash video player JS library like [flv-h265](https://www.npmjs.com/package/flv-h265)

## WebRTC

`WebRTC` is a real-time communication protocol developed and open-sources by Google.

>[!note] Note:
>WebRTC is essentially a standard for direct communication between two web browsers, mainly consisting of signaling and media protocols. Signaling deals with the negotiation of capabilities between two devices, such as supported encoding and decoding abilities. Media handles the encryption and low-latency transmission of media packets between devices.

WebRTC is a standard for direct communication between two web browsers, primarily involving signaling and media protocols. Signaling manages the negotiation of capabilities between devices, such as supported encoding and decoding abilities, while media handles the encryption and low-latency transmission of media packets. Additionally, WebRTC incorporates audio processing technologies like 3A, network congestion control methods such as NACK, FEC, and GCC, audio and video encoding and decoding, as well as technologies for smooth and low-latency playback.

There are several open-source implementations of WebRTC that made the protocol accessible, not only to the web browsers, but to mobile browsers and several native libraries like Python, Java, Go etc.

In reality, it is hard for two clients to communicate through WebRTC in present internet scenario. The data has to go through multiple networks, firewalls etc. making it difficult to maintain quality transmission of data. So for practical application, the data needs to be relayed through external servers.

There are several types of WebRTC servers optimized for certain use cases. Mostly it needs a `Signalling Server` which helps identifying and establishing a connection with a client, one of the transmission servers like `TURN Server`, `SFU Server` or `MCU Server` each of which are optimized for specific use case.

All of which is difficult to set up, and might encounter compatibility issues.

## Transferring frames over Web Socket

A WebSocket is a communication protocol that provides full-duplex communication channels over a single [TCP](https://www.pubnub.com/guides/tcp-ip/) connection. It enables real-time, event-driven connection between a client and a server.

Unlike traditional HTTP software, which follows a request-response model, Web Sockets allow two-way (bi-directional) communication. This means that the client and the server can send data to each other anytime without continuous polling.

The AI application captures the stream frame by frame using cv2. Each frame is a NumPy array. After the AI application processes each frame for object detection and classification, it can be sent to the front end along with the inferences through a web socket in JSON format.

# Displaying frames on the browser

## Base64

Before sending the frame to the browser, it has to be converted into a browser-understandable format. One such format is Base-64 encoding. First, the NumPy array has to be encoded into an image format such as JPEG, then this image has to be converted to a Base-64 encoded string.

```json
{
	"frame": "<base64 encoded string...>",
	"other data": "..." 
}
```

```html
<img src="data:image/jpeg;base64, <base64 encoded string...>" alt="Frame" />
```

The encoding algorithm of base64 is straightforward. It takes as an input one binary sequence and outputs another binary sequence. It doesn't care what the original bytes stood for – be it an image or a PDF or whatever. It just goes through the original binary stuff, splits it into chunks of 6 bits, and converts every chunk into a safe textual character (or, strictly speaking, a binary sequence that represents such a character). A "safe" character refers to one from a very limited set chosen from the ASCII. Decoding performs similar binary operations but in reverse. The restored data is guaranteed to be exactly the same as before encoding.

At the same time, base64 is a costly instrument. It makes data about 33% larger in terms of memory usage. So base64 is one of these little things that make software **slow**. That's why you should use it only when it's absolutely necessary. Base64 converts every 6-bit chunk of original data into a single ASCII character. If such a character is 8 bits long, it means that you are wasting 2 bits per every original chunk which is about 33%.

## ArrayBuffer and Blobs

The **`ArrayBuffer`** object is used to represent a generic raw binary data buffer. It is an array of bytes, often referred to in other languages as a "byte array". First, the NumPy array has to be encoded into an image format such as JPEG, then this image has to be converted to a byte array. This byte array will received as array buffer on the front end.

The contents of an `ArrayBuffer` cannot be manipulate directly; instead, it has to be converted into one of the [typed array objects](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray) or a [`DataView`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/DataView) object which represents the buffer in a specific format, and use that to read and write the contents of the buffer. The array buffer, in this case, is converted into an `Uint8Array` and this array is then converted into a `Blob`. Then the `createObjectURL()` static method of the [`URL`](https://developer.mozilla.org/en-US/docs/Web/API/URL) interface is used to create string containing a URL representing the blob. This ULR can then be used in the image tag to display the image.

>[!note] Note:
> Each time you call `createObjectURL()`, a new object URL is created, even if you've already created one for the same object. Each of these must be released by calling [`URL.revokeObjectURL()`](https://developer.mozilla.org/en-US/docs/Web/API/URL/revokeObjectURL_static "URL.revokeObjectURL()") when you no longer need them.
> Browsers will release object URLs automatically when the document is unloaded; however, for optimal performance and memory usage, if there are safe times when you can explicitly unload them, you should do so.

```javascript
let bytes = new Uint8Array("<ArrayBuffer>")
let url = URL.createObjectURL("<Blob>");
```

```html
<img src="<url>" alt="Frame"/>
```

This method of displaying the images on the browser is memory-efficient and performant. Although there are several other, perhaps more performant, methods; this will suffice and easy for our use case.

# Displaying bounding boxes and inferred data

Now that the frame is on the user's browser, we need to figure out a way to display bounding boxes around the detected objects and somehow, display the inferred data about the object in an efficient and user-friendly manner.

## Canvas

The **Canvas API** provides a means for drawing graphics via [JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript) and the [HTML](https://developer.mozilla.org/en-US/docs/Web/HTML) [`<canvas>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/canvas) element. Among other things, it can be used for animation, game graphics, data visualization, photo manipulation, and real-time video processing.

The Canvas API largely focuses on 2D graphics. The [WebGL API](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API), which also uses the `<canvas>` element, draws hardware-accelerated 2D and 3D graphics.