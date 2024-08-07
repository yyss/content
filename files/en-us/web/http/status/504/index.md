---
title: 504 Gateway Timeout
slug: Web/HTTP/Status/504
page-type: http-status-code
spec-urls: https://www.rfc-editor.org/rfc/rfc9110#status.504
---

{{HTTPSidebar}}

The HyperText Transfer Protocol (HTTP) **`504 Gateway Timeout`** server error response code indicates that the server, while acting as a gateway or proxy, did not get a response in time from the upstream server that it needed in order to complete the request.

> [!NOTE]
> A [Gateway](<https://en.wikipedia.org/wiki/Gateway_(telecommunications)>) might refer to different things in networking and a 504 error is usually not something you can fix, but requires a fix by the web server or the proxies you are trying to get access through.

## Status

```http
504 Gateway Timeout
```

## Specifications

{{Specifications}}

## See also

- [HTTP Status Code Definitions](https://httpwg.org/specs/rfc9110.html#status.504)
- {{HTTPStatus(502)}}
