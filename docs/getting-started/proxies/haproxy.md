# Using HAProxy as Reverse Proxy

!!! tldr ""
    HAProxy offers powerful routing options, but we cannot provide support for custom load-balancer setups. Consult the [HAProxy community](https://www.haproxy.org/#support) if you encounter proxy-specific issues.

The example below terminates TLS, enforces SNI routing, and proxies both long uploads and WebSocket connections to a local PhotoPrism service.

```haproxy
global
    log /dev/log local0
    maxconn 2000

defaults
    log     global
    mode    http
    option  httplog
    option  forwardfor
    timeout connect 5s
    timeout client  1m
    timeout server  1m
    timeout tunnel  5m

frontend https
    bind *:80
    bind *:443 ssl crt /etc/ssl/localcerts/wildcard.example.com.pem alpn h2,http/1.1
    redirect scheme https code 301 if !{ ssl_fc }

    acl host_photoprism hdr(host) -i photos.example.com
    use_backend photoprism if host_photoprism

backend photoprism
    server photoprism 127.0.0.1:2342 check
    http-request set-header X-Forwarded-Proto https
    http-response set-header Strict-Transport-Security "max-age=31536000; includeSubDomains"
    http-reuse safe
```

PhotoPrism’s WebSocket endpoint (`/api/v1/ws`) works automatically as long as you keep the backend in `mode http` and do not enable `option httpclose`. When forwarding to containers, replace `127.0.0.1:2342` with the container hostname or overlay VIP.

Store certificates in a PEM bundle (full chain + private key) referenced via `crt`. Use `crt-list` when hosting multiple domains on the same load balancer. More complex templates, including mutual TLS and ACL-based routing, are covered in the [HAProxy documentation](https://www.haproxy.com/documentation/).

## Why Use a Proxy? ##

If you install PhotoPrism on a public server outside your home network, **always run it behind a secure
HTTPS reverse proxy**. Your files and passwords will otherwise be transmitted in clear text and can be intercepted
by anyone, including your provider, hackers, and governments. Backup tools and file sync apps may refuse to
connect as well.

![](https://dl.photoprism.app/img/diagrams/reverse-proxy.svg){ class="w100" }

!!! example ""
    **Help improve these docs!** You can contribute by clicking :material-file-edit-outline: to send a pull request with your changes.
