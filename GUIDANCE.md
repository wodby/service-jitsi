# Jitsi Meet on Wodby

What Wodby sets up for Jitsi Meet on this service.

## Components

The service runs Jitsi's own images (`ghcr.io/jitsi/*`), all at the option's release, as four containers in one pod on
the `wodby/stateful` chart:

- `web` (primary, port 8000): Jitsi Meet's pages, and the proxy for the XMPP WebSocket (`/xmpp-websocket`) and BOSH.
- `prosody`: the XMPP server that holds rooms and accounts.
- `jicofo`: the conference focus that puts participants on the video bridge.
- `jvb`: the video bridge that relays every participant's audio and video on UDP port 10000 (`media`).

The components reach each other on `localhost`. The service runs one replica: one bridge carries every call through
the single published media port. A deployment restarts the pod, which ends calls in progress.

## Media port

Browsers send audio and video straight to the bridge over UDP, not through the domain. The `media` port must be
published; the bridge then advertises `<IP>:<public port>`:

- The IP is the **Media IP address** setting, or else the IPv4 address the app's domain resolves to. A domain behind a
  proxy such as Cloudflare resolves to the proxy, so set the cluster's public IP there.
- The port is the published port. Publishing or unpublishing it asks for a redeploy; deploy so the bridge advertises
  the new port.

Without a published port the pages and chat work, but calls carry no audio or video. There is no TURN server, so
networks that block outgoing UDP can't carry media.

## Accounts and access

- Authentication is on (`ENABLE_AUTH=1`, `AUTH_TYPE=internal`). Only a signed-in user can start a meeting.
- One moderator account, from the `moderator-username` and `moderator-password` settings, is created on every start of
  the `prosody` container; Prosody keeps no accounts between restarts. To change the password, change the setting and
  redeploy. Other accounts created by hand are lost on restart.
- `allow-guests` (`1` by default) lets anyone with a meeting link join once a moderator has started it. With `0`,
  everyone must sign in.
- The internal XMPP accounts for Jicofo and the bridge use generated tokens.

## Limits

- No recording, live streaming, SIP or transcription (Jibri and Jigasi are not included).
- No TURN server; media needs outgoing UDP to the published port.
- No JWT or LDAP sign-in.
- One bridge: the call size is limited by the pod's CPU (about 25 millicores of gateway CPU and somewhat more bridge
  CPU per participant at default quality).

## Check the result

- `curl -fs localhost:8080/about/health` in the `jvb` container succeeds once the bridge has joined.
- The `jvb` container log shows `[wodby] Browsers send audio and video to <IP>:<port> over UDP`.
- The `prosody` container log shows `[wodby] Created the moderator account <name>`.
