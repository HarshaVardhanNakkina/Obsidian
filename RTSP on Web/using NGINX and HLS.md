this is what the nginx configuration looks like:

```
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
        worker_connections 768;
        # multi_accept on;
}

http {

        ##
        # Basic Settings
        ##

        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        keepalive_timeout 65;
        types_hash_max_size 2048;
        # server_tokens off;

        # server_names_hash_bucket_size 64;
        # server_name_in_redirect off;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        ##
        # SSL Settings
        ##

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;

        ##
        # Logging Settings
        ##

        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;

        ##
        # Gzip Settings
        ##

        gzip on;

        # gzip_vary on;
        # gzip_proxied any;
        # gzip_comp_level 6;
        # gzip_buffers 16 8k;
        # gzip_http_version 1.1;
        # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

        ##
        # Virtual Host Configs
        ##

        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;

        map $sent_http_content_type $expires {
                default 1d;
                application/vnd.apple.mpegurl epoch;
        }

        server {
                listen 80;

                location / {
                        root /tmp/hls;
                        expires $expires;
                }
        }
}

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

publish the live stream to nginx using this command
```bash
ffmpeg -re -i <input> -c:v copy -c:a aac -ar 44100 -ac 1 -f flv rtmp://localhost/hls/linear_algebra
```

access the published stream as HLS like this:
```html
  <video id="my-video" class="video-js" controls preload="auto" width="1280" height="720" data-setup="{}">
    <source src="http://localhost/hls/linear_algebra.m3u8" type="application/vnd.apple.mpegurl" />
    <p class="vjs-no-js">
      To view this video please enable JavaScript, and consider upgrading to a
      web browser that
      <a href="https://videojs.com/html5-video-support/" target="_blank">supports HTML5 video</a>
    </p>
  </video>
```

pros:
1. easy to setup
2. no additional JS libraries are needed
cons:
1. delay between live stream and HLS