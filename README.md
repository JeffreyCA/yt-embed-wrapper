# yt-embed-wrapper

A static wrapper for embedding YouTube videos from `localhost` or a plain IP address.

YouTube blocks many label-owned videos when the player is embedded directly from a local development origin. This project hosts the player on GitHub Pages instead and lets the parent page control it with `postMessage`.

- [Demo](https://jeffreyca.github.io/yt-embed-wrapper/)
- [Wrapper](https://jeffreyca.github.io/yt-embed-wrapper/embed.html)

## Usage

```html
<iframe
  id="youtube"
  src="https://jeffreyca.github.io/yt-embed-wrapper/embed.html"
  allow="autoplay; encrypted-media; picture-in-picture"
  allowfullscreen
></iframe>

<script>
  const origin = 'https://jeffreyca.github.io';
  const frame = document.getElementById('youtube');

  window.addEventListener('message', (event) => {
    if (event.origin !== origin || event.data?.type !== 'yt-embed') return;

    if (event.data.event === 'ready') {
      frame.contentWindow.postMessage(
        { type: 'yt-embed', command: 'load', videoId: 'Zi_XLOBDo_Y' },
        origin,
      );
    }

    if (event.data.event === 'error') {
      console.error('YouTube player error:', event.data.code);
    }
  });
</script>
```

The player fills the iframe. Size the iframe or its container to fit your layout.

For a direct embed without messaging, use `embed.html?v=<videoId>`. It also accepts `autoplay=1`, `mute=1`, `controls=0`, `start=<seconds>`, and `nocookie=0`. Autoplay remains subject to browser media policies.

## Messages

Messages in both directions are objects with `type: 'yt-embed'`.

### Commands

| Command | Fields |
| --- | --- |
| `load` | `videoId`, optional `autoplay`, `start`, `muted`, and `controls` |
| `play`, `pause`, `mute`, `unmute` | None |
| `seek` | `seconds` |
| `volume` | `level` from `0` to `100` |
| `rate` | Positive playback speed in `rate`; YouTube selects the nearest supported value |

A muted load stays muted until `unmute` is sent. Passing `controls: false` hides the control bar and disables player keyboard controls, but only applies to the first `load` because YouTube fixes the player chrome when it is created.

### Events

| Event | Fields | Meaning |
| --- | --- | --- |
| `ready` | None | The iframe API loaded and `load` can be sent |
| `api-failed` | None | The iframe API script failed to load |
| `player-ready` | None | The YouTube player was created |
| `state` | `state`, `seconds` | Playback changed to a `YT.PlayerState` value |
| `time` | `seconds`, `duration` | Position update sent every 250 ms while playing |
| `error` | `code` | YouTube reported an error |

The first valid `load` binds the wrapper to its sender. Commands from other windows are then ignored, and events are sent only to the bound origin. The initial `ready` event uses `*` because no parent has been bound yet, so parent pages should validate `event.origin`.

## Privacy and limitations

The wrapper uses `youtube-nocookie.com` by default and has no analytics or storage. Pass `nocookie=0` to use the regular YouTube domain.

This only works around YouTube's hostname check. Videos whose uploader disabled embedding still fail with error `101` or `150`, and content blockers may prevent the API from loading. In either case, fall back to a link on YouTube.

The project is entirely static, so you can also host `embed.html` on your own HTTPS domain.

## License

MIT
