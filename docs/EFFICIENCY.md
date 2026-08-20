# Efficiency and tuning

The default settings are suitable for most Emby servers. WebSocket updates,
bounded caches, request coalescing, and separate coordinator intervals reduce
unnecessary API traffic without making playback state stale.

## Update sources

| Data | Default interval | Event support |
| --- | --- | --- |
| Client sessions | 10 seconds when polling is required | WebSocket session and playback messages |
| Server status | 300 seconds | Periodic reconciliation |
| Library counts | 3600 seconds | Library-change invalidation |
| Discovery data | 900 seconds | User-data and library invalidation |

When the WebSocket connection is healthy, it is the primary source of session
changes. HTTP polling remains as a slower consistency check and as the fallback
after a disconnect.

## WebSocket behavior

Keep WebSocket enabled unless the Emby endpoint cannot support it. It provides
the lowest-latency state changes with less repetitive HTTP traffic.

The receive and health-check loops are registered as config-entry background
tasks. They remain active for the lifetime of the integration without delaying
Home Assistant startup completion. Unloading the config entry cancels them.

After a disconnect, the integration continues polling and reconnects with
bounded backoff.

## Browse cache

Browse Media requests can produce large item trees. The integration uses a
bounded least-recently-used cache with time-based expiry.

| Property | Value |
| --- | --- |
| Lifetime | 5 minutes |
| Maximum entries | 500 |
| Invalidation | Expiry, explicit refresh, or library change |

All Movies and newest-first movie views return at most 1,000 items in one
response. Users with larger libraries can use the available filters rather
than loading every item into one frontend card.

## Request coalescing

If several entities ask for the same Emby resource at the same time, the client
shares the in-progress result instead of issuing duplicate HTTP requests. The
entry is removed as soon as the request completes or fails.

This is most useful during setup, coordinator refreshes, and dashboards that
load several related sensors together.

## Artwork traffic

Browse and discovery images stream from Emby through Home Assistant in bounded
chunks. The browser receives cache headers based on the image tag:

| Image | Cache lifetime |
| --- | --- |
| Tagged artwork | One year |
| Artwork without a tag | Five minutes |

The player requests a resized source up to 600 by 900 pixels. For
`layout=player`, the proxy reads that resized image into memory and embeds it in
a small SVG canvas. Ordinary browse and discovery requests remain streamed and
do not use the canvas branch.

## Recommended settings

### Typical server

Use the defaults:

```text
WebSocket: enabled
Session scan interval: 10 seconds
Discovery scan interval: 900 seconds
Library scan interval: 3600 seconds
Server scan interval: 300 seconds
```

### Low-power server

Keep WebSocket enabled and increase periodic refresh intervals:

```text
Discovery scan interval: 3600 seconds
Library scan interval: 86400 seconds
Server scan interval: 3600 seconds
```

### Faster polling fallback

Use a 5-second session interval only when WebSocket cannot remain connected and
the extra Emby API load is acceptable. Reducing the interval does not improve a
healthy WebSocket connection.

## Client filters

Use **Ignored devices** or **Ignore web players** when temporary or irrelevant
sessions create unnecessary entities. Filtering reduces entity updates and
dashboard noise, but it does not prevent those sessions from existing on Emby
Server.

## Diagnostics

Integration diagnostics include API timings, errors, WebSocket activity, cache
statistics, coalescing data, and coordinator update metrics.

To download them:

1. Open **Settings**, then **Devices & services**.
2. Select **Emby Media**.
3. Open the entry menu and select **Download diagnostics**.

Review the file before sharing it. Authentication fields are redacted, but
names and library details may still be private.

## When to tune

Change defaults only when measurements show a reason:

- Delayed state with a disconnected WebSocket
- High request volume on a low-power Emby server
- A very large library causing slow browse rendering
- Discovery data that does not need frequent refreshes

Use diagnostics and Emby Server logs to compare behavior before and after a
change. Larger intervals reduce traffic but delay periodic reconciliation.
