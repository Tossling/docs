# Changing the domain

A server can move to a new address without pairing the devices again. The apps follow it by themselves.

## How the apps follow

The health answer at `/v1/tossling/health` carries `url`, the address from `TOSSLING_BASE_URL`. When it names
another address than the one a device uses, the device:

1. accepts it only if it is `https`, or has the same scheme as now;
2. checks that the new address answers as a Tossling server and accepts the device's token;
3. switches to it and keeps everything else: rooms, keys, device ids.

If a check fails, the device stays where it is and tries again later. The Mac checks every 6 hours, and also after
waking up or a network change, at most every 10 minutes.

## Steps

1. Point the new name at the server, next to the old one. Both must work with HTTPS, so add the new name to the
   reverse proxy and get a certificate for it.
2. Set the new address in `docker-compose.yml`:

    ```yaml
        environment:
          TOSSLING_BASE_URL: https://new.example.com
    ```

3. Restart the server with `docker compose up -d`. Projects move to the new address by themselves.
4. Keep the old name working until every device has moved: open each app once, or give them a few days.
5. Drop the old name.

A device that was off the whole time and finds the old name gone has to be connected again.
