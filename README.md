# kong-auth-request
A Kong plugin that make POST auth request before proxying the original request.

## Description
kong-auth-request is a reincarnation of [ngx http auth request](http://nginx.org/en/docs/http/ngx_http_auth_request_module.html "ngx http auth request").    

The plug-in makes an http POST request with an authorization header and body that sends it to the authentication service (the URL is removed from the plug-in configuration); if the authentication response status code is 200, the plug-in forwards the original request to the upstream service with headers (header names are taken from the plugin's configuration) of the authentication response.
If the authentication response code is greater than 299, the authentication response will be returned to the client and the originating request will not be passed upstream.

## Installation

Install plugin by luarocks package manager.  
```luarocks install kong-auth-request```

Add kong-auth-request to Kong by environment variable
```KONG_PLUGINS=bundled,kong-auth-request```   

or add it to kong.conf:  
```plugins = bundled,kong-auth-request``` 

## Installation with Docker

Add the configurations below to the Kong Dockerfile:

```FROM kong:1.3.0-centos
RUN mkdir -p /usr/local/kong/plugins
COPY plugins /usr/local/kong/plugins
RUN cd /usr/local/kong/plugins/kong-auth-request \
    && /usr/local/bin/luarocks make *.rockspec make *.rockspec
ENV KONG_CUSTOM_PLUGINS kong-auth-request
```

Add kong-auth-request to the plugins in the Kong directory:

```mv ~/kong-auth-request ~/Folha3/folha3-kong/plugins```

Execute build command to the Kong service:

```docker-compose build kong```

## Configuration

```
curl -X POST \
--url "http://localhost:8001/services/fibery-api/plugins" \
--data "name=kong-auth-request" \
--data "config.auth_uri=http://auth.fibery.io/authorize" \
--data "config.auth_response_headers_to_forward[]=x-authorization"
--data "config.origin_request_headers_to_forward_to_auth[]=host"
```

| config parameter                                 | description                                                                                                                                                                                                                |
| ------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| config.auth_uri                                  | Plugin make a HTTP GET request with Authorization header to this URL before proxying the original request                                                                                                                  |
| config.auth_response_headers_to_forward          | If auth request was successful then plugin takes header names from auth_response_headers_to_forward collection, then finds them in auth response headers and adds them into origin request before proxying it to upstream. |
| config.origin_request_headers_to_forward_to_auth | Origin request headers to pass to auth uri                                                                                                                                                                                 |
Auth-request will execute a JSON POST request to the specified url with the following body:

| Attribute |                            Description                            |
| --------- | :---------------------------------------------------------------: |
| method    |  The http method of the original request. Ex: GET, POST e etc...  |
| path      | The url path of the original request. Ex: /service_id/resource/id |

## Author

Andray Shotkin

## License

The MIT License (MIT)
=====================

Copyright (c) 2019 Andray Shotkin

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
