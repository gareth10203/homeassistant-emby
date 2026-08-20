# Known limitations

There are no confirmed integration defects listed for version `0.6.5`.
The behaviors below are product or client limitations that can otherwise look
like faults.

## Playback clients are session based

A client media player exists when Emby Server reports a session for that
device. The integration cannot send an Emby playback request to a closed client
or launch the Emby application through the device operating system.

Open Emby on the target before using Home Assistant controls. Apple TV remote
playback is verified when the Emby app is open and connected.

## Client command support varies

Emby applications do not all implement the same remote commands. Basic
playback is widely available, while directional navigation, notifications,
queue operations, seeking, volume, and application-level commands can vary by
platform.

Home Assistant reports the features exposed by the integration, but the target
client has the final say on whether a command is applied.

## Library views have a response limit

All Movies and the two newest-first movie views return up to 1,000 items in a
single Home Assistant browse response. This avoids overwhelming the frontend.
Filtered views remain available for larger libraries.

## Player artwork uses a presentation canvas

Home Assistant crops media artwork differently in wide cards, square tiles,
and the More Info dialog. Now-playing artwork is therefore served on a
transparent 3:1 canvas. The cover itself is not altered, but opening the proxy
URL directly shows transparent or browser-colored space around it.

Browse and discovery artwork is not placed on this canvas.

## Session and library timing

WebSocket is the primary source of live changes. Polling is retained as a
fallback and for periodic reconciliation. A disconnected WebSocket can make a
state update arrive on the next polling cycle instead of immediately.

## Platform-specific verification

Apple TV is the primary playback target tested by this fork. Other Emby clients
are expected to work through the same server API, but results depend on their
remote-control implementation and network availability.

Report a reproducible problem at
[github.com/gareth10203/homeassistant-emby/issues](https://github.com/gareth10203/homeassistant-emby/issues)
with versions, logs, and redacted diagnostics.
