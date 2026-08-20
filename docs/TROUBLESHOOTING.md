# Troubleshooting

Work from the symptom that matches your problem. Restart Home Assistant after
replacing integration files, but do not repeatedly restart for a dashboard-only
problem.

## Confirm the installed code

Check this file first:

```text
/config/custom_components/embymedia/manifest.json
```

The `version` field must match the version you expect HACS or a manual copy to
have installed. HACS can show a commit hash instead of a release number. That
commit still contains a normal integration version in `manifest.json`.

If the files do not match the expected repository, redownload Emby Media from
HACS and restart Home Assistant.

## Setup cannot connect

### Cannot connect

1. Open `http://your-emby-server:8096` from another device on the same network.
2. Confirm the host contains only a hostname or IP address, without `/web` or
   another URL path.
3. Confirm the configured port and SSL setting match Emby Server.
4. Check that the Home Assistant host or container can reach the Emby network.
5. Check firewall and container-network rules.

When Home Assistant and Emby are in separate containers, a browser reaching
Emby does not prove that the Home Assistant container can reach it.

### Invalid API key

Create a new dedicated key in **Emby Server Dashboard**, then **Advanced**, then
**API Keys**. Copy the complete value without spaces. Use Home Assistant's
reauthentication flow when it appears.

Do not test by placing the API key in a dashboard URL or issue report.

### SSL verification failed

Confirm that the certificate is valid for the hostname used by Home Assistant.
For a self-signed certificate on a trusted local network, disable **Verify SSL**
for the Emby entry. Leave verification enabled for a public or CA-signed
certificate.

## A client entity is missing

Emby client entities represent active server sessions.

1. Open the Emby application on the target device.
2. Confirm that it is signed in to the same Emby server.
3. Play or browse something so the server creates a current session.
4. Check **Ignored devices** and **Ignore web players** in the integration
   options.
5. Reload the Emby Media config entry.

The integration cannot wake a client that has no Emby session. Apple TV must
have the Emby application open before it can receive a remote playback command.

Server sensors can remain available while client entities are absent.

## Controls work only while Emby is open

This is expected for clients that close their Emby session in the background.
Home Assistant sends control requests to the session reported by Emby Server.
It does not launch the application through the Apple TV operating system.

Keep Emby open on the target before using play, pause, browse playback, remote
navigation, or on-screen notifications.

## Browse selection does not start playback

1. Confirm the target player is still available and its Emby app is open.
2. Try pause or volume control to confirm the session accepts commands.
3. Select the item again from the target player's **Browse media** action.
4. Check Home Assistant logs for an Emby HTTP status or playback-info error.
5. Confirm Emby Server is at least version 4.9.1.90.

Playback requests require `ItemIds`, `StartPositionTicks`, and `PlayCommand` as
query parameters on Emby Server 4.9. The integration also sends the controlling
user and selected media source. Versions before the remote-play correction can
return success without delivering the request to Apple TV.

To replace something that is already playing, select the new film or episode.
The integration sends `PlayNow`; stopping the current item first is unnecessary.

Apple TV is the primary verified target for this fork. Support on another Emby
client depends on that client's remote-control implementation.

## Browse Media reports an unknown error

An `Unknown error` message is the frontend's generic response to an exception
inside a browse handler. Check Home Assistant logs at the same timestamp.

Useful checks:

1. Confirm the installed integration is current.
2. Open another browse branch to determine whether the failure is limited to a
   filter such as Movies by Year.
3. Retry after the integration has completed its first library refresh.
4. Confirm the selected Emby user can see the library in Emby itself.
5. Download diagnostics before reloading the entry.

Current movie browsing includes All Movies, Date Added newest first, Premiere
Date newest first, A to Z, Year, Decade, Genre, Studio, Collections, People,
and Tags. The Year branch and its container playback handler are supported in
this fork.

## Browse artwork is blank

Browse and discovery artwork should begin with:

```text
/api/embymedia/image/
```

Direct Emby image URLs can be blocked when Home Assistant is HTTPS and Emby is
HTTP. They can also expose an API key in frontend state. Update the integration
if a discovery sensor still publishes a direct Emby URL.

To test a proxy URL, prepend the Home Assistant address and open it in the same
browser. Use one slash between the port and `api`:

```text
https://home-assistant.example:8123/api/embymedia/image/...
```

A double slash before `api` can produce a false 404 result.

## Now-playing artwork is blank

Open **Developer tools**, then **States**, and inspect the Emby media player.
Both of these attributes should use the Emby image endpoint:

```text
entity_picture: /api/embymedia/image/...
entity_picture_local: /api/embymedia/image/...
```

If `entity_picture_local` begins with `/api/media_player_proxy/`, the installed
version is double-proxying a relative image URL. That generic Home Assistant
proxy can return HTTP 500. Install the current fork and restart Home Assistant.

For player artwork, the URL should include:

```text
maxWidth=600&maxHeight=900&layout=player
```

The player layout places the original cover on a transparent responsive canvas.
This prevents Home Assistant's media card, Overview tile, and More Info dialog
from cropping a portrait season cover.

If the direct URL works but an old crop remains, hard-refresh the browser after
restarting Home Assistant. The `layout=player` URL is distinct from the old
cached image URL.

## The wrong episode artwork is shown

Episode playback uses the current season's Primary image when Emby includes a
`SeasonId`. If it is missing, the integration uses the series Primary image,
then the episode image.

Check `media_series_title`, `media_season`, `media_episode`, and
`entity_picture` in Developer Tools. If the title data is correct but the image
belongs to another season, refresh metadata for that season in Emby Server.

## HACS cannot download the repository

For an error such as `Could not download, see log for details`:

1. Confirm the custom repository is
   `https://github.com/gareth10203/homeassistant-emby`.
2. Confirm the repository is public and available in a browser.
3. Confirm the requested commit has been pushed to GitHub.
4. Remove and re-add the custom repository in HACS.
5. Reload HACS, then retry the download.
6. Read the HACS log entry for the underlying HTTP or archive error.

A local commit cannot be downloaded by HACS until it has been pushed to the
repository.

## Home Assistant startup takes several minutes

Current versions register the permanent Emby WebSocket receive loop and health
check as config-entry background tasks. They should not hold Home Assistant in
the startup stage.

If startup still pauses for about five minutes:

1. Confirm the installed version and restart once.
2. Check logs for Emby tasks named `websocket_receive` or
   `websocket_health_check` in a bootstrap timeout.
3. Redownload the current integration if those tasks still block setup.
4. Check connectivity to Emby Server because repeated connection failures can
   produce separate retry messages.

## State updates are delayed

Check the integration options and logs:

1. Keep WebSocket enabled for normal use.
2. Confirm a successful WebSocket connection appears after setup.
3. Check that a reverse proxy between Home Assistant and Emby is not involved
   in the server-side connection. Home Assistant connects directly to Emby.
4. Increase polling frequency only after resolving WebSocket problems.

The integration falls back to HTTP polling after a WebSocket interruption and
reconnects automatically.

## Download diagnostics

1. Open **Settings**, then **Devices & services**.
2. Select **Emby Media**.
3. Open the entry menu and select **Download diagnostics**.

Diagnostics are designed to redact authentication fields, but review the file
before sharing it. Remove hostnames, usernames, library names, or client names
when they are private.

## Enable debug logging

Add this temporarily to `configuration.yaml`:

```yaml
logger:
  logs:
    custom_components.embymedia: debug
```

Restart Home Assistant, reproduce the problem once, collect the relevant log
section, then remove or lower debug logging to avoid unnecessary log volume.

## Report a problem

Search the fork's existing issues, then open a new issue at
[github.com/gareth10203/homeassistant-emby/issues](https://github.com/gareth10203/homeassistant-emby/issues).

Include versions, the affected client platform, a concise reproduction, logs,
and redacted diagnostics. Never include an Emby API key, Home Assistant bearer
token, signed entity-picture URL, or private SSH key.
