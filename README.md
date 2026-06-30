# Etincelle Receiver

A custom [Google Cast](https://developers.google.com/cast) **Web Receiver** (CAF) for the
[etincelle](https://github.com/renaudallard/etincelle) Android client and the
[Molotov TV Home Assistant integration](https://github.com/renaudallard/homeassistant_molotov_tv).

It plays Molotov / Fubo streams on a Chromecast, handles their Widevine (DRMtoday)
DRM, auto-selects the French audio track, and adds on-screen playback controls
driven by a TV remote. It is the fallback receiver ("Récepteur Arnor"), used when
the native Molotov streamer is unavailable.

## What it does

- **DRM** — reads `license_url` + `drm_token` from the cast `customData` and injects
  the `x-dt-auth-token` header on the Widevine license request (DRMtoday).
- **French audio** — auto-selects the French (VF) track when available.
- **Audio / subtitle tracks** — pick an audio track, or a subtitle track (or turn
  subtitles off), from the on-screen menu.
- **Remote controls** — an on-screen menu and direct D-pad seeking for devices with
  a remote (Chromecast with Google TV, Android TV).
- **Live rewind hand-off** — when a live show is cast after rewinding into its
  time-shift window, the receiver reads `live_rewind_sec` from the `customData` and
  resumes that far behind the live edge instead of jumping to the live edge.
- **Error notices** — surfaces playback errors briefly on screen instead of failing
  silently to a black screen.
- **Self-recovery** — recovers from a playback failure or a stall without
  intervention. On an error it first waits briefly for the phone to re-resolve
  fresh credentials and re-cast — the cached stream URL and DRM token are
  short-lived and cannot be refreshed on the device, so only the phone can mint
  new ones — and that fresh cast supersedes any reload. If no fresh cast arrives
  (e.g. the phone is away), it falls back to reloading the cached stream, backing
  off between attempts, and a reload resumes a replay where playback was. After
  repeated failures it stops and asks you to cast again rather than looping silently.
- **Deep buffering** — replays, recordings, and VOD buffer well ahead, so a brief
  network drop drains the buffer instead of stalling the show. Live is left small,
  since it cannot buffer past the live edge.

### Remote controls

| Key | Action |
|-----|--------|
| **◄ / ►** | Rewind 10 s / forward 30 s. **Hold to accelerate** — the step grows quickly the longer the button is held (capped high, so a held rewind crosses the 4 h time-shift window in a few seconds); the preview shows the target position and the speed (×N). Repeated presses accumulate and commit as a single seek. On a live channel a rewind **stops at the start of the current programme** so you do not overshoot into the previous show; once stopped there, press **◄** again to rewind further back into the time-shift window. |
| **▼** | Open the menu (rewind, forward, restart, audio track, subtitles). |
| **▲ / Back** | Close the menu. |
| **OK** | Open the menu, or confirm the highlighted item. |
| **Play/Pause** | Pause or resume playback. On a live channel, pausing drops behind the live edge into the time-shift window. |

While you seek on a live channel, a thin bar at the bottom is scaled to the **current programme**: the
bar's left edge is where the show begins, the part of the show up to the seek target is filled in, the
live edge sits part-way along with the not-yet-aired remainder dimmer past it, a tick marks the live
edge, and any stretch too old to rewind to is drawn in a distinct colour. A rewind stops at the left
edge (the show start) unless you press **◄** again to go further back. The bar appears alongside the
seek preview and clears once you stop seeking, so it never sits on screen during normal playback.

## Hosting

The receiver must be served over **HTTPS**. The simplest option is **GitHub Pages**:

1. In this repository's **Settings → Pages**, set the source to the `main` branch
   (root folder). It is then served at
   `https://renaudallard.github.io/etincelle_receiver/`.
2. That URL is your **Receiver Application URL**.

## Registering with Google Cast

1. Open the **Google Cast SDK Developer Console**: <https://cast.google.com/publish/>
   (a one-time $5 developer registration is required).
2. Create a **Custom Receiver** and set its URL to your hosted `index.html`.
3. Register your Chromecast as a **test device** to try it before publishing.
4. Copy the **Application ID** it issues.

Propagation of a new or updated receiver can take from 15 minutes to a few hours.

## Using it in the integration

Set your Application ID in the integration's `custom_components/molotov_tv/const.py`:

```python
CUSTOM_RECEIVER_APP_ID = "XXXXXXXX"  # your Cast Application ID
```

## Using it in etincelle (Android)

The [etincelle](https://github.com/renaudallard/etincelle) Android app casts to this receiver from
the phone by default; its Application ID is set in the app.

## Related

- etincelle Android client: <https://github.com/renaudallard/etincelle>
- Home Assistant integration: <https://github.com/renaudallard/homeassistant_molotov_tv>

## License

BSD 2-Clause — see [LICENSE](LICENSE).
