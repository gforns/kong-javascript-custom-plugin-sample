# Javascript Custom plugin for Kong

This repository is a sample repo to show how to run Kong with a Javascript Custom Plugin

This repository contains a `docker-compose.yaml` file that will start a kong node (Kong OSS 3.4) and will load a declarative config file from `kong.yaml`

This repository creates a custom Docker image, defined in the `Dockerfile` that will install 2 custom plugins:

 - `js-hello` plugin from:  https://github.com/Kong/kong-js-pdk/blob/master/examples/js-hello.js

 - `ts-hello` plugin from:  https://github.com/Kong/kong-js-pdk/blob/master/examples/ts-hello.ts


## Building a Docker image with the Custom Plugin

This sample creates a custom Kong Docker image based on `kong/kong:3.4` and installs the required dependencies to run nodeJS.

The Javascript custom plugin consists on 2 files: `package.json` and `js_hello.js` files that are copied inside the `/usr/local/kong/js-plugins/`.

This image is based on Ubuntu / Debian:

```
FROM kong/kong:3.4
USER root
RUN apt update && apt install -y nodejs npm && npm install -g kong-pdk
# Copy js-hello plugin  to /usr/local/kong/js-plugins
COPY --chown=kong --chmod=755 ./plugins/js_hello/package.json /usr/local/kong/js-plugins/package.json
COPY --chown=kong --chmod=755 ./plugins/js_hello/js_hello.js /usr/local/kong/js-plugins/js_hello.js
RUN cd /usr/local/kong/js-plugins/ && npm install
USER kong
```


For RHEL based image, the Dockerfile may be different:

```
FROM kong/kong:3.4-rhel
USER root
WORKDIR /usr/local/kong
RUN yum install -y nodejs
RUN npm install kong-pdk
# Copy js-hello plugin  to /usr/local/kong/js-plugins
COPY --chown=kong --chmod=755 ./plugins/js_hello/package.json /usr/local/kong/js-plugins/package.json
COPY --chown=kong --chmod=755 ./plugins/js_hello/js_hello.js /usr/local/kong/js-plugins/js_hello.js
RUN cd /usr/local/kong/js-plugins/ && npm install
USER kong
```




## Configuring Kong

For Debian / Ubuntu based images the required configuration is:

```
KONG_PLUGINS: bundled, js_hello
KONG_PLUGINSERVER_NAMES: js
KONG_PLUGINSERVER_JS_SOCKET: /usr/local/kong/js_pluginserver.sock
KONG_PLUGINSERVER_JS_START_CMD: /usr/local/bin/kong-js-pluginserver -v --plugins-directory /usr/local/kong/js-plugins
KONG_PLUGINSERVER_JS_QUERY_CMD: /usr/local/bin/kong-js-pluginserver --plugins-directory /usr/local/kong/js-plugins --dump-all-plugins
```

For RHEL based systems the plugin server is installed in a different path:

```
KONG_PLUGINS: bundled, js_hello
KONG_PLUGINSERVER_NAMES: js
KONG_PLUGINSERVER_JS_SOCKET: /usr/local/kong/js_pluginserver.sock
KONG_PLUGINSERVER_JS_START_CMD: /usr/local/kong/node_modules/.bin/kong-js-pluginserver -v --plugins-directory /usr/local/kong/js-plugins
KONG_PLUGINSERVER_JS_QUERY_CMD: /usr/local/kong/node_modules/.bin/kong-js-pluginserver --plugins-directory /usr/local/kong/js-plugins --dump-all-plugins
```


## Testing the plugin

You can bring the enviornment up

```
docker-compose up -d --build
```

Send a `curl -v` request to http://localhost:8000/anything, you should see `x-hello-from-javascript` and `x-javascript-pid` response headers added in the response:


```
$ curl -v http://localhost:8000/anything
```

```
* Connected to localhost (127.0.0.1) port 8000 (#0)
> GET /anything HTTP/1.1
> Host: localhost:8000
> User-Agent: curl/7.81.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Content-Type: application/json
< Content-Length: 442
< Connection: keep-alive
< x-hello-from-typescript: Typecript says typescript_hello to localhost:8000
< x-typescript-pid: 1292
< x-hello-from-javascript: Javascript says javascript_hello to localhost:8000
< x-javascript-pid: 1291
< Date: Fri, 03 May 2024 15:49:49 GMT
< Server: gunicorn/19.9.0
< Access-Control-Allow-Origin: *
< Access-Control-Allow-Credentials: true
< X-Kong-Upstream-Latency: 523
< X-Kong-Proxy-Latency: 102
< Via: kong/3.4.2
< 
{
  "args": {}, 
  "data": "", 
  "files": {}, 
  "form": {}, 
  "headers": {
    "Accept": "*/*", 
    "Host": "httpbin.org", 
    "User-Agent": "curl/7.81.0", 
    "X-Amzn-Trace-Id": "Root=1-6635079d-19141ff006eb0b6c293c14b0", 
    "X-Forwarded-Host": "localhost", 
    "X-Forwarded-Path": "/anything"
  }, 
  "json": null, 
  "method": "GET", 
  "origin": "192.168.80.1, 213.195.108.223", 
  "url": "https://localhost/anything/anything"
}

```

## More information 

 https://github.com/Kong/kong-js-pdk/