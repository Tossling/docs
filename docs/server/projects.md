# Projects

A project is a channel that your services publish events to: a CI build, a monitoring alert, a bot. Every Tossling
device shows the events as notifications. The list of projects is shared by all devices of the server.

Projects are ordinary ntfy messages. Unlike rooms, they are not encrypted.

## Create a project

```bash
docker compose exec tossling-server tossling-server project add backend-alerts "Backend"
```

Without Docker leave out `docker compose exec tossling-server`. The command creates the channel `backend-alerts` with
the name Backend, makes it appear on all devices and prints a token with a `curl` example:

```text
Project "Backend" on channel backend-alerts, publisher backend-alerts. Its token can only write to the channels of backend-alerts:

    tk_…

    curl -H "Authorization: Bearer tk_…" -H "Title: Build finished" -d "assembleRelease in 3 min" https://tossling.example.com/backend-alerts
```

The token is shown once. It can only publish to the channels of its publisher, and it cannot read anything.

A channel name has 1 to 64 letters, digits, `-` and `_`, and does not start with `tossling-` or `tossy-` (those are
rooms).

The same can be done in the [web panel](web-panel.md) under Projects, and from a Mac with
`tossling project add <channel> [name] [--publisher <name>]`.

## One token for several channels

Every project has a publisher, and the token belongs to the publisher. By default the publisher has the channel's
name. A service that feeds several channels keeps one token:

```bash
tossling-server project add backend-alerts "Backend" --publisher backend
tossling-server project add axrock AxRock --publisher backend
```

The first command creates the publisher `backend` and prints its token. The second one lets `backend` write to one
more channel and prints no new token.

## Publish an event

```bash
curl -H "Authorization: Bearer tk_…" \
     -H "Title: Build finished" \
     -d "assembleRelease in 3 min" \
     https://tossling.example.com/backend-alerts
```

The body is the message. Other ntfy headers work too, such as `Priority`, `Tags`, `Click` for a link to open, and
`Markdown: yes` for a message in Markdown.

## Webhooks

GitHub, Grafana and Alertmanager cannot set an `Authorization` header, so their addresses carry the token instead:

```text
https://tossling.example.com/<channel>?template=github&auth=<auth>
https://tossling.example.com/<channel>?template=grafana&auth=<auth>
https://tossling.example.com/<channel>?template=alertmanager&auth=<auth>
```

`<auth>` is the text `Bearer <token>` in URL-safe Base64 without padding. ntfy's built-in templates turn the JSON of
these services into a title and a message.

The panel page that shows a new publisher token lists these addresses ready to copy. To build one yourself:

```bash
printf 'Bearer %s' 'tk_…' | base64 | tr '+/' '-_' | tr -d '='
```

Keep these addresses as secret as the token itself.

## List and remove projects

```bash
tossling-server project list
tossling-server project remove backend-alerts
```

After `remove` nobody can publish to the channel. If the command stops halfway, run it again.

## What devices can do

Devices can read every channel on the server, but write only to their rooms and to the channels `mac` and `claude`.
Each device decides which projects to listen to: muting is local to the device. The Mac mutes `mac` and `claude` by
default.

The apps manage projects with the device token through the server's API:

| Request | What it does |
|---|---|
| `GET /v1/tossling/projects` | Lists projects with their names and publishers. |
| `POST /v1/tossling/projects` with `{"topic": "…", "name": "…", "publisher": "…"}` | Creates a project. The answer has the publisher token only when the publisher is new. |
| `DELETE /v1/tossling/projects/<channel>` | Deletes a project. |
