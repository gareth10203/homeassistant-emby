# Emby Media documentation

These guides describe the installed integration as a finished Home Assistant
product. Start with installation and configuration, then use the workflow and
reference documents as needed.

## Start here

| Guide | Use it for |
| --- | --- |
| [Installation](INSTALLATION.md) | Installing through HACS, changing from the upstream repository, manual installation, and updates |
| [Configuration](CONFIGURATION.md) | Connecting a server, selecting an Emby user, filtering clients, and tuning playback or refresh behavior |
| [Troubleshooting](TROUBLESHOOTING.md) | Resolving connection, playback, browsing, artwork, startup, and HACS problems |

## Everyday use

| Guide | Use it for |
| --- | --- |
| [Automation examples](AUTOMATIONS.md) | Playback-aware lighting, notifications, client control, and device triggers |
| [Services reference](SERVICES.md) | Standard media actions and every `embymedia` action |
| [Known limitations](KNOWN_ISSUES.md) | Client-session requirements and platform-dependent behavior |

## Technical reference

| Guide | Use it for |
| --- | --- |
| [Architecture](ARCHITECTURE.md) | Config entries, coordinators, WebSocket updates, entities, and API flow |
| [Efficiency](EFFICIENCY.md) | Polling, caching, request coalescing, and performance options |
| [Integration comparison](COMPARISON_MATRIX.md) | Neutral feature-level comparison with the Home Assistant core integrations |
| [Changelog](../CHANGELOG.md) | Released and unreleased changes |
| [Contributing](../CONTRIBUTING.md) | Local setup, checks, and contribution process |

## Core workflow

1. Install `gareth10203/homeassistant-emby` as a HACS custom integration.
2. Add Emby Media from **Settings**, then **Devices & services**.
3. Select an Emby user for library and discovery data.
4. Open the Emby application on the target playback device.
5. Use the target media player's **Browse media** action to select content.

Selecting a new item replaces content that is already playing. The target must
have an active Emby session so the server has somewhere to deliver the command.

## Browse and artwork behavior

The Movies browser includes All Movies, Date Added newest first, Premiere Date
newest first, A to Z, Year, Decade, Genre, Studio, Collections, People, and
Tags. Television, music, playlists, collections, and Live TV have their own
navigation trees.

Artwork is served through Home Assistant rather than directly from Emby. This
prevents HTTPS mixed-content failures and keeps the Emby API key out of browser
state. Episode playback prefers the season cover, then the series cover. A
responsive player layout keeps the full portrait image visible in media cards,
entity tiles, and the More Info dialog.

## Entity overview

| Platform | Typical use |
| --- | --- |
| `media_player` | Browse, start content, and control active Emby clients |
| `remote` | Navigate a compatible Emby client |
| `notify` | Show a message on a client |
| `button` | Refresh or scan the server library |
| `sensor` | Read library, session, recording, plugin, and server statistics |
| `binary_sensor` | Read server, update, scan, restart, and Live TV state |
| `image` | Display Next Up, Continue Watching, Recently Added, and Suggestions artwork |

## Getting help

Read [Troubleshooting](TROUBLESHOOTING.md) before opening an issue. Download
integration diagnostics and remove any private information before sharing.
Never publish API keys, access tokens, or private SSH keys.

Issues for this fork are handled at
[github.com/gareth10203/homeassistant-emby/issues](https://github.com/gareth10203/homeassistant-emby/issues).

Return to the [main README](../README.md).
