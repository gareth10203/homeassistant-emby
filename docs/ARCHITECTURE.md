# Architecture

Emby Media is a config-entry based Home Assistant integration. One config entry
represents one Emby Server and one selected Emby user context.

## Component map

```text
Home Assistant config entry
  EmbyClient
    HTTP API
    WebSocket connection
    browse cache
    request coalescer
    metrics collector
  session coordinator
    media_player
    remote
    notify
  server coordinator
    server sensors
    server binary sensors
  library coordinator
    library-count sensors
  discovery coordinator for the selected user
    discovery sensors
    discovery image entities
  image proxy view
  media source provider
  integration services
```

The integration uses `entry.runtime_data` to store an `EmbyRuntimeData`
instance. It holds the session, server, library, and per-user discovery
coordinators.

## Setup lifecycle

`async_setup_entry` performs the following work:

1. Create an `EmbyClient` with Home Assistant's shared `aiohttp` session.
2. Validate authentication and read server information.
3. Create the session, server, library, and discovery coordinators.
4. Complete each coordinator's first refresh.
5. Store the coordinators in config-entry runtime data.
6. Register the Emby Server device before child entities are added.
7. Register the image proxy and integration actions once per Home Assistant
   instance.
8. Forward setup to all entity platforms.
9. Start the WebSocket connection when enabled.
10. Register update and unload callbacks.

Temporary connection failures raise `ConfigEntryNotReady`. Authentication
failures raise `ConfigEntryAuthFailed` and use Home Assistant's
reauthentication flow.

## Coordinators

| Coordinator | Data | Default refresh | Primary consumers |
| --- | --- | --- | --- |
| Session | Active Emby sessions keyed by stable device ID | 10 seconds, reduced when WebSocket is healthy | Media player, remote, notify |
| Server | Version, tasks, plugins, recordings, status | 300 seconds | Server sensors and binary sensors |
| Library | Media counts and scan state | 3600 seconds | Library sensors |
| Discovery | User-specific media rows and counts | 900 seconds | Discovery sensors and image entities |

WebSocket messages can update or invalidate coordinator data before the next
scheduled refresh.

## Session model

An `EmbySession` represents one connected client. The stable `device_id` is used
for entity identity because Emby's `session_id` can change after reconnection.

The session contains:

- Client, device, application, and user details
- Remote-control capability and supported commands
- Current `EmbyMediaItem`
- Current `EmbyPlaybackState`
- Playable media types
- Queue item IDs and queue position
- Last activity timestamp

An `EmbyMediaItem` keeps the item, series, season, and album identifiers needed
for metadata, remote playback, and artwork fallback.

## Entity platforms

| Platform | Source and behavior |
| --- | --- |
| `media_player` | One entity per known client device, backed by session state |
| `remote` | Client navigation and remote commands |
| `notify` | Session-specific on-screen messages |
| `button` | Library refresh and scan actions |
| `sensor` | Session, library, server, recording, plugin, and watch data |
| `binary_sensor` | Server connection, update, restart, scan, and Live TV state |
| `image` | Artwork for discovery rows |

Media player state maps to `off` when the session is absent, `idle` when the
session has no active item, `paused` when Emby reports a paused play state, and
`playing` otherwise.

## Browse Media

`media_player.py` exposes the native Home Assistant Browse Media tree. Content
IDs encode the Emby item or navigation type so a later browse or play request
can reconstruct the target.

The Movies root includes complete-library and server-sorted views in addition
to filters. The complete and sorted views are capped at 1,000 items per
response. Other branches load child items as the user navigates.

Browse results use the local image proxy for thumbnails. They never include an
Emby API key or a direct HTTP artwork URL.

The separate media source provider exposes Emby libraries through Home
Assistant's Media panel.

## Remote playback

`async_play_media` resolves the selected item and target session, then:

1. Request playback information for the selected Emby user.
2. Select a media source that matches the configured direct-play or transcoding
   settings.
3. Build a `PlayRequest` containing the item, media source, controlling user,
   and start position.
4. Send the request to `/Sessions/{session_id}/Playing`.

Emby Server 4.9 expects `ItemIds`, `StartPositionTicks`, and `PlayCommand` in
the query string. A new browse selection uses `PlayNow`, so it replaces content
already playing on that session.

The client must have an active Emby session. The integration controls Emby, not
the operating system application launcher.

## Artwork delivery

The registered view has this route:

```text
/api/embymedia/image/{server_id}/{item_id}/{image_type}
```

The browser requests this same-origin route. Home Assistant resolves the
coordinator for `server_id`, then the proxy fetches the image from Emby with the
API key held only on the server side.

Browse and discovery artwork streams through the proxy with bounded chunk
sizes. Tagged images receive long cache headers. Untagged images receive a
short cache lifetime.

Now-playing artwork has additional behavior:

1. Episodes prefer `SeasonId`, then `SeriesId`, then the episode item.
2. Audio can fall back to its album.
3. The media player publishes the same proxy URL for `entity_picture` and
   `entity_picture_local`, avoiding Home Assistant's generic double proxy.
4. Player URLs request `layout=player`.
5. The proxy embeds the unmodified raster artwork in a transparent 3:1 SVG
   canvas with `preserveAspectRatio="xMidYMid meet"`.

The canvas lets Home Assistant apply its normal cover-style layout while
keeping the portrait poster inside the visible region of square and wide
surfaces.

## WebSocket lifecycle

The session coordinator owns the WebSocket connection. Session messages update
media players, while library and user messages invalidate or refresh related
coordinators.

The receive loop and health-check loop are created with the config entry's
background-task API. This marks them as permanent runtime work, so Home
Assistant does not wait for them during startup completion. Unload callbacks
cancel the loops and close the connection.

When WebSocket setup fails, the integration logs a warning and continues with
polling. Reconnection uses bounded backoff.

## Caching and request control

The client includes:

- A bounded Browse Media cache
- Time-based expiry and library-change invalidation
- Request coalescing for identical calls already in progress
- API, coordinator, and WebSocket metrics included in diagnostics

The artwork proxy streams ordinary images. The player-layout branch reads only
the resized player image into memory so it can embed it in the SVG canvas.

## Actions and administration

Integration actions are registered once and routed to the config entry selected
by the target media player. They cover client messaging, general commands,
watched state, favorites, playlists, collections, recordings, scheduled tasks,
queue clearing, and server administration.

Standard `media_player`, `remote`, and `notify` actions are implemented through
their normal Home Assistant entity platforms.

## Security boundaries

- The Emby API key stays in config-entry data and server-side requests.
- Diagnostics redact authentication data.
- Frontend state receives local artwork paths rather than authenticated Emby
  URLs.
- SSL verification is enabled by default.
- The player canvas embeds only a small allowlist of raster content types.

Users should still review diagnostics and logs before sharing them because
device, user, library, and host names may be private.

## Related documentation

- [Configuration](CONFIGURATION.md)
- [Efficiency](EFFICIENCY.md)
- [Services](SERVICES.md)
- [Troubleshooting](TROUBLESHOOTING.md)
