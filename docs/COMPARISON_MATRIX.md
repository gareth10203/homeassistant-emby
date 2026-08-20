# Choosing an Emby integration

Home Assistant users can choose between this custom Emby Media integration and
the Emby integration included with Home Assistant Core. Plex users should use a
Plex integration because the server API and client model are different.

This document describes the practical difference without treating the choice
as a feature score.

## This Emby Media fork

Choose this repository when Home Assistant needs to do more than observe an
active Emby session.

It provides:

- UI configuration with Emby user selection and reauthentication
- Per-session media player, remote, and notification entities
- Browse Media navigation for films, television, music, playlists,
  collections, people, tags, studios, genres, years, and decades
- Remote playback to a connected Emby client
- All Movies and newest-first movie views
- Server, library, session, recording, plugin, and watch-statistic sensors
- Discovery feeds for Next Up, Continue Watching, Recently Added, and
  Suggestions
- Playlist, collection, recording, scheduled-task, and server actions
- Same-origin artwork suitable for HTTPS Home Assistant dashboards
- WebSocket updates with HTTP polling fallback

The tradeoff is that it is a custom integration. HACS installation, custom
repository updates, and compatibility checks remain the user's responsibility.

## Home Assistant Core Emby

The core integration is appropriate when its smaller built-in feature set is
enough and avoiding custom code is the priority. Installation and compatibility
follow the Home Assistant release cycle.

Read the current core documentation before choosing because supported setup and
entity behavior can change with Home Assistant releases:

[Home Assistant Emby integration](https://www.home-assistant.io/integrations/emby/)

## Plex

Plex is not a drop-in alternative for an Emby server. Its authentication,
discovery, client control, and library API differ. Compare Plex only when the
media server itself is also a choice.

[Home Assistant Plex integration](https://www.home-assistant.io/integrations/plex/)

## Decision guide

| Requirement | Suggested choice |
| --- | --- |
| Use only integrations shipped with Home Assistant | Core Emby |
| Browse a large Emby library from a media player | This fork |
| Start or replace content on an active Apple TV Emby session | This fork |
| Create automations from detailed Emby server and library data | This fork |
| Avoid HACS and custom integration maintenance | Core Emby |
| Connect to a Plex server | Plex integration |

## Compatibility notes

This fork requires Home Assistant 2025.11.3 or newer and Emby Server 4.9.1.90
or newer. Client commands depend on the Emby application running on the target.
Apple TV is the primary remote-playback target verified by this fork.

This comparison was last reviewed in August 2026. Refer to each integration's
current documentation for changes made after that date.
