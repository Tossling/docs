# Instant delivery

Phone apps that are closed learn about new items through Firebase Cloud Messaging, which reaches an iPhone through
Apple's push service. The push carries no content. It only wakes the app, which then fetches the encrypted message
from your server.

## Without Firebase

Everything works without it, only later:

- New items reach the phone when the app opens.
- The Android app can keep a connection of its own instead, with its Keep a connection setting. It turns that on by
  itself when the server reports `"push": false`. Events then arrive at once even in Doze.
- The iPhone app works only while it is open.

## Set it up

The server needs the service account JSON of the Firebase project the apps are built with. If you build the apps
yourself, that is the project of your `google-services.json` (Android) and `GoogleService-Info.plist` (iOS). The iOS
app also needs an APNs key uploaded to that Firebase project.

1. In the Firebase console open Project settings, Service accounts, and generate a new private key. Save the file
   next to `docker-compose.yml`, for example as `firebase.json`.
2. Make it readable by user 10001, the user the server runs as:

    ```bash
    sudo chown 10001 firebase.json
    chmod 600 firebase.json
    ```

    If you set `TOSSLING_RUN_AS` to another user, give the file to that user.

3. Mount it into the container and point `TOSSLING_FIREBASE_KEY_FILE` at it:

    ```yaml
    services:
      tossling-server:
        image: ghcr.io/tossling/tossling-server:latest
        restart: unless-stopped
        ports:
          - "8090:8090"
        environment:
          TOSSLING_BASE_URL: https://tossling.example.com
          TOSSLING_BEHIND_PROXY: "true"
          TOSSLING_FIREBASE_KEY_FILE: /firebase.json
        volumes:
          - ./data:/data
          - ./firebase.json:/firebase.json:ro
    ```

4. Restart the server with `docker compose up -d`.

## Check it

The health answer says `"push": true`:

```bash
curl https://tossling.example.com/v1/tossling/health
```

The overview page of the [web panel](web-panel.md) shows the Firebase state too.
