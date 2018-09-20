# lua-resty-aws-signature

This library is based on the work of [Alan Grosskurth](https://github.com/grosskur)
at https://github.com/grosskur/lua-resty-aws.
It is basically forked from his repository and we change the HMAC library used.

## Overview

This library implements request signing using the [AWS Signature
Version 4][aws4] specification. This signature scheme is used by nearly all AWS
services.

## AWS documentation

[aws4]: http://docs.aws.amazon.com/general/latest/gr/signature-version-4.html

## Usage

This library uses standard AWS environment variables as credentials to
generate [AWS Signature Version 4][aws4].

```bash
export AWS_ACCESS_KEY_ID=AKIDEXAMPLE
export AWS_SECRET_ACCESS_KEY=AKIDEXAMPLE
```

To be accessible in your nginx configuration, these variables should be
declared in `nginx.conf` file.

```nginx
#user  nobody;
worker_processes  1;

error_log  /dev/fd/1 debug;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;

env AWS_ACCESS_KEY_ID;
env AWS_SECRET_ACCESS_KEY;

events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /dev/stdout;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```

You can then use the library to add AWS Signature headers and `proxy_pass` to a
given S3 bucket.

```nginx
set $bucket 'example';
set $s3_host ${bucket}.s3-eu-west-1.amazonaws.com;

location / {
  access_by_lua_block {
    require("resty.aws-signature").s3_set_headers(ngx.var.s3_host, ngx.var.uri)
  }

  proxy_set_header Host ${s3_host};
  proxy_pass https://${s3_host};
}
```

## Contributing

Check [CONTRIBUTING.md](CONTRIBUTING.md) for more information.

## License

Copyright 2018 JobTeaser

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

  [http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0)

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
