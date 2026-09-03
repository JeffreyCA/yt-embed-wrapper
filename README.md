# yt-embed-wrapper

If you've ever embedded a YouTube video on a page running on `localhost` or a plain IP address, you've probably hit "This video is unavailable" on anything from a major label. YouTube decides whether a video can be embedded based on the hostname of the page that contains the player, and localhost doesn't make the cut.

This repo is a small static page, hosted on GitHub Pages, that contains the player on your behalf. You embed the page in an iframe and tell it what to load over `postMessage`. Since the player lives on a real hostname, the videos play.

- Demo: <https://jeffreyca.github.io/yt-embed-wrapper/>
- Wrapper: <https://jeffreyca.github.io/yt-embed-wrapper/embed.html>

## Usage

```html
<iframe id="yt" src="https://jeffreyca.github.io/yt-embed-wrapper/embed.html"
        allow="autoplay; encrypted-media; picture-in-picture" allowfullscreen></iframe>
<script>
  const frame = document.getElementById('yt');
  const origin = 'https://jeffreyca.github.io';
  window.addEventListener('message', ({ origin: from, data }) => {
    if (from !== origin || data?.type !== 'yt-embed') return;
    if (data.event === 'ready') {
      frame.contentWindow.postMessage({ type: 'yt-embed', command: 'load', videoId: 'Zi_XLOBDo_Y' }, origin);
    }
    if (data.event === 'error') {
      // 101 or 150 means the uploader disabled embedding, so link to YouTube instead.
    }
  });
</script>
```

The player fills whatever size you give the iframe. If you just want to open a video without any messaging, `embed.html?v=<videoId>` works too, and you can add `autoplay=1`, `mute=1`, `controls=0`, or `start=<seconds>`.

## Messages

Every message in either direction is an object with `type: 'yt-embed'`.

Events sent by the wrapper:

| Event | Meaning |
|---|---|
| `ready` | The API has loaded and it's safe to send `load` |
| `api-failed` | The API script didn't load, usually because something blocked it |
| `player-ready` | The player has been created |
| `state` | A `YT.PlayerState` value in `state`, with the player's current time in `seconds` |
| `error` | A YouTube error code in `code` (2, 5, 100, 101, 150, 153) |
| `time` | `seconds` and `duration`, sent every 250 ms while playing |

Commands it accepts:

| Command | Fields |
|---|---|
| `load` | `videoId`, plus optional `autoplay`, `start`, `muted` (mute before the video starts and keep it muted while it plays, whatever the player's own chrome or an ad does, until an `unmute` command), and `controls: false` (no control bar and no player keys, so the viewer cannot unmute, scrub, or change the volume; the player's chrome is fixed when it is created, so this applies to the first `load` only) |
| `play`, `pause`, `mute`, `unmute` | `mute` also holds the mute like a muted load; `unmute` releases it |
| `seek` | `seconds` |
| `volume` | `level` from 0 to 100 |
| `rate` | `rate`, a playback speed multiplier; YouTube supports a fixed set (0.25 to 2 in steps) and picks the nearest |

The first `load` binds the wrapper to whichever window sent it. After that, commands from other windows are ignored and events are only posted back to that origin. The initial `ready` event goes to `*` since nothing has been bound yet.

## Privacy

The player uses `youtube-nocookie.com` by default, which means YouTube doesn't set any cookies until someone actually presses play. Pass `nocookie=0` if you'd rather use the regular domain. Beyond that, GitHub sees the request for the page and YouTube sees an ordinary embed with this page as the referrer. There's no analytics and nothing is stored.

## Limitations

This only helps with the hostname check. Videos whose uploader has turned off embedding will still fail with error 101 or 150 no matter where they're embedded, and content blockers can prevent the API script from loading, which shows up as `api-failed`. In both cases the sensible fallback is a link to the video on YouTube.

## License

MIT
