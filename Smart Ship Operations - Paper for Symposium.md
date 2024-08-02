# Abstract

This document explores various methods for streaming real-time video content from an RTSP stream to a web browser, focusing on different protocols and techniques. The primary method discussed is HTTP Live Streaming (HLS), a widely adopted protocol developed by Apple that segments video into smaller files for adaptive bitrate streaming. While HLS offers broad compatibility and high-quality video delivery, it introduces latency challenges that can impact live streaming experiences. Solutions to mitigate this latency, such as reducing fragment size or using Low Latency HLS, are also discussed.

Alternative approaches include encapsulating RTSP streams into FLV format using HTTP-FLV and leveraging WebRTC for low-latency, peer-to-peer communication. Each method presents unique advantages and challenges, such as the deprecation of Flash for FLV playback or the complexity of setting up WebRTC infrastructure. The use of WebSockets for frame transmission is also explored, highlighting the need to convert video frames into browser-compatible formats like Base64 or ArrayBuffers for efficient rendering.

The document further discusses techniques for displaying video frames and associated metadata, such as bounding boxes for detected objects, using the Canvas API and CSS. The challenges of dynamic resizing and interactivity in the browser are addressed, with potential future enhancements identified, including the use of OffscreenCanvas and WebGL for improved performance. Overall, this document provides a comprehensive overview of the technologies and considerations involved in streaming RTSP content to a web browser.
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

The base64 encoding algorithm is relatively simple. It takes a binary sequence as input and produces another binary sequence as output. The algorithm doesn't consider the nature of the original data—whether it's an image, a PDF, or something else. It simply processes the binary data, dividing it into 6-bit segments and converting each segment into a corresponding "safe" textual character. These safe characters are selected from a small subset of the ASCII character set. Decoding follows a similar process in reverse, ensuring that the original data is perfectly restored.

However, base64 encoding has a drawback: it increases the size of the data by about 33%, which can lead to higher memory usage and slower performance. This inefficiency arises because each 6-bit segment of data is represented by an 8-bit ASCII character, effectively adding 2 extra bits for every segment. Due to this overhead, base64 should be used only when absolutely necessary.

## ArrayBuffer and Blobs

The **`ArrayBuffer`** object is designed to hold raw binary data as a generic buffer. Essentially, it's a sequence of bytes, similar to what is commonly called a "byte array" in other programming languages. To process a NumPy array for the frontend, it first needs to be encoded into an image format like JPEG, which is then converted into a byte array. This byte array is sent to the frontend as an ArrayBuffer.

Direct manipulation of an `ArrayBuffer` is not possible; instead, it needs to be transformed into a [typed array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray) or a [`DataView`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/DataView) object, which allows the data to be interpreted and managed in a specific format. For instance, the array buffer is transformed into a `Uint8Array`, which is then converted into a `Blob`. Subsequently, the `createObjectURL()` method from the [`URL`](https://developer.mozilla.org/en-US/docs/Web/API/URL) interface generates a string URL representing the blob, which can be used in an image tag to display the image.

>[!note] Note:
> Every time `createObjectURL()` is called, a unique object URL is generated, even if the same object has been used to create one before. To manage these URLs properly, it's important to release them by calling `URL.revokeObjectURL()` once they are no longer needed. Although browsers automatically release object URLs when the document is unloaded, explicitly revoking them at safe points can help optimize performance and reduce memory usage.

```javascript
let bytes = new Uint8Array("<ArrayBuffer>")
let url = URL.createObjectURL("<Blob>");
```

```html
<img src="<url>" alt="Frame"/>
```

This method of displaying the images on the browser is memory-efficient and performant. Although there are several other, perhaps more performant, methods; this will suffice and easy for our use case.

# Displaying bounding boxes and inferred data

Now that the frame is on the user's browser, we need to figure out a way to display bounding boxes around the detected objects and somehow, display the inferred data about the object in an efficient and user-friendly manner. Here is a snippet of inference data received. here the `xmin`, `ymin` etc. values represent the top-left and bottom-right position (relative to the frame) of the object detected in the frame.

```json
{
	"...": "...",
	"list_of_targets": [
		{
			"xmin": "...",
			"ymin": "...",
			"xmax": "...",
			"ymax": "...",
			"type_of_target": "...",
			"...": "..."
		}
	],
	"...": "..."
}
```

## The Canvas API

The **Canvas API** enables the drawing of graphics using [JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript) and the [HTML](https://developer.mozilla.org/en-US/docs/Web/HTML) [`<canvas>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/canvas) element. It is versatile, allowing for tasks such as animation, game graphics, data visualization, image editing, and real-time video processing.

While the Canvas API is primarily geared towards 2D graphics, the [WebGL API](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API), which also operates through the `<canvas>` element, is designed for rendering both 2D and 3D graphics with hardware acceleration.

Drawing simple on a basic canvas element is not going to help for the following reasons:
- Elements drawn on the canvas are not interactive, they can not have HTML events on them.
- Elements can not update their positions dynamically, to update a position of an element on a canvas, the canvas has to wiped clean and painted again with element's new position which may look like the image is flickering and it is not a good user experience.
- Elements drawn on a basic canvas cannot be animated

Although canvas API looks promising, it takes some expertise to set up the graphics using WebGL. This approach not necessarily benefit our use-case significantly but it is worth exploring at later point of time.

## CSS

The `x,y` coordinates received for the objects are relative the frame, meaning, if the frame's resolution is `1920x1080`, the objects' coordinates will be within `(0,0)` to `(1920,1080)` range.

Using CSS positioning an HTML element can be used a container for the frame and the objects. A container element can be created in which the frame and bounding boxes can be absolutely positioned to the container but the problem here is that the device you viewing the output in, may not have the same resolution the camera. For example, the camera can capture the frame at `1920x1080` resolution but the device the application is running on, might have a resolution of `1280x720`.

This problem can be solved by mapping the coordinates of the bounding boxes from their original resolution to the device's resolution. Once these new coordinates are calculated, bounding boxes can be absolutely positioned to the container. Since the image is styled to take all the available width and height, it's parent (the container) also takes the same width and height and therefore the bounding boxes, most certainly, will be drawn at same position as they were originally inferred.

Since, each object is tracked by the AI application, it is assigned a unique id which will not change for as long as it is in the frame. This unique id can be used to update the corresponding object's position as it moves in the frame. Therefore, the bounding box used for each object need not be re-created for every update. Since the bounding box is an HTML element now, it can have HTML events for user interactions, and it can be animated like a regular HTML element for smooth user experience.

# Future Scope

- **Record and Replay:** Each incoming frame can be saved along with it's inference data. This data can be used for re-create a particular scenario.
- **Using the Canvas API:** A new API has been implemented called `OffscreenCanvas` which makes use of the browser's worker thread which takes the load off the main thread. This method can be explored to display the frame, once this new API comes to all the browsers.
- **Using the WebGL API:** The WebGL API uses the GPU. This API can be used to render the frame, bounding boxes for better performance.

# Conclusion

In conclusion, streaming real-time video content from an RTSP stream to a browser involves a complex interplay of various protocols and technologies, each with its unique strengths and challenges. The use of HLS (HTTP Live Streaming) provides a robust solution for delivering high-quality video content across a wide range of devices, thanks to its compatibility with the HTTP protocol and its ability to adapt video quality based on network conditions. However, its inherent latency poses a challenge for live streaming, which can be mitigated to some extent by reducing fragment size or using advanced techniques like Low Latency HLS.

Alternative approaches such as encapsulating RTSP into FLV or using WebRTC offer different benefits, such as lower latency and direct browser-to-browser communication, but they come with their own set of complexities and potential issues, particularly in terms of setup and compatibility. WebSockets, while offering real-time, bidirectional communication, require careful handling of data formats, such as converting frames to Base64 or using ArrayBuffers and Blobs for efficient transmission.

Displaying video frames in the browser, especially with added elements like bounding boxes for object detection, further complicates the process. The use of the Canvas API and CSS for rendering these elements provides flexibility and interactivity, though it requires precise management of coordinates and scaling to ensure accurate representation across devices with varying resolutions.

As technology evolves, exploring advanced APIs like OffscreenCanvas and WebGL could offer enhanced performance, particularly by leveraging GPU resources for rendering. Additionally, the possibility of implementing features like record and replay could significantly extend the functionality and usability of such a streaming setup, making it a versatile tool for various applications.