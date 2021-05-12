## EchoIP Docker Image
This is a simple docker image for looking up IP address base on https://github.com/mpolden/echoip

Demo: 
- https://ifconfig.co
- https://ip.zatp.com

You can easily pull the Docker Image I build:
```
docker pull hurt/echoip
docker run -d --rm --name echoip -p8080:8080 hurt/echoip
```
*GeoIP database version: 20210511*

### Build
#### Clone this project
```
git clone https://github.com/scenery/EchoIPDocker.git echoip
cd echoip
```

#### Download GeoIP data
You can get the newest GeoIP database from [MAXMIND](https://dev.maxmind.com/geoip/geoip2/geolite2/) At least three files are required as follows:
- GeoLite2-ASN.mmdb
- GeoLite2-City.mmdb
- GeoLite2-Country.mmdb

Then extract and place the files into ```geoip``` folder or you can edit ```run.sh``` to choose what you need.

**Usage:**
```
Usage of echoip:
  -C int
    	Size of response cache. Set to 0 to disable
  -H value
    	Header to trust for remote IP, if present (e.g. X-Real-IP)
  -a string
    	Path to GeoIP ASN database
  -c string
    	Path to GeoIP city database
  -f string
    	Path to GeoIP country database
  -l string
    	Listening address (default ":8080")
  -p	Enable port lookup
  -r	Perform reverse hostname lookups
  -t string
    	Path to template directory (default "html")
```

#### Build the container
```
docker build -t echoip .
```

### Run
```
docker run -d --name echoip -p 8080:8080 echoip
```

#### Nginx configuration

You can run this server with your own domain.
```
server {
    listen 80;
    listen [::]:80;
    ...

    location / {
        if ($http_user_agent !~* (curl|wget)) {
            return 301 https://$server_name$request_uri;
        }
        proxy_set_header  Host $host;
		proxy_set_header  X-Real-IP $remote_addr;
		proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header  X-Forwarded-Proto $scheme;
		proxy_pass  http://localhost:8080;
    }
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    ssl_certificate /path/to/ssl/chain.pem; 
    ssl_certificate_key /path/to/ssl/private.key;
    ...

    location / {
        proxy_set_header  Host $host;
		proxy_set_header  X-Real-IP $remote_addr;
		proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header  X-Forwarded-Proto $scheme;
		proxy_pass  http://localhost:8080;
    }
}
```