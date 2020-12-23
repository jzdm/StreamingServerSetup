## Software needed
apt install git certbot nginx libnginx-mod-rtmp ffmpeg

## NGINX
openssl dhparam -out /etc/nginx/dhparam-2048.pem 2048
mkdir -p /var/www/acme-challenge/.well-known/

# setup webroot-folder and recording location
mkdir -p /var/www/stream.example.com/records
# this folder must be writable by the nginx user. usually this is www-data
chown -r www-data:www-data /var/www/stream.example.com

# copy nginx.conf to /etc/nginx/nginx.conf
# copy stream.example.com.conf to /etc/nginx/sites-available/stream.example.com.conf
# adapt to your domain names

# enable virtual host
ln -s /etc/nginx/sites-available/stream.example.com.conf /etc/nginx/sites-enabled/stream.example.com.conf

# disable the HTTPS-part in stream.example.com.conf by commenting out lines 22 to 72, then get a Let's Encrypt certificate

# restart nginx
systemctl restart nginx.service

## Let's Encrypt
certbot certonly \
	--webroot -w /var/www/acme-challenge \
	--rsa-key-size 4096 \
	--renew-by-default \
	--agree-tos \
	-m mail@example.com \
	-d stream.example.com

# re-enable the HTTPS-part in stream.example.com.conf by un-commenting lines 22 to 72
# restart nginx
systemctl restart nginx.service

## Setup OBS
# go to Settings -> Stream
# Platform:
user-defined
# Server:
rtmp://stream.example.com:1935/src/
# Stream-Key:
live?psk=abcdef

# 'live' is the name under which the stream will be available later
# if needed adapt to own name. Do not forget to set correct name in the 'Streaming Server Authentication'
# part of stream.example.com.conf.
# 'psk' is a simple pre-shared key for authentication of streaming sources. should be updated to more secure phrase.
# if changed, must be updated as-well in the 'Streaming Server Authentication' part of stream.example.com.conf and 
# the rtmp -> server -> application src -> exec ffmpeg… part in nginx.conf. Also update all streaming-links published on websites.
# Restart nginx after updating these values!



## Start Streaming

# Streams can be accessed via HLS (dynamic resolution/bandwidth)
https://stream.example.com/hls/live.m3u8
# via HLS (static resolution/bandwidth)
https://stream.example.com/hls/live_720p/index.m3u8
https://stream.example.com/hls/live_480p/index.m3u8
https://stream.example.com/hls/live_240p/index.m3u8
https://stream.example.com/hls/live_144p/index.m3u8
# or RTMP
rtmp://stream.example.com:1935/hls/live_720p
rtmp://stream.example.com:1935/hls/live_480p
rtmp://stream.example.com:1935/hls/live_240p
rtmp://stream.example.com:1935/hls/live_144p
# audio-only, via RTMP
rtmp://stream.example.com:1935/audio/live
# audio-only via HLS does not work reliable

# recordings of streams will be saved to '/var/www/stream.example.com/records'
# do not forget to watch your disk usage
