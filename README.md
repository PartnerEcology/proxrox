# proxrox

[![Build Status](https://travis-ci.org/bripkens/proxrox.svg?branch=master)](https://travis-ci.org/bripkens/proxrox)

proxrox is a command line utility which starts a local nginx instance to
serve up static files, proxy one or many services under a single origin, use
SSL locally and, generally, to get a development environment that is
similar to a production environment.

proxrox achieves this using nginx. When proxrox is asked to start a server, it
will create an nginx config file in a temporary location and start an nginx
instance using this config file. This means that proxrox can theoretically
support all of nginx's features.

## Installation
In order to use proxrox you will need to have Node.js and NPM installed.
proxrox is currently only meant to be used as a command line utility, but may
export an API in the future for tool developers.

```
npm install -g proxrox
```

## Usage
Install nginx (platform specific, not yet supported for all platforms):
```
proxrox install
```

Serve the current working directory via `http`:
```
proxrox start
```

Serve the current working directory via `http` and fall back to a proxy
for all requests that couldn't be served from the working directory:
```
proxrox start --proxy http://127.0.0.1:8080
```

Enable server-side includes, transport layer security and SPDY support:
```
proxrox start --spdy --ssi
```

Start proxrox using a local [configuration file](#config-format):
```
proxrox start .proxrox.yaml
```

Stop the running nginx instances (stops all):
```
proxrox stop
```

## Why proxrox exists

### Production and development environment parity
Development environments should resemble production environments.
This means that server-side includes, transport layer security, compression
and more should exist during development. Not only is this important for
page speed optimizations, but it also allows you to find security
issues early, e.g. a secure page which references insecure content.

### Serving multiple services under a single [origin](https://tools.ietf.org/html/rfc6454)
Whether the app is service-oriented, micro service based,
[resource-oriented client architecture](http://roca-style.org/) like
or a single page app, the
[same-origin policy](https://www.w3.org/Security/wiki/Same_Origin_Policy)
is often an issue for local development. People circumvent this issue in
various ways. While most teams have good practices in place for production
environments, development environments often lack this. Solutions I have
seen range from [cross-origin resource sharing](http://www.w3.org/TR/cors/)
for local development activated via feature flags to completely disabling web
security in browsers.

### Extending the space of possible solutions
Many people don't know or use server-side includes. There are probably various
reasons for this. One thing that I noticed myself is that it just takes time
to setup a proper development environment with proxy servers.


## Config format
Proxrox can be started with a config from the file system. This file
can be shared in a team, e.g. placed in a repository, to ensure that
every team member is using the same configuration. Proxrox supports two
config file format:

 - JSON (recommended file name is `.proxrox.json`)
 - YAML (recommended file name is `.proxrox.yaml`)

All configuration options in this config file will take precedence over
proxrox's defaults and the CLI arguments. If you need to adapt specific
options to developers' machines, then you shouldn't put these options into
the config file as these values would otherwise take precedence.

A configuration file in the YAML format (in `.proxrox.yaml`) would look like
this. The following sections explain all the supported options.

```
# you can use YAML comments
proxy: 'http://127.0.0.1:8080'
spdy: true
ssi: true
root: './web'
```

List of supported options:

### serverName
This option is used to populate nginx's `server_name` directive. Most users
will not make use of this option and can use the default value.

 - **Type**: `string`
 - **Default**: `'example'`
 - **Nginx docs**:
   - http://nginx.org/en/docs/http/server_names.html
   - http://nginx.org/en/docs/http/ngx_http_core_module.html#server_name

### port
The port to bind to.

 - **Type**: `number`
 - **Default**: `4000`
 - **Nginx docs**:
   - http://nginx.org/en/docs/http/ngx_http_core_module.html#listen

### root
Defines the path to the directory which should be served via HTTP. You can
use absolute paths or relative paths that are resolved against the location
of the config file.

 - **Type**: `string`
 - **Default**: The current working directory
 - **Nginx docs**:
   - http://nginx.org/en/docs/http/ngx_http_core_module.html#root

### logDir
Defines the path to the nginx logs.

 - **Type**: `string`
 - **Default**: `'logs/'` (relative to nginx config dir)

### directoryIndex
Whether or not to generated a directory listing for requests to directories.

 - **Type**: `boolean`
 - **Default**: `true`
 - **Nginx docs**:
   - http://nginx.org/en/docs/http/ngx_http_autoindex_module.html#autoindex

### gzip
Whether or not to use gzip compression for responses (if possible).

 - **Type**: `boolean`
 - **Default**: `true`
 - **Nginx docs**:
   - http://nginx.org/en/docs/http/ngx_http_gzip_module.html#gzip

### proxy
Fall back to this proxy for every incoming request that could not be served
from the configured `root` directory. A named location will be created for
this proxy and used within nginx's `try_files` directive.

Use `null` or do not specify if you don't want o use a proxy.

 - **Type**: `string`
 - **Default**: `null`
 - **Nginx docs**:
   - http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass


### tls
When set, proxrox will generate a self-signed SSL certificate. Once started,
nginx will only accept `https` connections under the configured `port`.

 - **Type**: `boolean`
 - **Default**: `false`
 - **Nginx docs**:
   - http://nginx.org/en/docs/http/configuring_https_servers.html


### spdy
Proxrox can enable SPDY. Activating SPDY also implies the `tls` option.

 - **Type**: `boolean`
 - **Default**: `false`
 - **Nginx docs**:
   - http://nginx.org/en/docs/http/ngx_http_spdy_module.html

### ssi
Server-side includes are very useful and can be activated with the `ssi` flag.

 - **Type**: `boolean`
 - **Default**: `false`
 - **Nginx docs**:
   - http://nginx.org/en/docs/http/ngx_http_ssi_module.html
