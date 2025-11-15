# Using Caddy 2 as Reverse Proxy

!!! tip "Self-Updating Certificates"
    Caddy 2 [issues and renews](https://caddyserver.com/docs/caddyfile/directives/tls) Let’s Encrypt certificates automatically. Keep its data directory persistent so renewals survive container restarts.

WebSocket proxying works out of the box, making Caddy one of the easiest front ends for PhotoPrism. Disable PhotoPrism’s internal TLS (`PHOTOPRISM_DISABLE_TLS="true"`) and let Caddy terminate HTTPS.

## Example Caddyfile

```caddyfile
{
    email you@example.com
    auto_https disable_redirects
}

photos.example.com {
    encode zstd gzip
    header Strict-Transport-Security "max-age=31536000; includeSubDomains"

    reverse_proxy photoprism:2342 {
        flush_interval -1       # stream uploads
        health_body /api/v1/status
    }
}
```

When running PhotoPrism outside of Docker, replace `photoprism:2342` with the host/IP where PhotoPrism listens. You can also add `basicauth` blocks or global `tls` options (e.g., `tls internal`) depending on your trust model.

See the [Caddy reverse proxy docs](https://caddyserver.com/docs/caddyfile/directives/reverse_proxy) for advanced features such as mutual TLS, rate limiting, and header manipulation.

### Why Use a Proxy? ###

If you install PhotoPrism on a public server outside your home network, **always run it behind a secure
HTTPS reverse proxy**. Your files and passwords will otherwise be transmitted in clear text and can be intercepted
by anyone, including your provider, hackers, and governments. Backup tools and file sync apps may refuse to
connect as well.

![](https://dl.photoprism.app/img/diagrams/reverse-proxy.svg){ class="w100" }

!!! example ""
    **Help improve these docs!** You can contribute by clicking :material-file-edit-outline: to send a pull request with your changes.
