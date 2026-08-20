# Installation

HACS is the recommended installation method. It keeps the integration folder in
the expected location and makes later downloads straightforward.

## Requirements

| Component | Minimum version |
| --- | --- |
| Home Assistant | 2025.11.3 |
| Emby Server | 4.9.1.90 |
| Network | Home Assistant can reach the Emby host and port |

## Install through HACS

1. Open HACS in Home Assistant.
2. Open the menu and select **Custom repositories**.
3. Enter `https://github.com/gareth10203/homeassistant-emby`.
4. Choose **Integration** and add the repository.
5. Find **Emby Media** in HACS and select **Download**.
6. Restart Home Assistant.
7. Go to **Settings**, then **Devices & services**.
8. Select **Add integration** and search for **Emby Media**.

The downloaded files are stored in:

```text
/config/custom_components/embymedia
```

HACS may show a commit hash as the version when the repository has no matching
tagged release. That is a valid installation.

## Change from the upstream repository

Changing the HACS source does not require deleting the Home Assistant config
entry.

1. Remove the previous Emby Media custom repository from HACS.
2. Add `https://github.com/gareth10203/homeassistant-emby` as an Integration.
3. Download Emby Media from the new repository.
4. Confirm HACS reports the new commit.
5. Restart Home Assistant.

The existing server connection and entities remain registered. Removing and
re-adding the integration is only necessary when the config entry itself is
damaged or you want to discard its settings.

## Manual installation

1. Download the repository archive from
   `https://github.com/gareth10203/homeassistant-emby/archive/refs/heads/main.zip`.
2. Open the archive and locate `custom_components/embymedia`.
3. Copy the complete `embymedia` directory to
   `/config/custom_components/embymedia`.
4. Restart Home Assistant.

The resulting structure should begin like this:

```text
config/
  custom_components/
    embymedia/
      __init__.py
      manifest.json
      media_player.py
      image_proxy.py
      services.yaml
```

Do not copy the repository's outer directory into `custom_components`.

## Update an existing installation

### HACS

1. Open the Emby Media entry in HACS.
2. Select **Redownload** or install the offered update.
3. Check that the downloaded version or commit is the expected one.
4. Restart Home Assistant.

Home Assistant loads Python files in `custom_components` only during startup.
A dashboard refresh is not enough after an integration update.

### Manual installation

Replace the files in `/config/custom_components/embymedia` with the files from
the new archive, then restart Home Assistant. Preserve no individual old Python
files because mixed versions can fail during setup.

## Verify the installed version

Open:

```text
/config/custom_components/embymedia/manifest.json
```

The `version` field identifies the installed integration code. Home Assistant
logs also show the custom integration during startup.

After setup, verify the integration from **Settings**, then **Devices &
services**, then **Emby Media**. Server sensors should load even when no Emby
playback client is open.

## If a HACS download fails

If HACS reports `Could not download, see log for details`:

1. Confirm the repository URL is exactly
   `https://github.com/gareth10203/homeassistant-emby`.
2. Confirm the repository is public and opens in a browser.
3. Remove and re-add the custom repository in HACS.
4. Reload HACS or restart Home Assistant.
5. Check **Settings**, then **System**, then **Logs** for the detailed HACS
   error.
6. Check that `/config/custom_components/embymedia` is writable and does not
   contain a partial manual copy.

When HACS names a commit that has not been pushed to GitHub, the download will
fail because that commit exists only in the local checkout. Push the commit,
reload HACS, and try again.

## Remove Emby Media

There are two separate parts:

- The config entry contains the server connection and entity registration.
- The HACS installation contains the files in `custom_components`.

To remove both:

1. Open **Settings**, then **Devices & services**, then **Emby Media**.
2. Delete the config entry from its menu.
3. Open Emby Media in HACS and remove the downloaded integration.
4. Restart Home Assistant.

Deleting only the HACS repository entry does not delete the Home Assistant
config entry. Likewise, deleting the config entry does not necessarily remove
the files downloaded by HACS.

Continue with [Configuration](CONFIGURATION.md).
