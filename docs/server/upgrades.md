# Upgrades and backups

## Upgrade the server

```bash
docker compose pull
docker compose up -d
```

The image tag `latest` always points at the newest release. The server keeps its data in `./data` and upgrades it in
place. Devices that were offline during the restart catch up with the messages they missed.

To see the running version:

```bash
curl https://tossling.example.com/v1/tossling/health
```

Releases and their notes are on [GitHub](https://github.com/tossling/tossling-server/releases).

## Back up

Everything the server needs is in `./data`: the settings with the server token, users and tokens, the panel
password, projects and their publishers. Room keys are not there: they live only on the devices.

Stop the server for the copy, so the databases are consistent:

```bash
docker compose stop
tar czf tossling-data-$(date +%F).tar.gz data
docker compose start
```

`attachments/` holds encrypted files for at most 3 hours. You can leave it out of the backup with
`--exclude=data/attachments`.

To restore, stop the server, put the `data` directory back and start it. The server makes the files its own on start.

The backup contains the server token and every device and publisher token. Keep it as safe as the tokens.

## Upgrade from Tossy Server

The project was called Tossy before 0.2. A Tossy server upgrades in place:

- Point `docker-compose.yml` at `ghcr.io/tossling/tossling-server:latest` and keep the same data directory.
- `TOSSY_*` variables are still read. You can rename them to `TOSSLING_*` at any time.
- The settings stay in `tossy-server.json`.
- Paired devices keep working.

Since 0.3 the API lives at `/v1/tossling/…` and new rooms use `tossling-*` channels. The old `/v1/tossy/…` path and
`tossy-*` channels keep working for devices that still use them.

Since 0.4.0 room tokens never expire, and the first start of a newer server makes the room tokens of older versions
permanent.
