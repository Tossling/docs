# Configuration

Every setting is a command line flag and an environment variable. In Docker you set the variables in
`docker-compose.yml`. A flag wins over its variable.

## Settings

| Flag | Variable | Default | What it does |
|---|---|---|---|
| `-base-url` | `TOSSLING_BASE_URL` | none, required | The public address of the server, such as `https://tossling.example.com`. It goes into setup links, QR codes, attachment URLs and the health answer. |
| `-listen` | `TOSSLING_LISTEN` | `:8090` | The address and port to listen on. |
| `-data` | `TOSSLING_DATA` | `data` (`/data` in the Docker image) | The directory for the database, attachments and settings. |
| `-behind-proxy` | `TOSSLING_BEHIND_PROXY=true` | off | Takes the client address from `X-Forwarded-For` set by a reverse proxy in front. See [Reverse proxy](reverse-proxy.md). |
| `-firebase-key` | `TOSSLING_FIREBASE_KEY_FILE` | none | The Firebase service account JSON, for [instant delivery](instant-delivery.md) to phones. |
| `-demo` | `TOSSLING_DEMO=true` | off | A public demo server: files up to 10 MB, messages and attachments kept for an hour, rate limits for many guests sharing one token, no tokens of their own for devices. |
| none | `TOSSLING_RUN_AS` | `10001:10001` in the Docker image | The user to run as, see below. |

Variables from the time the project was called Tossy (`TOSSY_BASE_URL`, `TOSSY_LISTEN`, `TOSSY_DATA`,
`TOSSY_BEHIND_PROXY`, `TOSSY_FIREBASE_KEY_FILE`) are still read when the `TOSSLING_` name is not set.

### TOSSLING_RUN_AS

On Linux, when the server starts as root, it hands the data directory to the user in `TOSSLING_RUN_AS` and drops to
that user before doing anything else. The value is `uid` or `uid:gid`, both numeric and above 0.

The Docker image starts as root and sets `TOSSLING_RUN_AS=10001:10001`. So `./data` belongs to user 10001 after the
first start, even if an older image wrote it as root. Commands run with `docker compose exec`, which come in as root
too, drop to the same user.

To run the server as your own user, for example so that you can read `./data` without `sudo`:

```yaml
    environment:
      TOSSLING_RUN_AS: "1000:1000"
```

When the server does not start as root, `TOSSLING_RUN_AS` is ignored.

## Fixed limits

These are built in and have no setting.

| What | Limit |
|---|---|
| One attachment | 520 MB (10 MB on a demo server) |
| All attachments together | 5 GB (2 GB on a demo server) |
| How long attachments are kept | 3 hours (1 hour on a demo server) |
| A message without an attachment | 4,096 bytes, ntfy turns a longer body into an attachment |
| Tokens of the device user | 60, the server answers 409 instead of issuing more |

## Commands

`tossling-server` with no command runs the server. The other commands work on the same data directory, so they take
the same flags and variables. In Docker run them through `docker compose exec`, which sees the container's variables:

```bash
docker compose exec tossling-server tossling-server setup-link
```

| Command | What it does |
|---|---|
| `tossling-server setup-link` | Prints a new link to the [setup page](index.md#the-setup-link). |
| `tossling-server admin-password` | Sets the password of the [web panel](web-panel.md). It asks for the password, or reads it from stdin. At least 10 characters. |
| `tossling-server project add <channel> [name] [--publisher <name>]` | Creates a [project](projects.md) and prints its publisher token. |
| `tossling-server project list` | Lists projects and their publishers. |
| `tossling-server project remove <channel>` | Deletes a project. |

To set the panel password from a script:

```bash
echo 'a long password' | docker compose exec -T tossling-server tossling-server admin-password
```

## The data directory

| Path | What is there |
|---|---|
| `tossling-server.json` | The server's settings: the server token, the setup link, the panel password hash. A server from the Tossy days keeps `tossy-server.json`. |
| `user.db` | Users, tokens and access rules. |
| `cache.db` | Messages kept for devices that were offline. |
| `attachments/` | Encrypted attachments, deleted after 3 hours. |
| `tossling-server.lock` | Makes sure only one server uses the directory at a time. |

A second server started on the same directory stops with `another tossling-server already uses …`.
