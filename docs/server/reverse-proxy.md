# Reverse proxy

Put the server behind HTTPS. A reverse proxy such as nginx, Caddy or Nginx Proxy Manager holds the certificate and
forwards requests to the container on port 8090.

The proxy has to do four things.

1. Serve HTTPS. The devices send the token with every request. Apps also move to a new server address only when it is
   `https` (see [Changing the domain](domain.md)).
2. Pass the client address. Set `TOSSLING_BEHIND_PROXY: "true"` on the server and have the proxy send
   `X-Forwarded-For`. Without it the server counts every client as the proxy's address. ntfy answers 429 to every
   request from an address after 30 failed sign-ins, so one device with a dead token would cut off all the others.
3. Accept request bodies up to 600 MB. Files go up to 500 MB, the server keeps attachments up to 520 MB, and
   encryption adds a little on top.
4. Leave responses unbuffered. Devices listen with `GET /<topic>/json`, a response that stays open and sends one JSON
   line per event. A proxy that buffers responses holds those lines back, so nothing arrives until the buffer fills.
   Keep read timeouts at a few minutes. The server sends keep-alive events in between.

If the proxy runs on the same machine, publish the port on localhost only, so the server is reachable only through
the proxy:

```yaml
    ports:
      - "127.0.0.1:8090:8090"
```

The examples below follow
[ntfy's recommendations for running behind a proxy](https://docs.ntfy.sh/config/#behind-a-proxy-tls-etc).

## nginx

```nginx
# /etc/nginx/sites-available/tossling

server {
  listen 80;
  server_name tossling.example.com;
  return 301 https://$host$request_uri;
}

server {
  listen 443 ssl http2;
  server_name tossling.example.com;

  ssl_certificate /etc/letsencrypt/live/tossling.example.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/tossling.example.com/privkey.pem;

  location / {
    proxy_pass http://127.0.0.1:8090;
    proxy_http_version 1.1;

    # Stream /json subscriptions and uploads instead of buffering them
    proxy_buffering off;
    proxy_request_buffering off;
    proxy_redirect off;

    proxy_set_header Host $http_host;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    proxy_connect_timeout 3m;
    proxy_send_timeout 3m;
    proxy_read_timeout 3m;

    # Files up to 500 MB; ntfy itself uses 0, no limit
    client_max_body_size 600m;
  }
}
```

The first block sends plain HTTP to HTTPS. ntfy's own example keeps POST over plain HTTP for short `curl` commands;
Tossling devices always use the `https` address, so this example redirects everything.

## Caddy

Caddy gets a certificate by itself, accepts bodies of any size and sets `X-Forwarded-For`.

```caddyfile
tossling.example.com {
    reverse_proxy 127.0.0.1:8090 {
        # Send every line of a /json subscription at once
        flush_interval -1
    }
}
```

Caddy already flushes streaming responses of unknown length right away. `flush_interval -1` makes that explicit.

## Nginx Proxy Manager

Create a proxy host for `tossling.example.com` that forwards to the server on port 8090, turn on Websockets Support,
and request an SSL certificate with Force SSL. Then add this to the Advanced tab:

```nginx
proxy_buffering off;
proxy_request_buffering off;
client_max_body_size 600m;
proxy_read_timeout 3m;
```

## Check it

The health address answers without a token:

```bash
curl https://tossling.example.com/v1/tossling/health
```

```json
{"devices":true,"push":false,"server":"tossling-server","url":"https://tossling.example.com","version":"0.6.1"}
```

`url` must be the address from `TOSSLING_BASE_URL`. `push` says whether
[instant delivery](instant-delivery.md) is set up.
