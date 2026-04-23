NUB-MEDIA
=========

Media Session Control
---------------------

`draft`

**NUB ID:** NUB-MEDIA
**Namespace:** `window.napplet.media`
**Discovery:** `shell.supports("media")`

## Description

NUB-MEDIA provides media session management between napplets and the shell. Napplets that play audio or video need a way to expose playback state and metadata to the shell so the shell can display media controls (play/pause, skip, volume), aggregate multiple media sources across napplets, and enforce audio policy (e.g., mute on focus loss, duck overlapping sources).

The shell provides a media session registry and control surface. Napplets create sessions, report state and metadata, declare dynamic capabilities, and receive commands from the shell. Each napplet may have multiple concurrent sessions (e.g., a music player with background playback and a notification sound effect).

## API Surface

```typescript
interface NappletMedia {
  createSession(metadata?: MediaMetadata): Promise<MediaSessionResult>;
  updateSession(sessionId: string, metadata: Partial<MediaMetadata>): void;
  destroySession(sessionId: string): void;
  reportState(sessionId: string, state: MediaState): void;
  reportCapabilities(sessionId: string, actions: MediaAction[]): void;
  onCommand(sessionId: string, cb: (action: MediaAction, value?: number) => void): Subscription;
  onControls(sessionId: string, cb: (controls: MediaAction[]) => void): Subscription;
}

interface MediaMetadata {
  title?: string;
  artist?: string;
  album?: string;
  artwork?: { url?: string; hash?: string };
  duration?: number;
  mediaType?: 'audio' | 'video';
}
```

**Resource resolution.** The `artwork.url` field is a URL string. Napplets and shells that need the artwork bytes (for example, to render album art on a media controls surface) MUST fetch them through NUB-RESOURCE: `window.napplet.resource.bytes(url)`. The optional `artwork.hash` field, when present, MAY be used by shells as a content-addressed cache key but is not a substitute for the URL fetch — napplets address artwork by URL through the resource NUB. Direct `<img src="https://...">` loads will not work under the iframe sandbox model defined by NIP-5D (`sandbox="allow-scripts"`, no `allow-same-origin`); the shell is the sole network-fetch broker. Standard NUB-RESOURCE policy applies (private-IP block list at DNS-resolution time, MIME byte-sniffing, SVG rasterization, etc.).

```typescript

interface MediaState {
  status: 'playing' | 'paused' | 'stopped' | 'buffering';
  position?: number;
  duration?: number;
  volume?: number;
}

type MediaAction = 'play' | 'pause' | 'stop' | 'next' | 'prev' | 'seek' | 'volume';

interface MediaSessionResult {
  sessionId: string;
  error?: string;
}

interface Subscription {
  close(): void;
}
```

**`createSession(metadata?)`** -- Creates a new media session. The napplet provides optional metadata (title, artist, album, artwork, duration, mediaType). All metadata fields are optional -- a session can be created with no metadata and updated later. Returns the assigned `sessionId`. The shell MAY reject session creation (e.g., if the napplet exceeds a session limit).

**`updateSession(sessionId, metadata)`** -- Updates metadata for an existing session. Partial updates are supported -- only the fields provided are changed. Fire-and-forget.

**`destroySession(sessionId)`** -- Destroys a session. The shell removes it from the media control surface. Fire-and-forget.

**`reportState(sessionId, state)`** -- Reports the current playback state for a session. The napplet sends this whenever state changes (play/pause/stop/buffer transitions, position updates, volume changes). Fire-and-forget, high frequency during active playback.

**`reportCapabilities(sessionId, actions)`** -- Declares which media actions the session currently supports. Capabilities are dynamic -- a streaming source may not support `seek` initially but add it once buffered. The shell uses this to enable/disable media control buttons. Fire-and-forget.

**`onCommand(sessionId, callback)`** -- Listens for media commands from the shell. The shell sends commands based on its media control UI (user presses play, adjusts volume slider, etc.). The `value` parameter is used for `seek` (position in seconds) and `volume` (0.0 to 1.0). Returns a Subscription with `close()`.

**`onControls(sessionId, callback)`** -- Listens for the shell's control list. The shell tells the napplet which controls the shell supports, so the napplet can adapt its own UI (e.g., hide a next/prev button if the shell does not support it). Returns a Subscription with `close()`.

## Wire Protocol

`media.*` messages use the NIP-5D wire format (`{ "type": "domain.action", ...payload }`).

| Type | Direction | Payload fields |
|------|-----------|----------------|
| `media.session.create` | napplet -> shell | `id`, `sessionId`, `metadata?` |
| `media.session.create.result` | shell -> napplet | `id`, `sessionId`, `error?` |
| `media.session.update` | napplet -> shell | `sessionId`, `metadata` |
| `media.session.destroy` | napplet -> shell | `sessionId` |
| `media.state` | napplet -> shell | `sessionId`, `status`, `position?`, `duration?`, `volume?` |
| `media.capabilities` | napplet -> shell | `sessionId`, `actions` |
| `media.command` | shell -> napplet | `sessionId`, `action`, `value?` |
| `media.controls` | shell -> napplet | `controls` |

Key design notes:
- `media.session.create` / `media.session.create.result` use `id` for correlation. The napplet provides `sessionId` as a client-generated identifier.
- `media.session.update`, `media.session.destroy`, `media.state`, and `media.capabilities` are fire-and-forget (no `id` correlation).
- `media.command` is shell-initiated. The shell sends media control commands to the napplet.
- `media.controls` is shell-initiated. The shell pushes its supported control list so the napplet can adapt its UI.
- Multiple sessions per napplet are supported. Each session is identified by `sessionId`.
- Volume is dual: the napplet reports its own volume via `media.state`, and the shell can set volume via `media.command` with `action: 'volume'`. The effective volume is the product of both.

### Metadata

All metadata fields are optional. The `artwork` field supports two forms:

- `url` -- A direct URL to the artwork image.
- `hash` -- A Blossom hash (SHA-256) that the shell can resolve via its configured Blossom servers. If both are provided, the shell MAY prefer either.

```json
{
  "title": "Song Title",
  "artist": "Artist Name",
  "album": "Album Name",
  "artwork": { "url": "https://example.com/cover.jpg", "hash": "abc123..." },
  "duration": 240,
  "mediaType": "audio"
}
```

### Examples

**Create a session:**
```
-> { "type": "media.session.create", "id": "m1", "sessionId": "s1", "metadata": { "title": "My Song", "artist": "The Artist" } }
<- { "type": "media.session.create.result", "id": "m1", "sessionId": "s1" }
```

**Update metadata:**
```
-> { "type": "media.session.update", "sessionId": "s1", "metadata": { "title": "Updated Title" } }
```

**Report playback state:**
```
-> { "type": "media.state", "sessionId": "s1", "status": "playing", "position": 42.5, "duration": 240, "volume": 0.8 }
```

**Report capabilities:**
```
-> { "type": "media.capabilities", "sessionId": "s1", "actions": ["play", "pause", "seek", "volume"] }
```

**Shell sends a command:**
```
<- { "type": "media.command", "sessionId": "s1", "action": "pause" }
<- { "type": "media.command", "sessionId": "s1", "action": "seek", "value": 120 }
<- { "type": "media.command", "sessionId": "s1", "action": "volume", "value": 0.5 }
```

**Shell sends control list:**
```
<- { "type": "media.controls", "controls": ["play", "pause", "stop", "next", "prev", "seek", "volume"] }
```

**Destroy a session:**
```
-> { "type": "media.session.destroy", "sessionId": "s1" }
```

**Session creation rejected:**
```
<- { "type": "media.session.create.result", "id": "m1", "sessionId": "s1", "error": "session limit exceeded" }
```

### Error Handling

`media.session.create.result` includes an `error` field (string) on failure. Common errors: `"session limit exceeded"`, `"invalid session ID"`. When `error` is present, the session was not created.

Messages referencing an unknown `sessionId` (e.g., `media.state` for a destroyed session) MUST be silently ignored by both napplet and shell.

## Shell Behavior

- The shell MUST accept `media.session.create` messages and respond with `media.session.create.result` carrying the same `id`.
- The shell MUST track active sessions per napplet and remove them on `media.session.destroy` or when the napplet iframe is removed.
- The shell SHOULD display media controls (play/pause, skip, volume) for active sessions based on the napplet's reported capabilities.
- The shell SHOULD update its media control UI in response to `media.state` messages.
- The shell MAY send `media.command` messages to control napplet playback based on user interaction with the shell's media controls.
- The shell MAY send `media.controls` to inform the napplet which controls the shell supports. The napplet can adapt its own UI accordingly.
- The shell MAY enforce audio policy (e.g., mute sessions on focus loss, duck overlapping sources, enforce a single-active-session policy).
- The shell MAY limit the number of concurrent sessions per napplet.
- The shell MAY enforce ACL checks on `media` capabilities (e.g., restricting which napplets can create media sessions).
- The shell MUST silently ignore messages from unknown session IDs.
- The shell MUST clean up all sessions for a napplet when the napplet's iframe is removed.

## Security Considerations

- Media sessions expose metadata and playback state to the shell. Napplets SHOULD NOT include sensitive information in metadata fields.
- Artwork URLs are fetched by the shell, not the napplet (the napplet is sandboxed with no network access). The shell SHOULD validate URLs and enforce a content security policy for fetched resources.
- Blossom artwork hashes allow the shell to resolve artwork through its own Blossom infrastructure without the napplet needing network access. The shell controls which Blossom servers are queried.
- Volume control is advisory -- the napplet controls actual audio output. A malicious napplet could ignore volume commands. The shell MAY enforce volume limits at the iframe level using the Web Audio API or iframe attribute policies if available.
- Session creation is rate-limited by the shell. A napplet that creates excessive sessions can be throttled or denied.
- The `media.command` message allows the shell to control napplet behavior. The napplet trusts the shell (NIP-5D security model) but SHOULD validate the `action` against its declared capabilities.

## Implementations

- (none yet)
