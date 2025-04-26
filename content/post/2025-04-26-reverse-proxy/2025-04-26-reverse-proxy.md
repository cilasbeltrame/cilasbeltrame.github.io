+++
date = '2025-04-26T06:26:39-03:00'
title = 'The Host header: Details you should know when proxying requests'
subtitle = 'Understanding and handling the Host header in reverse proxies'
tags = ["reverse-proxy", "host-header", "http", "networking"]
+++

When proxying requests, it's crucial to understand the role of the Host header. This header specifies the target server for a request and can be altered by users to access different resources.

## The Host Header

The Host header in a request indicates the intended server. For example:

```
Host: example.com
GET /v1/users
```

## The Host Header in Reverse Proxies

In reverse proxies, the Host header determines the target server. If absent, the proxy uses the original Host header. When redirecting to a different host, ensure the Host header is updated accordingly.

Example:

Redirecting from `example.com/v1/users` to `api.test.com/v2/users` requires changing the Host header to `api.test.com`. Failing to update it results in a 404 error, as the resource isn't found on `example.com`.

## Managing the Host Header in Reverse Proxies

By default, the Host header isn't changed. You must explicitly update it when redirecting to a different host.

## Conclusion

Understanding the Host header's function is essential when proxying requests, as it directs the request to the correct server and can be manipulated to access various resources.


