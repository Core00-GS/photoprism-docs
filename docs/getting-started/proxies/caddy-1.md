# Using Caddy 1 as Reverse Proxy

!!! warning "Legacy Software"
    Caddy 1 reached end-of-life and only receives critical fixes. Consider upgrading to [Caddy 2](caddy-2.md) whenever possible. If you continue to run Caddy 1, the Caddy community—not PhotoPrism—must be your primary support channel.

Caddy 1 can still proxy WebSocket and HTTP/2 traffic for PhotoPrism. Enable the `websocket` and `transparent` options so request headers reach the app unchanged:

```caddyfile
example.com {
    gzip
    tls you@example.com

    proxy / photoprism:2342 {
        websocket
        transparent
        header_upstream X-Forwarded-Proto {scheme}
    }
}
```

The `tls` directive requests and renews certificates from Let’s Encrypt automatically. Use `tls internal` if you prefer to run your own CA, or `tls /path/fullchain.pem /path/privkey.pem` when supplying files manually.

Refer to the [Caddy 1 documentation](https://caddyserver.com/v1/docs/websocket) for additional directives and migration tips.

### Why Use a Proxy? ###

If you install PhotoPrism on a public server outside your home network, **always run it behind a secure
HTTPS reverse proxy**. Your files and passwords will otherwise be transmitted in clear text and can be intercepted
by anyone, including your provider, hackers, and governments. Backup tools and file sync apps may refuse to
connect as well.

![](https://dl.photoprism.app/img/diagrams/reverse-proxy.svg){ class="w100" }

!!! example ""
    **Help improve these docs!** You can contribute by clicking :material-file-edit-outline: to send a pull request with your changes.
