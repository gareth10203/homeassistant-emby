# Emby Media for Home Assistant

Emby Media connects an Emby server to Home Assistant as a set of native media
players, remotes, notifications, library sensors, discovery feeds, server
controls, and media sources.

This fork focuses on practical playback from Home Assistant. It can browse an
Emby library, start a film or episode on a connected client, replace content
that is already playing, and present artwork without exposing the Emby API key
to the browser.

Current integration version: `0.6.5`

## What it provides

| Area | Included functionality |
| --- | --- |
| Client control | Play, pause, stop, seek, skip, mute, volume, shuffle, repeat, and queue control |
| Remote playback | Start a selected Emby item on an active client, including Apple TV |
| Library browser | Movies, television, music, playlists, collections, people, tags, studios, genres, years, and decades |
| Movie views | All Movies, Date Added newest first, Premiere Date newest first, A to Z, and filtered views |
| Artwork | Same-origin image proxy, discovery thumbnails, season-first episode covers, and responsive player artwork |
| Discovery | Recently Added, Continue Watching, Next Up, and Suggestions |
| Server monitoring | Connection state, version, tasks, library counts, sessions, recordings, plugins, and activity |
| Home Assistant platforms | Media player, remote, notify, button, sensor, binary sensor, image, and media source |
| Administration | Library refresh, scheduled tasks, recording management, playlists, collections, restart, and shutdown |

State changes normally arrive over WebSocket. HTTP polling remains available
as a fallback when the WebSocket connection is unavailable.

## Requirements

| Component | Minimum version |
| --- | --- |
| Home Assistant | 2025.11.3 |
| Emby Server | 4.9.1.90 |
| HACS | Current version, when using the recommended installation method |

Home Assistant must be able to reach the Emby server over the configured host
and port.

## Install with HACS

This repository is installed as a custom HACS integration.

1. Open HACS in Home Assistant.
2. Open the menu and select **Custom repositories**.
3. Add `https://github.com/gareth10203/homeassistant-emby`.
4. Select **Integration** as the category.
5. Open **Emby Media** and select **Download**.
6. Restart Home Assistant.

HACS may identify a download by its commit hash when the repository does not
have a matching tagged release. This is expected.

### Moving from the upstream repository

An existing Emby Media config entry does not need to be removed.

1. Remove the old Emby Media repository from HACS custom repositories.
2. Add this repository using the URL above.
3. Download Emby Media again so HACS writes it to
   `/config/custom_components/embymedia`.
4. Restart Home Assistant.

The server connection, selected Emby user, entity registry, and dashboard
configuration remain in Home Assistant.

For manual installation and recovery steps, see
[Installation](docs/INSTALLATION.md).

## Connect an Emby server

Create a dedicated API key in the Emby Server Dashboard:

1. Open **Settings** in Emby Server.
2. Go to **Advanced**, then **API Keys**.
3. Create a key named `Home Assistant`.
4. Copy the key and keep it private.

In Home Assistant:

1. Go to **Settings**, then **Devices & services**.
2. Select **Add integration** and search for **Emby Media**.
3. Enter the host, port, SSL settings, and API key.
4. Select the Emby user whose library and discovery data should be used.
5. Complete the options step.

The integration validates the server version and API key before creating the
entry. Configuration can be changed later from the integration's **Configure**
button.

See [Configuration](docs/CONFIGURATION.md) for every available option.

## Use the media players

Each connected Emby client session becomes a Home Assistant media player.
Companion remote and notification entities are created for the same client.

The Emby application must be open and connected to the server before it can
receive a remote playback request. If the application is closed, there is no
active Emby session to target. Some clients may retain a session while idle,
but this behavior is controlled by the Emby client.

Apple TV is the primary verified playback target for this fork. Other Emby
clients can work when they implement the same Emby remote-control commands.

### Start or replace playback

1. Open a Home Assistant media player card for the Emby client.
2. Select **Browse media**.
3. Navigate to a film, episode, track, playlist, collection, or Live TV item.
4. Select the item to start it on that client.

Selecting another playable item sends a new play request to the same session.
It replaces the item that is currently playing. There is no need to stop the
first item manually.

Playback requests include the controlling Emby user and selected media source,
and send Emby's required `ItemIds`, `StartPositionTicks`, and `PlayCommand`
parameters. This behavior is compatible with Emby Server 4.9 and has been
verified on Apple TV.

## Browse the library

The media browser is available from each Emby media player and through Home
Assistant's Media panel.

| Library | Available navigation |
| --- | --- |
| Movies | All, Date Added, Premiere Date, A to Z, Year, Decade, Genre, Studio, Collections, People, and Tags |
| TV Shows | A to Z, Year, Decade, Genre, Studio, Series, Season, and Episode |
| Music | Artists, Albums, Genres, playlists, and alphabetical filters |
| Other | Collections, playlists, Live TV channels, and user libraries |

Movie sort views are requested directly from Emby and return up to 1,000 items
per view. The limit protects the Home Assistant frontend from exceptionally
large single responses.

## Artwork behavior

All frontend artwork uses the integration's Home Assistant image endpoint.
This avoids mixed-content failures when Home Assistant uses HTTPS and Emby uses
HTTP. It also keeps the Emby API key out of entity state and browser requests.

For an episode, the player uses this order:

1. Current season cover
2. Series cover
3. Episode artwork

Home Assistant uses different cover-cropped shapes for media cards, entity
tiles, and the More Info dialog. Player artwork is placed on a transparent
responsive canvas so a portrait season cover remains complete in square and
wide layouts. Browse and discovery thumbnails keep their original shape.

## Entities and data

| Platform | Purpose |
| --- | --- |
| `media_player` | Session state, metadata, artwork, playback control, and media browsing |
| `remote` | Directional and client navigation commands |
| `notify` | On-screen messages sent to a client |
| `button` | Library refresh and server scan actions |
| `sensor` | Server, library, session, recording, plugin, and watch statistics |
| `binary_sensor` | Server connection, restart, update, scan, and Live TV status |
| `image` | Discovery artwork for Next Up, Continue Watching, Recently Added, and Suggestions |

Client entities are session based. Server and library sensors remain available
when no playback client is connected.

## Automation example

Standard Home Assistant media player actions work with Emby client entities.

```yaml
automation:
  - alias: Pause Emby when the doorbell rings
    triggers:
      - trigger: state
        entity_id: binary_sensor.doorbell
        to: "on"
    actions:
      - action: media_player.media_pause
        target:
          entity_id: media_player.emby_apple_tv_emby_apple_tv
```

The integration also exposes device triggers for playback and session changes,
plus Emby-specific actions for messages, favorites, watched state, playlists,
collections, recordings, scheduled tasks, and server administration.

See [Automation examples](docs/AUTOMATIONS.md) and
[Services reference](docs/SERVICES.md).

## Configuration notes

Useful options include:

- WebSocket updates and subscription interval
- HTTP polling interval
- Ignored client names and browser-session filtering
- Direct play and transcoding profile
- Video container and bitrate limits
- Entity name prefixes
- Discovery sensors and refresh interval
- Library and server refresh intervals

WebSocket is enabled by default. When it is healthy, polling is reduced and
used as a fallback rather than the primary source of session updates.

## Support

Start with [Troubleshooting](docs/TROUBLESHOOTING.md). It covers installation,
client discovery, Apple TV playback, browse errors, artwork, HACS downloads,
WebSocket behavior, and diagnostics.

When reporting a problem, include:

- Home Assistant version
- Emby Media version
- Emby Server version
- Client name and platform
- Relevant Home Assistant log lines
- Diagnostics downloaded from the integration page

Do not include an Emby API key, Home Assistant access token, private SSH key,
or an unredacted diagnostics file.

Issues for this fork are tracked at
[gareth10203/homeassistant-emby](https://github.com/gareth10203/homeassistant-emby/issues).

## Documentation

| Document | Contents |
| --- | --- |
| [Documentation index](docs/README.md) | Guide to all user and technical documentation |
| [Installation](docs/INSTALLATION.md) | HACS, manual installation, upgrades, and repository changes |
| [Configuration](docs/CONFIGURATION.md) | Connection fields, users, filters, updates, and playback profiles |
| [Troubleshooting](docs/TROUBLESHOOTING.md) | Symptom-based checks and recovery steps |
| [Services](docs/SERVICES.md) | Standard and Emby-specific action reference |
| [Automations](docs/AUTOMATIONS.md) | Automation patterns and device triggers |
| [Architecture](docs/ARCHITECTURE.md) | Runtime components and data flow |
| [Changelog](CHANGELOG.md) | Version history |

## Project background

This repository is a maintained fork of
[troykelly/homeassistant-emby](https://github.com/troykelly/homeassistant-emby).
The original project established the integration architecture and broad Emby
feature set. This fork retains that work and adds its own playback, browsing,
artwork, startup, and maintenance changes.

Emby is a product of Emby LLC. Home Assistant is a project of the Open Home
Foundation. This integration is an independent community project and is not
endorsed by either organization.

The source is distributed under the [Apache License 2.0](LICENSE).
