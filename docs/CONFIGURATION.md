# Configuration

The Home Assistant config flow is the normal way to connect Emby Media. YAML
configuration is supported as an import path, but all ongoing settings are
managed from the integration page.

## Create an Emby API key

1. Open the Emby Server Dashboard.
2. Go to **Settings**, then **Advanced**, then **API Keys**.
3. Create a key named `Home Assistant`.
4. Copy the key exactly and store it securely.

The key grants access to the Emby API. Do not place it in dashboard YAML,
screenshots, issues, or exported diagnostics.

## Add the integration

In Home Assistant, go to **Settings**, then **Devices & services**, select
**Add integration**, and search for **Emby Media**.

| Field | Required | Default | Description |
| --- | --- | --- | --- |
| Host | Yes | None | Emby hostname or IP address without a URL path |
| Port | Yes | `8096` | Emby HTTP or HTTPS port |
| Use SSL | Yes | Off | Connect with HTTPS |
| API key | Yes | None | Dedicated key created in Emby Server |
| Verify SSL | Yes | On | Validate the HTTPS certificate |

Use port `8920` when that is the HTTPS port configured by Emby. If a local
server uses a self-signed certificate, SSL verification can be disabled, but a
trusted certificate is preferable.

The setup flow checks connectivity, authentication, and the minimum Emby Server
version before creating an entry.

## Select an Emby user

The selected user controls user-specific library data, including watched
state, favorites, Next Up, Continue Watching, Suggestions, and remote playback
authorization.

Choose the regular Emby account that should represent Home Assistant activity.
If more than one household profile needs separate data, add another Emby server
entry only when a separate server connection is appropriate. A server cannot
be added twice with the same unique server ID.

## Change integration options

Open **Settings**, then **Devices & services**, select **Emby Media**, and use
**Configure**.

### Updates and discovery

| Option | Default | Allowed range | Description |
| --- | --- | --- | --- |
| Session scan interval | 10 seconds | 5 to 300 seconds | HTTP session refresh interval when polling is required |
| Enable WebSocket | On | On or off | Receive near real-time session and library events |
| WebSocket interval | 1500 ms | 500 to 10000 ms | Emby session subscription interval |
| Enable discovery sensors | On | On or off | Create Next Up, Continue Watching, Recently Added, and Suggestions data |
| Discovery scan interval | 900 seconds | 300 to 3600 seconds | Refresh user discovery data |
| Library scan interval | 3600 seconds | 3600 to 86400 seconds | Refresh library statistics |
| Server scan interval | 300 seconds | 300 to 3600 seconds | Refresh version, task, and server status data |

Keep WebSocket enabled for normal use. When the WebSocket connection is
healthy, it carries immediate changes and polling acts as a fallback. The
integration automatically reconnects after an interruption.

### Client filtering

| Option | Default | Description |
| --- | --- | --- |
| Ignored devices | Empty | Comma-separated Emby device names that should not create client entities |
| Ignore web players | Off | Exclude browser-based Emby sessions |

Device filtering applies to Emby client names, not Home Assistant entity IDs.
After changing filters, reload the integration or restart Home Assistant.

### Playback profile

| Option | Default | Description |
| --- | --- | --- |
| Direct play | On | Prefer the source media when the target reports support |
| Video container | `mp4` | Preferred container: `mp4`, `mkv`, or `webm` |
| Transcoding profile | `universal` | Playback profile: `universal`, `chromecast`, `roku`, `appletv`, or `audio_only` |
| Maximum video bitrate | Empty | Optional positive bitrate limit in kbps |
| Maximum audio bitrate | Empty | Optional positive bitrate limit in kbps |

The integration requests playback information from Emby before starting an
item. It sends the chosen media source and controlling user to the target
session. Actual direct-play and transcoding decisions remain subject to Emby
Server and client capabilities.

Use the `appletv` profile when Apple TV playback needs an explicit compatible
profile. Start with `universal` unless a client requires different behavior.

### Entity names

The media player, remote, notification, and button platforms each have an
option to prefix generated names with `Emby`. Prefixes are enabled by default.

Changing a prefix does not always rename an entity ID that Home Assistant has
already stored in the entity registry. Rename an existing entity from its Home
Assistant entity settings when necessary.

## Client session behavior

Client media players represent live Emby sessions. The Emby app must be open
and connected before the client can receive browse playback, navigation, or
notification commands.

If content is already playing, select another item in **Browse media**. The new
request uses `PlayNow` and replaces the current item. A separate stop action is
not required.

Apple TV remote playback has been verified with Emby Server 4.9. Other clients
depend on the remote-command support provided by their Emby application.

## Artwork and HTTPS

Artwork is published through `/api/embymedia/image/...` on the Home Assistant
origin. This design supports an HTTPS Home Assistant dashboard with an HTTP
Emby server and prevents the Emby API key from appearing in frontend state.

Episode playback prefers season artwork. The player URL includes
`layout=player`, which keeps the complete portrait cover visible across square
and wide Home Assistant layouts. Discovery and Browse Media images use their
normal dimensions.

No separate external Home Assistant URL is required for this image path.

## YAML import

UI setup is recommended. A minimal YAML entry can be imported into a config
entry at startup:

```yaml
embymedia:
  host: emby.local
  port: 8096
  ssl: false
  verify_ssl: true
  api_key: !secret emby_api_key
```

The connection is represented as a Home Assistant config entry after import.
Use the integration's **Configure** action for the remaining options.

## Reauthentication

If the Emby API key is revoked or replaced, Home Assistant starts a
reauthentication flow. Enter the new key when prompted. The existing config
entry, selected user, devices, and entity registry are retained.

## Applying changes

Most option changes reload the config entry. When Python files in
`custom_components/embymedia` have changed, a full Home Assistant restart is
required. A browser refresh alone cannot load new integration code.

Continue with [Troubleshooting](TROUBLESHOOTING.md) if a server or client does
not behave as expected.
